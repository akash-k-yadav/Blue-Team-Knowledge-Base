# Windows Authentication Mechanisms
## What is it?
- Windows authentication is a way for Windows environments to authenticate users. It is done by an authentication authority to verify that credentials provided are valid and provide system access to the desired resource.
### Domain Account Authentication
- For domain accounts, the domain controller acts as the authentication authority; its job is to verify credentials.
- Giving access to the resource is handled by the client workstation.
### Local Account Authentication
- For local account authentication, the standalone machine or computer is responsible for verifying credentials and giving access to services based on permissions.
## Logs Generated During Account Logon
- When authentication is approved or initiated, Windows records an **account logon** event in the security event log of the system that verifies credentials (for domain accounts: DC, and for local accounts: the client computer).
- When a system provides access to a resource or service based on the result or permissions, Windows records a *logon event* in the security event log of the system providing access to the resource.
- **For example:** If a domain account requests access to a file on file_server_1, an account logon event will be recorded on the domain controller, and a logon event on file_server_1.
- *Conclusion:* Think of account logon events as authentication events and logon events as records of actual logon activity.
## NTLM Authentication (NT LAN Manager Version 2)
- NTLM is an older authentication mechanism/protocol used by Windows to verify credentials and give access to resources. It is the **fallback** protocol used when Kerberos is not possible (e.g., access via IP address, or DC unreachable). Kerberos is the default for domain environments.
- In this protocol, the user's password NT hash is used as a shared secret between the domain controller, client workstation, and the requested service.
### How Does it Work?
- **Step 1:** The user enters their credentials in plain text (username and password) on their client workstation.
- **Step 2:** The client workstation converts the password into an NT hash.
- **Step 3:** The client sends the username to the requested service. The service responds by sending a **challenge** (a random nonce) to the client only.
- **Step 4:** The client encrypts the challenge using the NT hash of the typed password and sends this encrypted response back to the server.
- **Step 5:** The server forwards three things to the domain controller: the **username**, the **original challenge**, and the **client's encrypted response**.
- **Step 6:** The domain controller looks up the stored NT hash for that username, encrypts the forwarded challenge using it, and compares the result to the client's encrypted response.
- **Step 7:** The domain controller sends the authentication result (success or failure) back to the server, and records a respective event.
- *NOTE:* Only the challenge and ciphertext move over the network — the NT hash and plain text password never do. The DC and client never communicate directly in NTLM.