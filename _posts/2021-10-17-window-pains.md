---
title: "Window Pains: An Introduction to Volatility"
date: 2021-10-17T00:00:00-00:00
categories:
  - dfir
tags:
  - forensics
  - volatility
---

This is a writeup on 4 forensics challenges at DEADFACE CTF 2021. I really enjoyed these challenges because they introduced me to Volatility, an essential memory analysis tool for anyone looking to go into digital forensics. 

# Window Pains

> One of De Monne's employees had their personal Windows computer hacked by a member of DEADFACE. The attacker managed to exploit a portion of a database backup that contains sensitive employee and customer PII.  Inspect the memory dump and tell us the Windows Major Operating System Version, bit version, and the image date/time (UTC, no spaces or special characters). Submit the flag as flag{OS_BIT_YYYYMMDDhhmmss}.

We typically begin triaging a memory dump by getting more information about the system we dumped the memory from. 

We do so by running the following Volatiltiy command, since we already know that the questions is a Windows question:
```bash
python3 vol.py -f <file> windows.info
```

![windows.info](/assets/images/windowpains_1.png)

From the screenshot we can see `MajorOperatingSystemVersion` is 10, and we also have `System Time`, which gives us our flag.

# Window Pains 2
> Using the memory dump file from Window Pains, submit the victim's computer name. Submit the flag as flag{COMPUTER-NAME}.

After a quick Google search, I learnt that the ComputerName is typically stored in the following registry: `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\ComputerName\ActiveComputerName`. 

We use the `PrintKey` functionality to try and access the key. We started at the SYSTEM hive and slowly descend down the values to find the key that we want. 

The final key value is `\REGISTRY\MACHINE\SYSTEM\ControlSet001\Control\ComputerName\ActiveComputerName` and the value is `DESKTOP-IT8QNRI`. 

![windows.printkey](/assets/images/windowpains_2.png)