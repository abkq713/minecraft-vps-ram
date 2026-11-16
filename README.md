# Minecraft VPS: how much RAM you need, what it costs, and how to set one up (full Vultr plan table and $300 credit terms)

A vanilla world for two to five friends runs on 2 GB of RAM. A Create-heavy modpack for eight people wants 8 GB or more. That gap is why "how much do I need" has no single answer, and it's also the difference between $12 and $48 a month.

On Vultr the floor is a **$5/month** instance with 1 vCPU, 1 GB RAM, 25 GB SSD and 1 TB of transfer, billed at $0.007/hour. A 4 GB High Performance instance at **$24/month** is where a small group stops fighting the server. The 8 GB tier at **$48/month** is the first size that carries a modpack without constant garbage-collection stutter. What follows is how to choose between those, what hourly billing actually saves you, and the commands to have a server running tonight.

> **Most readers:** 2–5 friends on vanilla or Paper fit on the 4 GB High Performance instance at $24/month. Past ten players, or once a modpack is involved, move to 8 GB at $48/month. Both bill hourly, so a test evening costs cents.
> 👉 [Compare Vultr's current plans and hourly rates](https://www.vultr.com/pricing/?ref=9738262-9J)

## Vultr plans that suit a Minecraft server

| Instance | RAM | vCPU | Storage | Price | Buy |
| --- | --- | --- | --- | --- | --- |
| Regular Cloud Compute | 1 GB | 1 | 25 GB SSD | $5.00/mo | [Cheapest that boots](https://www.vultr.com/pricing/?ref=9738262-9J) |
| High Performance | 4 GB | 2 | 100 GB NVMe | $24.00/mo | [4 GB plan](https://www.vultr.com/pricing/?ref=9738262-9J) |
| High Performance | 8 GB | 4 | 180 GB NVMe | $48.00/mo | [8 GB plan](https://www.vultr.com/pricing/?ref=9738262-9J) |
| High Performance | 12 GB | 4 | 260 GB NVMe | $72.00/mo | [12 GB plan](https://www.vultr.com/pricing/?ref=9738262-9J) |
| High Performance | 16 GB | 8 | 350 GB NVMe | $96.00/mo | [16 GB plan](https://www.vultr.com/pricing/?ref=9738262-9J) |
| High Performance | 24 GB | 12 | 500 GB NVMe | $144.00/mo | [24 GB plan](https://www.vultr.com/pricing/?ref=9738262-9J) |

Vultr also lists smaller and larger sizes than these, including a 2 GB High Performance instance at $12/month and a 2 GB Regular instance at $10/month. Every size bills hourly, so the monthly figure is what you pay only if the server runs the whole month. Prices move; confirm on the pricing page before you order.

Two things about this lineup matter more than the price column. High Performance instances run on current-generation Intel Xeon or AMD EPYC cores with NVMe disks, while Regular instances use older cores and plain SSDs. For a game that lives or dies on single-thread speed, that difference is worth more than an extra vCPU.

## How much RAM does a Minecraft VPS need?

Hosting companies disagree, and the disagreement tracks what they sell. Liquid Web's guide sets the floor at a dual-core 3.0 GHz CPU, 4 GB of RAM, 10–20 GB of SSD and 10 Mbps of bandwidth, with 6–8 GB for modded servers. Gandi recommends 4 GB for five players. Cloudzy's FAQ says vanilla for a few friends is comfortable on 2 GB, modpacks want 4–8 GB, and a busy Spigot or Paper network wants 8–16 GB.

None of those is wrong; they describe different servers. Sorting by workload instead of by player count gets you closer:

- **Vanilla or Paper, whitelisted, 2–5 friends.** 2 GB works with a view distance around 8. 4 GB removes the anxiety and costs $24/month on Vultr. The $5 instance will boot a vanilla server for two or three players if you cut view and simulation distance, but it is not a comfortable long-term home.
- **5–20 players on Paper with a handful of plugins.** 8 GB. Plugins that render maps or run economies add memory on top of the base game.
- **A modpack (ATM, FTB, anything with 200+ mods).** Start at 8 GB and expect to need 12–24 GB once five or more people log in simultaneously. Light packs run on 4 GB.
- **A public network with BungeeCord or Velocity in front.** Budget 8–16 GB per Paper backend, and give each backend its own instance and IP rather than stacking them on one box.

Player count is the weakest of the three variables. View distance, simulation distance, how much of the map is pre-generated, and how many entities your farms keep loaded will move memory use by gigabytes on servers with identical player counts. If TPS drops after you build a gold farm, the fix is usually a setting, not a bigger plan.

👉 [Check what the 8 GB tier costs right now](https://www.vultr.com/pricing/?ref=9738262-9J)

## Single-core speed decides your TPS, not vCPU count

Minecraft's main thread is largely single-threaded, so per-core speed matters more than how many cores you rent. That is the single most useful sentence in any host's documentation, and it changes how you read a plan table: a 4 vCPU instance on older cores can tick worse than a 2 vCPU instance on newer ones.

Applied to Vultr, it means choosing High Performance over Regular at the same RAM before you choose more RAM. It also means the $72 12 GB plan, which has the same 4 vCPU as the $48 8 GB plan, is only worth the extra $24 if a modpack genuinely needs the memory. Paying for cores Minecraft won't use is the most common way people overspend on a Minecraft VPS.

## Setting up a Minecraft server on a VPS

Vultr's Marketplace includes a Minecraft app that takes server type (vanilla, Paper, Forge, Fabric or Bedrock) and memory allocation as deploy-time variables. Take that route and you can skip to step 7. The manual route takes about half an hour.

1. Deploy Ubuntu 24.04 in the region closest to most of your players. Latency is the one problem more RAM cannot fix.
2. Connect: `ssh root@your-server-ip`
3. Update the system: `sudo apt update && sudo apt upgrade -y`
4. Install Java: `sudo apt install openjdk-21-jre-headless -y`. Java 17 or newer is required by current server builds; 21 is the safe default.
5. Create a directory and drop the jar in: `mkdir ~/minecraft && cd ~/minecraft`, then download `server.jar` from Mojang or from your mod loader's site.
6. Run it once so it generates config: `java -Xmx4G -Xms4G -jar server.jar nogui`. It will exit and write `eula.txt`.
7. Accept the EULA by setting `eula=true` in that file, then edit `server.properties` for `view-distance`, `simulation-distance`, `max-players` and `online-mode`.
8. Open the ports: `sudo ufw allow 25565/tcp` for Java Edition, plus `sudo ufw allow 19132/udp` if you run Bedrock or GeyserMC.
9. Start it inside a persistent session so it survives logout: `screen -S mc java -Xmx4G -Xms4G -jar server.jar nogui`.
10. Add Aikar's JVM flags, the de facto G1GC tuning for Paper and Spigot, and schedule a nightly backup that copies the world folders off the machine.

Set `-Xmx` about 1 GB below the plan's RAM so the operating system and Java itself have room. Connecting from the client uses the instance's IP and port 25565.

## What it really costs, and where the $300 credit goes

Vultr bills by the hour, and a month is counted as 672 hours. The $5 instance works out to $0.007/hour, which is what makes short-lived servers practical: spin up a 24 GB box for a weekend event, destroy it Sunday night, pay for two days.

**Stopping an instance does not stop billing.** Charges begin when the server is deployed and continue while it is powered off. You have to destroy the instance to end the meter. This is the most expensive misunderstanding in VPS hosting and it's written plainly in Vultr's billing documentation.

Automated backups are optional and cost extra. Vultr's refund policy does not cover services, and promotional credit is non-refundable and expires.

The $300 offer is a credit for new accounts, not a discount on the $5 plan. On Vultr's coupons page the code **FLY300VULTR** grants $300 of free credit to new customers, with a card or PayPal linked, one promotional code per account, select products only, and no combining with other offers. Unused credit expires, and the offer terms we checked put that at 30 days. That expiry is the part worth doing arithmetic on: a 30-day window caps what you can consume, so a $48/month server captures roughly $48 of the $300 before the balance lapses. The credit is worth most to someone running a large instance flat out for a month, or several of them. Other codes on the same page include VULTRMATCH (dollar-for-dollar match on your first deposit, up to $100), FLYTWOHUNDRED, and 250VULTRFLY.

👉 [Check the current $300 credit terms and code requirements](https://www.vultr.com/coupons/?ref=9738262-9J)

## How Vultr compares with Cloudzy, LightNode and Liquid Web

These are vendor-stated prices collected while writing, and they change.

| Provider | Plan | Spec | Price | Notes |
| --- | --- | --- | --- | --- |
| Cloudzy | 2 GB | 1 vCPU, 60 GB NVMe, 3 TB | $7.48/mo (page showed 50% off; list $14.95) | 13 regions, hourly billing, 14-day money-back |
| Cloudzy | 4 GB | 2 vCPU, 120 GB NVMe, 5 TB | $14.48/mo | Same terms |
| LightNode | 2 GB | 1 high-frequency vCPU, 50 GB NVMe, 1 TB | $7.71/mo | 40+ locations, hourly billing, 99.95% stated uptime |
| Liquid Web | 2 GB | 1 vCPU, 60 GB, 2 TB | $11/mo | Monthly pricing shown on the page |
| Liquid Web | 4 GB | 2 vCPU, 80 GB, 3 TB | $17/mo | Same |
| Liquid Web | 8 GB | 4 vCPU, 240 GB, 5 TB | $45/mo | Same |

Cloudzy and LightNode are cheaper per gigabyte of RAM and both bill hourly, which suits a small vanilla server on a tight budget. The Cloudzy discount is a promotional state rather than a standing price, so budget for the crossed-out figure if you stay past the promo. Liquid Web's monthly-only pricing makes less sense for a server you might run for three weeks. Vultr's advantage is the tier structure: when single-core speed matters more than RAM, you can buy newer cores without buying a bigger plan.

## Who should not rent a VPS for Minecraft

If you want a control panel with one-click modpack installs, automated backups and no SSH, a managed Minecraft host costs more per gigabyte but removes the sysadmin work entirely. A VPS gives you root and nothing else.

If the server is for a two-week holiday session, pay hourly and destroy it afterwards rather than prepaying a year. And if the group is two people who play vanilla on weekends, the $5 instance with reduced view distance will do the job; the 24 GB tier exists for modpacks, not for prestige.

## Picking a plan in one minute

- Two to five friends, vanilla or Paper: **4 GB High Performance, $24/month**.
- Five to twenty players, plugins but no modpack: **8 GB High Performance, $48/month**.
- Any modpack at all: start at 8 GB. Five or more concurrent players on a heavy pack: 12–24 GB.
- Testing a pack or hosting a weekend event: deploy hourly, destroy it when you're done.

Before you pay, confirm two things: how long the $300 credit stays valid on your account, and that you understand destroy-versus-stop billing. Those two facts decide whether the cheap month stays cheap.

👉 [Pick your size and deploy in the region closest to your players](https://www.vultr.com/pricing/?ref=9738262-9J)
