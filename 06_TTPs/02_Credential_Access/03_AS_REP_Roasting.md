# AS-REP Roasting

## What is AS-REP Roasting?
- AS-REP Roasting is a credential access attack that targets Active Directory
accounts configured with "Do not require Kerberos pre-authentication" enabled.

### What is Pre-Authentication?
- In the AS-REQ, the client sends a timestamp encrypted with a shared secret
to the domain controller for authentication. This is known as pre-authentication
and it proves that the client possesses the correct password.
- Pre-authentication can be disabled for some accounts. When disabled, the Key
Distribution Center on the domain controller will send back an AS-REP to anyone
who requests it for that account, without requiring any proof of identity.

### Attacker Role
- **Step 1:** The attacker enumerates domain accounts via LDAP, querying for
accounts with the `DONT_REQUIRE_PREAUTH` flag set in `userAccountControl`.
- **Step 2:** For each identified account, the attacker sends an AS-REQ to the
KDC without the encrypted timestamp.
- **Step 3:** The domain controller responds with an AS-REP containing an
encrypted session key encrypted with the user's password hash.
- **Step 4:** The attacker takes this encrypted portion offline and cracks it
using tools such as Hashcat or John the Ripper to recover the plaintext
credentials. Rubeus is commonly used to perform the initial request and capture
the AS-REP in a crackable format.
- **Note:** Unlike Kerberoasting, AS-REP Roasting does not require a valid
domain account to perform. As long as the target account has pre-authentication
disabled, anyone can request the AS-REP and attempt to crack it offline.

## MITRE ATT&CK
- Adversaries may reveal credentials of accounts that have disabled Kerberos
pre-authentication by password cracking Kerberos messages. For each account
found without pre-authentication, an adversary may send an AS-REQ message
without the encrypted timestamp and receive an AS-REP message with data
encrypted with the user's long-term key, which may be vulnerable to offline
password cracking attacks similarly to Kerberoasting.
- ID: T1558.004
- Sub-technique of: T1558
- Tactic: Credential Access

## Detection

### Event IDs to Monitor
| Event ID | Source | Description |
|---|---|---|
| 4768 | Windows Security | TGT requested, contains pre-authentication type and encryption type |
| 4738 | Windows Security | User account modified, detects if pre-authentication was recently disabled |

### Important Fields
- **Pre-Authentication Type:** whether pre-authentication is disabled (0x0
indicates disabled)
- **Ticket Encryption Type:** which encryption algorithm was used to encrypt
the AS-REP data

### Correlation Framework
**Step 1:** Filter for Event ID 4768 with Pre-Authentication Type = 0x0.
This indicates a TGT was issued without pre-authentication.

**Step 2:** Check the Ticket Encryption Type. If it is 0x17 (RC4) that is
a strong indicator of malicious activity. RC4 is an older and weaker algorithm
that is easier to crack offline. Attackers target accounts with
pre-authentication disabled and RC4 encryption as these produce hashes that
are easier to crack.

**Step 3:** Check the Client IP field for the IP address that requested the
ticket and determine whether that IP is associated with a known compromised
host.

**Step 4:** Cross reference with Event ID 4738 to check if pre-authentication
was recently disabled on the targeted account. If yes, the attacker likely
disabled it themselves after gaining sufficient privileges rather than finding
an already misconfigured account.

**Step 5:** Contact the owner of the account for confirmation if the activity
cannot be confirmed as malicious from available logs alone.

## Key Notes
- Any account with pre-authentication disabled is a potential AS-REP Roasting
target. No valid domain credentials are needed to attack it.
- 0x17 (RC4) in the Ticket Encryption Type field is the primary red flag
alongside Pre-Authentication Type 0x0. AES encryption makes offline cracking
significantly harder.
- Event ID 4738 showing pre-authentication was recently disabled, followed by
a 4768 with Pre-Authentication Type 0x0, means the attacker set up the
condition themselves rather than finding an existing misconfiguration. Treat
this as higher severity than finding an already misconfigured account.