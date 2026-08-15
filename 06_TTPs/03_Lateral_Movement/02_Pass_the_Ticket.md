# Pass the Ticket

## What is "Pass the Ticket"
Pass the Ticket is a technique used by attackers that lets them impersonate the owner of a stolen ticket (TGT). The adversary steals an already-issued, legitimate TGT belonging to another user and uses it to complete authentication or request service tickets.

### How Attacker Steals the Tickets
- **LSASS.exe Dumping:** since `lsass.exe` stores the TGT for interactive logons, the attacker can use tools such as Mimikatz or Procdump to dump `lsass.exe` and steal the issued tickets (see [LSASS Dumping](../02_Credential_Access/01_LSASS_Dumping.md))

### Why Attacker Uses This
- Pass the Ticket doesn't require a shared secret or plaintext password, only the Kerberos TGT, so there's no need for offline cracking
- Authentication happens through the normal Kerberos process, which makes it look like standard service ticket request activity, creating less noise
- The stolen ticket can be used to request service tickets and access services while impersonating the legitimate owner in the domain

### How the Attack Works
1. Attacker steals the ticket by dumping `lsass.exe`
2. Tools such as Mimikatz or Rubeus let the adversary load the stolen ticket into their own session, impersonating the ticket's owner
3. The stolen ticket is presented to the domain controller with a request for a service ticket
4. The domain controller verifies the TGT is valid (it was encrypted using the krbtgt account's password representation, and it's within its validity period), then issues a service ticket to the requester (the adversary), containing the PAC details of the stolen TGT's owner
5. The attacker presents this service ticket to the remote system to gain access with the privileges of the account the TGT was stolen from
6. The issued TGT has a validity period (10 hours by default) and can be renewed for up to 7 days, after which it's no longer usable

**Important Note:** when a stolen ticket is injected into an existing user session, it creates an anomaly on that endpoint, a mismatch between the TGT being presented and the active logon session's associated user account.

## MITRE ATT&CK
Pass the ticket authenticates to a system using a stolen Kerberos ticket instead of the account's password, bypassing the normal credential-based logon steps.

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
| 4770 | Windows Security | Kerberos service ticket (TGT renewal) was renewed |
| 4624 | Windows Security | An account successfully logged on |
| 4648 | Windows Security | A logon was attempted using explicit/stolen credentials |

### Important Fields
- **Account Name / Client Address (on 4768):** identifies which user requested a TGT from which host, this pairing is what you correlate against later 4769/4770 events
- **Ticket Options:** field containing the Renew flag, value `0x40000000`, confirms a renewal request
- **Ticket lifetime:** default max is 10 hours (TGT) with renewal allowed up to 7 days, a ticket presented or renewed outside this window is a forgery/reuse indicator

### Detection Framework

**On endpoint (more reliable)**
- **Step 1 :-** look at the current logon session on the system, using `Get-LoggedOnUsers` or `whoami`
- **Step 2 :-** use `klist` (or `klist -li` with a session ID) to inspect the Kerberos tickets tied to that session
- **Step 3 :-** look for a mismatch, for example the active session belongs to user *akash* but the cached TGT belongs to user *sky*, pass-the-ticket detected

**With logs (behavioral, less reliable alone)**
- **Step 1 :-** look for LSASS dumping activity, since pass the ticket always starts with stealing the ticket from `lsass.exe` (see [LSASS Dumping](../02_Credential_Access/01_LSASS_Dumping.md))
- **Step 2 :-** baseline normal `4768` TGT requests per Account/Client pair, this is what a legitimate user generates once per host they log into
- **Step 3 :-** look for `4769` (service ticket request) or `4770` (TGT renewal) for an Account/Client pair that has no matching `4768` in the prior ~10 hours, this is the strongest DC-only indicator, since in a real pass-the-ticket attack the attacker never requests their own TGT, they only reuse the stolen one
- **Step 4 :-** if a `4770` renewal appears, check the ticket's age against domain Kerberos policy (default 10hr TGT, 7-day max renewal), a renewal older than policy maximum signals a forged or long-lived stolen ticket
- **Step 5 :-** behavioral, look for an abnormal number of `4769` requests from one account, or a service ticket request for a service the account has never accessed before
### Key Notes
- Endpoint-based detection (klist session mismatch) is more reliable than DC log correlation alone, since legitimate cross-domain or multi-host activity can create similar-looking log patterns
- DC-based detection depends on baselining normal Account/Client TGT request patterns first, without that baseline the Account/Client pair correlation in Step 3 won't work
- Ticket lifetime is a useful secondary signal across all Kerberos ticket abuse (Pass the Ticket, Golden Ticket, Silver Ticket), a ticket older than the domain's max renewal window is never legitimate