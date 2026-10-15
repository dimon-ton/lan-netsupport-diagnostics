---
name: lan-netsupport-diagnostics
description: Diagnose authorized LAN clients and network inventory, including router WAN health, NetSupport School remote diagnostics, and false Windows "No Internet" status.
---

# LAN NetSupport Diagnostics

Use this skill when the user asks to diagnose LAN computers that cannot reach the Internet, inventory an authorized local network, or investigate a false Windows "No Internet" indicator. It supports environments that use a router web interface and NetSupport School for authorized remote administration.

## Scope and access

- Treat router credentials and NetSupport access as authorization only for the routers and clients the user names.
- Do not attempt to retrieve Windows credentials. A router login does not grant Windows access.
- For inventory, scan only the explicitly authorized subnet or the local connected subnet derived from the active interface. Do not perform credential guessing, login attempts, or configuration changes. Documented factory credentials may be reported only after identifying the exact model; do not test them.
- Use read-only network checks first. Ask before applying a fix that changes client, router, proxy, firewall, DNS, or registry settings.
- Never take over, disconnect, or interrupt an active Tutor session. Use a NetSupport Tech Console when available.

## Authorized network inventory

When the user requests a LAN scan or wants to find network equipment:

1. Derive the active IPv4 CIDR and default gateway. Start with the neighbor/ARP table, then use a bounded, rate-limited discovery sweep only when needed.
2. Use read-only identification: MAC/OUI vendor, ICMP response, and a small set of common management services. Do not treat missing ICMP as proof that a host is absent.
3. Identify device roles only from evidence. A gateway with a router/ONT web interface can be reported as such; an IP-addressable device with a management interface is not automatically a switch.
4. State that unmanaged switches usually have no IP address and cannot be found or confirmed by an IP scan. For a managed switch, require a vendor/model banner, management UI, SNMP evidence obtained with user-supplied access, or physical confirmation before declaring it a switch.
5. Report IP address, MAC/vendor when available, observed management services, likely role, confidence, and any uncertainty. Do not include passwords, serial numbers, session tokens, or unnecessary client details.

## Triage

1. Determine the local IPv4 subnet and default gateway. Use the neighbor/ARP table to identify devices that are present on the LAN.
2. Check router WAN status before changing any client. A healthy WAN connection has an active uplink, address, gateway, DNS, and no disconnect reason.
3. For a named client, test LAN reachability, its default gateway, a public IP such as `1.1.1.1`, and DNS resolution such as `example.com`. Interpret the results together:
   - gateway fails: client LAN, addressing, Wi-Fi, cable, or VLAN issue;
   - public IP fails while gateway works: router WAN or upstream issue;
   - public IP works but DNS fails: DNS or filtering issue;
   - both work: Internet is working for that client.

Avoid treating missing ICMP replies alone as failure; hosts and firewalls can block ping.

## Router checks

When browser automation is available, access only the user-provided gateway and log in with credentials explicitly supplied by the user. Capture the WAN connection state and connected-client/ARP entries for the requested IPs. Do not expose router serial numbers, passwords, session tokens, or other secrets in the result.

## NetSupport School

NetSupport School Clients normally listen on TCP port `5405`. An open 5405 service and an entry in the Tech Console establish that the client is available for authorized remote diagnostics.

- Prefer the NetSupport Tech Console (`nssadmui.exe`) over a Tutor session if a Tutor is already active for the room.
- Match named IPs to the Tech Console client list before opening a Remote Command session.
- Use Remote Command for read-only checks: `hostname`, `ipconfig /all`, gateway/public-IP pings, DNS lookup, and proxy inspection.
- A raw TCP connection or a binary banner is not a substitute for an authorized NetSupport session.

## False "No Internet" status

If gateway, public-IP, and DNS tests all succeed but Windows still says "No Internet," report that Internet access works and investigate Windows Network Connectivity Status Indicator (NCSI). Check whether the environment permits the Microsoft probes:

- DNS: `dns.msftncsi.com`
- HTTP: `http://www.msftconnecttest.com/connecttest.txt`

Also check for an unintended WinHTTP or user proxy and DNS/web filtering. Do not reset IP settings or replace gateway/DNS settings merely to clear the status icon. Apply a configuration change only after identifying the blocked probe or proxy and receiving authorization.

## Handoff

Report each client separately with LAN, public Internet, and DNS results. For inventory requests, use the device fields specified above. Distinguish an actual connectivity outage from an NCSI status-only problem. Flag duplicate hostnames because they can break name-based access even if Internet connectivity works.
