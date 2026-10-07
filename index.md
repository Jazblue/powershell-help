# PowerShell Help Guide

## Essential PowerShell Commands for Windows Administration

A quick reference for common PowerShell tasks, especially useful for system administration, troubleshooting, and automation.

---

## **Getting Started**

- **Launch PowerShell as Admin**: Right-click Start → Windows PowerShell (Admin) or Windows Terminal (Admin)
- **Check version**: `$PSVersionTable.PSVersion`
- **Update Help**: `Update-Help` (run as admin, requires internet)
- **Get help for a cmdlet**: `Get-Help Get-Process -Full` or `Get-Process -?`

---

## **Basic Navigation & Information**

| Cmdlet | Description | Example |
|--------|-------------|---------|
| `Get-Command` | List all available cmdlets | `Get-Command *-Service*` |
| `Get-Member` | See properties and methods of an object | `Get-Process | Get-Member` |
| `Get-Process` | List running processes | `Get-Process | Sort-Object CPU -Descending \| Select-Object -First 5` |
| `Get-Service` | List services and their status | `Get-Service \| Where-Object {$_.Status -eq 'Running'}` |
| `Get-EventLog` | Read event logs | `Get-EventLog -LogName Application -Newest 20` |
| `Get-WmiObject` / `Get-CimInstance` | Access WMI/CIM information | `Get-CimInstance -ClassName Win32_OperatingSystem` |
| `Get-HotFix` | List installed Windows updates | `Get-HotFix \| Sort-Object InstalledOn` |
| `Get-Location` / `Set-Location` | Get/set current directory (like pwd/cd) | `Set-Location C:\Scripts` |
| `Get-ChildItem` | List files and directories (like ls/dir) | `Get-ChildItem -Path C:\ -Filter *.log -Recurse` |

---

## **File & Directory Operations**

| Cmdlet | Description | Example |
|--------|-------------|---------|
| `New-Item` | Create a new file or directory | `New-Item -Path "C:\Test" -ItemType Directory -Force` |
| `Copy-Item` | Copy files or directories | `Copy-Item -Path "C:\File.txt" -Destination "D:\Backup\" -Recurse` |
| `Move-Item` | Move or rename files/directories | `Move-Item -Path "C:\OldName.txt" -Destination "D:\NewName.txt"` |
| `Remove-Item` | Delete files or directories | `Remove-Item -Path "C:\Temp\*" -Recurse -Force` |
| `Rename-Item` | Rename a file or directory | `Rename-Item -Path "C:\Old.txt" -NewName "New.txt"` |
| `Set-Content` / `Add-Content` | Write or append to a file | `Set-Content -Path "C:\log.txt" -Value "Start of log"` |
| `Get-Content` | Read a file | `Get-Content -Path "C:\log.txt" -Tail 10` |
| `Test-Path` | Check if a path exists | `if (Test-Path "C:\File.txt") { "File exists" }` |

---

## **Working with Objects & Pipeline**

PowerShell works with objects, not just text.

- **Select specific properties**: `Get-Process \| Select-Object Name, ID, CPU, WorkingSet`
- **Filter objects**: `Get-Service \| Where-Object {$_.Status -eq 'Running' -and $_.StartType -eq 'Automatic'}`
- **Sort objects**: `Get-Process \| Sort-Object CPU -Descending`
- **Group objects**: `Get-Process \| Group-Object -Property Company`
- **Measure objects**: `Get-Process \| Measure-Object -Property CPU -Average -Sum -Maximum`
- **Export to CSV/JSON/XML**: `Get-Process \| Export-Csv -Path "processes.csv" -NoTypeInformation`
- **Import from CSV**: `$procs = Import-Csv -Path "processes.csv"`
- **ForEach-Object / %**: Apply script block to each object | `Get-Service \| ForEach-Object { $_.DisplayName }`
- **Where-Object / ?**: Filter objects | `Get-Process \| Where-Object {$_.CPU -gt 100}`

---

## **Common Administration Tasks**

### **Managing Services**
```powershell
# Start a service
Start-Service -Name "wuauserv"

# Stop a service
Stop-Service -Name "wuauserv" -Force

# Restart a service
Restart-Service -Name "Spooler"

# Set startup type
Set-Service -Name "Telephony" -StartupType Automatic

# Check if a service exists
Get-Service -Name "Dnscache" -ErrorAction SilentlyContinue
```

### **Managing Features & Roles (Windows Server)**
```powershell
# List available features
Get-WindowsFeature

# Install a feature
Install-WindowsFeature -Name "Web-Server" -IncludeManagementTools

# Remove a feature
Remove-WindowsFeature -Name "Web-Server"

# Get installed features
Get-WindowsFeature | Where-Object {$_.InstallState -eq "Installed"}
```

### **Managing Local Users & Groups**
```powershell
# List local users
Get-LocalUser

# Create a local user
New-LocalUser -Name "HomeUser" -Password (Read-Host -AsSecureString "Enter password") -FullName "Home User" -Description "Standard home account"

# Disable a user
Disable-LocalUser -Name "Guest"

# Add user to a group
Add-LocalGroupMember -Group "Administrators" -Member "HomeUser"

# List group members
Get-LocalGroupMember -Group "Administrators"
```

### **Managing Scheduled Tasks**
```powershell
# List scheduled tasks
Get-ScheduledTask | Where-Object {$_.State -eq 'Ready'}

# Create a simple scheduled task
$action = New-ScheduledTaskAction -Execute "powershell.exe" -Argument "-Command \"Get-Process | Out-File C:\Temp\proc.txt\""
$trigger = New-ScheduledTaskTrigger -Daily -At 2am
Register-ScheduledTask -Action $action -Trigger $trigger -TaskName "DailyProcessLog" -Description "Log processes daily at 2am" -RunLevel Highest

# Remove a scheduled task
Unregister-ScheduledTask -TaskName "DailyProcessLog" -Confirm:$false
```

### **Working with the Registry**
```powershell
# Read a registry key
Get-ItemProperty -Path "HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion" -Name "ProductName"

# Set a registry value
Set-ItemProperty -Path "HKCU:\Console" -Name "FaceTransparency" -Value 100 -Type DWord

# Create a new registry key
New-Item -Path "HKCU:\Software\MyApp" -Force

# Delete a registry value
Remove-ItemProperty -Path "HKCU:\Console" -Name "FaceTransparency"
```

### **Network Diagnostics**
```powershell
# Test network connection to a host
Test-Connection -ComputerName "google.com" -Count 4

# Get detailed network adapter info
Get-NetIPConfiguration | Format-Table -AutoSize

# Reset TCP/IP stack
netsh int ip reset
netsh winsock reset

# Flush DNS cache
Clear-DnsClientCache

# Release/Renew IP address (if using DHCP)
ipconfig /release
ipconfig /renew
```

### **Windows Update via PowerShell**
```powershell
# Install PSWindowsUpdate module if needed (first time)
# Install-Module -Name PSWindowsUpdate -Scope CurrentUser -Repository PSGallery -Force

# Import module
Import-Module PSWindowsUpdate

# Check for updates
Get-WindowsUpdate

# Install all updates
Install-WindowsUpdate -AcceptAll -AutoReboot

# Install only critical updates
Install-WindowsUpdate -AcceptAll -AutoReboot -MicrosoftUpdate -IgnoreReboot

# View update history
Get-WUHistory | Sort-Object Date -Descending | Select-Object -First 20
```

---

## **Useful One-Liners for Troubleshooting**

```powershell
# Find processes using the most CPU
Get-Process | Sort-Object CPU -Descending | Select-Object -First 10

# Find services that are set to Automatic but are not running
Get-Service | Where-Object {$_.StartType -eq 'Automatic' -and $_.Status -ne 'Running'}

# List all listening TCP ports
Get-NetTCPConnection | Where-Object {$_.State -eq 'Listen'} | Select-Object LocalAddress,LocalPort,OwningProcess

# Find the process using a specific port (e.g., 8080)
Get-Process -Id (Get-NetTCPConnection -LocalPort 8080).OwningProcess

# Get BIOS/UEFI information
Get-CimInstance -ClassName Win32_BIOS

# Get motherboard information
Get-CimInstance -ClassName Win32_BaseBoard

# Get installed RAM
Get-CimInstance -ClassName Win32_PhysicalMemory | Measure-Object -Property Capacity -Sum

# Get disk usage for all drives
Get-PSDrive -PSProvider FileSystem | Select-Object Name,@{Name='Size(GB)';Expression={[math]::Round($_.Used/1GB,2)}},@{Name='Free(GB)';Expression={[math]::Round($_.Free/1GB,2)}},@{Name='PercentUsed';Expression={[math]::Round(($_.Used/$_.Maximum)*100,2)}}

# List all installed programs (like Add/Remove Programs)
Get-WmiObject -Class Win32_Product | Select-Object Name,Version,Vendor | Sort-Object Name

# Find large files (>100MB) in a directory
Get-ChildItem -Path "C:\" -Recurse -ErrorAction SilentlyContinue | Where-Object {$_.Length -gt 100MB} | Select-Object FullName,@{Name='SizeMB';Expression={[math]::Round($_.Length/1MB,2)}} | Sort-Object SizeMB -Descending

# Check if a specific hotfix/KB is installed
Get-HotFix -Id "KB5001330"

# Export all startup items (registry and startup folders)
Get-CimInstance -ClassName Win32_StartupCommand | Select-Object Name,Command,Location,User

# Get environment variables
Get-ChildItem Env: | Sort-Object Name

# Find duplicate files by hash (simple example)
Get-ChildItem -Path "D:\Docs" -Recurse -File -ErrorAction SilentlyContinue | 
  Get-FileHash | 
  Group-Object Hash | 
  Where-Object {$_.Count -gt 1} | 
  Select-Object -ExpandProperty Group | 
  Select-Object Hash,Path
```

---

## **PowerShell Profiles**

Customize your PowerShell session with a profile.

- **Check profile paths**: `$PROFILE | Format-List *`
- **Create profile if not exists**: 
  ```powershell
  if (!(Test-Path $PROFILE)) { New-Item -ItemType File -Path $PROFILE -Force }
  ```
- **Edit profile**: `notepad $PROFILE`
- **Common profile additions**:
  ```powershell
  # Set console colors
  $Host.UI.RawUI.BackgroundColor = "DarkBlue"
  $Host.UI.RawUI.ForegroundColor = "White"
  Clear-Host

  # Add useful aliases
  Set-Alias ll Get-ChildItem
  Set-Alias .. "Set-Location .."
  Set-Alias md mkdir
  Set-Alias rd Remove-Item
  Set-Alias h history
  Set-Alias eh Start-Process notepad.exe

  # Set window title
  $Host.UI.RawUI.WindowTitle = "PowerShell - $env:USERNAME@$env:COMPUTERNAME"

  # Load PSReadLine for better tab completion (if not already loaded)
  if (Get-Module -ListAvailable -Name PSReadLine) { Import-Module PSReadLine }
  ```

---

## **Execution Policies**

Check and set execution policy for running scripts.

```powershell
# See current policy
Get-ExecutionPolicy
Get-ExecutionPolicy -List  # Shows all scopes

# Set policy for current user (recommended for home use)
Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy RemoteSigned -Force

# Common policies:
# Restricted - No scripts can run (default)
# AllSigned - Only scripts signed by trusted publisher can run
# RemoteSigned - Downloaded scripts must be signed; local scripts can run unsigned
# Unrestricted - All scripts can run (risky)
# Bypass - Nothing is blocked; no warnings or prompts
```

> **Note:** For home use, `RemoteSigned` for `CurrentUser` is a good balance of security and usability.

---

## **Creating Reusable Scripts & Functions**

### **Simple Function Example**
```powershell
function Get-MyIP {
    # Get public IP address via external service
    try {
        $ip = Invoke-RestMethod -Uri 'https://api.ipify.org' -TimeoutSec 5
        return "Your public IP address is: $ip"
    } catch {
        return "Unable to determine public IP: $_"
    }
}

# Usage: Get-MyIP
```

### **Parameterized Function**
```powershell
function Get-ProcessByName {
    param(
        [Parameter(Mandatory=$true)]
        [string]$Name
    )
    
    Get-Process -Name $Name -ErrorAction SilentlyContinue
}

# Usage: Get-ProcessByName -Name "chrome"
```

### **Saving Functions to Profile**
Add functions to your `$PROFILE` so they're available in every session.

### **Script Blocks & Advanced Usage**
```powershell
# Script block stored in variable
$checkDisk = {
    param([string]$DriveLetter = "C:")
    Get-PSDrive -Name $DriveLetter | 
        Select-Object Name,@{Name='FreeGB';Expression={[math]::Round($_.Free/1GB,2)}}
}

# Invoke script block
& $checkDisk -DriveLetter "D:"
```

---

## **Common Gotchas & Tips**

1. **Aliases aren't saved**: Aliases set in a session are lost when you close PowerShell. Add them to your profile to make them permanent.
2. **Case insensitive**: PowerShell is case-insensitive for cmdlets and parameters (but not always for values).
3. **Pipeline output**: Most cmdlets write objects to the pipeline; you can capture them in variables or pipe to other cmdlets.
4. **Error handling**: Use `-ErrorAction SilentlyContinue` to suppress errors, or `try { } catch { }` for more control.
5. **Updating help**: Run `Update-Help` monthly or whenever you install new modules.
6. **Finding commands**: Use `Get-Command -Noun *Service*` to find all cmdlets with a specific noun.
7. **Exploring objects**: Pipe to `Get-Member | Where-Object {$_.MemberType -eq 'Property'}` to see only properties.
8. **Running .ps1 files**: By default, you can't double-click to run. Right-click → Run with PowerShell, or call from PowerShell: `& "C:\Scripts\MyScript.ps1"`.

---

## **Recommended Modules to Install**

```powershell
# Install from PowerShell Gallery (run once as admin or with -Scope CurrentUser)
Install-Module -Name PSReadLine -Scope CurrentUser -Force      # Better tab completion and history
Install-Module -Name Terminal-Icons -Scope CurrentUser -Force # Pretty icons in ls/output
Install-Module -Name oh-my-posh -Scope CurrentUser -Force     # Fancy prompts
Install-Module -Name PSWindowsUpdate -Scope CurrentUser -Force # Windows Update via PS
Install-Module -Name ActiveDirectory -Scope CurrentUser -Force # If you have AD (RSAT)
Install-Module -Name SqlServer -Scope CurrentUser -Force      # If you work with SQL Server
```

Then add to profile:
```powershell
Import-Module PSReadLine
Import-Module Terminal-Icons
Import-Module oh-my-posh
```

---

## **Resources for Further Learning**

- **Official Docs**: https://learn.microsoft.com/powershell/
- **PowerShell Gallery**: https://www.powershellgallery.com/ (find and install modules)
- **SS64.com PowerShell**: https://ss64.com/ps/ (command reference)
- **PowerShell.org**: https://powershell.org/ (community and articles)
- **YouTube**: PowerShell.tv, Learn PowerShell channels
- **Books**: 
  - *Learn PowerShell in a Month of Lunches* by Don Jones
  - *PowerShell in Depth* by Richard Siddaway, et al.
  - *The PowerShell Scripting and Toolmaking Book* by Don Jones & Jeff Hicks

---

*This guide covers PowerShell 5.1 (built into Windows 10/11) and PowerShell 7+ (cross-platform). Most commands work in both versions.*

*Last updated: October 2026*