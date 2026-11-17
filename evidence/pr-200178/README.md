# PR #200178 — redacted E2E evidence index

Candidate Gateway head `fae22bedc776fadbbbcfb7b0e8d2226d0b1969f6` (2026.11.4). This evidence lives on an **independent evidence branch**, not in the PR source. All reports preserve test chronology and distinguish protocol probes, actual graphical-app observations and manual provisioning.

| Real test / scenario | Result | Evidence |
| --- | --- | --- |
| Original 2026.11.3 Serve operator UI without Cloudflare headers | Missing proxy identity / rejected | [baseline report](E2E_REPORT.md), [structured WS traces](redacted-e2e-traces.json) |
| PR 2026.11.4 real Chromium Control UI over managed Serve | Authenticated / Control UI loaded | [browser notes](browser-screenshot-notes.md), [browser visual pair](screenshots/) |
| PR 2026.11.4, **real Cloudflare Access browser login** | **Authenticated** Control UI, `auth=trusted-proxy` (23:20 and 23:21) | [Cloudflare browser E2E](cloudflare-access/README.md), [WS receipts](cloudflare-access/redacted-ws-events.json) |
| Native graphical app 2026.10.10 operator UI | Connected over Tailscale and trusted proxy | [native app screenshots and receipts](native-macos/README.md) |
| Matching native app 2026.11.4 operator UI and SSH Node | Both connected with respective credentials | [SSH / Direct comparison](native-macos/transport-comparison.md) |
| Matching native app 2026.11.4 Direct Node **before** Node token association | Rejected, `auth=none` | [negative Direct results](native-macos/transport-comparison.json) |
| Matching native app 2026.11.4 Direct Node **after** manually associating existing approved role token | **Connected** with `auth=device-token`, also after app restart | [positive Direct Node proof](native-macos/direct-node-provisioned-20261117.md), [JSON receipts](native-macos/direct-node-provisioned-20261117.json) |

## Browser screenshot evidence

Unmodified, already-redacted original screenshots: [before](https://github.com/user-attachments/assets/f3e2b8e0-5376-4240-8357-cf2b23545edf) and [after](https://github.com/user-attachments/assets/aabdae04-c604-49f3-98d2-67dd374bbfa2).

Separate annotated copies of the **same real, redacted images**: [before annotation](screenshots/serve-before-annotated.png) and [after annotation](screenshots/serve-after-annotated.png). Annotations add only a caption and highlights. Original before capture shows a cropped error card; after capture shows a wider Control UI, therefore this is **not** a precisely viewport-matched visual regression fixture. Independently matched Gateway WS logs substantiate network authentication.

## Boundaries and unresolved items

- Node permission requires its own scoped token; verified Tailnet operator identity alone did not admit Node access. The manual client-side route association is a **test-only** procedure, not general-supported automatic onboarding.
- Full native recovery GUI, automatic first-run Direct Node pairing and Node command invocation remain **unproven**. The tester reports successful real Cloudflare Access SSO and the Gateway separately corroborates `auth=trusted-proxy` Control UI admission; **Cloudflare edge audit/JWT cryptographic verification was not independently captured**.
- The independent Serve-vs-Cloudflare allowlist trust policy needs **maintainer acceptance**. Neither positive tests nor ClawSweeper scores constitute policy approval.
- No raw logs, credentials, backup databases, JWTs, IP addresses, hostnames or private keys published.

Detailed timeline and caveats: [E2E_REPORT.md](E2E_REPORT.md).
