# Active Directory

## What is Active Directory
- Before writing anything, the scope of Active Directory in this file and repo 
is related to a cybersecurity level and objective, not deep admin or system 
operator level.
- Active Directory (AD) is Microsoft's centralized service that manages who can 
access what in a network, handling user accounts, computers, groups, and 
permissions all from one place.
- It stores information about objects such as groups, users, servers, computers, 
and devices in a domain environment.
- It performs authentication using mechanisms such as Kerberos or NTLM (older 
authentication protocol).
- *Important Note:* AD is important for cybersecurity professionals as almost 
every organization uses Microsoft AD in their domain environment, making it 
attractive to attackers. Compromising AD means compromising the entire 
organization.

## Core Concepts

### Domain
- A domain is the core organizational unit in Active Directory. It is a logical 
grouping of users, computers, and other objects that share the same AD database 
and are governed by the same policies.
- Every domain has at least one domain controller that manages authentication 
and stores the domain database.

### Forest
- A forest is the top-level container in Active Directory. It is a collection 
of one or more domains that share a common schema, configuration, and global 
catalog.
- The first domain created in a forest is called the forest root domain.
- From a security perspective, a forest is considered a security boundary in 
Active Directory, not a domain. This means that by default, a domain admin in 
one domain does not automatically have rights in another domain within the same 
forest, but a forest admin (Enterprise Admin) does.

### Trusts
- Trusts define how authentication and access work between domains or forests. 
If Domain A trusts Domain B, users in Domain B can potentially access resources 
in Domain A.
- **Transitive Trust:** the trust relationship extends beyond the two domains 
involved. If Domain A trusts Domain B and Domain B trusts Domain C, then Domain 
A also implicitly trusts Domain C. By default, domains within the same forest 
have transitive trusts with each other.
- **Non-Transitive Trust:** the trust is limited only to the two domains 
directly involved and does not extend further.
- **One-Way Trust:** only one domain trusts the other. If Domain A trusts 
Domain B (one-way), users in Domain B can access resources in Domain A, but 
not the other way around.
- **Two-Way Trust:** both domains trust each other, allowing users from either 
domain to access resources in the other.
- *Security Note:* Trusts are a common lateral movement and privilege escalation 
path. An attacker who compromises a lower-trust domain can potentially abuse 
trust relationships to move into a higher-value domain.

## Domain Controller
- A domain controller is the central authority server that manages everything, 
including users, groups, permissions, and authentication handling.

### NTDS.dit
- It is the actual file or database that stores all information about the 
domain, such as users, their credentials (NTLM password hash), accounts, 
groups, computer objects, and machines.
- *Security Note:* Extracting NTDS.dit is one of the primary objectives of 
an attacker in a domain environment as it contains password hashes of every 
account in the domain, including domain admins.

### Global Catalog
- The global catalog is a domain controller that stores a partial copy of all 
objects in the forest, allowing it to answer queries about any object in any 
domain within the forest without having to contact the domain controller of 
that specific domain.
- It is used during logon and searches across the forest.

## Objects in Active Directory

### Users
- Users are the most common object type in Active Directory. Users are a type 
of object known as security principals, meaning they can be authenticated by 
the domain and can be assigned privileges over resources such as files or 
printers.
- Users in Active Directory represent entities such as people (generally 
representing a person or employee in an organization or domain).

### Machines
- Machines are another type of object within Active Directory. For every 
computer that joins the Active Directory domain, a machine object will be 
created. Machines are also considered security principals and are assigned an 
account just like any regular user.
- Machine account names always end with a $ symbol (e.g., WORKSTATION01$).

### Service Accounts
- Service accounts are user accounts created specifically to run services, 
scheduled tasks, or applications rather than being used by an actual person.
- Service accounts are often over-privileged in real environments, meaning 
they are granted more permissions than the service actually needs.
- Service accounts are associated with **SPNs (Service Principal Names)**. 
An SPN is a unique identifier that links a service to the account running it 
in the domain. For example, a web server service running under a service 
account would have an SPN registered to that account.
- *Security Note:* SPNs are what make service accounts a target for 
**Kerberoasting**. Any domain user can request a service ticket for any SPN 
in the domain. That service ticket is encrypted with the service account's 
password hash, meaning an attacker can request it and attempt to crack it 
offline to recover the plaintext password.

### Organizational Units
- Organizational Units (OUs) are container objects that allow classification 
of users and machines.
- OUs are mainly used to define sets of users with similar policy requirements, 
for example employees in finance or HR departments.
- OUs are useful for applying policies to users and computers, which include 
specific configurations that apply to sets of users depending on their 
particular role in the enterprise.

## Privileged Groups

### Domain Admins
- The Domain Admin group is a built-in group in the domain environment. Members 
of this group have full administrative rights over every machine in the domain.
- Compromising a Domain Admin account means full control over the entire domain.

### Enterprise Admins
- Enterprise Admins is a group that only exists in the forest root domain. 
Members of this group have administrative rights across every domain in the 
entire forest.
- This is the highest level of privilege in an Active Directory environment.
- *Security Note:* This group should be empty in normal operations. Any account 
added to this group should be treated as a high-priority alert.

### Schema Admins
- Schema Admins is a group that only exists in the forest root domain. Members 
have the ability to modify the Active Directory schema, which defines the 
structure of all objects and attributes in the forest.
- Abuse of this group can result in persistent and hard-to-detect modifications 
to the AD structure.

### Backup Operators
- Backup Operators is a built-in group whose members have the ability to back 
up and restore files regardless of file permissions, and can also log on to 
domain controllers locally.
- *Security Note:* Backup Operators can back up the NTDS.dit file, making this 
group a high-value target for attackers despite not being an obvious 
administrative group.

### Account Operators
- Account Operators is a built-in group whose members can create, modify, and 
delete user accounts and groups, but cannot manage Domain Admin or higher 
privileged accounts.
- *Security Note:* This group can be abused to create new accounts or modify 
existing accounts for persistence in the domain.

## LDAP (Lightweight Directory Access Protocol)
- LDAP is the protocol used to query and communicate with Active Directory. 
Anything that wants to talk to Active Directory uses this protocol.
- LDAP enumeration is one of the first steps an attacker takes after gaining 
a foothold. It involves using LDAP to map the domain environment and retrieve 
information such as users, group memberships, privileges, systems, and admins.
- It involves sending LDAP queries to the domain controller for information, 
and the DC responds.
- By default, the DC responds even to the least privileged account, meaning 
if an attacker compromises any domain account they can use LDAP queries to 
fully map the domain environment without requiring elevated privileges.

## Group Policy (GPO)
- Group Policy is a feature of Active Directory that allows administrators to 
define and enforce specific configurations and settings for users and computers 
across the domain.
- A **Group Policy Object (GPO)** is the container that holds these policy 
settings. GPOs can be linked to the domain, a site, or an OU, and they apply 
to all users and computers within the linked scope.
- Common uses include enforcing password policies, restricting software 
execution, configuring firewall rules, and deploying software.
- *Security Note:* GPOs are a high-value target for attackers. A user or 
group with GPO modification rights can push malicious configurations to every 
machine the GPO applies to, making it a powerful persistence and lateral 
movement mechanism. Attackers who gain rights to modify a GPO linked to the 
domain root effectively have a way to push changes to every machine in the 
domain.