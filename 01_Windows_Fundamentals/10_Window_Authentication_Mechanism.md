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
- *NOTE:* Only the challenge and ciphertext move over the network , the NT hash and plain text password never do. The DC and client never communicate directly in NTLM.
## Kerberos
- Kerberos is an authentication protocol that uses encrypted tickets instead of passwords to verify the identity of users and services. After a user logs in, the Kerberos server issues tickets that allow access to network resources without repeatedly sending the user's password across the network. It is the default authentication protocol used in Active Directory domains.

### Components in Kerberos
- **Client:** the user's machine
- **KDC:** Key Distribution Center, runs on the domain controller and has two internal services: the AS (Authentication Service) and TGS (Ticket Granting Service)
- **KRBTGT account:** it is a special built-in domain account. Its password hash is used to encrypt and sign every TGT in the domain. This account never logs in interactively; it only exists to sign tickets.
- **TGT:** A Ticket Granting Ticket (TGT) is the first ticket issued by the Kerberos Authentication Server (AS) after a user successfully logs in. It proves that the user has already been authenticated and allows them to request access to other services without sending their password again. The TGT is presented to the Ticket Granting Server (TGS) whenever the user needs a service ticket.
- **Service Ticket:** A Service Ticket is a ticket issued by the Ticket Granting Server (TGS) that allows a user to access a specific network service, such as a file server, web server, or database. It contains the information needed for the service to verify the user's identity and is only valid for the service it was issued for.
- **PAC:** Privilege Attribute Certificate is the data structure that contains information about a user account or system, such as group membership, policy information, identifiers, and other security details. It is embedded into TGTs and service tickets.

### Step by Step Flow of Kerberos

### Step 1: AS-REQ (Client to Authentication Service)
- This is the first step, in which the user enters their password and username on their client machine or workstation. The machine converts the entered password into an *NT hash representation*.
- This NT hash is used as a key to encrypt the current timestamp (the output of this will be referred to as TS ciphertext in this file).
- TS ciphertext and username are transmitted to the Authentication Service on the domain controller.
- The domain controller decrypts the TS ciphertext using the user's NT hash representation (stored in the NTDS.dit file) and based on the result (current timestamp) authenticates the user.
- **Event ID
### Step 1 Response: AS-REP
- In case of successful authentication, the DC sends two things:
  - 1 **Session Key:** session key encrypted with the user's NT hash
  - 2 **TGT:** Ticket Granting Ticket encrypted with the NT hash representation of the KRBTGT account. It contains information about the user such as username, privileges, group membership, and expiry time.
- This TGT is now used to request resource access.
- **NOTE:** All of this is stored in memory (RAM) on the client machine, managed by the LSASS process.

### Step 2: TGS-REQ (Client to TGS)
- Now suppose the user wants to request or access some resource (in our example, a file server). The client machine sends the request to the TGS component of the KDC on the DC with the following:
  - 1 **An Authenticator:** client username and current timestamp encrypted using the session key (from Step 1 response). By encrypting with the session key, the DC knows the request comes from an already authenticated user.
  - 2 **SPN:** the identifier of the service (in this case, the file server identifier)
  - 3 **TGT:** from Step 1 response
- **DC Role:**
  - DC decrypts the TGT using the KRBTGT account's hash representation
  - Reads the session key inside it
  - Uses the session key to decrypt the authenticator
  - Validates the timestamp
  - By this, the DC knows the user is legitimate and authenticated, as the session key is only shared with its legitimate owner and only the DC has the KRBTGT account hash
  - Copies the PAC details inside the TGT into the service ticket and encrypts it using the NT hash representation of the requested service

### Step 2 Response: TGS-REP
- The DC sends two things to the client machine:
  - 1 **New Session Key:** new session key encrypted with the user's session key from Step 1 response
  - 2 **Service Ticket:** service ticket encrypted with the hash representation of the service's password (file server hash representation), containing the username, privileges, new session key, and expiry time

### Step 3: AP-REQ (Client Machine to Requested Service)
- The client sends the service ticket directly to the service (file server).
- The file server decrypts the service ticket using its own hash representation of the password.
- Reads the PAC inside it.
- Based on its content such as user privileges and local policies of the server, it grants or denies the request.

### Important Points
- In Kerberos, to issue the TGT from the KDC, multiple types of shared secrets are used, such as the following:
  - 1 **NT Hash:** RC4-HMAC enabled or NTLM hash
  - 2 **AES128 Key:** modern Kerberos, 128-bit key derived from the password
  - 3 **AES256 Key:** 256-bit key derived from the password, preferred in modern Kerberos
  - 4 **Long Term Key:** symmetric key derived from the password. This is the actual shared secret used by Kerberos.
- **Note:** In the Kerberos flow above, NT hash representation is used as the shared secret, but any of the above can be used. There are also more representations but they are beyond the scope of this file.
- By default, a TGT that is properly encrypted is considered valid and no additional checking is done when the TGT is presented. Only after 20 minutes does the DC check again to verify the account has not been disabled since the TGT was issued.

### Key Characteristics of Kerberos
- The account's plain text password, hash representation of the password, and shared secret are never transmitted over the wire, solving the key distribution problem.
- TGT allows an already authenticated user to request service tickets without authenticating again.
- Service tickets allow the requested service to know the user or system is already authenticated, and based on service local policies and user privileges, grant or deny access.
- The domain controller acts as a trusted entity in the domain. Every system, user, and service trusts the domain controller and the tickets issued by it.

## Authentication Event IDs
### Kerberos Authentication
- **4768** fires when a TGT is requested (AS-REQ) and contains the result (failed or successful issuance).
   - The network information section of the event contains additional information about the remote host in the event of a remote logon attempt.
   - The keyword field indicates whether the authentication attempt was successful or failed. In case of a failed attempt, the result code in the event description provides additional information about the reason for failure.
- **4769** fires when a service ticket is requested by a user account for a specified resource.
   - The event contains information about the source IP of the system that made the request, the user account used, and the service to be accessed.
   - The event also indicates whether the request was successful or failed, and the code provides additional information about the failure.
- **4770** fires when a service ticket is renewed. The account name, service name, client IP address, and encryption type are recorded.
- **4771** fires for failed Kerberos pre-authentication. Depending on the reason for a failed Kerberos logon, either event ID 4768 or event ID 4771 is created. In both cases, the code provides additional information about the failure.

### NTLM Authentication
- **4776** is logged for NTLM authentication events.
   - The network information section of the event contains additional information about the remote host in the event of a remote logon attempt.
   - The keyword field indicates whether the authentication attempt succeeded or failed, and the error code provides a description of the failure.

## Normal Baseline Scenario of Kerberos Authentication
- In this scenario we will look at what events are logged during Kerberos authentication where a user sits on Client_Workstation_1 and attempts logon in a domain environment.

### On Domain Controller
- **4768** for TGT request (AS-REQ)
- **4769 (For Client_Workstation_1)** service ticket requested for the client machine the user is sitting at, as that system needs to be accessed for the logon to complete
- **4769 (For DC)** service ticket requested for the domain controller itself, as the client needs to access DC services to complete authentication
- **4769 (For krbtgt)** service ticket requested for the krbtgt service, which is responsible for authentication and issuance of tickets
- **4624 Logon Type 3** remote logon to the domain controller required for processing authentication
- **4634/4647** records the end of the remote logon session. The logon ID in this event will match the logon ID in the corresponding 4624 event, allowing correlation of the session start and end

### On Client_Workstation_1
- **4624 Logon Type 2** for interactive logon. Credentials and TGT are cached in memory and managed by the LSASS process
- **4634/4647** recorded when the user logs off from Client_Workstation_1