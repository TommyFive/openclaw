# Real Gateway E2E evidence — OpenClaw PR #200178

- **Commit:** `e0b9b75428f136819dab42fbc370862c0f8924b2` (Draft).
- **Date:** 2026-11-17 (UTC), actual tailnet certificate-verified TLS / WSS using a second physical MacBook Air (untagged human tailnet node).
- **Baseline:** Installed OpenClaw 2026.11.3, trusted-proxy + required Cloudflare JWT header, managed Serve, allowTailscale true.
- **Candidate:** Compiled OpenClaw 2026.11.4 from the exact PR SHA, same ingress/auth policy, separately isolated Gateway state, no channels/plugins/hooks.
- **Client:** Custom, protocol-faithful Ed25519 v3 handshake over real WSS, not the graphical/native OpenClaw.app itself.
- **TLS:** Validated certificate and WebSocket HTTP/1.1 Upgrade 101 from the MacBook Air.
- **Credentials:** Independent short-lived test device, operator.read and node role-specific pairing tokens stored only in the isolated state, cleared after tests.

## Before — original 2026.11.3

| Signed client and route | Result | Auth detail |
|---|---|---|
| Native `openclaw-macos` `ui` / managed Serve | Denied | `AUTH_IDENTITY_HEADER_REQUIRED` |
| Browser `openclaw-control-ui` / managed Serve | Denied | `AUTH_IDENTITY_HEADER_REQUIRED` |
| Node without device credential / managed Serve | Denied | `AUTH_IDENTITY_HEADER_REQUIRED` |

Actual before responses are preserved in the local `signed-before-patch.json`.

**Same-device cross-check after rollback:** The same physical untagged MacBook Air also independently repeated native UI and node WSS handshakes against restored 2026.11.3; both returned `AUTH_IDENTITY_HEADER_REQUIRED`. This is in `macbook-original-2026-11-3-signed-baseline.json`. The negative result was recorded after the positive PR test, eliminating the tagged-versus-untagged client difference as an explanation.

## After — 2026.11.4 E2E on real tailnet

| Case | Result | Code / detail | Granted scopes |
|---|---|---|---|
| `macbook_ui_paired_operator_token` | PASS (hello-ok) | `-` | operator.read |
| `macbook_node_without_token` | Denied | `AUTH_IDENTITY_HEADER_REQUIRED` | - |
| `macbook_node_operator_token_only` | Denied | `AUTH_DEVICE_TOKEN_MISMATCH` | - |
| `macbook_node_paired_node_token` | PASS (hello-ok) | `-` |  |

**Positive proof:** Correctly paired native-app UI accepts `hello-ok` with exactly `operator.read`, and a separately approved node accepts `hello-ok` with no operator scopes. **Negative proof:** no node token or operator token for node role is rejected.

When the client had a valid signed identity but no paired credential, native UI reached `PAIRING_REQUIRED` instead of failing at Cloudflare's missing-header gate. This demonstrates the original auth-ordering fix independently of the later device credential grant.

## Browser policy / origin from the second MacBook

| Case | Result | Error | Scopes |
|---|---|---|---|
| `browser_control_ui_allowed_origin` | Denied | `PAIRING_REQUIRED` | - |
| `browser_control_ui_bad_origin` | Denied | `CONTROL_UI_ORIGIN_NOT_ALLOWED` | - |

The same-origin browser probe with the correct Control UI build ID reaches device pairing (`PAIRING_REQUIRED`), while a deliberately invalid Origin is rejected by `CONTROL_UI_ORIGIN_NOT_ALLOWED`. The browser Control UI was *not* given separate pairing approval. The earlier `PROTOCOL_MISMATCH` from a synthetic browser client was a harness error (missing mandatory `client.buildId`) and was corrected.

## Tagged nodes versus human Tailnet clients

The Mac mini itself and two test VMs use tagged tailnet identities. Their WSS requests reached the candidate Gateway but lacked a `tailscale-user-login` (expected for tagged clients), yielding `AUTH_TAILSCALE_IDENTITY_MISSING`. This is not evidence of an admission regression for untagged end users. The actual untagged MacBook Air supplied sufficient verified identity and reached pairing / successful admission.

## Startup compatibility and production safety

A direct in-place 2026.11.3→2026.11.4 restart failed before serving traffic during a 6.9 GB / 53,839-file legacy state upgrade, when a concurrently changing historical log was missing (`ENOENT`). No production files were forced or patched. A private isolated state allowed the exact built PR Gateway to start with the same auth ingress policy. This is **a deployment/upgrade limitation, not proven to originate in this PR's auth changes**.

The original 2026.11.3 package, state, config, and LaunchAgent were backed up. After the live proof, the original LaunchAgent and configuration hashes were restored; the 2026.11.3 process, port 18789, local HTTP 200, and managed Serve HTTPS 200 (TLS verify=0) were verified.

Full raw Gateway logs and internal tailnet status are retained **locally only**, not uploaded to GitHub. Local raw Gateway log SHA-256: `1becc6b73e3c74d24e31188a61d27a7cfba14cc55ec5b798d491b8cebb42c03b` (51721396 bytes). This hash permits matching the original confidential evidence if an authorized maintainer needs a sanitized excerpt.

## Additional interactive browser UI evidence — tester screenshots

1. The managed-Tailscale-Serve Chromium screenshot shows the Control UI Model Setup page and a footer identifying the PR build, version 2026.11.4 at e0b9b75. Correlating raw Gateway logs record two **real successful browser-class WebSocket connections**, authenticated by Tailscale at 2026-11-17 15:35:23.335 +08:00 and 15:35:40.350 +08:00. These are separate from the synthetic protocol-client E2E tests.
2. The Cloudflare-facing edge URL displays OpenClaw's **Approve this browser** screen. This is proof the browser reached OpenClaw device pairing, but **not** proof that a Cloudflare Access JWT was accepted, that a device was approved, or that this connection reached authenticated hello-ok.

The two source images were tester-supplied; sanitized copies with private hostname and device-request ID masked were produced separately. The full confidential logs are still held locally and not uploaded to GitHub.

## Remaining limitations and owner decision

1. Actual macOS native application UI/onboarding screenshots and its pairing recovery flow were not captured. The actual app installed on the MacBook is an older release and was not launched or modified.
2. No successful authenticated browser session through the separate **Cloudflare Access** route was exercised in the live experiment. Unit tests cover its allowlist and header gate; the production Cloudflare proxy was not touched.
3. Need maintainer approval for independent managed Serve authentication in trusted-proxy mode: tailnet user membership is intentionally *not* equivalent to Cloudflare `trustedProxy.allowUsers`.
4. Full upstream CI remains blocked by external fork `action_required`, despite 354 earlier targeted local tests.
5. Native Node token in the synthetic isolated test had to be loaded by restarting the isolated Gateway after out-of-process test pairing; a normal RPC pairing workflow could differ. Once loaded, positive Node admission passed.

This report offers concrete before/after production-route proof, not a claim of merge readiness or a guaranteed 6/6 score.
