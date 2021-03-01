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

The event log record object you get from Get-WinEvent includes a method to create an XML version. This document has properties that expose the data used to construct the event log record:

<img src="https://raw.githubusercontent.com/dfirale/dfirale.github.io/master/assets/images/GetWinEvent/nodes.png" width="100"/>

We can use these data nodes to do a more precise and unrestricted query with a simple logic. It also gives us more options to display the results. This can be accomplished by selecting the nodes using [XPath](https://devblogs.microsoft.com/scripting/understanding-xml-and-xpath/) queries as seen in next image.

Here is an example of querying Sysmon process create events where whoami.exe was executed as system user:

<img src="https://raw.githubusercontent.com/dfirale/dfirale.github.io/master/assets/images/GetWinEvent/whoami.PNG" width="100"/>