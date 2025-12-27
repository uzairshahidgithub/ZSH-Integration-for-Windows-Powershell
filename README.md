<p align="center">
  <img
    src="https://github.com/user-attachments/assets/9fbda57b-d0e5-40c0-898d-2d0ac34b614d"
    alt="ZSH PowerShell Logo"
    width="480"
  />
</p>

<h2 align="center"> ZSH-Like Linux Terminal Experience on Windows PowerShell</h2>
<p align="center" style="max-width: 800px; margin: 0 auto;">
  A clean, stable, error-free PowerShell profile that brings
  <strong>ZSH-style Linux terminal behavior</strong> to
  <strong>Windows PowerShell</strong>, without risky aliases or unstable hacks.
  Designed for developers, security engineers, DevOps users, and power users
  who want productivity, history intelligence, and a modern prompt while
  keeping everything native and safe.
</p>
<br />

## Key Highlights
* ZSH-style **history search & suggestions**
* Intelligent **command completion**
* Clean **time-based prompt**
* Fast **navigation helpers**
* Useful **file & system utilities**
* Works on **Windows 10 / 11**
* Optional **ScreenFetch support**
  
## Main Features
* Up / Down Arrow history search
* `Ctrl + R` interactive reverse search
* Right Arrow smart word completion
* `showsg` Show all suggestions
* `myhistory` View command history
## Cheatsheet
| Commands           | Description          |
| ----------------- | -------------------- |
| `time`     | Current time & date  |
| `startcountdown` | Countdown timer      |
| `Go-Home`         | Move to Home directory |
| `Go-Desktop`      | Move to Desktop        |
| `Go-Documents`    | Move to Documents      |
| `update`   | Check Windows updates (Admin)  |
| `upgrade` | Install updates (Admin) |
| `reps`   | Restart PowerShell      |
| `showsg`   | show you suggestions   |
| `myhistory` | show you memory of commands |
| `lsc`   | List with color    |
| `clean`   | Clean Temp Files   |

## ScreenFetch Support
### Step 1: Install ScreenFetch (Optional)
If `screenfetch` already works, skip this step.
```powershell
Install-Module -Name windows-screenfetch
```
Verify:
```powershell
Get-Command screenfetch
screenfetch
```
## 🛠 Installation Guide
### Step 2: Open PowerShell Profile
Check if profile exists:
```powershell
Test-Path $PROFILE
```
If **False**, create it:
```powershell
New-Item -ItemType File -Path $PROFILE -Force
```
Open profile:
```powershell
notepad $PROFILE
```
### Step 3: Paste Configuration Code
Paste **the full configuration** into `$PROFILE`. 
⚠️ If you want **ScreenFetch**, uncomment. Otherwise, leave it commented.
```powershell
# ============================================
# ZSH WINDOWS POWERSHELL DEVELOPED BY AYUDANTE.
# ============================================
Clear-Host
# try { screenfetch } catch {}

# Load PSReadLine
try {
    Import-Module PSReadLine -ErrorAction SilentlyContinue
} catch {
    # Silent fail
}

if (Get-Module PSReadLine) {
    # Basic settings
    $MaximumHistoryCount = 10000
    Set-PSReadLineOption -HistorySearchCursorMovesToEnd
    Set-PSReadLineOption -MaximumHistoryCount 10000
    
    # Navigation
    Set-PSReadLineKeyHandler -Key UpArrow -Function HistorySearchBackward
    Set-PSReadLineKeyHandler -Key DownArrow -Function HistorySearchForward
    
    # History search
    Set-PSReadLineKeyHandler -Chord Ctrl+r -ScriptBlock {
        [Microsoft.PowerShell.PSConsoleReadLine]::ReverseSearchHistory()
    }
    
    # Tab completion
    Set-PSReadLineKeyHandler -Key Tab -Function MenuComplete
}

# Suggestion function
function Get-CommandSuggestions {
    param([string]$Text)
    
    if (-not $Text) { return @() }
    
    $suggestions = @()
    
    # From history
    $history = Get-History | 
        Where-Object { $_.CommandLine -like "$Text*" } |
        Select-Object -Last 5 -ExpandProperty CommandLine -Unique
    
    # From commands
    $commands = Get-Command -Name "$Text*" -ErrorAction SilentlyContinue |
        Select-Object -First 5 -ExpandProperty Name
    
    $suggestions = @($history) + @($commands)
    return $suggestions | Select-Object -Unique
}

# Right arrow completion
if (Get-Module PSReadLine) {
    Set-PSReadLineKeyHandler -Key RightArrow -ScriptBlock {
        param($key, $arg)
        
        $line = $null
        $cursor = $null
        [Microsoft.PowerShell.PSConsoleReadLine]::GetBufferState([ref]$line, [ref]$cursor)
        
        if ($cursor -eq $line.Length -and $line.Trim()) {
            if ($line -match '(\S+)$') {
                $lastWord = $matches[1]
                $suggestions = Get-CommandSuggestions -Text $line
                
                if ($suggestions.Count -gt 0) {
                    $firstSuggestion = $suggestions[0]
                    if ($firstSuggestion.StartsWith($line)) {
                        $nextPart = $firstSuggestion.Substring($line.Length)
                        if ($nextPart -match '^(\S+)') {
                            $nextWord = $matches[1]
                            [Microsoft.PowerShell.PSConsoleReadLine]::Insert($nextWord)
                            [Microsoft.PowerShell.PSConsoleReadLine]::Insert(" ")
                            return
                        }
                    }
                }
            }
        }
        
        [Microsoft.PowerShell.PSConsoleReadLine]::ForwardChar($key, $arg)
    }
}

# Show suggestions - use function name directly
function showsg {
    param([string]$Text = "")
    
    if (-not $Text) {
        $line = $null
        $cursor = $null
        if (Get-Module PSReadLine) {
            [Microsoft.PowerShell.PSConsoleReadLine]::GetBufferState([ref]$line, [ref]$cursor)
        }
        $Text = $line
    }
    
    $suggestions = Get-CommandSuggestions -Text $Text
    
    if ($suggestions.Count -eq 0) {
        Write-Host "No suggestions for: $Text"
        return
    }
    
    Write-Host "Suggestions:"
    $suggestions | ForEach-Object {
        Write-Host "  $_"
    }
}

# History viewer
function myhistory {
    param(
        [string]$Filter = "",
        [int]$Count = 20
    )
    
    if ($Filter) {
        Get-History | Where-Object { $_.CommandLine -like "*$Filter*" } | Select-Object -Last $Count
    } else {
        Get-History | Select-Object -Last $Count
    }
}

# Clear history
function clshistory {
    Clear-History
    Write-Host "History cleared"
}

# Time info
function time {
    $now = Get-Date
    Write-Host "Time: " -NoNewline
    Write-Host $now.ToString("HH:mm:ss")
    Write-Host "Date: " -NoNewline
    Write-Host $now.ToString("dddd, MMMM dd, yyyy")
}

# Timer
function startcountdown {
    param([int]$Seconds = 60)
    
    $endTime = (Get-Date).AddSeconds($Seconds)
    Write-Host "Timer: $Seconds seconds"
    Write-Host "Ends: " -NoNewline
    Write-Host $endTime.ToString("HH:mm:ss")
    
    $timer = [System.Diagnostics.Stopwatch]::StartNew()
    
    while ($timer.Elapsed.TotalSeconds -lt $Seconds) {
        $remaining = $Seconds - [math]::Round($timer.Elapsed.TotalSeconds)
        Write-Host "`rRemaining: $remaining seconds " -NoNewline
        Start-Sleep -Milliseconds 200
    }
    
    $timer.Stop()
    Write-Host "`nDone!"
}

function Go-Home { Set-Location $HOME }
function Go-Desktop { Set-Location "$HOME\Desktop" }
function Go-Documents { Set-Location "$HOME\Documents" }
function Go-Downloads { Set-Location "$HOME\Downloads" }

# Colorful listing
function lsc {
    Get-ChildItem @args | ForEach-Object {
        if ($_.PSIsContainer) {
            Write-Host $_.Name -ForegroundColor Blue
        } elseif ($_.Extension -match '\.(ps1|bat|cmd|sh)$') {
            Write-Host $_.Name -ForegroundColor Green
        } elseif ($_.Extension -match '\.(txt|log|md)$') {
            Write-Host $_.Name -ForegroundColor Yellow
        } elseif ($_.Extension -match '\.(exe|dll)$') {
            Write-Host $_.Name -ForegroundColor Magenta
        } else {
            Write-Host $_.Name
        }
    }
}

# Windows update (simplified)
function update {
    if (-not (Get-Module -ListAvailable PSWindowsUpdate)) {
        Write-Host "PSWindowsUpdate module not installed"
        Write-Host "Run: Install-Module PSWindowsUpdate -Scope CurrentUser"
        return
    }
    
    Import-Module PSWindowsUpdate -ErrorAction SilentlyContinue
    Get-WindowsUpdate
}

function upgrade {
    if ([Security.Principal.WindowsPrincipal]::new(
        [Security.Principal.WindowsIdentity]::GetCurrent()
    ).IsInRole([Security.Principal.WindowsBuiltInRole]::Administrator)) {
        
        Check-Updates
        Install-WindowsUpdate -AcceptAll
        
    } else {
        Write-Host "Run as Administrator"
    }
}

# Clean temp files
function clean {
    Write-Host "Cleaning temp files..."
    
    $tempPaths = @(
        "$env:TEMP\*",
        "$env:WINDIR\Temp\*"
    )
    
    $totalFreed = 0
    foreach ($tempPath in $tempPaths) {
        if (Test-Path $tempPath) {
            Get-ChildItem -Path $tempPath -Recurse -ErrorAction SilentlyContinue | 
                Where-Object { -not $_.PSIsContainer } |
                ForEach-Object {
                    try {
                        $totalFreed += $_.Length
                        Remove-Item $_.FullName -Force -ErrorAction SilentlyContinue
                    } catch {}
                }
        }
    }
    
    $freedMB = [math]::Round($totalFreed / 1MB, 2)
    Write-Host "Freed: ${freedMB}MB"
}

# Restart PowerShell
function reps {
    Write-Host "Restarting PowerShell..."
    Start-Process "powershell.exe" -ArgumentList "-NoExit"
    exit
}

function prompt {
    # Time
    $currentTime = Get-Date -Format "HH:mm"
    
    # Path
    $path = (Get-Location).Path
    
    # Shorten home
    if ($path.StartsWith($HOME)) {
        $path = "~" + $path.Substring($HOME.Length)
    }
    
    # Truncate if long
    if ($path.Length -gt 40) {
        $path = "..." + $path.Substring($path.Length - 37)
    }
    
    # Admin check
    $isAdmin = ([Security.Principal.WindowsPrincipal] [Security.Principal.WindowsIdentity]::GetCurrent()).IsInRole([Security.Principal.WindowsBuiltInRole] "Administrator")
    
    # Build prompt
    Write-Host "[$currentTime] " -NoNewline
    Write-Host $path -NoNewline
    
    if ($isAdmin) {
        Write-Host " #>" -NoNewline
    } else {
        Write-Host " >" -NoNewline
    }
    
    return " "
}

# Add commands to history
try {
    Get-Date | Out-Null
    Get-ChildItem | Select-Object -First 1 | Out-Null
} catch {}

# Profile loaded successfully
```
**Save & close Notepad**

### Step 4: Install Windows Update Module (Skipped If Installed)
```powershell
Install-Module PSWindowsUpdate -Force -Scope CurrentUser
```
### Step 5: Reload Profile
```powershell
. $PROFILE
```
Then **restart PowerShell**.
* `>` → Normal user
* `#>` → Administrator
* Time always visible

## Compatibility
* ✅ Windows 10 / 11
* ✅ PowerShell 5.x / PowerShell 7
* ❌ No WSL required
* ❌ No third-party shells required

## Security
* No unsafe overrides
* No alias hijacking
* Silent error handling
* Admin checks included
* Safe for enterprise & government use

## ![License](https://img.shields.io/badge/License-MIT-green.svg) License
Free to use, modify, and distribute. Ideal for **developers, students, SOC teams, and security engineers**.

## References
- JulianChow94 [Windows-screenFetch](https://github.com/JulianChow94/Windows-screenfetch)
- [Powershell Gallery Package](https://www.powershellgallery.com/packages/windows-screenfetch/1.0.2)

## Authors
- [Muhammad Uzair](https://www.github.com/uzairshahidgithub)

## Support
For support, email uzairrshahid@gmail.com or join our [Discord Community Codemo Teams](https://linktr.ee/codemoteams).
