# Phishing

## What is it?
- Phishing is a type of social engineering attack in which adversaries send
emails, messages, or SMS to trick victims into revealing sensitive information
or performing malicious tasks. In phishing, adversaries create a sense of panic
or urgency, or impersonate a legitimate entity to gain trust.

### MITRE ATT&CK
- Tactic: Initial Access
- Technique ID: T1566
- Sub-Techniques:
  - **T1566.001 :- Spearphishing Attachment:** phishing email contains a
  malicious attachment such as a document, PDF, or executable.
  - **T1566.002 :- Spearphishing Link:** phishing email contains a malicious
  URL that redirects the victim to a malicious site or triggers a download.
  - **T1566.003 :- Spearphishing via Service:** adversaries use third-party
  services such as social media, messaging platforms, or file sharing services
  instead of email to deliver the phishing content.
  - **T1566.004 :- Spearphishing Voice:** adversaries use phone calls or voice
  messages to trick victims into revealing information or performing actions,
  also known as vishing.

## Why Attackers Use It
- Phishing is one of the most used tactics to gain an initial foothold because
it targets the human element, which no technical control can fully eliminate.
Even with strong perimeter defenses, a single employee clicking a link or
opening an attachment bypasses all of them.
- Every employee in a domain environment has an email address, and these are
often publicly available on the organization's website, making target
identification trivial for an attacker.
- AI tools such as ChatGPT and Claude have made phishing significantly more
effective by removing the language and grammar errors that previously served
as detection signals. Adversaries can now generate convincing, localized, and
contextually accurate phishing content at scale with minimal effort, lowering
the barrier for both volume and targeted campaigns.

## How Phishing Works
- There are many phishing scenarios. Covering all cases is not within the scope
of this repo, so the most commonly used ones are covered below.

**Scenario 1: Malicious URL**
- The attacker embeds a URL or link to a malicious website in the email.

```
Victim clicks URL ---> Redirected to malicious site ---> Payload downloaded
onto victim machine
```

**Scenario 2: Malicious Attachment**
- Adversaries embed malware or macro/VBS/PowerShell scripts inside documents
(PDF, JPG, DOCX, XLSX, etc.).
- The script either executes and initiates an outbound network connection to
download a payload, or contains the actual malware itself.

## Detection

### Event IDs to Monitor
| Event ID | Source | Description |
|---|---|---|
| 1 | Sysmon | Process creation with full command line (requires Sysmon deployed) |
| 4688 | Windows Security | Process creation (requires command line auditing enabled) |
| 3 | Sysmon | Network connection initiated by a process |
| 7 | Sysmon | Image loaded (DLL loaded by a process) |
| 11 | Sysmon | File created |

### Suspicious Indicators
- Office applications (winword.exe, excel.exe, powerpnt.exe) spawning
scripting engines or shells:
  - winword.exe → powershell.exe
  - winword.exe → cmd.exe
  - excel.exe → wscript.exe
  - excel.exe → mshta.exe
- PowerShell launched with encoded commands (Base64 in command line arguments)
- Outbound network connections initiated by Office applications or scripting
engines (Event ID 3)
- Files written to temp directories or user AppData by Office processes
(Event ID 11)
- Execution of scripts from temp or download directories

### Correlation Framework
**Step 1:** Alert fires on Event ID 4688 or Sysmon Event ID 1. Identify the
process that was created and note the parent process.

**Step 2:** Check the parent-child relationship. Is the parent process an
Office application, browser, or email client? If yes, treat as high suspicion.
Abnormal parent-child combinations are the primary signal here.

**Step 3:** Analyze the command line arguments of the created process. Look
for:
- Base64 encoded strings
- Download cradles (Invoke-WebRequest, IEX, DownloadString, curl, wget)
- Execution policy bypass flags (-ep bypass, -ExecutionPolicy Bypass)
- Unusual file paths (temp, AppData, ProgramData)

**Step 4:** If a network connection is present (Sysmon Event ID 3), pivot to
the destination IP or domain:
- Check reputation on VirusTotal and WHOIS
- Check domain age (newly registered domains are a red flag)
- Check if the domain is impersonating a legitimate service

**Step 5:** Trace the process tree fully. Map every child process spawned from
the initial suspicious process. The goal is to reconstruct the full execution
chain from the initial click to the final payload.

**Step 6:** Check for file creation events (Sysmon Event ID 11) around the
same timestamp. Note what files were written, where, and by which process.
Cross reference with VirusTotal if hashes are available.

**Step 7:** Correlate with email gateway logs if available. Identify the
sender, subject, and delivery time. This confirms the initial vector and
helps scope whether other users received the same email.