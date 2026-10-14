# Day 18 — Commands (Extended Hands-on Lab)

## Actually executed, with outputs

### 1. Read five historical Sysmon network events

```powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'Microsoft-Windows-Sysmon/Operational'
    Id = 3
} -MaxEvents 5 | Select-Object TimeCreated, Id
```

Learner typed `-MaxEvent 5`; PowerShell accepted this parameter abbreviation. Prefer complete `-MaxEvents` for readability. Five event records were shown on 2026-10-14 around 20:23 local.

### 2. Inspect three historical network events

```powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'Microsoft-Windows-Sysmon/Operational'
    Id = 3
} -MaxEvents 3 | Format-List TimeCreated, Message
```

Three shown events: all Spotify.exe, all UDP. One historical PID 21148 was marked Initiated false and had source ending .255 (LAN broadcast address) with local destination [REDACTED_PRIVATE_IP], destination port 57621. Two event records involved historical PID 21452, ProcessGuid [REDACTED_PROCESSGUID], destination 35.186.224.24:443, Initiated true. Same PID and ProcessGuid means same process instance for those two, not proof of the cause of repeated logging.

`Format-List TimeCreated, Message` shows these two selected top-level properties in list layout; it does NOT automatically show every property. `Message` itself contains many Sysmon fields. `Format-List *` can display all object properties.

### 3. Check historical Discord PID against current process list

```powershell
Get-CimInstance Win32_Process -Filter "ProcessId = 8808" |
Select-Object Name, ProcessId, ParentProcessId, ExecutablePath, CommandLine
```

User reports no result for the past PID 8808. This does not invalidate historical Sysmon ID 3.

### 4. Enumerate current Discord.exe process instances

```powershell
Get-CimInstance Win32_Process -Filter "Name = 'Discord.exe'" |
Select-Object Name, ProcessId, ParentProcessId, ExecutablePath
```

Six current Discord.exe processes, PIDs 16220, 17708, 19312, 9144, 20744, 21412. ParentProcessId for five was 16220; the 16220 process had parent PID 15444. Historical PID 8808 is distinct from all six. Current parent process does NOT establish the historical parent of PID 8808.

### 5. List ten live UDP endpoint records

```powershell
Get-NetUDPEndpoint |
Select-Object -First 10 LocalAddress, LocalPort, OwningProcess
```

Endpoint snapshot was obtained. `OwningProcess` stores current process PID. Actual local IP addresses are not reproduced.

### 6. Check whether historical PID 8808 currently owns UDP endpoints

```powershell
Get-NetUDPEndpoint -OwningProcess 8808 |
Select-Object LocalAddress, LocalPort, OwningProcess
```

Returned `No MSFT_NetUDPEndpoint objects found ... OwningProcess equal to 8808`. This means no matching CURRENT UDP endpoint, not no past activity.

### 7. Check executable path

```powershell
$file = Join-Path $env:LOCALAPPDATA 'Discord\app-1.0.9261\Discord.exe'
Test-Path -LiteralPath $file
```

Result: `True` for current file.

### 8. Read file metadata

```powershell
Get-Item -LiteralPath $file |
Select-Object Name, Length, CreationTime, LastWriteTime
```

Observed: Discord.exe, Length 220542392 bytes, CreationTime 2026-10-08 16:02:01, LastWriteTime 2026-10-08 16:02:04 (local). These are filesystem timestamps, not verified first execution times.

### 9. Check Authenticode digital signature

```powershell
Get-AuthenticodeSignature -LiteralPath $file |
Format-List Status, StatusMessage, SignerCertificate
```

Observed `Status: Valid`; `StatusMessage: Signature verified.`; signer subject Discord Inc. Verification pertains to file checked at time of test and does not guarantee benign network behavior.

### 10. Calculate SHA256

```powershell
Get-FileHash -LiteralPath $file -Algorithm SHA256
```

Observed: Hash successfully calculated, SHA256 begins `08F12BDED9410C45...`. A hash is a fingerprint, not an antivirus verdict. Do not assume historical file bytes are identical without verification.

## Commands assigned in lab but whose output was NOT supplied

The time-window `Get-WinEvent` commands using `StartTime`, `EndTime`, and `Where-Object` were in the lesson. Learner answered P17–P19, but did not provide actual outputs for these two commands; therefore no verified extra historical Discord records can be documented.

## Privacy

All personal local names, hostnames, host-specific ProcessGuid values, local/private IP addresses, and personally identifying profile paths are redacted in repository material.
