# Overpass the Hash

## What is it
Overpass the Hash is a technique where a stolen NTLM hash is used to obtain a Kerberos ticket (TGT) from the domain controller, and the obtained ticket is then used to access resources or request service tickets.

### How Adversary Steals the Hash
There are a few ways an attacker can steal the NTLM hash representation of a password:
- **LSASS.exe Dumping:** using tools like Procdump or Mimikatz, the adversary can dump `lsass.exe`, which stores the hash representation for interactive logons
- **Sniffing Challenge-Response:** intercepting the challenge-response exchange during NTLM authentication and cracking it offline
- **Local Security Account (SAM):** extracting hashes from the SAM file

There are also other ways such as NTDS extraction, DCSync, etc.

## Why Attackers Use It
- Converts a stolen NTLM hash into a legitimate-looking Kerberos ticket, so it works against resources or environments that don't accept NTLM authentication at all
- Kerberos traffic blends in better than raw NTLM, especially in environments that specifically monitor or restrict NTLM usage
- Tickets can be requested with strong encryption (AES-256), which is harder to flag as suspicious than the RC4-based ticket a straight NTLM hash normally produces

## How the Attack Works
1. Attacker steals the NTLM hash using one of the methods above
2. Using the stolen hash and a tool such as Mimikatz, the attacker requests a TGT by sending an AS-REQ to the domain controller
3. The attacker obtains a TGT encrypted using RC4. This is possible because Microsoft supports RC4-HMAC-MD5-encrypted Kerberos tickets keyed off the NTLM hash
4. The attacker can also request a TGT encrypted with a stronger algorithm such as AES-256, depending on the adversary's tooling, tickets encrypted with AES-256 are harder to flag as suspicious
5. Once the TGT is obtained, the attacker issues service tickets as needed for their objective

## MITRE ATT&CK
Overpassing the hash uses a stolen NTLM password hash to authenticate as a user while also using that same hash to create a valid Kerberos ticket, bridging Pass the Hash into Kerberos-based Pass the Ticket.

**Note:** MITRE doesn't give Overpass the Hash its own sub-technique ID, it's documented as a variant under Pass the Ticket (T1550.003), which is why this file cites the same ID as the Pass the Ticket file.

- **ID:** T1550.003
- **Sub-technique of:** T1550
- **Tactic:** Lateral Movement
- **Platform:** Windows

## Detection

### Event IDs to Monitor
| Event ID | Source | Description |
|---|---|---|
| 4768 | Windows Security | Kerberos TGT was requested |
| 4769 | Windows Security | Kerberos service ticket was requested |
| 4624 | Windows Security | An account successfully logged on |
| 4648 | Windows Security | A logon was attempted using explicit/stolen credentials |
| 4672 | Windows Security | Special privileges assigned to new logon |

### Important Fields
- **Logon Type:** value `9` (NewCredential) on the source host is the main anomaly indicator, normal NTLM authentication uses Logon Type `3` or `2`
- **Logon Process:** identifies which trusted authentication component/module on the machine processed and validated the logon request, consistently shows as `seclogo` for pass the hash tools like Mimikatz
- **Authentication Package:** shows as `Negotiate` on the source host during the initial hash injection
- **GrantedAccess (Sysmon Event ID 10):** access rights requested against `lsass.exe`, values `0x1010` or `0x1038` indicate credential extraction access
- **Ticket Encryption Type (on 4768/4769):** `0x17` = RC4, `0x11` = AES128, `0x12` = AES256, RC4 on an account/environment that should be using AES is a stronger signal than the event ID alone

### Correlation Framework
- **Step 1:** on the source machine, filter for `4624` with Logon Type `9`, this is the primary trigger, since Overpass the Hash starts with the same hash injection as Pass the Hash
- **Step 2:** confirm the Logon Process field shows `seclogo` and Authentication Package shows `Negotiate`, both together with Logon Type `9` is a strong indicator rather than a false positive
- **Step 3:** check for a matching Sysmon Event ID `10` on the same source host around the same time, GrantedAccess `0x1010` or `0x1038` against `lsass.exe` confirms hash extraction took place (see [LSASS Dumping](../02_Credential_Access/01_LSASS_Dumping.md))
- **Step 4:** check whether `4648` is recorded on the source machine, this depends on which tool/technique is used, e.g. Mimikatz spawns a new process with the stolen credential
- **Step 5:** check `4672` on the source machine, if it fires for the logged-on user rather than the impersonated account, that's a key anomaly
- **Step 6:** pivot to the domain controller, look for `4768` (TGT request) or `4769` (service ticket request) for the impersonated account instead of `4776` (NTLM), this is what separates Overpass the Hash from plain Pass the Hash
- **Step 7:** check the Ticket Encryption Type field on that `4768`/`4769`, `0x17` (RC4) is the classic signal since it's what an NTLM-hash-derived ticket uses by default, but don't rule out AES-encrypted tickets, some tooling can request those too
- **Step 8:** if the activity can't be confirmed from logs alone, contact the account owner for confirmation

### Detection Nuance: Overpass the Hash vs Pass the Hash
The outcome and endpoint-level detection footprint of Overpass the Hash and Pass the Hash are the same, both start from the identical hash injection. The main difference is where they diverge at the domain controller: Pass the Hash produces NTLM authentication (`4776`), Overpass the Hash produces Kerberos ticket activity (`4768`/`4769`) instead. The Ticket Encryption Type field at the DC (`0x17` RC4, `0x11` AES128, `0x12` AES256) can help distinguish it further, but isn't reliable on its own since legitimate AES-encrypted tickets are common in a properly hardened environment.

The most reliable detection is still endpoint-based: `4624` with Logon Type `9`, optionally combined with Sysmon LSASS access for fewer false positives. From there, pivot to the domain controller and check whether that same logon produced `4776` (Pass the Hash) or `4768`/`4769` (Overpass the Hash).

### Key Notes
- Overpass the Hash is functionally a bridge technique: NTLM hash theft feeding into Kerberos-based lateral movement, not a separate credential theft method
- If NTLM is disabled or restricted in the environment, Overpass the Hash becomes the attacker's only path to use a stolen hash, since straight Pass the Hash won't work
- Cross-reference with Pass the Ticket, once the attacker has a TGT from this technique, everything downstream (ticket reuse, injection into other sessions) follows the same detection logic as Pass the Ticket