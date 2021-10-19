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

# Window Pains 3

> Using the memory dump file from Window Pains, find out the name of the malicious process. Submit the flag as flag{process-name_pid} (include the extension). Example: flag{svchost.exe_1234}

This task is asking us to find a malicious process. I initially thought that the process would be easy to spot and started to look for clearly suspicious behaviour via the PsTree module. Here, I was hoping to see some form of suspicious behaviour, for example Notepad.exe spawning a PowerShell process or something clearly malicious. However, we did not find anything of that sort. 

![windows.PsTree](/assets/images/windowpains_3.png)

After closer enumeration of the individual processes, one particular process sticks out is `userinit.exe` PID 8180. I noticed that there was already 1 other `userinit.exe` running. Let's take a look at the two different `userinit.exe` processes. 

![userinit.exe1](/assets/images/windowpains_4.png)

We note that this `userinit.exe` is the child process of `winlogon.exe`.

![userinit.exe2](/assets/images/windowpains_5.png)

However, for this `userinit.exe` PID 8180, the parent process 2252 is not on the list shown by PsTree or PsScan. In order to further enumerate this suspicious behaviour, I went to do some further research on `userinit.exe` as a process.

## A Primer on USERINIT.EXE 
`userinit.exe` is responsible for user initialization, launching logon scripts and launching the windows shell. It is typically spawned from parent process `winlogon.exe`. Typically, the number of instances should be 0, so having 2 instances of it here is indeed strange.

The second `userinit.exe` with PID 8180 is extra suspicious since its parent process is also not `winlogon.exe`. 

## Back To Triage
One more useful scan we can run to try and enumerate suspicious behaviour is the inbuilt Malfind scan. Let's see what Volatility's documentation says about the Malfind module.

> Malfind module lists process memory ranges that potentially contain injected code. 

We run the scan and verify that our malicious process also appears there. 

![malfind](/assets/images/windowpains_6.png)

# Window Pains 4

> We want to see if any other machines are infected with this malware. Using the memory dump file from Window Pains, submit the SHA1 checksum of the malicious process. Submit the flag as flag{SHA1 hash}. CAUTION Practice good cyber hygiene! Use an isolated VM to download/run the malicious process. While the malicious process is relatively benign, if you're using an insecurely-configured Windows host, it may be possible for someone to compromise your machine if they can reach you on the same network.

We dump the entire process as a single file via the following command:

```bash
python3 vol.py -f ../cases/memdump/physmemraw windows.pslist --pid 8180 --dump
```

We use `sha1sum` to calculate the SHA1 hash of the file and run it through VirusTotal. Our findings indeed reflect that this file is known to be malicious and a suspected Trojan. 

![dump](/assets/images/windowpains_8.png)

![virustotal](/assets/images/windowpains_9.png)

# Conclusion 
This quick series of CTF challenges has been a great introduction to using Volatilty for memory analysis. I hope this article has also introduced you to some of the Volatility features and I look forward to further learning of what else this tool can do. 