# NTDS Extraction

## What is it?
- It is the actual file or database that stores all information about the
domain, such as users, their credentials (NTLM password hash), accounts,
groups, computer objects, and machines.

## MITRE ATT&CK
- Adversaries may attempt to access or create a copy of the Active Directory
domain database in order to steal credential information, as well as obtain
other information about domain members such as devices, users, and access
rights. By default, the NTDS file (NTDS.dit) is located in
%SystemRoot%\NTDS\Ntds.dit of a domain controller.
- ID: T1003.003
- Sub-technique of: T1003
- Tactic: Credential Access

## Why Attackers Extract NTDS.dit
- Extracting NTDS.dit gives the attacker password representations of all
accounts in the domain, including all users, domain admins, the KRBTGT
account, and all service accounts.
- When an attacker extracts NTDS.dit, the entire domain can be considered
compromised:
  - The attacker can forge silver or golden tickets to achieve long-term
  persistence in the domain.
  - Domain admin account credentials can be used to move laterally to any
  machine in the domain.
- **Note:** To detect and investigate NTDS extraction, it helps to first
fully understand NTDS.dit and how it is extracted.

## What Attackers Actually Need to Extract Password Representations

### Snapshot of NTDS.dit
- When Active Directory Domain Services is running, Windows locks the file
so it cannot be copied directly or accessed by any process other than
*lsass.exe*.
- To solve this, adversaries abuse a legitimate Windows service named
**Volume Shadow Copy**.

#### Volume Shadow Copy (vssadmin.exe)
- It was developed for legitimate use as a backup mechanism while AD is
still running.
- It works by asking the file system driver to create a shadow copy, a
point-in-time read-only snapshot of an entire volume taken at the block
level.
- Once the shadow copy exists, it is mounted as a separate static device
path. As it is a frozen snapshot, file locks and active transactions no
longer apply.
- Attackers abuse this by using it to create a shadow copy containing
NTDS.dit and the SYSTEM registry hive.
- Commands look like:

```
C:\Windows\System32> vssadmin create shadow /for=C:
C:\Windows\System32> copy \?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\Windows\NTDS\ntds.dit C:\temp\ntds.dit
C:\Windows\System32> copy \?\GLOBALROOT\Device\HarddiskVolumeShadowCopy1\Windows\System32\config\SYSTEM C:\temp\SYSTEM
```

- **Note:** vssadmin.exe requires local admin rights to create a volume
shadow copy.

### ntdsutil.exe
- A legitimate AD management tool designed for domain controller
maintenance. It is the official maintenance interface for NTDS.dit, its
logs, and related DC level settings.
- Attackers abuse ntdsutil.exe with the IFM (Install From Media) feature
to produce a copy of NTDS.dit and the SYSTEM hive, saving them to a local
directory.
- Command looks like:
```
C:\Windows\System32> ntdsutil "ac i ntds" "ifm" "create full C:\temp" q q
```

- **Note:** ntdsutil requires local domain admin or equivalent permissions.

### Decrypting NTDS.dit
- The NTDS.dit file contains ciphertext of password representations, not
the actual plaintext or raw NTLM hashes.
- These password representations are encrypted using the PEK (Password
Encryption Key), and the PEK itself is encrypted with the bootkey.
- The bootkey is derived from values scattered across the SYSTEM registry
hive `HKLM\SYSTEM`, four subkeys whose values are combined
algorithmically to produce the bootkey.
- Therefore, NTDS.dit alone is not enough. Attackers need the SYSTEM hive
to decrypt it.

- **Hence** attackers need two things:
  - A snapshot of NTDS.dit, obtained using vssadmin.exe or ntdsutil.exe
  - A copy of the SYSTEM registry hive to extract the bootkey, which is
  used to decrypt the PEK and ultimately recover the password hashes

### Offline Parsing
- Once both files are obtained, attackers parse them offline using tools
such as Impacket's secretsdump, DSInternals, or Mimikatz to extract the
actual NTLM password hashes of every account in the domain.
- This step happens entirely outside the domain environment and generates
no additional Windows event logs.

## Detection

### Event IDs to Monitor
| Event ID | Source | Description |
|---|---|---|
| 1 | Sysmon | Process creation, identifies the tool used to perform the dump |
| 11 | Sysmon | File creation, to locate where the snapshot is saved on disk |

### Investigating ntdsutil Malicious Use
**Step 1:** Filter for Event ID 1 with image containing `ntdsutil.exe`.

**Step 2:** Check the user, parent image, image, command line, and
timestamp.

**Step 3:** Correlate with previous findings. Is the user account that
started this process already compromised? If uncertain, contact the account
owner for confirmation.

**Step 4:** Look for file creation events (Event ID 11) with image
containing `ntdsutil.exe` and target filename containing `ntds.dit` to
identify where the NTDS.dit snapshot was saved.

**Step 5:** Look for a copy of the SYSTEM hive using process creation
events (Event ID 1).

### Investigating vssadmin Shadow Copy
**Step 1:** Search for process creation events (Event ID 1) with image
containing `vssadmin.exe` and command line containing `create shadow`.

**Step 2:** Check the user, parent image, image, command line, and
timestamp.

**Step 3:** Correlate with previous findings. Is the user account that
started this process already compromised? If uncertain, contact the account
owner for confirmation.

**Step 4:** Look for copy commands by filtering for Event ID 1 with command
line containing `HarddiskVolumeShadowCopy`, `ntds`, or `SYSTEM`.

### Final Notes
- Volume shadow copy creation or ntdsutil usage alone is not sufficient
evidence of malicious activity, as these are legitimate Windows and Active
Directory management tools.
- Detecting and investigating NTDS extraction is about correlating shadow
copy creation and SYSTEM hive copy events with context, whether the account
used to run those commands is already compromised, or by communicating with
the account owner to verify whether they performed those operations
legitimately.

