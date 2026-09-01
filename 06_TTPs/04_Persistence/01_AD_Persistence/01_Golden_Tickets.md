# Golden Tickets

## What is Golden Tickets
Golden tickets are forged Kerberos Ticket Granting Tickets (TGT) created by an attacker who has compromised the krbtgt account's password hash in an Active Directory environment. By exploiting this critical secret, adversaries can generate cryptographically valid tickets that let them impersonate any user, including domain administrators, and gain unrestricted, persistent access to the entire domain without triggering standard authentication checks.

### How Golden Tickets Work
- A TGT is a special case of service ticket issued by and for the krbtgt service on the domain controller. Its security depends on encrypting and signing it with the long-term key (the hash representation of the krbtgt account password). Since only the krbtgt service has access to that password representation, only the krbtgt service should be able to encrypt or decrypt TGT details
- If the domain controller itself is compromised and the krbtgt account's password representation is stolen, the security that the entire domain's authentication and authorization depends on is broken
- Once an attacker has the krbtgt account's password representation, they can use that long-term key to forge a TGT at will, containing any information they want in the associated PAC
- They can impersonate any legitimate user, providing accurate group membership details and a valid security identifier in the PAC
- They could also generate completely fictitious user or PAC entries

**Important Note:**
- By default in Active Directory, a properly encrypted TGT is considered valid and no additional checking is done when it's presented, unless the TGT is 20 minutes old. After 20 minutes, the account is checked again to ensure it hasn't been disabled since the TGT was issued
- As long as the attacker keeps creating a new TGT every 20 minutes, those tickets will be trusted throughout the domain and can be used to request any service ticket desired

(see [Windows Authentication Mechanism](../../../01_Windows_Fundamentals/10_Window_Authentication_Mechanism.md))

## MITRE ATT&CK
Adversaries who have the krbtgt account password hash can forge Kerberos ticket-granting tickets, known as a golden ticket, to generate authentication material for any account in Active Directory. Using a golden ticket, adversaries then request TGS tickets to access specific resources, which still requires interacting with the KDC.

- **ID:** T1558.001
- **Sub-technique of:** T1558
- **Tactic:** Credential Access
- **Platform:** Windows

## Detection

### Event IDs to Monitor
| Event ID | Source | Description |
|---|---|---|
| 4769 | Windows Security | Kerberos service ticket was requested |
| 4768 | Windows Security | Kerberos TGT was requested (used as the baseline for comparison, not expected to be present for a forged ticket) |

### Important Fields
- **Ticket Encryption Type:** `0x12`/`0x11` = AES256/AES128 (expected in a modern domain), `0x17` = RC4-HMAC, forging tools commonly default to RC4 unless explicitly configured otherwise, so RC4 on a privileged account in an AES-enforced environment is a strong indicator
- **Ticket lifetime / validity period:** default Kerberos ticket lifetime is 10 hours, forging tools often set arbitrarily long lifetimes (years instead of hours), an abnormal validity period is a major red flag
- **Account Name / SIDs in the PAC:** forged tickets can contain fictitious usernames or group memberships (e.g. Domain Admins SID) that don't match the account's actual AD record

### Correlation Framework
- **Step 1 :-** since the ticket is forged offline using the stolen krbtgt hash, there will be no `4768` on the domain controller for it, detection has to start from the absence of this event, not its presence
- **Step 2 :-** filter for `4769` (service ticket requested) events and check whether there's a matching `4768` from the same account and source host within the expected ticket lifetime window, a `4769` with no corresponding `4768` for that account/session is the primary golden ticket indicator
- **Step 3 :-** for any unmatched `4769` found, check the Ticket Encryption Type field, `0x17` (RC4) on a privileged account in a domain that otherwise enforces AES is a strong secondary indicator, since most forging tools default to RC4
- **Step 4 :-** check the ticket lifetime/validity period against domain Kerberos policy, an abnormally long validity period (well beyond the default 10 hours) points to a forged ticket rather than a legitimately issued one
- **Step 5 :-** cross-reference the username and group memberships/SIDs in the PAC against the actual AD user and group inventory, a fictitious username or a SID for a group the account was never actually added to confirms forgery
- **Step 6 :-** if a golden ticket is confirmed, treat it as a full domain compromise, not an isolated account issue, since forging one requires the krbtgt hash to already be stolen, most likely through DCSync or direct domain controller compromise, and requires a krbtgt password reset (twice) plus broader incident response

### Key Notes
- Golden ticket detection is inherently about absence and anomaly, not a single clean event ID signature, since the forged ticket itself never touches the DC until it's used to request a service ticket
- RC4 encryption alone isn't proof, some legacy applications and accounts genuinely use RC4, it only becomes a strong indicator when paired with a missing `4768` or when seen on a privileged account in an otherwise AES-only environment
- Confirming a golden ticket means the krbtgt account is already compromised, response has to include resetting the krbtgt password twice (each reset invalidates existing golden tickets, but a single reset can be bypassed since the KDC retains the previous password for a grace period)