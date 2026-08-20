# Security Report: Plaintext Autopilot Private Keys in localStorage

**Target:** https://limen.finance/agent  
**Severity:** High

When a user starts an unattended Autopilot session, the app stores a raw
secp256k1 private key in localStorage:

- limen-agent-session-key-v1
- limen-agent-session-key-v1:<marketId>

Format: 0x + 64 hex. Also exposed via Backup Key in the UI.
