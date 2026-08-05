# Blue Team Knowledge Base

Windows internals, attack types, security frameworks, firewall, IDS, and Splunk. Mostly everything a cybersecurity professional needs to know.

This repo is my personal documentation of everything I've learned in my journey. It covers the theory and concepts that actually matter in a real security operations environment.

I already have a separate repo for networking fundamentals and protocol analysis labs. This one picks up from there and goes deeper into the defensive security side.

## What's Inside

### Windows Fundamentals
How Windows actually works under the hood: architecture, file system, user accounts, processes, services, registry, event logs, and Active Directory. Written from a SOC perspective, so the focus is always on what matters for detection and investigation.

### Attack Types
Social engineering, password attacks, DoS and DDoS, malware, and MITM attacks. Each one covers how the attack works and what to look for.

### Security Frameworks
Pyramid of Pain, Cyber Kill Chain, Unified Kill Chain, and MITRE ATT&CK. These are the frameworks that SOC teams actually use to structure their thinking, documented the way I understand and use them.

### Firewall and IDS
Firewall concepts, IDS vs IPS, Snort basics, network segmentation, and defense in depth. The defensive architecture side of blue teaming.

### Splunk and SIEM
How SIEM works, Splunk architecture, SPL queries with real examples, and how to use Splunk the way a SOC analyst actually uses it.

### TTPs
Technique-level writeups mapped to MITRE ATT&CK, covering Initial Access, Credential Access, Lateral Movement, Persistence, and Defense Evasion. Each technique covers what it is, why attackers use it, how the attack works, and detection: event IDs, fields to check, and correlation steps.