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

Prerequisites

Windows AD Domain.

Shared folder for installer and logs:

\\Server\IT\SophosSetup.exe 
\\Server\IT\Logs


Installer must be unblocked:

Unblock-File "\\Base\IT\SophosSetup.exe"


Target computers must have Read access to the installer and Write access to the logs folder.

Step 1 — Install AD and GPMC Tools (Windows 11)
# Install Active Directory and Group Policy Management tools
Add-WindowsCapability -Online -Name "RSAT:ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0"
Add-WindowsCapability -Online -Name "RSAT:GroupPolicy.Management.Tools~~~~0.0.1.0"

# Or launch GUI
gpmc.msc

Step 2 — Create GPO for Sophos Deployment

Open GPMC → Group Policy Objects → New → Install_Sophos_Target.

Edit → Computer Configuration → Windows Settings → Scripts (Startup) → Add install_sophos.bat.

Step 3 — Startup Script (install_sophos.bat)
@echo off
REM === Check if Sophos already installed ===
IF EXIST "C:\Program Files\Sophos\Sophos Endpoint Agent" exit /b 0

REM === Run the Sophos installer silently and log output ===
\\Base\IT\SophosSetup.exe --quiet >> "\\Base\IT\Logs\%COMPUTERNAME%_sophos.log" 2>&1


Notes:

Runs silently with --quiet.

Prevents re-install if Sophos is already installed.

Creates a log per computer in \\Base\IT\Logs.

Step 4 — Configure Security Filtering

Remove Authenticated Users.

Add Sophos_Deployment_Computers (AD group of target PCs).

Ensure the group has Read & Apply group policy in Delegation.

Step 5 — Deployment

PCs in the target group will install Sophos automatically on next reboot.

No admin password or user interaction required.

Logs are written per PC:

\\Base\IT\Logs\<PCNAME>_sophos.log

Step 6 — Verification

On each PC, verify installation and GPO application:

# Verify folder exists
Test-Path "C:\Program Files\Sophos\Sophos Endpoint Agent"

# Verify Sophos services are running
Get-Service *sophos*

# Confirm GPO applied
gpresult /r /scope:computer

License

MIT License – feel free to use and adapt for your organization.
