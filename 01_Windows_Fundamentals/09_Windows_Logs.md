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

