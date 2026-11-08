# Reaching private or on-premises tools

Mindflow runs in the cloud. A step can only reach what is reachable from the internet,
unless a private route has been set up. Users often meet this as a cryptic network error
on an HTTP Request or integration step pointing at an internal tool (an on-prem
ServiceNow, a SIEM, an internal API, a host with a private IP or an internal DNS name).

## Recognising it

The step fails with a low-level network error rather than an HTTP answer from the tool:

| Error in the step's log | Usually means |
|---|---|
| `Timeout of …ms exceeded` (often after every retry) | The address isn't reachable from Mindflow: private IP (10.x, 172.16–31.x, 192.168.x), or a firewall silently dropping the traffic |
| `getaddrinfo ENOTFOUND <host>` | The host name only exists in the company's internal DNS |
| `connect ECONNREFUSED` / `ECONNRESET` / `ETIMEDOUT` | The host is reachable but nothing accepts the connection, or a firewall rejects it |
| `403` / an HTML block page from a proxy or WAF | The tool is public but only accepts listed IP addresses |
| `self-signed certificate` / `unable to verify the first certificate` | The tool uses an internal certificate authority |

The step's **Response** has no body from the tool in the first cases: the request never
arrived. Ask the user where the tool is hosted and what address they use from their own
network.

## The two ways to reach it

### 1. Make it reachable from Mindflow, restricted to Mindflow's addresses

For a tool that can be exposed on the internet (or already is, behind a firewall):
- Publish it (or a reverse proxy in front of it) on a public address with HTTPS.
- **Allow only Mindflow's outbound IP addresses** in the firewall, WAF or the tool's own
  IP allowlist. These addresses are fixed per Mindflow platform deployment: ask Mindflow
  support or the customer's Mindflow contact for them; don't guess.
- Keep authentication on the tool (API key, OAuth…): the IP allowlist is a second layer,
  not a replacement.
- Then point the step (or the credential's Custom URL) at the public address.

Best when the tool is a SaaS with an IP allowlist, or when the company already publishes
services through a DMZ or reverse proxy.

### 2. Relay Agent (no inbound opening)

For tools that must stay private. Mindflow's **Relay Agent** is a **Cloudflare Tunnel
(`cloudflared`)** connector that the customer runs inside their network:
- It connects **outbound only** to Cloudflare; no inbound firewall port is opened.
- Only the internal services mapped to it are exposed to Mindflow.
- The host needs at least **4 CPU cores and 4 GB RAM**, with outbound **port 7844 (TCP
  and UDP)** allowed (443 optional).
- It's **set up together with Mindflow** (there's no self-service screen in the Studio):
  the user contacts Mindflow support or their Mindflow contact to get the connector
  configuration and the addresses to use in their steps.

Best for on-prem tools, private data centres and internal-only DNS names.

## Certificates

If the tool uses an internal certificate authority, fixing the route isn't enough: the
TLS check will fail. Prefer a certificate the public trust store accepts (on the reverse
proxy or the tunnel's public side). Some steps have an option to allow insecure
certificates; use it only for tests, never in production.

## How to answer the user

1. Confirm it's a reachability problem (error type above, no response body).
2. Ask where the tool lives and whether it may be exposed with an IP allowlist.
3. Recommend option 1 if it can be exposed, option 2 if it must stay private, and say
   that both involve their network team and, for the Relay Agent, Mindflow.
4. Meanwhile, they can keep building with the step **mocked** (Mock tab) so the rest of
   the flow is ready when the route opens.
