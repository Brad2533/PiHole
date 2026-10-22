# PiHole

> Public snapshot of a private development repository. The full commit history is private; a walkthrough is available on request.

Home network Pi-hole project on a Raspberry Pi 4B, sitting behind a separate tri-band Wi-Fi 5 router (bypassing the Starlink second-gen router).

## Planned build order

1. Install Pi-hole, give the Pi a static IP, point the router's DNS at it, and confirm ads are actually being blocked.
2. Add security-focused blocklists (malware and phishing domains).
3. Add Suricata alongside Pi-hole.
4. Add a small status display showing ads blocked, queries today, and the Pi's static IP / hostname.

Decided against Unbound and a WireGuard VPN for now in favor of the security tooling above.

The Pi should only run Pi-hole plus security software (Suricata) and light helpers like health checks. The Minecraft server goes on a separate machine.

## Suricata pass rules for competitive games

Latency matters for racing and competitive shooters, so game traffic should skip Suricata inspection. Everything still goes through Pi-hole DNS blocking, which adds no meaningful latency.

- Suricata pass rules match source/destination, ports, or protocol and let matching traffic (and the rest of a TCP flow) bypass all other rules. They are checked first.
- Match on **hostname** (e.g. TLS SNI) rather than IP or port, so server IP changes do not silently break the rule. Exempting a whole PC by IP would disable inspection for its browsing and downloads too, so avoid that.
- Do not guess the hostnames. Once Pi-hole is running, watch the query log while playing each game and build the rules from what actually shows up.
- Deployment tradeoff: a mirror-port IDS adds zero latency but can only alert, not block. Inline IPS can block but adds latency and is limited by the Pi's throughput. Mirror mode needs a switch that supports port mirroring.

| Game | Expected domains (unverified) |
|---|---|
| Le Mans Ultimate | `racecontrol.gg` (assumption: it is the account/server layer, but live race traffic may use a different domain) |
| Valorant, League of Legends | Riot-owned domains |
| Counter-Strike, Deadlock | Valve matchmaking/relay domains |

## Late-stage project: document LMU's network traffic

No public documentation of Le Mans Ultimate's live race domains/IPs seems to exist. Toward the end of the project, capture real traffic during races (Pi-hole query log, Wireshark), write it up, and publish it somewhere visible rather than leaving it in this private repo: a public gist or repo, Steam Community discussions, the LMU / Studio 397 forums, and the LMU or RaceControl Discords.

## Health checks and alerting

Run self-hosted Uptime Kuma (free, nothing leaves the network) or a similar lightweight check to alert on Pi-hole or Suricata going down. Healthchecks.io's free tier is the hosted alternative.

## Backups

- Keep Pi-hole and Suricata config files (rules, blocklist list, settings) in this repo.
- Pi-hole's runtime database does not belong in Git. Back it up with a script on my computer that pulls the backup from the always-on Pi over SSH/SCP once a day, and runs at next startup if that day's run was missed.

## Note: separating ad blocks from security blocks

Pi-hole's API reports overall stats (total queries, queries blocked, top blocked domains) but does not break blocks down by which blocklist caused them. During gravity updates, all list domains are merged into one blocked set, and there is an open feature request upstream for per-list statistics.

To report "ads blocked" and "security blocked" separately:

- Keep ad lists and security lists in separate Pi-hole **groups** from the start.
- Pull blocked queries from the query log / FTL database and join each blocked domain against the group's list contents myself in a small custom API.
- Set up the grouping up front, since it is the only piece that is hard to retrofit.

nenad is gone
