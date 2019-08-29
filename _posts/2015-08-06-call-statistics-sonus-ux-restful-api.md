---
permalink: /call-statistics-sonus-ux-restful-api
title: "Call Statistics from Sonus UX 1000/2000 using the RESTful API"
subHeading: "We all love statistics"
description: "We all love statistics"
heroImage: "/assets/img/sonus-monitor.jpg"
---

### Introduction

Challenged to collect call statistics from our Sonus UX 1000 SBC the first thought was to use SNMP. After enabling SNMP and pointing an SNMPwalk tool with MIB to the SBC, I struggled to find anything useful that could be used by our monitoring software with its native SNMP sensor. Why taunt me with your call counters in the web administration interface and not expose them through SNMP?

My colleagues have been using the Sonus UX REST API and PowerShell to load up configurations when deploying so thought I'd give it a go. Not much browsing time later and I had the API calls needed:

*   CAS Signalling Group Resource - [https://support.sonus.net/display/UXAPIDOC/Resource+-+cassg](https://support.sonus.net/display/UXAPIDOC/Resource+-+cassg "Resource - cassg")
*   ISDN Signalling Group Resource - [https://support.sonus.net/display/UXAPIDOC/Resource+-+isdnsg](https://support.sonus.net/display/UXAPIDOC/Resource+-+isdnsg "Resource - isdnsg")
*   SIP Signalling Group Resource - [https://support.sonus.net/display/UXAPIDOC/Resource+-+sipsg](https://support.sonus.net/display/UXAPIDOC/Resource+-+sipsg "Resource - sipsg")

This is what I ended up with.

### The Script

Nothing much to it.

```
.\Get-SonusUxCallCounters.ps1 -Username "restuser" -Password "restpassword" -ServerFqdn "sbc.contoso.com" -Type SIP -Id 1
```

Type can be SIP, ISDN or CAS.

### PRTG

Choose the "EXE/Script Advanced" sensor type and fill in the settings:

<div>![](/assets/img/prtg.png)</div>

I've used the Linux username and password of the parent device to pass through the REST credentials.

A little while later...

### Stats!

<div>![](/assets/img/graph.png)</div>

Guess everyone went home around 5.30 :)

<div>![](/assets/img/table.png)</div>

That'll keep the boss entertained.

### Download

Downoad [Get-SonusUxCallCounters.ps1](/assets/misc/get-sonusuxcallcounters.zip "Get-SonusUxCallCounters.ps1")