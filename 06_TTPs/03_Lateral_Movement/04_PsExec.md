# PsExec

## What is PsExec
PsExec is an administration tool that leverages SMB to remotely execute commands on other systems. It allows remote execution of programs over an authenticated network connection when provided with administrator credentials valid on the remote system. If the executable doesn't already exist on the target, it can be copied there.

Syntax:

```
psexec \<target_ip> [-c] [-d] [-e] [-s]
-u [domain\user]
-p [password]
<command>
```

- `-c` : copy the executable to the remote system
- `-d` : run the command in a non-interactive manner, don't wait for the process to terminate
- `-e` : disable creation of a user profile on the remote system, the logon won't leave a mounted profile
- `-s` : run the remote process in the context of the System account instead of the authenticating account
- `-u` : specify username
- `-p` : specify password (recorded in plaintext if auditing is enabled)

**Note:** use of `-u` causes the system to treat the logon as interactive, meaning the credential is stored in memory.

## Why Attackers Use It
- Works with legitimate administrative functionality already built into Windows, so it doesn't require dropping custom malware to get remote code execution
- Only needs valid admin credentials, no exploit or vulnerability required
- Widely used by IT admins, so activity can blend into normal administrative traffic if not baselined
- Provides an easy path to run payloads with SYSTEM-level privileges on the target via the `-s` flag

### Metasploit Version of PsExec
Metasploit also has a version of PsExec that requires valid admin credentials to execute remote commands (an exploit module that leverages or abuses admin privileges). In the absence of valid credentials, it will attempt to log in as guest.

### How the Metasploit Module of PsExec Works
- Uses valid admin credentials
- Copies the executable to the targeted system
- Creates a service to load the desired payload
- Deletes the service it created
- Deletes the uploaded payload

This records a service creation and deletion event, along with the username and executable used.

### Sysinternal Version of PsExec
By default, the Sysinternal version of PsExec installs itself as a service with the service name PSEXEC and an executable named `psexesvc` written to disk, making it easy to spot in EventCode `7045`. Unlike other tools, Sysinternal PsExec does not automatically delete the service upon completion. By default, it also causes a user profile to be created on the remote system if one does not already exist for the account credential used.

## MITRE ATT&CK
PsExec is cataloged as Software S0029. Its lateral movement mechanism relies on copying and executing content via SMB/Windows Admin Shares, using the ADMIN$ share to transfer the executable and the Service Control Manager to run it remotely.

- **ID:** T1021.002
- **Sub-technique of:** T1021
- **Tactic:** Lateral Movement
- **Platform:** Windows

## Detection

### Event IDs to Monitor
| Event ID | Source | Description |
|---|---|---|
| 7045 | System Log | New service installed |
| 4697 | Windows Security | Service installed |
| 7036 | System Log | Service state change |
| 1 | Sysmon | Process creation |
| 5145 | Windows Security | Detailed file share access |
| 4648 | Windows Security | A logon was attempted using explicit credentials (only if `-u` is used) |
| 17 | Sysmon | Named pipe created (PsExec uses named pipes to relay stdin, stdout, and stderr between the source and destination) |

### Correlation Framework
We can detect PsExec lateral movement by starting on the destination side and then pivoting to the source host for further context.

- **Step 1 :-** filter for EventCode `7045` (service installation) and hunt for a service name of `PSEXESVC`
- **Step 2 :-** identify the destination host from Step 1
- **Step 3 :-** filter for EventCode `1` on the destination host where ParentImage is `PSEXESVC`
- **Step 4 :-** check the command line of the spawned process to see what's actually being executed, malicious use of PsExec would often contain command-line discovery commands such as `whoami`, `net localgroup administrators`, etc.
- **Step 5 :-** filter for EventCode `17`, check host, image, and pipename, the pipename contains the name of the source host the remote command is coming from
- **Step 6 :-** filter for EventCode `5145` on the destination host identified in Step 2, this logs the specific files and objects accessed through the share, shown in the Relative_Target_Name field
- **Step 7 :-** now that the source host is identified, look for EventCode `1` where the image is `psexec`, to see the actual commands used by the attacker remotely

### Key Notes
- Attackers know security tools look for `PSEXESVC` by name. PsExec's `-r` flag lets the operator specify a custom service name, running `PsExec -r renamed_psexec \\target cmd` creates a service called `renamed_psexec` instead of `PSEXESVC`
- The more reliable pattern to hunt for is any new service with Service_Type of "user mode service" and Service_Start_Type of "demand start," the service name and binary can change, but this signature stays consistent
- Named pipe detection (Step 5) is one of the few artifacts that survives service renaming, since the pipe still carries the source hostname regardless of what the service is called