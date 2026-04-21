# Cyber Kill Chain 

## What is Cyber Kill Chain?
- Cyber Kill Chain is a security framework that breaks down the lifecycle of a cyberattack into 7 distinct linear stages  
- It defines the steps used by adversaries or malicious actors in cyberspace  
- To succeed, an adversary typically goes through multiple phases of an attack  
- Developed by Lockheed Martin in 2011  

---

## Phases in Cyber Kill Chain 

### 1. Reconnaissance 
- Reconnaissance is the research and planning phase of an attack, used to gather information about the target and identify vulnerabilities  

**Types of reconnaissance:-**
- Active :- involves direct interaction with the target  
  - methods :- social engineering, port scanning, banner grabbing  
- Passive :- involves no direct interaction with the target  
  - methods :- social media, OSINT, data breaches  

**Examples :-** harvesting email addresses, mapping network architecture, finding subdomains and IP addresses  



### 2. Weaponization 
- This phase involves turning information gathered during reconnaissance into a usable attack by creating malware or combining exploits into a payload  
- *Payload* is the malicious code executed on the target system  

**Examples:-**
- tailoring phishing attacks  
- creating infected documents  
- setting up command and control infrastructure  



### 3. Delivery 
- Delivery refers to the method used to transmit the payload to the target system  

**Examples :-**
- phishing emails  
- USB drops  
- watering hole attacks  



### 4. Exploitation 
- Exploitation refers to taking advantage of vulnerabilities to execute malicious code on the target system  

**Signs:-**
- unexpected process creation  
- registry changes or new service creation  
- suspicious command line arguments  



### 5. Installation
- In this phase, the attacker installs malware or a backdoor to establish persistence and maintain long term access after exploitation  

**Examples:-**
- installing a web shell  
- installing a backdoor  
- creating or modifying Windows services  
- modifying registry keys  



### 6. Command & Control 
- After gaining persistence, the attacker connects the compromised system to a command and control server to remotely control it and execute commands  

**Techniques :-** DNS tunneling, DNS beaconing  


### 7. Actions on Objectives 
- This is the final phase where the attacker achieves their goal  

**Examples:-**
- credential harvesting  
- privilege escalation  
- internal reconnaissance  
- lateral movement  
- data collection and exfiltration  

---

## Why Cyber Kill Chain 
- The primary purpose of the Cyber Kill Chain is to help security teams understand and disrupt attacks by breaking them into stages  
- It also acts as a foundation for more advanced frameworks like Unified Kill Chain and MITRE ATT&CK  