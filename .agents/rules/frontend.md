# Frontend Rules

- The browser is an untrusted client. It must never receive an SSH private key, password, passphrase, raw SSH configuration secret, or direct TCP access to EC2.
- Keep the interface focused: sign in, instance list, add/edit connection profile, connection status, terminal view, and session close.
- The browser communicates with Express over HTTPS. Use REST for profile/lifecycle operations and an authenticated WebSocket for interactive terminal input/output.
- Do not persist secrets in localStorage, sessionStorage, IndexedDB, URLs, or client-side logs. Credential entry, if implemented, must be a secure one-time backend-mediated flow.
- Display sanitized, actionable errors. Do not render raw server stack traces.
- Render terminal output as text through a terminal emulator; never insert output as HTML. Treat all terminal output as untrusted.
- Provide clear connecting, connected, failed, and disconnected states.
- Make controls keyboard accessible and the layout responsive.
- Inspect the existing stack before introducing a frontend framework.
