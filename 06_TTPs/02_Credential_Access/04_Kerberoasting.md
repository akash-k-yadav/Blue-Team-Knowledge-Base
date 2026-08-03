# Kerberoasting

## What is Kerberoasting?
- Kerberoasting is a credential harvesting attack in which an attacker requests
service tickets from the KDC on the domain controller, takes those tickets
offline to crack them, and attempts to recover the plaintext password of the
associated service account. The service ticket is encrypted with the service
account's long-term key, making it a target for offline password cracking.

## How Kerberoasting Actually Works
- Service tickets are encrypted and signed with the long-term key of the
associated service, usually the password representation of the associated
service account.
- Since each ticket is signed, when it is correctly decrypted, the hash stored
in the signature can be used to verify that everything in the ticket was
received and decrypted accurately.
- By receiving a service ticket, the attacker has the ciphertext of the ticket
and the hash value associated with the correctly decrypted version of the
ticket's data.
- By iterating through possible passwords, if one of those passwords can be
used to decrypt the ciphertext so that the hash in the signature matches the
hash of the decrypted data, the attacker has successfully identified the
plaintext password of the associated service account.

### Attacker Role
- **Step 1:** Attacker enumerates SPNs in the domain to identify service
accounts to target.
- **Step 2:** Attacker requests service tickets for those services from the
KDC. Rubeus is commonly used to request tickets and extract them in a
crackable format.
- **Step 3:** Attacker takes the encrypted tickets offline and cracks them
using tools such as Hashcat or John the Ripper.
- **Step 4:** If cracking succeeds, the attacker recovers the plaintext
password of the service account.

### Important Note
- Many default services use the associated system computer account for their
security context. Computer accounts are directly managed by Active Directory,
have extremely long randomized passwords, and are rotated by default every
30 days, making them computationally infeasible to crack through offline
password cracking.
- Service accounts that are manually created, for example for MySQL, Apache,
or Microsoft SQL Server, may have been created with weak passwords and left
unchanged for long periods of time, making them attractive targets for
attackers.

## Why Kerberoasting?
- Service tickets can be requested by any authenticated user in the domain
regardless of their group membership or privileges. There is no need to
compromise the domain controller or the system hosting the service in advance.
- The attacker can also use the recovered service account credentials to
directly log on to systems where that account has permissions.
- With the recovered password of a service account, the attacker can forge
silver tickets to maintain long-term persistence.

## MITRE ATT&CK
- Adversaries may abuse a valid Kerberos ticket-granting ticket (TGT) or sniff
network traffic to obtain a ticket-granting service (TGS) ticket that may be
vulnerable to brute force. Service Principal Names (SPNs) are used to uniquely
identify each instance of a Windows service. To enable authentication, Kerberos
requires that SPNs be associated with at least one service logon account, an
account specifically tasked with running a service.
- ID: T1558.003
- Sub-technique of: T1558
- Tactic: Credential Access

## Detection

### Event IDs to Monitor
| Event ID | Source | Description |
|---|---|---|
| 4769 | Windows Security | Service ticket requested |
| 4770 | Windows Security | Service ticket renewal |

### Important Fields to Monitor
- **Ticket Encryption Type:** which encryption algorithm was used to encrypt
the service ticket (0x17 or 0x18 = RC4, 0x12 = AES256)
- **Account Name:** the account requesting the service ticket
- **Service Name:** the service the ticket was requested for
- **Client IP:** source IP of the request

### Correlation Framework
**Step 1:** Filter for Event ID 4769. Flag any account requesting an
unusually high number of service tickets for different services in a short
time frame.

**Step 2:** Check the Ticket Encryption Type. If it is 0x17 or 0x18 (RC4)
that is a strong indicator of malicious activity. Attackers deliberately
request RC4 encrypted tickets as they are easier to crack offline than AES.

**Step 3:** Retrieve the account name and source IP. Check whether they are
already associated with a known compromised account or host.

**Step 4:** Check whether the account normally accesses the requested service.
Requests for service tickets to services the account has never accessed before
make the 4769 entries more suspicious.

**Step 5:** Check the service name. Requests targeting manually created
service accounts such as SQL service accounts or backup accounts are higher
priority than requests for computer account backed services.

**Step 6:** Look for unusual account logon and logon events involving service
accounts, as abnormal use of a service account may indicate it has already
been compromised and is being used for malicious activity.

**Step 7:** Contact the owner of the account for confirmation if the activity
cannot be confirmed as malicious from available logs alone.