# Security Report: Plaintext Autopilot Private Keys in localStorage

**Target:** https://limen.finance/agent

**Severity:** High

**Component:** Unattended Autopilot / Mission Control session signer

**Client source:** `/_next/static/chunks/9543-*.js`

## Summary

When a user starts an unattended Autopilot session, the app generates a secp256k1 private key in the browser and stores it in plaintext in `window.localStorage` under:

- `limen-agent-session-key-v1`
- `limen-agent-session-key-v1:<marketId>`

The stored value is a raw private key matching `/^0x[0-9a-f]{64}$/i`.

The same secret is exposed in the UI through **Backup Key**.

Anyone who can read `localStorage` for `https://limen.finance` (XSS, malicious extension, shared device) can control the Autopilot session signer and drain the session budget.

This affects the Autopilot session hot wallet only, not the main MetaMask / Circle wallet.

## Impact

1. Session budget can be drained if the stored key is read.
2. No encryption-at-rest for the private key.
3. Backup Key returns the same plaintext secret.
4. XSS becomes a direct key-theft path.

## Reproduction

1. Open https://limen.finance/agent
2. Connect a wallet
3. Start **Start unattended**
4. DevTools → Application → Local Storage → `https://limen.finance`
5. Find `limen-agent-session-key-v1` or `limen-agent-session-key-v1:<marketId>`
6. Value format: `0x` + 64 hex characters

Do not submit real private keys, seeds, or passwords with this report.

## Evidence (from client bundle)

Storage key names:

```
limen-agent-session-key-v1
limen-agent-session-key-v1:<marketId>
limen-agent-session-addrs-v1
```

Load key from localStorage (minified logic):

```
function V(e) {
  let a = window.localStorage.getItem(U(e));
  // migrates legacy key if needed
  return a !== null && /^0x[0-9a-f]{64}$/i.test(a) ? a : null;
}
```

Start unattended session (critical line):

```
let n = V(e.id) ?? generatePrivateKey();
let r = privateKeyToAccount(n);
window.localStorage.setItem(U(e.id), n);  // raw private key saved
```

Backup Key:

```
em = useCallback(() => V(e.id), [e.id]);
```

Error string from the same flow:

```
this browser blocks local storage — the session key could not be saved, so no funds were moved
```

## Recommended fix

1. Do not persist raw private keys in localStorage.
2. Prefer non-extractable WebCrypto keys or a constrained server-side session signer.
3. Remove any UI that exports the raw secret (Backup Key).
4. Add strong CSP and enforce HTTPS + HSTS.
