---
layout: post
title: Finding evil with Powershell and Get-WinEvent
description: Logs, logs, logs
category: DFIR
tags: [DFIR, Powershell, Sigma, Rules]
---

# Get-WinEvent usage

PowerShell is natively installed in Windows Vista and newer, and includes the Get-WinEvent cmdlet by default.

You can use Get-WinEvent cmdlet to scan local or remote eventlogs with specified criteria e.g. log source, event id, time and some specific keywords.

With the FilterHashtable parameter a simple query looks something like this:

<img src="https://raw.githubusercontent.com/dfirale/dfirale.github.io/master/assets/images/GetWinEvent/pic1.PNG" width="100"/>

If you use FilterHashtable the data field must be an exact match. Wildcards are not accepted:

<img src="https://raw.githubusercontent.com/dfirale/dfirale.github.io/master/assets/images/GetWinEvent/pic2.PNG" width="100"/>