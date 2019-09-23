---
permalink: /skype-for-business-lync-web-services-warmup
title: "Skype for Business (Lync) Web Services Warmup"
subHeading: "Application Initialisation is my new favourite Windows technology"
description: "Using IIS application initialization feature to keep Skype for Business (Lync) web services running"
heroImage: "/assets/img/iispreload.png"
---

### Problem

This blog runs on Umbraco and due its popularity, or lack thereof, IIS was shutting down the application pool frequently. Not the best first impression if a visitor has to wait 10 seconds for the application to start before they've even received a single byte of HTML. The same issue applies to Skype for Business (Lync) and all of its web services (meet, dialin, lyncdiscover, scheduler, ucwa, ...).

### Solution

While writing a script that would call the website to be executed from a scheduled task, I happened across a technology called IIS Application Initialization. As explained by the documentation, "<span>The IIS 8.0 Application Initialization feature enables website administrators to configure IIS 8.0 to proactively perform initialization tasks for one or more web applications</span>".

No need to document the steps required, it's all here: [http://www.iis.net/learn/get-started/whats-new-in-iis-8/iis-80-application-initialization](http://www.iis.net/learn/get-started/whats-new-in-iis-8/iis-80-application-initialization "IIS 8.0 Application Initialization")

Now when I visit this blog, even after an iisreset, it's always warm and ready to go.

<span>Next I turned my attention to Skype for Business and there are two warm up solutions that I have used, the original by [Drago Totev](http://www.lynclog.com/2013/12/user-might-experince-delay-when-join.html "User might experince delay when join Lync Online Meeting") and the improved version by [Greig Sheridan](https://greiginsydney.com/new-lyncmeetingwarmup/ "New-LyncMeetingWarmup") however they both focus on just the meeting join URLs. </span>

### Script

The Powershell script will configure each of the Skype for Business or Lync application pools to be "Always running". It will then iterate through each of the applications / virtual directories and set preload enabled. The script does not enable preload for CSCP, PowerShell remoting and legacy services like MCX. Tested on Windows Server 2012 and 2012 R2 with both Lync Server 2013 and Skype for Business Server 2015.

It's up to you to install the Application Initialization Windows feature as detailed in the [IIS documentation](http://www.iis.net/learn/get-started/whats-new-in-iis-8/iis-80-application-initialization "IIS 8.0 Application Initialization") and do an iisreset.

Whether you _should_ enable Application Initialization on your Skype for Business or Lync front-end servers... Well... Caveat emptor. IIS will use more memory but so long as you have more than 2.5GB free afterwards, then you should be fine.

If this sounds too risky for you, perhaps stick with [Greig Sheridan's](https://greiginsydney.com/new-lyncmeetingwarmup/ "New-LyncMeetingWarmup") solution.

### Revision History

v1.0: 13 Sep 2015: Initial Release

### Download

Download [Set-CsApplicationInitialization.ps1](/assets/misc/set-csapplicationinitialization.zip "Set-CsApplicationInitialization.zip").

### Credits

New-LyncMeetingWarmup - [https://greiginsydney.com/new-lyncmeetingwarmup/](https://greiginsydney.com/new-lyncmeetingwarmup/ "New-LyncMeetingWarmup")

User might experince delay when join Lync Online Meeting - [http://www.lynclog.com/2013/12/user-might-experince-delay-when-join.html](http://www.lynclog.com/2013/12/user-might-experince-delay-when-join.html "User might experince delay when join Lync Online Meeting")