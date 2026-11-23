# Security

Models rarely add a Content Security Policy or cross-origin isolation unless asked. Even when asked,
they often get the details wrong. **Propose a CSP for any app you build**, and ask about COOP/COEP
when the app could use `SharedArrayBuffer` or high-resolution timers.

## 9. DOM XSS defenses and HTML sanitization

### Don't
- Write your own sanitizer with `DOMParser` and a tag/attribute allowlist, whether as the main path,
  as a `setHTML()` fallback or inside a Trusted Types policy.
- Import DOMPurify eagerly on every page load when most browsers can sanitize natively.
- Register a pass-through Trusted Types policy such as `createHTML: (input) => input`. It defeats sink enforcement.
- Protect a static page with `script-src 'self'` host allowlists next to Trusted Types. Hash the inline scripts instead.
- Ship a nonce-based `script-src` without `'strict-dynamic'` and `'report-sample'`.

### Do

```js
// Native sanitizer first, vetted library as a lazy fallback
async function setSafeHTML(el, untrusted) {
  if ('setHTML' in Element.prototype) {
    el.setHTML(untrusted); // sanitizes with the browser's safe defaults
    return;
  }
  const { default: DOMPurify } = await import('dompurify');
  el.innerHTML = DOMPurify.sanitize(untrusted);
}
```

```js
// A Trusted Types policy that really sanitizes
if (globalThis.trustedTypes?.createPolicy) {
  const { default: DOMPurify } = await import('dompurify');
  trustedTypes.createPolicy('default', {
    createHTML: (input) => DOMPurify.sanitize(input, { RETURN_TRUSTED_TYPE: false }),
  });
}
```

**Static pages:** use a hash-based CSP.

```
Content-Security-Policy:
  script-src 'sha256-<base64 hash of each inline script>';
  object-src 'none';
  base-uri 'none';
  require-trusted-types-for 'script';
```

**Server-rendered pages:** use a nonce-based strict CSP.

```
Content-Security-Policy:
  script-src 'nonce-{RANDOM_PER_RESPONSE}' 'strict-dynamic' 'report-sample' https: 'unsafe-inline';
  object-src 'none';
  base-uri 'none';
  require-trusted-types-for 'script';
  report-to csp-endpoint;
```

(`https:` and `'unsafe-inline'` are ignored by browsers that support nonces and `'strict-dynamic'`.
They only keep older browsers working.)

Cross-origin isolation, when needed:

```
Cross-Origin-Opener-Policy: same-origin
Cross-Origin-Embedder-Policy: require-corp
```

## 10. Passkey sign-in, reauthentication and credential lifecycle

### Don't
- Hand-roll `atob()`/`btoa()` Base64URL helpers to convert WebAuthn options and responses,
  as the main path or as a "just in case" fallback where the JSON helpers exist.
- Ignore "unknown credential" responses from the server. Deleted or revoked passkeys then stay in the user's password manager.
- Render "Create a passkey" UI without checking what the client supports.
- Fire step-up reauth `navigator.credentials.get()` calls with no way to cancel them.
- Use the outdated `mediation: "immediate"` option.

### Do

```js
// Feature-detect before showing passkey UI
const caps = await PublicKeyCredential.getClientCapabilities?.() ?? {};
if (caps.passkeyPlatformAuthenticator || caps.userVerifyingPlatformAuthenticator) {
  showCreatePasskeyButton();
}

// Registration
const createOpts = PublicKeyCredential.parseCreationOptionsFromJSON(await (await fetch('/webauthn/register/options')).json());
const cred = await navigator.credentials.create({ publicKey: createOpts });
await fetch('/webauthn/register', { method: 'POST', body: JSON.stringify(cred.toJSON()) });

// Sign-in / step-up reauth, cancellable
const controller = new AbortController();
const getOpts = PublicKeyCredential.parseRequestOptionsFromJSON(await (await fetch('/webauthn/login/options')).json());
const assertion = await navigator.credentials.get({ publicKey: getOpts, signal: controller.signal });
const res = await fetch('/webauthn/login', { method: 'POST', body: JSON.stringify(assertion.toJSON()) });

// Server says the credential ID is unknown: tell the password manager
if (res.status === 404 && PublicKeyCredential.signalUnknownCredential) {
  await PublicKeyCredential.signalUnknownCredential({
    rpId: getOpts.rpId ?? location.hostname,
    credentialId: assertion.id, // base64url
  });
}
```

For streamlined "immediate" sign-in prompts, use the `uiMode: "immediate"` option,
not `mediation: "immediate"`. Also consider `PublicKeyCredential.signalAllAcceptedCredentials()`
and `signalCurrentUserDetails()` to keep password managers in sync after account changes.

Only fall back to manual Base64URL conversion in browsers where `parseCreationOptionsFromJSON` is
missing. Feature-detect it; don't run both paths.
