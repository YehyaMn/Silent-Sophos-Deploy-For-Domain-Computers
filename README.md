# Silent-Sophos-Deploy-For-Domain-Computers
Automated deployment of Sophos Endpoint across targeted domain computers using a GPO startup script. Includes step-by-step instructions, startup script, and verification commands for Windows AD environments.

GPO Sophos Deployment

Automated deployment of Sophos Endpoint across targeted domain computers using a GPO startup script. Includes step-by-step instructions, the startup script, and verification commands for Windows AD environments.

Table of Contents

Prerequisites

Step 1 — Install AD and GPMC Tools

Step 2 — Create GPO for Sophos Deployment

Step 3 — Startup Script

Step 4 — Configure Security Filtering

Step 5 — Deployment

Step 6 — Verification

License


# Prerequisites:
Windows AD Domain.
Put the Sophos installer in a Shared folder for installer and logs to check who the code ran on (optional):
```
\\Server\IT\SophosSetup.exe 
\\Server\IT\Logs
```

Installer must be unblocked:
![WhatsApp Image 2025-10-03 at 3 35 57 PM](https://github.com/user-attachments/assets/51eed5ff-01b3-44d0-89a8-8c606476f9b7)

```
Unblock-File "\\Server\IT\SophosSetup.exe"
```
Or from properties and put checkbox on the Unblock

Target computers must have Read access to the installer and Write access to the logs folder.

# Step 1 — Install AD and GPMC Tools (Windows 11)
 Install Active Directory and Group Policy Management tools
```powershell
Add-WindowsCapability -Online -Name "RSAT:ActiveDirectory.DS-LDS.Tools ~~~0.0.1.0"
Add-WindowsCapability -Online -Name "RSAT:GroupPolicy.Management.Tools~~~~0.0.1.0"
```
 Or launch GUI
``gpmc.msc``

# Step 2 — Create GPO for Sophos Deployment
``
Open GPMC → Group Policy Objects → New → Install_Sophos_Target.
``
Enforce the GPO
![WhatsApp Image 2025-10-03 at 3 18 32 PM](https://github.com/user-attachments/assets/ee29ca21-ba51-4436-8dad-2710139b5009)

Edit → Computer Configuration → Windows Settings → Scripts (Startup) → Add install_sophos.bat.

# Step 3 — Startup Script (install_sophos.bat)
```
@echo off
REM === Check if Sophos already installed ===
IF EXIST "C:\Program Files\Sophos\Sophos Endpoint Agent" exit /b 0

REM === Run the Sophos installer silently and log output ===
\\Server\IT\SophosSetup.exe --quiet >> "\\Server\IT\Logs\%COMPUTERNAME%_sophos.log" 2>&1
```

Notes:

Runs silently with ``--quiet.``

Prevents re-install if Sophos is already installed.

Creates a log per computer in ``\\Server\IT\Logs.``

# Step 4 — Configure Security Filtering
![WhatsApp Image 2025-10-03 at 3 18 33 PM](https://github.com/user-attachments/assets/cf196fd9-966d-4dee-9ec6-c53473a269ba)

Remove Authenticated Users.
Add ``Sophos_Deployment_Computers`` (AD Security group of target PCs).
![WhatsApp Image 2025-10-03 at 3 18 34 PM](https://github.com/user-attachments/assets/a5d20b86-88c5-45fa-a5ca-ab92bbec360f)

Ensure the group has Read & Apply group policy in Delegation.

# Step 5 — Deployment

PCs in the target group will install Sophos automatically on next reboot.

No admin password or user interaction required.

Logs are written per PC:

``\\Server\IT\Logs\<PCNAME>_sophos.log``

# Step 6 — Verification

On each PC, verify installation and GPO application:

# Verify folder exists
``Test-Path "C:\Program Files\Sophos\Sophos Endpoint Agent"``

# Verify Sophos services are running
``Get-Service *sophos*``
![WhatsApp Image 2025-10-03 at 3 18 35 PM](https://github.com/user-attachments/assets/0c75ed3c-0aaf-47ef-ac5e-1a43bb3102fc)

Or just check the Program files\Sophos

Or Check task manager
![WhatsApp Image 2025-10-03 at 3 33 37 PM (1)](https://github.com/user-attachments/assets/2860ae95-f1b2-4a57-ab9f-db96006a5cf7)

Or use Procmon if SophosSetup is working as shown below
![WhatsApp Image 2025-10-03 at 3 33 36 PM](https://github.com/user-attachments/assets/81ba6d01-ab4e-4349-8d02-df393a2a93bd)

# Confirm GPO applied
``gpresult /r /scope:computer``

# License
MIT License – feel free to use and adapt for your organization.
