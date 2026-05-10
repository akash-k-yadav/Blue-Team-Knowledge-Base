# Windows Event Log


## What is a Log?

So basically a log is just a file where the system keeps writing down whatever is happening. Like every action, every error, every login attempt — it gets recorded with a time and date.

Think of it like a notebook where your computer is just constantly writing stuff down in the background without you even noticing. If something breaks or something weird happens, you go check that notebook and try to figure out what went wrong.

Without logs you're basically guessing. You have no idea what happened, when it happened, or what caused it.

---

## What is Windows Event Log?

Windows Event Log is the logging system that comes built into Windows. You don't install it, it's just there. It runs in the background on every Windows machine and keeps track of pretty much everything — errors, warnings, logins, service crashes, hardware issues, all of it.

You can open it by running `eventvwr.msc` which opens the Event Viewer. It's a GUI tool where you can browse through all the logs. You can also query logs using PowerShell if you prefer that.

Each thing that gets recorded is called an **Event**. Every event has some basic info attached to it like — what happened, when it happened, which app or service caused it, and how serious it is.

---

## Purpose of Windows Event Log

The main reason event logs exist is so you have something to look at when things go wrong. If a service crashes or an app stops working, the log usually tells you why. Without it you'd have no starting point for debugging.

It's also useful for security stuff. Like if someone is trying to brute force a login or an account keeps getting locked out, that all shows up in the logs. Security people use this to catch suspicious activity before it becomes a bigger problem.

Another use is auditing. In some organizations they need to keep a record of who accessed what and when. Event logs give you that history.

Honestly most people ignore event logs until something breaks, but once you start using them regularly they save a lot of time troubleshooting.

## Log Types

Windows doesn't just dump everything into one single log file. It splits things up into different logs based on where the event is coming from. Here are the main ones you'll actually see.

**Application Log**
This one has events from applications and programs running on the system. So if some software crashes or throws an error, it usually ends up here. Any app that is built to write to the event log will write to this one.

**System Log**
This is for events coming from Windows itself — like drivers, core services, hardware stuff. If a driver fails to load or a windows service goes down, you'll find it in here. This one is probably the most useful when you're troubleshooting system level problems.

**Security Log**
This one is different from the others because not just anyone can write to it. Only Windows and authorized security software can. It records things like login attempts, failed logins, account lockouts, file access if auditing is turned on. If you're doing anything security related this is the first place you check.

**Setup Log**
This one records stuff related to Windows installation and updates. Honestly you don't check this one often but it's useful when an update fails and you want to know why.

**Forwarded Events**
This one is more of an enterprise thing. When multiple machines are set up to send their logs to one central machine, those logs land here. So instead of checking 50 machines individually you check one place.