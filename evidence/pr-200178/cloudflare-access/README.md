# Real Cloudflare Access browser E2E — trusted-proxy operator connection

**Observed result: PASS for actual browser Control UI over the Cloudflare-facing trusted-proxy ingress.** On 17 November 2026, the tester reported successfully completing the real **Cloudflare Access sign-in**. The same PR Gateway independently recorded **two authenticated real browser-class WebSocket connections** using `auth=trusted-proxy`. This is different from the Tailscale Serve `auth=tailscale` and macOS `auth=device-token` routes.

## Real test and config provenance

Candidate: OpenClaw **2026.11.4**, exact PR source head `fae22bedc776fadbbbcfb7b0e8d2226d0b1969f6`. The isolated live Gateway's checked configuration had `gateway.auth.mode=trusted-proxy`, configured `trustedProxies` and `trustedProxy.allowUsers`, and a required `cf-access-jwt-assertion` identity header. `allowTailscale=true` and managed Serve remained enabled in parallel. The Cloudflare-facing public origin was separate from Serve.

The Gateway's `src/gateway/auth.ts` trusted-proxy branch checks that the request comes from the configured trusted source, configured identity headers are present and nonempty, a proxy user exists and is allowed, and the ingress is attributed to the trusted-proxy listener. A real `auth=trusted-proxy` authenticated connection means these server-side checks passed.

## Privacy-redacted Gateway receipts

| China time, 17 Nov (UTC+08) | Real client/mode | Role | Gateway auth method | Result |
| --- | --- | --- | --- | --- |
| 23:20:20.366 | Browser | Operator pairing gate | — | Pairing transition observed |
| **23:20:22.561** | `openclaw-control-ui` / `webchat` | operator | **`trusted-proxy`** | **Connected** |
| 23:21:01.269 | Browser | Operator pairing gate | — | Pairing transition observed |
| **23:21:03.417** | `openclaw-control-ui` / `webchat` | operator | **`trusted-proxy`** | **Connected** |

Source: local Gateway log verified by exact timestamp and client, mode, role, auth and result predicates; summarized in [sanitized structured WS events](redacted-ws-events.json). No user name, IP, hostname, JWT, header value, device identity or raw log is published.

## Scope and limits

**Established:** A tester-performed Cloudflare Access sign-in was followed by two successfully admitted **trusted-proxy-authenticated Control UI WebSockets** in the real browser on the patched Gateway. This closes the former **Gateway-side Cloudflare ingress functional E2E** gap; the original browser pairing screenshot alone did not prove a successful authenticated handshake.

**Not independently audited:** Neither Cloudflare Access edge login/audit records nor JWT signatures/claims were separately retrieved. The Gateway requires the relevant identity header but does **not** cryptographically validate the Cloudflare Access JWT. The tester's SSO observation plus actual trusted-proxy gateway receipts is functional evidence, not an independent Cloudflare security audit.

**Still requires maintainer approval:** The separate policy choice allowing verified managed-Serve operator UI identity independently of `trustedProxy.allowUsers`. Cloudflare-side success does not authorize that broader ingress policy.

This publication changes no Gateway config, token, device grant, server process or PR source code.
