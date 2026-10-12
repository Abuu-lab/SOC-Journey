# Day 16 — Commands

**Topic:** Sysmon Health, Event Distribution, Process Create Analysis, Telemetry Gap.

## Commands actually executed by learner

### 1. Count Sysmon records by Event ID
~~~powershell
Get-WinEvent -LogName 'Microsoft-Windows-Sysmon/Operational' |
Group-Object Id |
Select-Object Name, Count |
Sort-Object Name
~~~

Observed output (no user/hostname retained):

| Event ID | Count |
|---|---:|
| 1 | 71 |
| 2 | 29 |
| 3 | 108 |
| 4 | 1 |
| 5 | 3 |
| 11 | 4 |
| 13 | 5 |
| 16 | 1 |
| 22 | 352 |

**Meaning:** 108 historical Event ID 3 records are present. Recording Event ID 3 has been demonstrated; it does not guarantee every network connection is captured.

### 2. Verify Sysmon service status
~~~powershell
Get-Service -Name Sysmon,Sysmon64 -ErrorAction SilentlyContinue |
Select-Object Name, Status, StartType
~~~

Observed: Sysmon, Running, Automatic.

### 3. Inspect Win32_Service details
~~~powershell
Get-CimInstance Win32_Service |
Where-Object {$_.Name -in @('Sysmon', 'Sysmon64')} |
Select-Object Name, State, StartMode, ProcessId, PathName
~~~

Observed: Sysmon; Running; Auto; PID 26508 (historical snapshot); PathName C:\Windows\Sysmon.exe.

### 4. Retrieve latest historical Process Create
~~~powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'Microsoft-Windows-Sysmon/Operational'
    Id = 1
} -MaxEvents 1 |
Format-List TimeCreated, Id, Message
~~~

Observed: EpicGamesUpdater.exe (PID 21144) with parent EpicGamesLauncher.exe (PPID 15924), IntegrityLevel Medium, CommandLine EpicGamesUpdater.exe queryinstallation. User account and unique GUIDs withheld.

## Optional next steps — taught but NOT executed yet in Day 16

~~~powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'Microsoft-Windows-Sysmon/Operational'
    Id = 3
} -MaxEvents 1 |
Format-List TimeCreated, Message
~~~

Read a Network Connection event, if available. Correlate its ProcessGuid to an Event ID 1 rather than rely on PID alone.

## Security/privacy
Never commit raw hostname, local username, user's profile-directory path, or raw logs containing them. Substitute [REDACTED_HOST], [REDACTED_USER] where applicable.
