# LSASS Dumping

## What is it?
- lsass.exe is a Windows process responsible for authentication. It also stores
password representations and tickets (TGT) in memory in case of an interactive
logon (logon type 2 and logon type 10).
- As it stores Kerberos tickets and NTLM representations of user passwords in
memory, adversaries extract these tickets and hashes from memory using tools
such as Mimikatz, and then use the stolen hashes or tickets for lateral movement
and privilege escalation in attacks such as Overpass-the-Hash and Pass-the-Hash.
- This process of extracting password representations and tickets from LSASS is
known as LSASS dumping.

## Why Attackers Use It
- lsass.exe is a legitimate Windows process. When a user interactively logs on
to a compromised workstation or host, an attacker can use Mimikatz to dump LSASS
without the victim knowing their tickets or hashes have been stolen. No phishing
email or brute force is required.
- With stolen tickets, adversaries can access services while impersonating the
legitimate owner of the ticket. On the surface it appears as normal user activity.
- With stolen hashes, adversaries can request TGTs or service tickets mimicking
the legitimate owner of the credential.
- **Note:** Mimikatz is the most well-known tool but adversaries also use
procdump, rundll32, and comsvcs.dll for the same purpose.

## Prerequisites
- LSASS dumping requires administrative or SYSTEM level privileges on the target
machine. This means that by the time LSASS dumping is observed, the attacker has
already achieved privilege escalation on that host. Scope your investigation
accordingly — LSASS dumping is not an initial access technique, it is a
post-compromise credential harvesting step.

## MITRE ATT&CK
- Adversaries may attempt to access credential material stored in the process
memory of the Local Security Authority Subsystem Service (LSASS). After a user
logs on, the system generates and stores a variety of credential materials in
LSASS process memory. These credential materials can be harvested by an
administrative user or SYSTEM and used to conduct lateral movement using
alternate authentication material.
- Technique ID: T1003.001
- Sub-technique of: T1003 (OS Credential Dumping)
- Tactic: Credential Access
- Platform: Windows

## Detection

### Event IDs to Monitor
| Event ID | Source | Description |
|---|---|---|
| 10 | Sysmon | Process access :- primary detection signal for LSASS access |
| 1 | Sysmon | Process creation :- identifies the tool used to perform the dump |
| 11 | Sysmon | File creation :- tools like ProcDump write a .dmp file to disk |
| 13, 14 | Sysmon | Registry key modification :- attackers modify the WDigest registry key to force Windows to cache plaintext passwords in LSASS memory before dumping |

### WDigest Registry Key
- The key targeted is:
`HKLM\SYSTEM\CurrentControlSet\Control\SecurityProviders\WDigest\UseLogonCredential`
- Setting this value to 1 forces Windows to store plaintext passwords in LSASS
memory. By default in modern Windows this is disabled. Modification of this key
before a dump significantly increases the value of what the attacker extracts.
- Event IDs 13 or 14 on this specific key path should be treated as a high
priority alert.

### Important Fields
- **SourceImage:** full path of the process performing the access
- **SourceUser:** user account running that process
- **TargetImage:** full path of the process being accessed
- **GrantedAccess:** what level of access was granted to the target process
- **CallTrace:** call stack of the accessing process at the time of access
- **CommandLine:** common dumping syntax such as rundll32, procdump,
comsvcs.dll, Invoke-Mimikatz

### GrantedAccess Values
- **0x1010:** PROCESS_VM_READ and PROCESS_QUERY_INFORMATION, commonly
associated with Mimikatz (sekurlsa module) reading LSASS memory.
- **0x1410:** Adds PROCESS_DUP_HANDLE, typical of ProcDump or Task Manager
creating a memory dump.
- **0x1FFFFF or 0x1F1FFF:** PROCESS_ALL_ACCESS, requesting maximum permissions,
often seen in comsvcs.dll MiniDump attacks. Primary red flag.
- **0x1438 / 0x143A:** Used by Mimikatz (lsadump module) for credential dumping
operations.

### Abnormal CallTrace
- A normal CallTrace shows a clean chain of known, named Windows modules such
as ntdll.dll, kernelbase.dll, and kernel32.dll.
- A suspicious CallTrace contains memory addresses that are not backed by any
named module on disk. These appear as raw hex addresses or as UNKNOWN in the
trace, for example:

```
C:\Windows\SYSTEM32\ntdll.dll+0xabcd|UNKNOWN(0000000000000000)
```

- This indicates code running from injected or reflectively loaded memory,
which is the primary indicator of tools like Mimikatz that load and execute
entirely from memory without touching disk.
- comsvcs.dll appearing in the CallTrace is a specific indicator of the
rundll32 MiniDump technique.

### Correlation Framework
**Step 1:** Look for process access events (Sysmon Event ID 10).

**Step 2:** Filter for TargetImage containing lsass.exe to narrow results
to LSASS access specifically.

**Step 3:** Check the CommandLine for common dumping tools or syntax such
as procdump, mimikatz, rundll32, comsvcs.dll, or Invoke-Mimikatz.

**Step 4:** Check the GrantedAccess value. 0x1FFFFF or 0x1F1FFF are the
primary red flags as they indicate maximum access. Treat all values listed
above as suspicious in the context of lsass.exe access.

**Step 5:** Examine the CallTrace. Any raw hex addresses or UNKNOWN entries
in the trace confirm code running from unbacked memory, which is strong
evidence of in-memory tooling such as Mimikatz.

**Step 6:** Check SourceUser running the process. Correlate with known
compromised accounts. Retrieve the user's role, privilege, and group
membership to evaluate the severity of the dump.

**Step 7:** Check for Event ID 13 or 14 on the WDigest registry key path.
If this modification preceded the LSASS access event, the attacker
deliberately enabled plaintext password caching before dumping, indicating
a more deliberate and prepared attack.

**Step 8:** Check for file creation events (Event ID 11) around the same
timestamp. A .dmp file written to disk confirms a file-based dump using
tools like ProcDump rather than an in-memory only approach.

**Step 9 — Post Dumping:** Review account logon and logon events for the
user whose credentials were stolen to identify lateral movement attempts
following the dump.