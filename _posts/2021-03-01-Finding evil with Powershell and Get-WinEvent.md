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

When you use FilterHashtable, the data field must be an exact match. Wildcards are not accepted:

<img src="https://raw.githubusercontent.com/dfirale/dfirale.github.io/master/assets/images/GetWinEvent/pic3.PNG" width="100"/>

For better performance, <u>always specify atleast the event id you are looking for</u>. I have total of 215 512 Sysmon logs which 12 027 are process create events (1). Yes, I'm a hoarder.. Here is a comparison between specifying the id and piping it to where-object:

<img src="https://raw.githubusercontent.com/dfirale/dfirale.github.io/master/assets/images/GetWinEvent/perf.png" width="100"/>

[Microsoft documentation](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.diagnostics/get-winevent?view=powershell-7.1)

# Parsing the results

If you navigate to Windows Event viewer and toggle the XML View of an event, it looks like this:

<img src="https://raw.githubusercontent.com/dfirale/dfirale.github.io/master/assets/images/GetWinEvent/xmlview.png" width="100"/>

These are the same data fields that you can use in your Get-WinEvent query.

Luckily, the event log record object you get from Get-WinEvent includes a method to create an XML version. This document has properties that expose the data used to construct the event log record:

<img src="https://raw.githubusercontent.com/dfirale/dfirale.github.io/master/assets/images/GetWinEvent/nodes.png" width="100"/>

We can use these data nodes to do a more precise and unrestricted query with a simple logic. It also gives us more options to display the results. This can be accomplished by selecting the nodes using [XPath](https://devblogs.microsoft.com/scripting/understanding-xml-and-xpath/) queries as seen in next image.

Here is an example of querying Sysmon process create events where whoami.exe was executed as system user:

<img src="https://raw.githubusercontent.com/dfirale/dfirale.github.io/master/assets/images/GetWinEvent/whoami.PNG" width="100"/>

# Finding the evil

For the baseline and proof of concept I wanted to use [Florian Roth's](https://twitter.com/cyb3rops) [Godmode Sigma rule](https://github.com/Neo23x0/sigma/blob/master/other/godmode_sigma_rule.yml). It basically includes the most effective search queries to detect malicious activity - following the principle: if you had only one shot, what would you look for?

By breaking down the Sigma rule conditions into arrays, we can loop through the eventlogs and print results when specific criteria is met.

Example of Sigma CommandLine conditions:

<img src="https://raw.githubusercontent.com/dfirale/dfirale.github.io/master/assets/images/GetWinEvent/cmdline.PNG" width="100"/>

Let's do some regex magic and put that into an array:

<img src="https://raw.githubusercontent.com/dfirale/dfirale.github.io/master/assets/images/GetWinEvent/cmdlineps.PNG" width="100"/>

Extract the needed nodes (CommandLine/ParentCommandLine) using [XPath](https://devblogs.microsoft.com/scripting/understanding-xml-and-xpath/) queries and look for matches in array (if match --> send-syslog):

<img src="https://raw.githubusercontent.com/dfirale/dfirale.github.io/master/assets/images/GetWinEvent/cmdlineps1.PNG" width="100"/>

Same thing without Sysmon can be accomplished by using native [Windows Process creation auditing](https://docs.microsoft.com/en-us/windows-server/identity/ad-ds/manage/component-updates/command-line-process-auditing) with commandline auditing enabled:

<img src="https://raw.githubusercontent.com/dfirale/dfirale.github.io/master/assets/images/GetWinEvent/4688.PNG" width="100"/>

You could also look for malware droppers launched by Office programs (winword.exe launching powershell.exe etc. etc.):

<img src="https://raw.githubusercontent.com/dfirale/dfirale.github.io/master/assets/images/GetWinEvent/odropper.PNG" width="100"/>

<img src="https://raw.githubusercontent.com/dfirale/dfirale.github.io/master/assets/images/GetWinEvent/odropperlogic.PNG" width="100"/>