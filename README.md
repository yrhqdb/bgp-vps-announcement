# vps with bgp sessions: how to announce your own IP prefixes from a VPS, what the session really costs, and which plans are worth it

Running a BGP session from a VPS is how you announce your own IPv4/IPv6 prefixes to the internet without renting a rack, a router, or a colo cage. You bring an ASN and (usually) your own IP space, the provider gives you an eBGP session from their edge router to your VM, and your prefixes start following you across reboots, provider changes, and even data center moves.

The catch: providers differ wildly on whether BGP costs extra, what prefix sizes they'll actually carry, and how much of the setup is self-service. Below is what we could verify about V.PS (the brand behind the vps.hosting site, owned and operated by xTom OÜ), plus how it stacks up against the providers that publicly document their BGP offerings.

> **Quick picks**
> - Most BGP VPS buyers want a solid base VM plus a session they can attach later. The V.PS 4-core/4 GB/40 GB plan at **€9.95/mo** with 1 TB traffic on a 1 Gbps port is the best value in their current lineup, and a BGP session can be added to it as a paid extra.
> - If you don't own an ASN or prefixes at all, a standard VPS plus a managed BYOIP-style arrangement will get you further than a raw session. Virtua offers exactly that on every plan.
> - 👉 [Check V.PS plans and current pricing](https://vps.hosting/?affid=1666)

## What "VPS with BGP sessions" actually means

A BGP VPS gives you an eBGP session between your VM and the provider's routers, over which you advertise your own prefixes. Instead of using the provider's IP addresses, traffic for *your* prefixes routes to *your* VM. That's the whole point: the IPs are yours, so you can move workloads between providers, run anycast, keep your mail/API sender reputation to yourself, or build a small ISP-style service.

Three things every provider handles differently, and each one affects whether a plan works for you:

1. **ASN requirement.** Most providers verify your ASN through IRR objects before the session goes live. Some are stricter than others: Catalyst Host rejects private, reserved, and documentation ASNs outright. Pufferfish Host takes the opposite approach — if you don't have an AS number, they'll issue a private AS for you, as long as you provide a Letter of Authority for the space. Virtua sits in the middle with a "managed BYOIP" mode that needs no ASN from you at all: they announce your prefixes under their own AS35661, fully automated.
2. **Prefix size.** This one trips up more people than ASN ownership does. Announcing anything shorter than a /24 (IPv4) or /48 (IPv6) is pointless — filters across the internet will drop it, so Virtuasys requires /24 and /48 minimums, and Catalyst Host only propagates IPv4 /19–/24 and IPv6 /32–/48 toward transit, even though their portal lets you register longer ranges. If your allocation is a /25 or shorter, no provider on this list can make it globally reachable.
3. **RPKI.** Every provider here enforces it. Your prefix needs a valid ROA before it will be accepted — Catalyst validates each prefix at add time and rejects anything without a matching ROA, and Pufferfish warns that their upstreams can take up to 24 hours to pull updated RPKI data, during which your prefix simply won't work.

## V.PS plans and pricing (verified)

V.PS runs KVM virtualization on Proxmox VE, publishes a 99.9% SLA, and operates 11 data centers: Singapore, Tokyo, Osaka, Hong Kong, Sydney, Amsterdam, Frankfurt, London, Düsseldorf, Tallinn, and San Jose. Every VPS includes 1 TB of traffic on a 1 Gbps port, counted in both directions.

The core Linux plans, all billed monthly in EUR:

| Plan | CPU / RAM / Storage | Traffic / Port | Price | BGP session | Order |
| --- | --- | --- | --- | --- | --- |
| Starter | 2 cores / 1 GB / 20 GB NVMe | 1 TB / 1 Gbps | €6.95/mo | Paid add-on | [ Order Starter](https://vps.hosting/?cmd=cart&action=add&id=91&affid=1666) |
| 2-core / 2 GB | 2 cores / 2 GB / 30 GB NVMe | 1 TB / 1 Gbps | €7.95/mo | Paid add-on | [ Order 2 GB plan](https://vps.hosting/?cmd=cart&action=add&id=105&affid=1666) |
| Pro | 4 cores / 4 GB / 40 GB NVMe | 1 TB / 1 Gbps | €9.95/mo | Paid add-on | [ Order Pro](https://vps.hosting/?cmd=cart&action=add&id=106&affid=1666) |
| 8-core / 8 GB | 8 cores / 8 GB / 80 GB NVMe | 1 TB / 1 Gbps | €19.95/mo | Paid add-on | [ Order 8 GB plan](https://vps.hosting/?cmd=cart&action=add&id=107&affid=1666) |

The official documentation also references other product lines — Mini and Mini Pro plans (500–600 GB traffic on 400–500 Mbps ports), Storage KVM VPS (starting at 5 TB traffic), Performance KVM VPS, and an Edge KVM line that ships with one backup slot. Those lines have their own pricing pages, and Edge/Performance plans include 1–2 backup slots each.

One thing worth knowing before you click order on any of these: **V.PS does not take backups of your data by default.** The FAQ states this plainly. If the VM holds anything you care about, set up your own offsite backups. A 2024 hands-on review of the service recommends doing exactly that, using weekly backups pushed to external storage.

## What the BGP session itself costs

On V.PS, the BGP session is a paid network add-on, not a bundled feature. The official FAQ lists it alongside IXP ports (LocIX, DE-CIX, AMS-IX) as purchasable extras, available in locations including Düsseldorf, Frankfurt, Amsterdam, Tallinn, London, San Jose, Tokyo, Osaka, Hong Kong, and Sydney, each with additional charges. A hands-on review published in August 2024 reports the BGP session add-on at **€50 setup**. Treat that figure as reported rather than official — confirm the current amount on the order page before committing.

Compare that to the two providers that publish their BGP pricing openly:

- **Virtua** includes BGP sessions on every VPS plan, starting from €5.00/mo. Their stated logic: the sessions are fully automated from the customer panel, set up in minutes, redundant by default, and they support FRR, Quagga, and BIRD with ready-made configs, plus one-click VyOS, pfSense, and Mikrotik CHR installs. Managed BYOIP is also free.
- **Catalyst Host** includes BGP sessions and BYOIP only on their vDedi plans, starting at $29/mo — no per-session or per-prefix fees beyond that, and no setup fee. Their Starter VPS line at $10–18/mo does not include BGP.
- **Virtuasys** doesn't publish prices at all. They quote per configuration (vCPU/RAM, NVMe, port speed, extra IPv4, BGP setup) and return a firm quote within one business day, claiming no resale margin since they operate their own network, AS35661.

There's a second cost layer people forget: **the IP space itself.** If you don't own prefixes, V.PS sells extra single addresses (a /32 IPv4 and /48 IPv6 with setup and monthly fees) — fine for a static IP, useless for BGP announcements, which need at least a /24. Catalyst rents /24 blocks at $1 per IP per month, which works out to $256/mo for a full block. Virtua's managed BYOIP lets you rent space from IPXO and have them announce it. Whichever route you take, the IPs usually cost more than the VM.

If you've decided the V.PS Pro tier fits — 4 cores, 4 GB, 40 GB NVMe at €9.95/mo with the session added on top — 👉 [order the Pro plan here](https://vps.hosting/?cmd=cart&action=add&id=106&affid=1666).

## The limits that only show up in the docs

These are the details that rarely make it into comparison articles, and several of them change how usable a BGP VPS actually is.

**V.PS operational restrictions** (from their official FAQ):

- Outbound port 25 is blocked by default. If you plan to run your own mail server on your announced IPs — a common reason people chase IP reputation through BGP — you need to open a ticket or use a third-party email service.
- Only one IPv4 and one IPv6 come with each VPS. Changing your IPv4 is free once within the first 24 hours, then €8 per change, max one per month.
- Plans bought at a discount can't be upgraded at the discount price, and cross-location migration doesn't exist — you buy fresh in the target city, move data, then cancel.
- A VPS with an active BGP or IXP service can't be transferred to another account, even after you cancel the service. If you resell or hand off servers, this matters.

**Cross-provider limits worth knowing:**

- Catalyst caps each session at a max-prefix of 100 IPv4 and 50 IPv6 routes, allows 100 prefixes per bulk-add batch, and revalidates RPKI only once per 24 hours per ASN.
- Pufferfish's BGP feature is currently limited to their Germany location; elsewhere you have to ask support.
- Senko Digital's published rules forbid using the session as transit for someone else's networks, and they say they'll shut sessions down over it. Expect similar terms elsewhere — a BGP session is for announcing *your* space.

None of these are dealbreakers, but the port 25 block plus the no-default-backup policy on V.PS are the two I'd weigh against the low base prices. 👉 [Check the Starter plan if you want to test the platform cheaply](https://vps.hosting/?cmd=cart&action=add&id=91&affid=1666) before adding a session to anything.

## What the setup process actually looks like

Once you have a plan, an ASN, and prefixes with valid ROAs, the flow looks roughly like this at every provider:

1. **Verify ASN ownership** — usually automatic via IRR objects. Catalyst does this at session creation; Virtuasys handles LOA, IRR route objects, and RPKI validation through their in-house LIR team; Virtua's ASN registration service can register one for you via RIPE if you're starting from zero.
2. **Create the session** — Catalyst's portal lets you set an MD5 secret (up to 126 characters), AS-prepend (0–3 times), and communities, then add your VPS as a peer. Pufferfish requires a ticket and gives you two redundant route servers to peer with.
3. **Configure your daemon** — BIRD 2, FRR, or OpenBGPD. Every provider here publishes config examples; Catalyst embeds a copy-paste BIRD 2.x guide in the portal. On Pufferfish you'll also bind your prefixes to a dummy interface in `/etc/network/interfaces` so the kernel accepts traffic for them.
4. **Wait.** This is the part nobody warns you about. A 2024 first-person writeup of the process documents that full internet propagation can take up to 72 hours — until then you can't ping your announced addresses or even use them as a source IP. The NLNOG looking glass is the standard tool for watching your routes crawl across the table.

For redundancy, peer the same prefix from two or more VMs. Pufferfish documents that same-region announcements load-balance across all your servers and fail over automatically when one dies; add AS-prepend on the one you want deprioritized. Catalyst goes further, suggesting private VLANs between nodes for keepalived/VRRP setups, and supporting blackholing (RTBH) so a single attacked address can be null-routed with a community string instead of dragging down the whole prefix.

## How the providers compare

| Provider | BGP cost | Automation | Minimum prefixes | ASN flexibility |
| --- | --- | --- | --- | --- |
| V.PS | Paid add-on (€50 setup reported) | Ticket-based add-on | Single IPs only; BGP needs your own /24+/48 | Standard |
| Virtua | Free on all plans (from €5/mo) | Fully automated, minutes; managed BYOIP needs no ASN | /24+/48 for announcements | ASN registration service; managed BYOIP alternative |
| Catalyst Host | Included on vDedi ($29/mo+) | Self-service portal, team review | /19–/24 IPv4, /32–/48 IPv6 reach transit | Own ASN required, verified via IRR |
| Virtuasys | Quote-based | LIR team handles LOA/IRR/RPKI | /24 IPv4, /48 IPv6 | RIPE ASN sponsorship available |
| Pufferfish Host | Not published | Ticket setup, dual route servers | Own prefixes required | Private AS offered with LoA |

Honest read: if BGP is the entire reason you're buying, Virtua's free, automated sessions on a €5 base plan are hard to beat on cost, and their managed BYOIP removes the ASN barrier completely. V.PS makes more sense when you primarily want the VM — the €9.95 four-core tier is competitive per-spec across the EU and Asian locations — and BGP is a secondary add-on. Catalyst is the pick for network engineers who want granular portal control (prepend, communities, MD5, blackholing) and are fine paying $29/mo for it.

## FAQ

**Do I need my own ASN for a BGP VPS?**
Usually yes, and it must be verifiable — Catalyst rejects private and documentation ASNs, and most providers check IRR records. Exceptions: Pufferfish issues private AS numbers with a Letter of Authority, Virtua can register an ASN for you via RIPE, and Virtua's managed BYOIP skips the ASN entirely by announcing your space under theirs.

**Can I announce a prefix shorter than /24?**
No, not usefully. Global BGP filters drop IPv4 prefixes longer than /24 and IPv6 longer than /48, so anything smaller stays unreachable regardless of provider. Catalyst additionally only carries /19–/24 (IPv4) and /32–/48 (IPv6) toward transit.

**How long until my prefixes are visible on the internet?**
Expect up to 72 hours for full propagation after the session comes up, based on a documented 2024 setup walkthrough. RPKI changes can add up to another 24 hours of delay at some providers, during which the prefix won't route at all.

**Is a BGP VPS worth it for just one server?**
Rarely. A single session on a single VM is a single point of failure — if the VPS reboots, your announcement vanishes with it. It starts making sense when you peer the same prefix from two or more servers, run anycast, or need portable IP space across providers. If you just want one stable IP for email reputation, buying an extra address is cheaper than the whole BGP apparatus.

## Before you order

- Own an ASN and a /24 (or larger) with valid ROAs, and you're a genuine BGP case → pick the provider whose session terms fit your workflow: V.PS if you want the VM value first, Virtua if you want the session free and automated, Catalyst if you want portal-level control over prepends and communities.
- Don't own IP space yet → decide between renting a /24 (Catalyst at $1/IP/mo, IPXO rental via Virtua's managed BYOIP) or abandoning BGP and buying extra single IPs instead. On V.PS the latter is the cheaper path.
- On V.PS specifically, plan your backups from day one (nothing is backed up by default), and remember a BGP-carrying VPS can never be transferred to another account.

If that matches what you need, 👉 [open the V.PS order page and pick your plan](https://vps.hosting/?affid=1666) — the session add-on can be attached when you configure the server.
