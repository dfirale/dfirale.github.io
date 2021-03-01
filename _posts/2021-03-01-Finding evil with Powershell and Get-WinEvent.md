---
layout: post
title: Finding evil with Powershell and Get-WinEvent
description: Logs, logs, logs
category: DFIR
tags: [DFIR, Powershell, Sigma, Rules]
---

# Basic Get-WinEvent usage

PowerShell is natively installed in Windows Vista and newer, and includes the Get-WinEvent cmdlet by default.

You can use Get-WinEvent cmdlet to scan local or remote eventlogs with specified criteria e.g. log source, event id, time and some specific keywords.

With the FilterHashtable parameter a simple query looks something like this:

<img src="https://raw.githubusercontent.com/dfirale/dfirale.github.io/master/assets/images/GetWinEvent/pic1.PNG" width="100"/>

If you use FilterHashtable the data field must be an exact match. Wildcards are not accepted:

<img src="https://raw.githubusercontent.com/dfirale/dfirale.github.io/master/assets/images/GetWinEvent/pic3.PNG" width="100"/>

For better performance, <u>always specify atleast the event id you are looking for</u>. I have total of 215 512 Sysmon logs which 12 027 are process create events (1). Yes I'm a hoarder.. Here is a comparison between specifying the id and piping it to where-object:

<img src="https://raw.githubusercontent.com/dfirale/dfirale.github.io/master/assets/images/GetWinEvent/perf.png" width="100"/>