---
name: lan-netsupport-diagnostics
description: Diagnose Internet problems on LAN Windows clients, including router WAN health, client reachability, NetSupport School remote diagnostics, and false Windows “No Internet” status.
---

# Lan Netsupport Diagnostics

Use this skill when the user asks to diagnose LAN computers that cannot reach the Internet, or Windows clients that display a false “No Internet” indicator. It supports environments that use a router web interface and NetSupport School for authorized remote administration.

## Scope and access

- Treat router credentials and NetSupport access as authorization only for the routers and clients the user names.
- Do not attempt to retrieve Windows credentials. A router login does not grant Windows access.
- Use read-only network checks first. Ask before applying a fix that changes client, router, proxy, firewall, DNS, or registry settings.
- Never take over, disconnect, or interrupt an active Tutor session. Use a NetSupport Tech Console when available.

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

## False “No Internet” status

If gateway, public-IP, and DNS tests all succeed but Windows still says “No Internet,” report that Internet access works and investigate Windows Network Connectivity Status Indicator (NCSI). Check whether the environment permits the Microsoft probes:

- DNS: `dns.msftncsi.com`
- HTTP: `http://www.msftconnecttest.com/connecttest.txt`

Also check for an unintended WinHTTP or user proxy and DNS/web filtering. Do not reset IP settings or replace gateway/DNS settings merely to clear the status icon. Apply a configuration change only after identifying the blocked probe or proxy and receiving authorization.

## Handoff

Report each client separately with LAN, public Internet, and DNS results. Distinguish an actual connectivity outage from an NCSI status-only problem. Flag duplicate hostnames because they can break name-based access even if Internet connectivity works.
