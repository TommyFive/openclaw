# Real macOS 2026.11.4: SSH node succeeds, Tailscale Direct node denied

Reproduced 17 November 2026 (China time, UTC+08) with actual graphical macOS debug app and Gateway from PR commit fae22bedc776fadbbbcfb7b0e8d2226d0b1969f6.

| Test | Real screenshot and Gateway corroboration | Outcome |
| --- | --- | --- |
| Native operator UI, managed Tailscale Serve | App showed green Remote Gateway direct, Auth: device token (paired device), Connected via paired device. Gateway WS 21:50:49.868, role=operator, mode=ui, auth=device-token, connected | PASS: actual UI with paired operator device token |
| Native Mac node, SSH transport | App showed 2026.11.4 node paired and connected, with capabilities. Gateway WS 21:44:22.351, role=node, mode=node, auth=password, connected | PASS: actual Mac node functions through authenticated SSH route |
| Same native Mac node, direct Tailscale transport after switching back | App showed Mac node unavailable, unauthorized, trusted-proxy node needs a device token. Gateway WS 21:50:53.374, 21:50:56.066 and 21:50:59.851, role=node, auth=none, trusted_proxy_missing_header_cf-access-jwt-assertion | NOT PASSED: direct node lacks valid endpoint-scoped node credentials |
| Gateway node inventory after switching | Isolated nodes status shows version 2026.11.4, paired=true, connected=false | Paired is not equivalent to connected |

This is a reproducible and useful NEGATIVE authorization result, not proof of successful node admission via Tailscale Direct. Operator identity is not silently promoted to node access. Successful SSH password authentication does not establish that a reusable node credential exists for the direct route.

The current macOS 2026.11.4 app shows the helpful missing-node-token error, but full recovery workflow/role-specific bootstrap over Direct is NOT proven. No extra device privileges, token copies or gateway security modifications were made.

Source screenshots: paired operator-device-token UI, paired-and-connected node while on SSH, missing-node-token native menubar error after switching back. Crops exclude sensitive hostnames, addresses, user/device identifiers. Raw screenshots and full Gateway logs are retained privately.

Structured redacted log evidence: [transport-comparison.json](transport-comparison.json).
