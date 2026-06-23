## 03 - Network Connectivity Troubleshooting

A teller called the helpdesk reporting they could not reach any websites or the core banking application. Their machine showed a valid network connection but nothing was loading. The task was to isolate the root cause using a structured troubleshooting methodology — testing from the network layer up before escalating.

---

## Step 1: Test Basic IP Connectivity

`ping 8.8.8.8` was run first to test whether the machine could reach the internet by IP address, bypassing DNS entirely. If this fails the issue is at the network or gateway level. If it succeeds, the network path is intact and DNS is the likely cause.

```cmd
ping 8.8.8.8
```

All 4 packets returned successfully — basic connectivity confirmed. The network and default gateway are not the issue.

<img width="1918" height="790" alt="ping 8888" src="https://github.com/user-attachments/assets/70ba8570-5a63-4b97-979e-d7a6e6e070d2" />


---

## Step 2: Test DNS Resolution

`ping google.com` was run to test whether the machine could resolve a hostname to an IP address. Unlike pinging by IP, this command goes through the DNS server first. A successful result means DNS is working.

```cmd
ping google.com
```

google.com resolved to 142.250.176.78 and returned 4 successful replies — DNS resolution is functioning correctly on this machine.

<img width="1918" height="861" alt="ping google" src="https://github.com/user-attachments/assets/d56fbef8-051b-4bfd-a26a-6fdb9d934625" />


---

## Step 3: Identify the DNS Server with nslookup

`nslookup` was used to identify exactly which DNS server the machine is querying and whether it is returning valid responses. This is more specific than ping — it shows the DNS server address, the resolved name, and the returned IP addresses.

```cmd
nslookup google.com
```

The server returned Unknown for the server name but successfully resolved google.com — this indicates the reverse DNS lookup for the DNS server itself did not resolve, but forward resolution is working correctly.

<img width="1918" height="800" alt="nslookup google" src="https://github.com/user-attachments/assets/2cf43b52-fb6a-4e00-847f-abf254e8e485" />


---

## Step 4: Check Full Network Configuration with ipconfig /all

`ipconfig /all` was run on Client01 to verify the machine's full network configuration — IP address, subnet mask, default gateway, DHCP server, and DNS server assignment.

```cmd
ipconfig /all
```

Key values confirmed:
- **IPv4 Address:** 192.168.10.101
- **Default Gateway:** 192.168.10.5 (DC01)
- **DNS Server:** 192.168.10.5 (DC01)

In an Active Directory environment the domain controller serves as the DNS server because AD depends on DNS to locate domain resources. This confirms the machine is using the correct DNS server.

<img width="1672" height="832" alt="ipconfig command to confirm dns server" src="https://github.com/user-attachments/assets/b66db251-3102-4f82-b7a6-79487cdbec8e" />


---

## Step 5: Verify DNS Forwarders on DC01

`Get-DnsServerForwarder` was run on DC01 to confirm that forwarders are configured. Forwarders tell the DNS server where to send queries it cannot answer locally — for example, external names like google.com that are outside the mydomain.com zone.

```powershell
Get-DnsServerForwarder
```

DC01 is configured with 8.8.8.8 and 8.8.4.4 as forwarders — external DNS queries are being forwarded to Google's public DNS servers correctly.

<img width="1876" height="751" alt="retrieved dns server forwarder" src="https://github.com/user-attachments/assets/b792778d-76e9-4213-84d8-59395f1c586e" />


---

## Step 6: Verify the DNS Service Status

`Get-Service DNS` was run on DC01 to confirm the DNS Server service is running. A stopped DNS service would cause all name resolution to fail for every client on the domain — not just one user.

```powershell
Get-Service DNS
```

Status returned Running — the DNS service is healthy and the issue is not a DNS service outage.

<img width="1896" height="857" alt="Get-service Dns" src="https://github.com/user-attachments/assets/5c955dba-a21b-44b6-b37f-b656d351791d" />


---

## Escalation Note

In this lab all checks passed — basic connectivity, DNS resolution, forwarders, and DNS service status were all healthy. In a real scenario where DNS resolution was failing despite all these checks passing, the next step would be a **Tier 2 escalation to the network team**. The suspected cause would be a firewall blocking outbound DNS traffic on port 53, which sits outside helpdesk scope.

---

## Summary

| Step | Command | Purpose |
|---|---|---|
| Test IP connectivity | ping 8.8.8.8 | Confirms network path and gateway — bypasses DNS |
| Test DNS resolution | ping google.com | Confirms DNS is resolving hostnames |
| Identify DNS server | nslookup google.com | Shows which DNS server is being queried |
| Check network config | ipconfig /all | Confirms IP, gateway, and DNS server assignment |
| Verify forwarders | Get-DnsServerForwarder | Confirms DC01 can forward external DNS queries |
| Verify DNS service | Get-Service DNS | Confirms DNS Server service is running on DC01 |

Always work from the network layer up — confirm IP connectivity before testing DNS, and confirm DNS before investigating application-level issues. If all network and DNS checks pass, escalate to Tier 2 for firewall investigation.
