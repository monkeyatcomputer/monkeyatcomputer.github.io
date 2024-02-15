---
permalink: /surface-hub-2s-with-teams-rooms-on-windows
title: "Transforming Surface Hub 2S into the Microsoft Teams Rooms on Windows experience"
description: "A blog post about installing the Microsoft Teams Rooms on Windows experience on a Surface Hub 2S"
image:
    path: "/assets/img/surface-hub.jpg"
    alt: "Surface Hub"
categories: [Microsoft Teams]
tags: [microsoft teams, microsoft teams rooms, mtr, surface hub]
---

## The Microsoft Teams Rooms on Windows Experience

Back in March 2023, Microsoft announced some updates to the Surface Hub line with: the new Surface Hub 3, a Surface Hub 2S upgrade pack, and a software-only upgrade for the Surface Hub 2S. The new device and upgrades are based on the same Microsoft Teams Rooms on Windows experience we're all too familiar with.

Microsoft have delivered on the new Surface Hub 3 and the Surface Hub 2S upgrade pack, but we don't yet have a date for when we'll be able to convert our Surface Hub 2S's without the upgrade pack.

What follows is how I've managed to get our Surface Hub 2S running the Microsoft Teams Rooms on Windows experience. It goes without saying that there are some risks with this unsupported approach, so before we begin, a disclaimer...

> Please be advised that applying the Microsoft Teams Rooms on Windows experience to a Surface Hub 2S device through methods not officially supported by Microsoft involves significant risks and potential drawbacks. Users who choose to pursue these methods do so at their own risk, understanding that:
> **Windows Activation**: You will need to supply your own Windows 11 Enterprise key the same as if you were to convert a Surface Hub to Windows 10/11 Pro or Enterprise. Surface Hub 2S has the Windows 10 Team key baked in to the UEFI BIOS but not Windows 11 IoT Enterprise.
> 
> **Voided Warranty**: Implementing an unsupported configuration may void any remaining warranty on the Surface Hub 2S. Microsoft may not cover damages or issues arising from the use of unsupported software or hardware modifications.
> 
> **Software Incompatibility**: The Surface Hub 2S is designed to operate with a specific set of software and hardware configurations. Using unsupported methods to install or run Microsoft Teams Rooms on Windows may lead to compatibility issues, software instability, or the inability to access certain features and updates.
> 
> **Security Vulnerabilities**: Unsupported modifications can expose the device to increased security risks, including vulnerabilities that may not be addressed by standard Microsoft security updates. This could potentially compromise the security of your device and the data it contains.
> 
> **Performance Issues**: The Surface Hub 2S hardware and software are optimized for its intended use. Running unsupported software may result in suboptimal performance, including slower response times, reduced reliability, and a degraded user experience.
> 
> **Lack of Support**: Microsoft and its affiliates will not provide technical support for issues that arise from the use of unsupported methods. Users will be solely responsible for troubleshooting and resolving any problems that may occur.
> 
> **Compliance Issues**: Depending on your organization's policies and regulatory requirements, using an unsupported configuration may breach compliance obligations, leading to potential legal and financial repercussions.
> 
> Before proceeding with any unsupported method to apply the Microsoft Teams Rooms on Windows experience to a Surface Hub 2S, carefully consider the potential impacts and ensure you have the necessary skills and resources to manage the risks involved. It is highly recommended to consult with IT professionals and consider alternatives that are supported by Microsoft to achieve your objectives while maintaining device integrity and security.
{: .prompt-danger }

... with that out of the way, the high-level steps required are:

 * Prepare a signing certificate required for the UEFI configuration package.
 * Create UEFI configuration USB using the Microsoft Surface UEFI Configurator.
 * Apply UEFI configuration to the Surface Hub 2S allowing boot to Windows 10/11 Pro/Enterprise.
 * Recovery using Windows 11 IoT Enterprise with the Microsoft Teams Rooms on Windows experience recovery files.
  
The instructions that follow have been reproduced and adapted from the Microsoft Learn article [Migrate to Windows 10/11 Pro or Enterprise on Surface Hub](https://learn.microsoft.com/en-us/surface-hub/surface-hub-2s-migrate-os).

## Pre-requisites

 * Surface Hub 2S [serial number](https://support.microsoft.com/help/4036293/surface-find-the-serial-number-on-surface) to download correct recovery image.
 * USB drive formatted as FAT32 for the UEFI configuration boot disk.
 * 32GB USB drive formatted as FAT32 for the recovery files.
 * Surface Hub 2S with [UEFI version *694.2938.768.0* or later](https://learn.microsoft.com/en-us/surface-hub/surface-hub-2s-migrate-os#verify-the-uefi-version-on-surface-hub-2s).  
![Your Surface](/uploads/2024-02-13-surface-hub-2s-with-teams-rooms-on-windows/shm-fig1.png){: width="250" }
* Download and install latest *SurfaceUEFI_Configurator_vx.xxx.xxx.x_x64.msi* from the [Surface Tools for IT page](https://www.microsoft.com/download/details.aspx?id=46703).

> I was unable to install the Surface Hub UEFI Configurator on a Windows 11 Enterprise PC with the installer detecting the CPU as ARM64 instead of x64. Installing on another PC with Windows 10 was successful.
{: .prompt-tip }

* Download *Surface Hub 2S Demo - Windows 11 IoT Enterprise with the Microsoft Teams Rooms on Windows experience* from [Surface Recovery Image Download](https://support.microsoft.com/en-us/surface-recovery-image) and extract the ZIP archive to a 32GB FAT32 formatted USB.  
![Select your Surface](/uploads/2024-02-13-surface-hub-2s-with-teams-rooms-on-windows/shm-fig2.png){: width="250" }
![Recovery Image Download](/uploads/2024-02-13-surface-hub-2s-with-teams-rooms-on-windows/shm-fig3.png){: width="250" }

> The dowload is 22GB and appears to include both Windows 10 Team and Windows 11 IoT Enterprise editions. Depending on the Surface Hub 2S's UEFI configuration, the restore will either write Windows 10 Team or Windows 11 IoT Enterprise to the Surface Hub internal disk.
{: .prompt-info }

## Prepare the SEMM certificate

You must prepare a certificate if this is the first time you have used Surface UEFI Configurator. This certificate ensures that after a device is enrolled in SEMM, you can modify UEFI settings only by using packages created with the approved certificate.

I just used a self-signed certificate as it we were not intending to do mass configuration of Surface devices. Run the following PowerShell commands to create a PFX in the current folder.

> This script creates a certificate with a password of 12345678. The certificate generated by this script isn't recommended for production environments.
{: .prompt-info }

```powershell
$pw = ConvertTo-SecureString "12345678" -AsPlainText -Force

$TestUefiV2 = New-SelfSignedCertificate `
  -Subject "CN=Surface Hub UEFI, O=Generation-e, C=AU" `
  -Type SSLServerAuthentication `
  -HashAlgorithm sha256 `
  -KeyAlgorithm RSA `
  -KeyLength 2048 `
  -KeyUsage KeyEncipherment `
  -KeyUsageProperty All `
  -Provider "Microsoft Enhanced RSA and AES Cryptographic Provider" `
  -NotAfter (Get-Date).AddYears(25) `
  -TextExtension @("2.5.29.37={text}1.2.840.113549.1.1.1") `
  -KeyExportPolicy Exportable
  
$TestUefiV2 | Export-PfxCertificate -Password $pw -FilePath ".\SurfaceHubUefi.pfx"
```
> For use with SEMM and Microsoft Surface UEFI Configurator, the certificate must be exported with the private key and with password protection. Microsoft Surface UEFI Configurator prompts you to select the SEMM certificate file (.pfx) and certificate password.
{: .prompt-tip }

> Keep the certificate and password somewhere safe!
{: .prompt-danger }

## Create UEFI Configuration USB

Open Microsoft Surface UEFI Configurator and select the following options:

 * **Surface Devices**  
![Surface Devices](/uploads/2024-02-13-surface-hub-2s-with-teams-rooms-on-windows/shm-fig4.png)
 * **Configuration Package**  
![Configuration Package](/uploads/2024-02-13-surface-hub-2s-with-teams-rooms-on-windows/shm-fig5.png)
 * **DFI File**  
![DFI File](/uploads/2024-02-13-surface-hub-2s-with-teams-rooms-on-windows/shm-fig6.png)
 * **Certificate Protection** - select the PFX file created earlier when prompted  
![Certificate Protection](/uploads/2024-02-13-surface-hub-2s-with-teams-rooms-on-windows/shm-fig7.png)
![Select Certificate](/uploads/2024-02-13-surface-hub-2s-with-teams-rooms-on-windows/shm-fig8.png)
 * Enter your **certificate password**  
![Certificate Password](/uploads/2024-02-13-surface-hub-2s-with-teams-rooms-on-windows/shm-fig9.png)
 * Skip password protection  
 * On the Surface Type selection page, only turn on **Surface Hub 2S**  
![Surface Type](/uploads/2024-02-13-surface-hub-2s-with-teams-rooms-on-windows/shm-fig10.png)
 * Turn on **EnableOsMigration**  
![EnableOsMigration](/uploads/2024-02-13-surface-hub-2s-with-teams-rooms-on-windows/shm-fig12.png)
 * Select/check your USB drive, and select **Build**  
![Build](/uploads/2024-02-13-surface-hub-2s-with-teams-rooms-on-windows/shm-fig14.png)
 * Make a copy or screen shot of the last page as we need the two characters highlighted when importing to the Surface Hub  
![Done](/uploads/2024-02-13-surface-hub-2s-with-teams-rooms-on-windows/shm-fig15.png)

## Apply UEFI Configuration

 * With the Surface Hub 2S powered off, and the UEFI Configuration USB plugged in to the USB-C or USB-A ports on the rear, press and hold **Volume +**, and then press and release the **Power** button. Keep holding **Volume +** until the UEFI menu appears  
![UEFI](/uploads/2024-02-13-surface-hub-2s-with-teams-rooms-on-windows/shm-fig16.png)
 * From the UEFI menu, select **Management**, and then select **Install from USB**  
![Management](/uploads/2024-02-13-surface-hub-2s-with-teams-rooms-on-windows/shm-fig17.png)
* Select **Restart now**  
![Restart](/uploads/2024-02-13-surface-hub-2s-with-teams-rooms-on-windows/shm-fig18.png)
* Press and release the **Power** button. In the red dialog box, choose to activate Surface Enterprise Management Mode. Enter your two-character certificate thumbprint and select **OK**. The Surface Hub will reboot again.  
![Activate](/uploads/2024-02-13-surface-hub-2s-with-teams-rooms-on-windows/shm-fig19.png)

## Install Microsoft Teams Room on Windows Experience

* With the Surface Hub 2S powered off, and the recovery USB plugged in to the USB-C or USB-A ports on the rear, press and hold **Volume -**, and then press and release the **Power** button. Keep holding **Volume -** until the spinner below the Windows logo appears.  
![USB Boot](/uploads/2024-02-13-surface-hub-2s-with-teams-rooms-on-windows/shm-fig20.png)
* On the language selection screen, select the display language for your Surface Hub.
* Select **Recover from a drive** and **Just remove my files**, and then select **Recover**. Surface Hub reboots several times and can take an hour or more to complete the recovery process.

When the Microsoft Teams Room OOBE appears, go ahead and set up the MTR as you normally would.

![USB Boot](/uploads/2024-02-13-surface-hub-2s-with-teams-rooms-on-windows/shm-fig21.jpg){: width="250" }

> Windows will need activation! Presumably the official process coming some time this year will include activation for Windows 11 IoT Enterprise.
{: .prompt-warning }

## Final Thoughts

As tempting as it might be to jump on some quick-fix hacks to get the Teams Rooms experience up and running, it's a bit like skating on thin ice. Going down the unofficial route is risky business — it's not just about the potential for a few glitches here and there; it's about the bigger picture of keeping your device and network safe and sound.

In wrapping up, the journey to amp up your Surface Hub with that sleek Microsoft Teams Rooms experience is all about striking the right balance. It's about aiming for those tech goals with an eye on the safe route, sticking to the playbook, and avoiding the tech equivalent of "shortcuts through the sketchy part of town." After all, the goal is to enhance your collaborative space, not to turn it into a cautionary tale. So, here's to making those meeting rooms shine—safely and smartly!

## P.S.

If you need to go back to Windows 10 Team, first [Unenroll Surface devices from SEMM
](https://learn.microsoft.com/en-us/surface/unenroll-surface-devices-from-semm), and follow [Reset & recovery for Surface Hub 2S & Surface Hub 3](https://learn.microsoft.com/en-us/surface-hub/surface-hub-recover-reset).

