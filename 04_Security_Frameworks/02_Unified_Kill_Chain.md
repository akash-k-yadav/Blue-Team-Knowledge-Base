# Unified Kill Chain 

## What is Unified Kill Chain 
- Unified Kill Chain is a cybersecurity framework that expands the original 7 stage Cyber Kill Chain into 18 phases, organized across 3 arcs, and maps each phase to specific techniques from MITRE ATT&CK  

---

### Purpose of Unified Kill Chain 
- give defenders a more complete and realistic model of how modern attacks unfold  
- bridge strategic phases with operational level techniques  
- cover attack patterns the original kill chain could not model, such as insider threats, cloud attacks, supply chain attacks, and long dwell times  
- help defenders understand two things at once: where in the attack they are and what exactly the attacker is doing  


### Phases vs Technique 
- the UKC tells us what stage the attacker is at, while the mapped ATT&CK techniques explain how they performed the action  



### Note:
- the 18 phases are a complete map of possible attacker actions, not a mandatory checklist  
- real attacks choose their own path  

for example:
- a basic ransomware attack may only use 6–8 phases from Arc 1 and Arc 3  
- a nation state attacker may move through most phases over a long period  
- attackers can skip phases, repeat phases, or run multiple phases in parallel  

- *think of it like a map of a city. not every driver takes every road, but to understand all possible paths, you need the full map*  



### The Arcs 
- the UKC organizes its 18 phases into 3 arcs  
- each arc represents a major part of the attack from the attacker’s goal perspective  
- arcs are not strictly sequential, attackers may loop within an arc or move between arcs  

---

## Arc 1 (Initial Foothold)

- this arc involves going from zero knowledge to an established and hidden presence inside the target  
- all Arc 1 activities happen before the attacker gains stable internal access  


### Phase 1. Reconnaissance 
- includes information gathering about the target  

- passive :- no direct interaction with the target (OSINT, data breaches, social media)  
- active :- direct interaction with the target (port scanning, banner grabbing, subdomain enumeration) 


### Weaponization 
- crafting the attack payload such as malicious documents, rootkits, or custom droppers  
- no direct contact with the target yet  



### Delivery 
- sending the payload to the target via phishing emails, watering hole attacks, USB drops, or supply chain compromise  



### Social Engineering 
- refers to techniques used to trick users into revealing sensitive information or executing malware  
- attackers exploit human psychology by creating urgency or mimicking trusted services  


### Exploitation 
- taking advantage of vulnerabilities to gain access or execute malicious code  



### Persistence 
- persistence refers to techniques used to maintain long term access to a compromised system  
- includes installing backdoors, modifying startup items, or changing registry keys  



### Defense Evasion 
- techniques used to avoid detection by security tools like EDR, firewall, IDS, or IPS  
- includes deleting logs, abusing trusted processes, or using protocols like DNS and HTTP  



### Command and Control 
- after gaining persistence, the attacker connects the system to a command and control server to remotely control it  

**Techniques :-** DNS tunneling, DNS beaconing  



### Pivoting 
- pivoting is when an attacker uses a compromised machine as a bridge to access internal systems not directly reachable from the internet  
- the compromised host forwards attacker traffic into the internal network, making it appear as trusted  

**Common pivoting techniques:-**
- SSH tunneling, SOCKS proxy, port forwarding, double pivot, DNS tunneling  

**Why pivoting is hard to detect:-**
- traffic appears to originate from a trusted internal system  
- encrypted tunnels like SSH or HTTPS hide payload content  
- DNS tunneling blends into normal traffic and is difficult to block  

---

## Arc 2: Network Propagation 

- the goal of this arc is to expand from one compromised host to control of multiple internal systems  
- this is where sophisticated attackers spend most of their time  
- it loops repeatedly, each loop moving the attacker to a more privileged host  



### Discovery 
- this phase includes mapping the internal network from the pivot host  
- attacker uses the pivot host from Arc 1 to identify vulnerabilities and discover assets like database servers, file servers, etc  



### Privilege Escalation 
- privilege escalation refers to gaining access to higher permission accounts on the compromised host  
- techniques include kerberoasting, token impersonation, exploiting local misconfigurations  


### Execution 
- execution involves running attacker tools and payloads  
- examples include running PowerShell commands, mshta, psexec  



### Credentials Access 
- this phase involves stealing usernames, passwords, or authentication tokens  
- techniques include LSASS dumping, pass the hash, keylogging, accessing credential stores  



### Lateral Movement 
- lateral movement refers to moving to new and more privileged hosts using stolen credentials  
- techniques include pass the hash, kerberoasting, SMB abuse, RDP  

### The Arc 2 Loop 
- this is the most dangerous part of the attack  

```
Dicovery  -------> privilege esclation ------> Credentials Access
     ^                                             |    
     |                                             |    
     |                                             |
     |                                             V
new compromised host  <-------------------- lateral movement
```


---

### Real World Example: Arc 2
- **loop 1:-** from a web server (pivot), discover a developer workstation, steal a cached session token, move laterally  
- **loop 2:-** on the workstation, BloodHound reveals a service account with admin rights on multiple machines, kerberoast and crack password  
- **loop 3:-** on one of those machines, a domain account logged in recently, dump its hash from LSASS using mimikatz  
- **loop 4:-** with domain admin hash, take control of the domain controller, now Arc 3 begins  



## Arc 3: Action on Objectives 
- this is where the attacker achieves their goal  
- this arc varies depending on attacker motive  



### Collection 
- attacker gathers target data such as documents, emails, databases, credentials, or intellectual property  


### Exfiltration 
- attacker sends collected data outside the network  
- methods include HTTPS, cloud storage, DNS tunneling, or SFTP  



### Impact 
- this phase focuses on damaging the target  
- examples include ransomware, disk wiping, OT/ICS disruption, DDoS  



### Objectives
- this represents the final goal of the attacker  
- examples include financial gain, espionage, sabotage, hacktivism, ransom  



### Arc varies by attacker motives
- ransomware :- collection + exfiltration + impact  
- espionage :- collection + exfiltration (stay hidden, no impact)  
- sabotage :- impact only  


## Key Points 
- UKC covers both phases and techniques, phases show where and techniques show how  
- 18 phases are grouped into 3 arcs: foothold, propagation, and action on objectives  
- Arc 2 loops, representing dwell time that traditional kill chain misses  
- pivoting connects Arc 1 to Arc 2  
- not every attack uses all phases, attacker goals determine the path  

