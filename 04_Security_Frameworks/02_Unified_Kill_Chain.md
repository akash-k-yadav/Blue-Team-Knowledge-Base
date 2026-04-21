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