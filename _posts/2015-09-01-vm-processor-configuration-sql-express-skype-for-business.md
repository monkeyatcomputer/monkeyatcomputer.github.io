---
permalink: /vm-processor-configuration-sql-express-skype-for-business
title: "VM Processor Configuration and SQL Server Express with Skype for Business (Lync)"
description: "A lesson in Microsoft licensing"
image:
    path: "/assets/img/vspherecores.png"
    alt: "A lesson in Microsoft licensing"
categories: [Unified Communications, Monitoring]
tags: [skype for business, sql server, sql server express]
---

## Problem
While investigating an SQL issue with a customers Lync environment, discovered the following in the local SQL Express instance error logs:

~~~~
SQL Server detected 8 sockets with 1 cores per socket and 1 logical processors per socket, 8 total logical processors; using 1 logical processors based on SQL Server licensing. This is an informational message; no user action is required.
~~~~

Hmmm. That's a little strange. You're only using a single processor on a 8 core virtual machine.

Quick check of the capacity limits of SQL Server Express:

[https://msdn.microsoft.com/en-AU/library/ms143760(v=sql.120).aspx](https://msdn.microsoft.com/en-AU/library/ms143760(v=sql.120).aspx "https://msdn.microsoft.com/en-AU/library/ms143760(v=sql.120).aspx")

Little bit confusing but I read "Limited to lesser of 1 Socket or 4 cores" to mean that SQL Server Express is limited to a single socket and no more than 4 cores.

This customer had configured their VMware guest as a single socket per core based on this advice:

[https://blogs.vmware.com/vsphere/2013/10/does-corespersocket-affect-performance.html](https://blogs.vmware.com/vsphere/2013/10/does-corespersocket-affect-performance.html "https://blogs.vmware.com/vsphere/2013/10/does-corespersocket-affect-performance.html")

Not usually a bad idea except when licensing restrictions require a different configuration.

## Solution
After a scheduled outage and some reconfiguration, our SQL Server Express instances have access to the expected number of cores:

~~~~
SQL Server detected 2 sockets with 4 cores per socket and 4 logical processors per socket, 8 total logical processors; using 4 logical processors based on SQL Server licensing. This is an informational message; no user action is required.
~~~~

Customer reports presence is being updated normally and SQL errors appear to have disappeared.