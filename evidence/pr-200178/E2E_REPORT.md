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

## Subsequent real graphical native app test

The real app's operator UI connected through Serve. Its Node was rejected **before** manual route-scoped Node token association; the matching-build Node then connected successfully over SSH and Direct (including post-restart), without any Gateway auth-policy modification. See [native macOS E2E](native-macos/README.md). Automatic first-run recovery remains unproven.

## Later positive Direct Node evidence (manually provisioned)

Following the initial matching-app SSH success and Direct rejection, the MacBook's existing approved Node token was associated with the verified Direct route through the OpenClawKit credential API, without changing Gateway policy. The real native macOS 2026.11.4 Node connected with `auth=device-token` at **22:50:19.573** and **22:51:17.600** (after app restart), UTC+08. Gateway status showed `paired=true`, `connected=true`.

See [positive Direct Node report](native-macos/direct-node-provisioned-20261117.md) and [structured sanitized receipts](native-macos/direct-node-provisioned-20261117.json). Manual token association does **not** prove native first-run auto-onboarding or successful GUI recovery; those remain limitations.

## Successful Cloudflare Access browser route (subsequent live E2E)

Following the earlier Cloudflare-facing browser pairing screenshot, the tester completed the actual Cloudflare Access login. At **23:20:22.561** and **23:21:03.417 (+08:00)** on November 17, the real `openclaw-control-ui` `webchat` operator connected to the patched Gateway with `auth=trusted-proxy`. Two pairing-gate transitions were also observed. The Gateway's trusted-proxy path verifies the configured required header, trusted source, user allowlist and ingress attribution. This is separate from the previously proved managed-Serve `auth=tailscale` ingress.

[Cloudflare E2E report](cloudflare-access/README.md) · [structured sanitized receipts](cloudflare-access/redacted-ws-events.json). The tester's SSO completion and Gateway admission are supported; provider-side Cloudflare edge logs/JWT cryptographic verification were not separately audited.

## Remaining limitations and owner decision

1. The older graphical macOS app (2026.10.10) established operator UI access. A later matching-build 2026.11.4 native app connected as Node over SSH and, after a manual route-scoped token association, over Tailscale Direct (including a post-restart connection). **Automatic first-run Node provisioning and the complete matching-build recovery GUI flow remain unproven.** See native-macos/README.md.
2. **Subsequent Cloudflare Access browser success:** The tester completed real Cloudflare Access sign-in; the Gateway recorded authenticated browser `auth=trusted-proxy` connections at 23:20:22 and 23:21:03 (+08:00). Independent Access edge audit and JWT cryptographic checks were **not** performed. See [Cloudflare browser E2E](cloudflare-access/README.md).
3. Need maintainer approval for independent managed Serve authentication in trusted-proxy mode: tailnet user membership is intentionally *not* equivalent to Cloudflare `trustedProxy.allowUsers`.
4. Full upstream CI remains blocked by external fork `action_required`, despite 354 earlier targeted local tests.
5. Native Node token in the synthetic isolated test had to be loaded by restarting the isolated Gateway after out-of-process test pairing; a normal RPC pairing workflow could differ. Once loaded, positive Node admission passed.

This report offers concrete before/after production-route proof, not a claim of merge readiness or a guaranteed 6/6 score.
