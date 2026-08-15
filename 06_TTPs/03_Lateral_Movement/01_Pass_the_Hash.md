# Pass the Hash Attack

## What is it
In a pass the hash attack, the adversary steals the NTLM hash representation of a password and uses this stolen hash directly to complete authentication in a domain environment, without ever needing the cleartext password.

### How Adversary Steals the Hash
There are a few ways an attacker can steal the NTLM hash representation of a password:
- **LSASS.exe Dumping:** using tools like Procdump or Mimikatz, the adversary can dump `lsass.exe`, which stores the hash representation for interactive logons
- **Sniffing Challenge-Response:** intercepting the challenge-response exchange during NTLM authentication and cracking it offline
- **Local Security Account (SAM):** extracting hashes from the SAM file

There are also other ways such as NTDS extraction, DCSync, etc.

## Why Attackers Use It
- Doesn't require the cleartext password, only the hash, so there's nothing to crack before authenticating
- Many environments reuse the same local admin password across multiple machines, so one stolen hash can authenticate against many hosts
- Authentication completes through the domain's normal NTLM process, so it looks like standard account activity instead of an exploit
- Useful for lateral movement, resource access, and code execution once authenticated

## How the Attack Works
- 1. Adversary has already stolen the NTLM hash representation of an account's password
- 2. Adversary uses tools such as Metasploit or Mimikatz to inject this stolen hash and perform NTLM authentication as that account
- 3. Once successfully authenticated, the adversary abuses the stolen account's privileges for lateral movement, resource access, code execution, etc.

## MITRE ATT&CK
Pass the hash lets an adversary authenticate as a user by directly using a stolen password hash instead of a cleartext password, skipping the normal steps that require entering a plaintext credential.

- **ID:** T1550.002
- **Sub-technique of:** T1550
- **Tactic:** Lateral Movement
- **Platform:** Windows

## Detection

### Event IDs to Monitor
| Event ID | Source | Description |
|---|---|---|
| 4776 | Windows Security | NTLM authentication |
| 4624 | Windows Security | An account successfully logged on |
| 4648 | Windows Security | A logon was attempted using explicit/stolen credentials |
| 4672 | Windows Security | Special privileges assigned to new logon |

### Important Fields
- **Logon Type:** value `9` (NewCredential) on the source host is the main anomaly indicator, normal NTLM authentication uses Logon Type `3` or `2`
- **Logon Process:** identifies which trusted authentication component/module on the machine processed and validated the logon request, consistently shows as `seclogo` for pass the hash tools like Mimikatz
- **Authentication Package:** shows as `Negotiate` on the source host during a pass the hash attempt
- **GrantedAccess (Sysmon Event ID 10):** access rights requested against `lsass.exe`, values `0x1010` or `0x1038` indicate credential extraction access

### Correlation Framework
- **Step 1:** on the source machine, filter for `4624` with Logon Type `9`, this is the primary trigger for a pass the hash alert
- **Step 2:** confirm the Logon Process field shows `seclogo` and Authentication Package shows `Negotiate`, both together with Logon Type `9` is a strong indicator of pass the hash rather than a false positive
- **Step 3:** check for a matching Sysmon Event ID `10` on the same source host around the same time, GrantedAccess `0x1010` or `0x1038` against `lsass.exe` confirms hash extraction took place (see [LSASS Dumping](../02_Credential_Access/01_LSASS_Dumping.md))
- **Step 4:** check whether `4648` is recorded on the source machine, this depends on which tool/technique is used, e.g. Mimikatz spawns a new process with the stolen credential to authenticate
- **Step 5:** check `4672` on the source machine, if it fires for the logged-on user rather than the impersonated account, that's a key anomaly, normal privileged logons show `4672` for the account actually being used
- **Step 6:** pivot to the target/destination host, look for `4624` with Logon Type `3` and Authentication Package `NTLM` for the impersonated account
- **Step 7:** check the domain controller for `4776` without a matching `4768`/`4769` pair, the absence of Kerberos ticket requests is a secondary indicator only, it can also occur for legitimate reasons like cross-domain or non-trusted domain authentication, so don't rely on it alone
- **Step 8:** if the activity can't be confirmed from logs alone, contact the account owner for confirmation

### Key Notes
- Detecting pass the hash is about correlating logs across source host, target host, and domain controller, no single event ID confirms it on its own
- Logon Type 9 + Logon Process `seclogo` + Authentication Package `Negotiate` occurring together, correlated with a concurrent LSASS access event, is the most reliable combination
- Logon Type 9 alone can produce false positives from legitimate applications that use impersonation