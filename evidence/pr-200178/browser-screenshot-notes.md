# Real browser evidence — Draft PR #200178

PR under test: `e0b9b75428f136819dab42fbc370862c0f8924b2`.

The three cropped, redacted image screenshots were supplied in the contributor's test conversation.
The original screenshots contain private endpoint information and, in one case, a browser approval request ID; they are **not** committed here.

| Screenshot | Claim supported | Independent log evidence |
| --- | --- | --- |
| Baseline: "Proxy-Authentifizierung erforderlich" on Tailnet URL | Original installed 2026.11.3 presents proxy auth error | Signed MacBook WSS handshake returns `AUTH_IDENTITY_HEADER_REQUIRED` |
| Patched: OpenClaw 2026.11.4 Model Setup page | Real browser reaches Control UI on managed Serve | Two actual `openclaw-control-ui` authenticated WebSocket connections, `auth=tailscale`, `connected`, at 15:35:23.335 and 15:35:40.350 +08:00 |
| Patched: "Diesen Browser genehmigen" on Cloudflare-facing edge | Edge browser route reached OpenClaw's device pairing gate | NOT evidence of completed Cloudflare Access JWT verification, approved device, or hello-ok |

The public textual report and redacted structured traces in this evidence branch contain no tokens, raw Gateway logs, private endpoints, or device identities.
**For fully visual proof, the separately redacted image files need to be attached to the PR through a GitHub image upload.** This branch intentionally does not claim unhosted screenshots are available here.
