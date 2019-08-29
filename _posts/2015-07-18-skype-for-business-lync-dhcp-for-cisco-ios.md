---
permalink: /skype-for-business-lync-dhcp-for-cisco-ios
title: "Skype for Business (Lync) device DHCP for Cisco IOS"
subHeading: "Some PowerShell scripting"
heroImage: "/assets/img/powershell.jpg"
---
### Introduction

While designing a Skype for Business deployment for a customer, a requirement was discovered that meant we would need to provide a DHCP configuration to their WAN vendor. Each of the sites would be using Polycom VVX devices with IP addresses obtained from a Cisco router.

[Neill Pert describes the process](https://neillpert.wordpress.com/2011/11/30/configuring-a-cisco-isr861s-dhcp-service-for-lync-edition-handsets/ "Configuring a Cisco ISR861’s DHCP Service for Lync Edition Handset") of taking output from DHCPUtil and massaging to a usable format but wouldn't it be nice if we didn't have to mess around with concatenating strings and counting lengths?

Introducing my first non-commercial contribution to the internets: Get-CsCiscoIosDhcp.ps1

### Creating the Configuration

Fire up PowerShell and run the script:

```
.\Get-CsCiscoIosDhcp.ps1
```

The script will then ask for DHCP options needed for a successful Lync Phone Edition/Qualified deployment with PIN authentication:

~~~~
DNS suffix (eg. contoso.local): contoso.local
UTC/GMT Offset Hours (eg. +/-10): 10
NTP server address (eg. 192.168.0.1): 192.168.0.254
NTP server address (eg. 192.168.0.1): 
DNS search domain (eg. contoso.local): contoso.local
DNS search domain (eg. contoso.local): contoso.com
DNS search domain (eg. contoso.local):
SfB/Lync SIP FQDN (eg. sip.contoso.com): sip.contoso.com
SfB/Lync Web FQDN (eg. lyncwebint.contoso.com): sip.contoso.com
~~~~

Done:

~~~~
-------------------------------------------------------
Cisco IOS DHCP configuration for the following values :
-------------------------------------------------------
DNS suffix : contoso.local
Time Offset : 10
NTP Servers : 192.168.0.254
DNS Search List : contoso.local contoso.com
SIP Server : sip.contoso.com
Web Server : sip.contoso.com
-------------------------------------------------------
domain-name "contoso.local"
option 2 hex 00008ca0
option 42 ip 192.168.0.254
option 119 hex 07636f6e746f736f056c6f63616c0007636f6e746f736f03636f6d00
option 120 hex 000373697007636f6e746f736f03636f6d00
option 43 hex 010c4d532d55432d436c69656e7402056874747073030f7369702e636f6e746f736f2e636f6d040334343305252f4365727450726f762f4365727450726f766973696f6e696e67536572766963652e737663
-------------------------------------------------------
~~~~
### Revision History

v0.1: 19 July 2015: Initial release

### Download

[Download Get-CsCiscoIosDhcp.ps1](/assets/misc/get-csciscoiosdhcp.zip "Download")

### Credits

Configuring a Cisco ISR861’s DHCP Service for Lync Edition Handset - [https://neillpert.wordpress.com/2011/11/30/configuring-a-cisco-isr861s-dhcp-service-for-lync-edition-handsets/](https://neillpert.wordpress.com/2011/11/30/configuring-a-cisco-isr861s-dhcp-service-for-lync-edition-handsets/ "Configuring a Cisco ISR861’s DHCP Service for Lync Edition Handset")

Cisco support community post by Michael Solomonides describing option 119 - [https://supportforums.cisco.com/discussion/11213696/dhcp-pool-dns-search-list-option-119](https://supportforums.cisco.com/discussion/11213696/dhcp-pool-dns-search-list-option-119)

Setting Up DHCP for Devices - [https://technet.microsoft.com/en-us/library/gg398369(v=ocs.14).aspx](https://technet.microsoft.com/en-us/library/gg398369(v=ocs.14).aspx "Setting Up DHCP for Devices")