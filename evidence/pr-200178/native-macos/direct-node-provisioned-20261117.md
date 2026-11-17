# Native macOS 2026.11.4: direct node after manual route-scoped credential association

**Result:** PASS for native Tailscale Direct Node authorization using a separately approved `node` device token. The real matching-build macOS app made two successful WS connections, including one **after a full app stop/relaunch**.

**Scope:** PR #200178, candidate `fae22bedc776fadbbbcfb7b0e8d2226d0b1969f6`, 17 November 2026, China time (UTC+08:00). Prior SSH-positive / Direct-negative findings in [transport-comparison.md](transport-comparison.md) are accurate **pre-provisioning** results, not a contradiction.

## Authentic client execution

1. Confirmed the same existing primary signed device identity was used for SSH and Tailscale Direct, and the two routes reached the same intended Gateway. An SSH-route `node` token and Direct-route `operator` token were already present. The direct Node token was missing.
2. Before the write, made a consistent SQLite backup of the test app credentials on the MacBook Air; integrity check passed. Stopped **only** the ad-hoc debug app.
3. Associated the existing, approved SSH Node token with the verified Direct-route scope via the first-party OpenClawKit `DeviceAuthStore.storeTokenPersisted` API, conditionally creating only the missing `node` entry. No server-side token was minted, rotated or approved, no existing token overwritten, no SQL row was modified directly, and no Gateway auth policy changed.
4. Relaunched the real macOS app. The Gateway admitted the native Node over managed Serve Direct with a `device-token` credential. Stopped/restarted the app again and observed a second successful Node handshake.

## Redacted live Gateway WebSocket receipts

| Time (UTC+08:00, Nov 17) | Real client | Role/mode | Auth | Result |
| --- | --- | --- | --- | --- |
| 22:50:19.573 | `openclaw-macos` | `node/node` | `device-token` | **Connected** |
| 22:51:17.600 | `openclaw-macos` | `node/node` | `device-token` | **Connected after app restart** |

Live isolated Gateway node inventory after the restart: macOS app **2026.11.4**, `paired=true`, `connected=true`; **10 capabilities and 22 advertised commands**.

The corresponding [sanitized JSON E2E proof](direct-node-provisioned-20261117.json) contains source/method/result metadata; full raw Gateway logs, token values, device IDs, credentials, hostnames and MacBook backups are **not public**.

## What this proves and does not prove

- **PASS:** a real native Node can authenticate over Tailscale Direct using its explicitly approved and correctly scoped device credential; the result survives a macOS app restart.
- **PASS:** the uncredentialed Direct Node was previously denied; operator UI tokens were not promoted into Node authorization. Server auth and pairing policy stayed unchanged.
- **NOT PROVEN:** automatic first-run Direct Node provisioning, complete success of the proposed native GUI recovery workflow, remote Node command execution, or a fully authenticated Cloudflare Access browser round trip.
- **Security caveat:** manually associating an existing Node token to a separately verified route is a controlled E2E test, **not** a supported default onboarding recommendation. Product/security maintainers must approve any broader policy or user workflow.

A checked local SQLite backup and a narrowly scoped rollback method are available to the tester. Neither was uploaded to GitHub.
