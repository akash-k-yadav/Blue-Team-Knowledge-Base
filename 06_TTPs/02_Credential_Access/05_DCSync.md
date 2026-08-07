# DCSync

## What is it
- DCSync is a technique in which an attacker impersonates a Domain Controller using the Active Directory replication protocol to steal password data, without touching the DC disk, without creating volume shadow copies, and without needing physical access to the DC.

**Note:** Large organizations run multiple domain controllers for load balancing, backups, or different physical locations. For all domain controllers to stay in sync, they must maintain identical copies of the NTDS.dit file. They constantly synchronize to update changes such as password resets, new users, or object modifications. This process is called directory replication and uses MS-DRSR (Directory Replication Service Remote Protocol).

## Why Attackers Use It
- Pulls credentials directly from AD replication traffic, without touching NTDS.dit or LSASS, avoiding the detection points tied to those two areas.
- Allows retrieval of the KRBTGT hash, which enables Golden Ticket attacks.
- No need for physical or interactive access to the domain controller itself.
- Only requires an account with the right replication permissions, not code execution on the DC.

## How the Attack Works / Attacker Role
- Attacker needs an account with Replicating Directory Changes and Replicating Directory Changes All rights. By default this means Domain Admins, Enterprise Admins, or DC machine accounts, or any account these rights have been delegated to.
- Attacker's machine sends a replication request over DRSUAPI (Directory Replication Service Remote Protocol interface), pretending to be a domain controller asking to sync directory data.
- The real DC checks the requesting account's permissions. If valid, it responds with the requested data, including password hashes.
- No enumeration step is required beforehand if the attacker already has a compromised account with the right privileges. If not, privilege escalation or credential theft to obtain such an account happens first.

## Mitre Attack
- Adversaries may attempt to access credentials and other sensitive information by abusing a Windows Domain Controller's API to simulate the replication process from a remote domain controller, a technique called DCSync.
- Members of the Administrators, Domain Admins, and Enterprise Admins groups, or computer accounts on the domain controller, are able to run DCSync to pull password data from Active Directory, which may include current and historical hashes of accounts such as KRBTGT and Administrators.
- ID: T1003.006
- Sub-technique of: T1003
- Tactic: Credential Access

## Detection

### Event IDs to Monitor
| Event ID | Source | Description |
|---|---|---|
| 4662 | Windows Security | An operation was performed on an object |
| 4624 | Windows Security | An account successfully logged on |

### GUIDs
- **1131f6ad-9c07-11d1-f79f-00c04fc2dcd2** is DS-Replication-Get-Changes-All
- **1131f6aa-9c07-11d1-f79f-00c04fc2dcd2** is DS-Replication-Get-Changes
- **89e95b76-444d-4c62-991a-0facbeda640c** is DS-Replication-Get-Changes-In-Filtered-Set

### Important Fields to Monitor
- **User:** the account performing replication
- **Access Mask:** the type of access requested
- **Properties:** GUID of the replication right requested
- **Logon ID:** used to correlate with 4624

### Correlation Framework
- **Step 1:** Filter for Event ID `4662` where Properties contains one of the GUIDs above and the account is not a machine account (does not end in "$").
- **Step 2:** If the machine initiating replication is not a domain controller, that alone is a strong indicator of malicious activity, since DCSync is normally initiated only between domain controllers.
- **Step 3:** Check the Access Mask. If it equals `0x100`, this corresponds to the Control Access right, which is required to invoke DS-Replication-Get-Changes-All. Seeing this mask outside of DC-to-DC replication is a strong indicator of DCSync.
- **Step 4:** Retrieve the username and check whether it is associated with a known compromised account.
- **Step 5:** Retrieve the Logon ID and filter for Event ID `4624` with the same Logon ID to get the source IP of the attacker.

### Key Points
- Detection centers on replication initiated by an account that is not a machine account, combined with Access Mask `0x100`. That combination alone is a strong indicator of DCSync.