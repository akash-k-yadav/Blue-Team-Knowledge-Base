# RDP Brute Force

## What is RDP?
- Remote Desktop Protocol (RDP) is a proprietary protocol developed by Microsoft
that provides a user with a graphical interface to connect to and control another
computer over a network connection. It enables users to access applications,
files, and desktops on a remote device as if they were physically sitting in
front of it.
- Generally used by administrators for administrative tasks or IT support for
troubleshooting.
- Attackers can use RDP to blend in and avoid detection by masquerading as
legitimate remote administrative activity.

## Why Attackers Use It
- A successful RDP brute force gives the attacker a fully interactive session
on the compromised machine, indistinguishable from legitimate remote
administrative activity.
- RDP is enabled on almost every Windows machine in enterprise environments
and many organizations expose it directly to the internet, creating a large
attack surface that requires no victim interaction.
- RDP is so commonly abused as a ransomware entry point that it is informally
referred to by defenders as **"Ransomware Deployment Protocol."** Groups
behind Ryuk, Dharma, and SamSam all relied heavily on exposed RDP as their
primary initial access vector.

### What is RDP Brute Force?
- If the RDP service is exposed to the internet there is a high chance it is
being targeted by botnets.
- In RDP brute force, botnets try common usernames (admin, support,
administrator) and common or leaked passwords against the exposed service in
order to gain an initial foothold.
- This generates noise , multiple failed login attempts with logon type 3 or 10
in a short time period from the same IP address.

## Detection

### Event IDs to Monitor
| Event ID | Source | Description |
|---|---|---|
| 4625 | Windows Security | Failed logon attempt, logon type 3 or 10 |
| 4624 | Windows Security | Successful logon attempt |
| 4778 | Windows Security | RDP session reconnected |
| 4779 | Windows Security | RDP session disconnected |
| 1 | Sysmon | Process creation (post-compromise) |

### Suspicious Indicators
- Multiple failed logon attempts (4625) using common usernames in a short time
period with logon type 3 or 10 from the same external IP is a strong indicator
of an RDP brute force attempt.
- Multiple failed logon attempts (4625) in a short time period followed by a
successful logon (4624) from the same IP indicates a successful RDP brute force.
- Logon type 10 specifically indicates an interactive remote logon via RDP,
making it more specific than logon type 3 for RDP identification.

### Correlation Framework
**Step 1:-** Look for failed logon attempts (4625) with logon type 3 or 10
originating from external IPs.
- Note the source IP address and check its reputation on VirusTotal and WHOIS.
- Note the usernames being attempted. Common usernames (admin, administrator,
support) in rapid succession confirm brute force behavior.

**Step 2::-** Filter for successful logon (4624) with logon type 10 from the
same flagged IP.
- Analyze the timeline , failed attempts followed closely by a success confirms
a successful brute force.
- Note the username, logon ID, and timestamp.

**Step 3:-** Retrieve information about the compromised account , its privileges,
owner, and group memberships. A high-privilege account being compromised
significantly increases severity.

**Step 4 :- Post-Compromise:**
- Filter process creation events (Sysmon Event ID 1) correlated to the noted
logon ID to identify what actions the adversary performed after gaining access.
- Look for reconnaissance commands (whoami, net user, ipconfig, nltest),
lateral movement tools, or any persistence mechanisms being established.

**Step 5:** Isolate the host and the compromised account if you have the
authority to do so, or escalate to a senior analyst for deeper investigation.

### Valid Credential Based RDP Access
- If the adversary already has valid credentials obtained through phishing or
credential harvesting from leaked databases, the attack will not generate
failed logon noise, making it significantly harder to detect.

**Detection Approach:**

**Step 1:** Hunt for successful logon (4624) with logon type 10 from external
IP addresses. Any external IP with a successful RDP session that is not a
known admin IP should be treated as suspicious.

**Step 2:** Check the reputation of the source IP:
- VirusTotal: https://www.virustotal.com
- WHOIS: https://who.is

**Step 3:** Check if the IP is a known proxy, VPN, or Tor exit node:
- ip2proxy: https://www.ip2proxy.com
- Spur: https://spur.us

- If the IP is flagged as a proxy or VPN it is a strong indicator of
compromise, as attackers use these to mask their true location or make
their geolocation appear consistent with the victim organization's region.

**Step 4:** Check the geolocation of the source IP and compare it against
the legitimate user's known login locations. A successful RDP logon from
a geography inconsistent with the user's normal pattern .for example,
the account is regularly used from Mumbai but this session originates from
Russia , is a strong indicator of compromise even with no failed attempts
preceding it.