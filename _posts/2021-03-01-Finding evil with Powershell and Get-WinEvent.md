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

With the FilterHashtable parameter a simple query for Sysmon logs looks something like this:

<img src="https://raw.githubusercontent.com/dfirale/dfirale.github.io/master/assets/images/GetWinEvent/pic1.PNG" width="100"/>

When you use FilterHashtable, the data field must be an exact match. Wildcards are not accepted:

<img src="https://raw.githubusercontent.com/dfirale/dfirale.github.io/master/assets/images/GetWinEvent/pic3.PNG" width="100"/>

For better performance, <u>always specify atleast the event id you are looking for</u>. I have total of 215 512 Sysmon logs, 12 027 of which are process create events (1). Here is a comparison between specifying the event id inside the FilterHashtable parameter and piping it to Where-Object:

<img src="https://raw.githubusercontent.com/dfirale/dfirale.github.io/master/assets/images/GetWinEvent/perf.png" width="100"/>

Read more about Get-WinEvent cmdlet: [Microsoft documentation](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.diagnostics/get-winevent?view=powershell-7.1)

# Parsing the results

If you navigate to Windows Event Viewer and toggle the XML View of an event, it looks like this:

<img src="https://raw.githubusercontent.com/dfirale/dfirale.github.io/master/assets/images/GetWinEvent/xmlview.png" width="100"/>

These are the same *Data* fields which can be used in Get-WinEvent query.

Luckily, the event log record object you get from Get-WinEvent includes a method to create an XML version. This document has properties that expose the data used to construct the event log record:

<img src="https://raw.githubusercontent.com/dfirale/dfirale.github.io/master/assets/images/GetWinEvent/nodes.png" width="100"/>

We can use the *Data* nodes to do a more precise and unrestricted query with a simple logic. It also gives us more options to display the results. This can be accomplished by selecting the nodes using [XPath](https://devblogs.microsoft.com/scripting/understanding-xml-and-xpath/) queries as seen in next image.

Here is an example of querying Sysmon process create events where whoami.exe was executed as system user:

<img src="https://raw.githubusercontent.com/dfirale/dfirale.github.io/master/assets/images/GetWinEvent/whoami.PNG" width="100"/>

`You can select the wanted node attributes with .SelectSingleNode and match with their values.`

# Finding the evil

For the baseline and proof of concept I wanted to use [Florian Roth's](https://twitter.com/cyb3rops) [Godmode Sigma rule](https://github.com/Neo23x0/sigma/blob/master/other/godmode_sigma_rule.yml). It basically includes the most effective search queries to detect malicious activity - following the principle: if you had only one shot, what would you look for?

By dividing the Sigma rule conditions into arrays, we can loop through the eventlogs and send the results via Syslog when specific node value is matched in our predefined array. To recieve Syslog packets, you can use your own receiver or the simple powershell and python receivers provided in my [repository](https://github.com/dfirale/evtscanner).

Example of Sigma CommandLine conditions:

<img src="https://raw.githubusercontent.com/dfirale/dfirale.github.io/master/assets/images/GetWinEvent/cmdline.PNG" width="100"/>

Let's do some regex magic and put that into an array:

<img src="https://raw.githubusercontent.com/dfirale/dfirale.github.io/master/assets/images/GetWinEvent/cmdlineps.PNG" width="100"/>

Extract the needed nodes (CommandLine/ParentCommandLine) using [XPath](https://devblogs.microsoft.com/scripting/understanding-xml-and-xpath/) queries and look for matching values in array (if node value match --> send-syslog):

<img src="https://raw.githubusercontent.com/dfirale/dfirale.github.io/master/assets/images/GetWinEvent/cmdlineps1.PNG" width="100"/>

Same thing without Sysmon can be accomplished by using native [Windows process creation auditing](https://docs.microsoft.com/en-us/windows-server/identity/ad-ds/manage/component-updates/command-line-process-auditing) with commandline auditing enabled:

<img src="https://raw.githubusercontent.com/dfirale/dfirale.github.io/master/assets/images/GetWinEvent/4688.PNG" width="100"/>

You could also look for malware droppers launched by Office programs (winword.exe launching powershell.exe etc.):

<img src="https://raw.githubusercontent.com/dfirale/dfirale.github.io/master/assets/images/GetWinEvent/odropper.PNG" width="100"/>

<img src="https://raw.githubusercontent.com/dfirale/dfirale.github.io/master/assets/images/GetWinEvent/odropperlogic.PNG" width="100"/>

Let's fire it up and see what the results are. I'm going to use a receiver on the same host so the ip parameter is pointed to localhost: `.\evtscanner.ps1 -ip 127.0.0.1`

After running the script for a while we already get some results. First one is a `whoami.exe` executed as `system` user and the second one is triggered by a run key `Microsoft\Windows\CurrentVersion\Run` on commandline:

<img src="https://raw.githubusercontent.com/dfirale/dfirale.github.io/master/assets/images/GetWinEvent/results.png" width="100"/>

Results are always presented in the same format:

`<event time> WinEvtlog: <channel>: EVENT-ID(<id>): <provider>: COMPUTER: <computer>: <full log in one line> CollectTime: <time when scanned>`

The script execution time really depends on how many process create(1), registry(12,13) and file create(11) Sysmon events you have. With ~77k of those events the execution time is around 9 minutes and 33 seconds:

<img src="https://raw.githubusercontent.com/dfirale/dfirale.github.io/master/assets/images/GetWinEvent/performance.png" width="100"/>

I'll try to update this script with some more Sigma rules in future. Hat tip to [Florian](https://twitter.com/cyb3rops) and other contributors for maintaining the [Sigma](https://github.com/SigmaHQ/sigma) rule base.

Don't forget to check out the script described in this blog post! [https://github.com/dfirale/evtscanner](https://github.com/dfirale/evtscanner)

<img src="https://raw.githubusercontent.com/dfirale/dfirale.github.io/master/assets/images/GetWinEvent/logwell.jpg" width="100"/>