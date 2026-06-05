*PowerShell_Forensic_Report.md:*
Real-World Problem Solving with PowerShell

**Student Name:** Kikie  
**Course:** Cybersecurity  
**Tutor:** Mr. Dennis  
**Date:** May 31, 2026

---

Table of Contents
1. [Abstract](#1-abstract)
2. [Introduction](#2-introduction-why-powershell-is-important-in-cybersecurity)
3. [Methodology](#3-methodology)
4. [Practical Tasks](#practical-tasks--powershell-solutions)
   - [Task 1: System Exploration](#task-1-system-exploration)
   - [Task 2: User Analysis](#task-2-user-analysis)
   - [Task 3: Hidden Files Investigation](#task-3-hidden-files-investigation)
   - [Task 4: Recent Activity Check](#task-4-recent-activity-check)
   - [Task 5: Executable File Scan](#task-5-executable-file-scan)
   - [Task 6: Large File Analysis](#task-6-large-file-analysis)
   - [Task 7: Temporary File Check](#task-7-temporary-file-check)
   - [Task 8: Startup Behavior Check](#task-8-startup-behavior-check)
5. [Real-World Problem Questions](#real-world-problem-questions---powershell-answers)
6. [Practical Challenges](#powershell-practical-challenges)
7. [Challenges Faced](#challenges-faced)
8. [Conclusion](#conclusion)
9. [Recommendation](#recommendation)


1. *Abstract*
This project demonstrates real-world cybersecurity investigation using Windows PowerShell. Eight practical tasks were completed to simulate forensic analysis of a Windows endpoint using only built-in PowerShell tools.

2.*Introduction:* Why PowerShell is Important in Cybersecurity?
PowerShell is critical for cybersecurity investigations because it has direct access to Windows OS objects, registry, event logs, and processes without needing external tools. 

**Key points:**
- Automates forensics + queries WMI/CIM data
- Runs remotely via WinRM  
- Attackers abuse it for "fileless malware"
- Defenders must master it to detect abuse
- Built into Windows = always available during IR

3. Methodology
**Tool:** Windows PowerShell run as Administrator  
**Method:** Native cmdlets. `-ErrorAction SilentlyContinue` used to skip access-denied folders. Results exported for documentation.


*Practical Tasks & PowerShell Solutions*

Task 1: System Exploration
**Objective:** Explore C: drive and identify purpose of each main directory.
```powershell
Get-ChildItem -Path C:\ -Directory | Select-Object Name, LastWriteTime
*Analysis:* Lists Windows, Users, Program Files. Helps spot malware hiding in wrong directories.

Task 2: User Analysis
*Objective:* Check all folders inside C:\Users for unknown/suspicious profiles.
Get-ChildItem C:\Users | Select-Object Name, LastWriteTime
*Analysis:* Unknown folders may indicate backdoor accounts. `LastWriteTime` shows creation date.

Task 3: Hidden Files Investigation
*Objective:* Find all hidden files in user directories.
Get-ChildItem C:\Users -Force -Recurse -File -ErrorAction SilentlyContinue
*Analysis:* Legit apps hide configs, but malware hides payloads/keyloggers. Hidden `.exe` in AppData = suspicious.

Task 4: Recent Activity Check
*Objective:* Identify files modified in last 24 hours.
Get-ChildItem C:\Users -Recurse -File | Where-Object {$_.LastWriteTime -gt (Get-Date).AddHours(-24)}
*Analysis:* Ransomware modifies many files fast. Spikes outside business hours = red flag.

Task 5: Executable File Scan
*Objective:* Find all .exe files and check if legit.
Get-ChildItem C:\ -Recurse -Filter *.exe -ErrorAction SilentlyContinue
*Analysis:* `.exe` in Documents/Downloads is suspicious. `AppData\Local\Temp` is classic malware drop zone.

Task 6: Large File Analysis
*Objective:* Identify largest files in system.
Get-ChildItem C:\Users -Recurse -File -ErrorAction SilentlyContinue | Sort-Object Length -Descending | Select-Object FullName, Length -First 10
*Analysis:* Large unknown files in Temp/AppData could be exfiltrated data or ransomware.

Task 7: Temporary File Check
*Objective:* Investigate Temp folders for suspicious files.
Get-ChildItem $env:TEMP -Recurse -File -ErrorAction SilentlyContinue | Select-Object FullName, Length, LastWriteTime
*Analysis:* Malware drops payloads in Temp due to write access. Check random names + recent dates.

Task 8: Startup Behavior Check
*Objective:* Find files that run at startup.
Get-CimInstance Win32_StartupCommand | Select-Object Name, Command, Location, User
*Analysis:* Malware needs persistence. Unknown `.exe` in Startup or Registry `Run` key = executes after reboot.


*Real-World Problem Questions - PowerShell Answers*

*1. Unknown .exe in Downloads. Investigation steps?*
Get-Item C:\Users\YourUser\Downloads\unknown.exe | Select-Object Name, Length, LastWriteTime
Get-FileHash C:\Users\YourUser\Downloads\unknown.exe -Algorithm SHA256

*2. File in AppData changes every hour. Why suspicious?*
Get-ChildItem $env:APPDATA -Recurse -File | Where-Object {$_.LastWriteTime -gt (Get-Date).AddHours(-1)}
Get-Process | Where-Object {$_.Path -like "*AppData*"}

*3. Large file reappears in Temp after deletion. What does it indicate?*
Get-ScheduledTask | Where-Object {$_.Actions -like "*Temp*"}
Get-CimInstance Win32_StartupCommand | Where-Object {$_.Command -like "*Temp*"}

*4. New unknown user folder in C:\Users. Explanations?*
Get-ChildItem C:\Users | Select-Object Name, LastWriteTime, CreationTime
Get-LocalUser | Select-Object Name, Enabled, LastLogon

*5. System slow + multiple unknown executables running. Approach?*
Get-Process | Sort-Object CPU -Descending | Select-Object Name, Id, CPU, Path -First 10
Get-NetTCPConnection | Where-Object {$_.State -eq "Established"}
---

*PowerShell Practical Challenges*

*Challenge 1:* List all hidden files in system
Get-ChildItem -Path C:\ -Force -Recurse -File -ErrorAction SilentlyContinue | Where-Object {$_.Attributes -match "Hidden"}

*Challenge 2:* Find all .exe files in user directories only
Get-ChildItem C:\Users -Recurse -Filter *.exe -ErrorAction SilentlyContinue

*Challenge 3:* Show files modified in last 7 days
Get-ChildItem C:\Users -Recurse -File | Where-Object {$_.LastWriteTime -gt (Get-Date).AddDays(-7)}

*Challenge 4:* Top 10 largest files in C:\Users
Get-ChildItem C:\Users -Recurse -File -ErrorAction SilentlyContinue | Sort-Object Length -Descending | Select-Object FullName, Length -First 10

*Challenge 5:* Scan AppData for suspicious executables + export
Get-ChildItem $env:APPDATA -Recurse -Filter *.exe | Select-Object FullName, Length, LastWriteTime | Out-File -FilePath C:\suspicious_exe.txt
---

Challenges Faced
1. *Access Denied* → Solved with `-ErrorAction SilentlyContinue`
2. *Long scan times* → Limited searches to `C:\Users` instead of full `C:\`
3. *Documentation* → Used `Export-Csv` / `Out-File` to save evidence

Conclusion
PowerShell alone is powerful for initial Windows endpoint forensics. Key folders to check first: AppData, Temp, and Startup. The 5 real-world questions + 5 challenges prove why these tasks matter in actual SOC work.

Recommendation
Enable PowerShell Script Block Logging and use `Get-FileHash` + VirusTotal for all unknown files.

Appendixes
...
