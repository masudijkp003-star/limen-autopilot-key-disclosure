---

## Evidence — verbatim minified excerpts

From `https://limen.finance/_next/static/chunks/9543-*.js`:

```js
// Constants
let M="limen-agent-session-key-v1",
    R="limen-agent-session-market-v1",
    U=e=>"limen-agent-session-key-v1:".concat(e),
    F="limen-agent-session-addrs-v1";

// Load + migrate + validate raw private key
function V(e){
  try{
    let a=window.localStorage.getItem(U(e));
    if(null===a){
      let t=window.localStorage.getItem(R),
          n=window.localStorage.getItem(M);
      null!==n&&(t===e||null===t)&&(
        window.localStorage.setItem(U(e),n),
        window.localStorage.removeItem(M),
        window.localStorage.removeItem(R),
        a=n
      )
    }
    return null!==a&&/^0x[0-9a-f]{64}$/i.test(a)?a:null
  }catch(e){return null}
}

// Start unattended: reuse or generate, then setItem
let n=null!==(t=V(e.id))&&void 0!==t?t:(0,w.w)(),
    r=(0,g.L)(n);
try{
  window.localStorage.setItem(U(e.id),n),
  function(e){
    try{
      let a=[e,...T().filter(a=>a.toLowerCase()!==e.toLowerCase())].slice(0,6);
      window.localStorage.setItem(F,JSON.stringify(a))
    }catch(e){}
  }(r.address)
}catch(e){
  throw Error("this browser blocks local storage — the session key could not be saved, so no funds were moved")
}

// Backup Key
em=(0,n.useCallback)(()=>V(e.id),[e.id])
```

---

## Recommended fix

1. **Do not persist raw private keys in `localStorage`.**
2. Prefer non-extractable WebCrypto keys, hardware-backed keys, or a constrained server-side/session signer.
3. If a browser-held key is required, avoid any UI that exports the raw secret (“Backup Key”).
4. Add strong **CSP**, enforce **HTTPS + HSTS**, and treat Autopilot keys as high-value secrets.
