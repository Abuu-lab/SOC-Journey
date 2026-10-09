# Day 15 — Commands

**Tema:** Process-to-Network Investigation | Windows TCP + Sysmon

## Commands yang benar-benar dijalankan

### 1. Melihat satu koneksi TCP Established
\`\`\`powershell
$conn = Get-NetTCPConnection -State Established | Select-Object -First 1
$conn | Format-List LocalAddress, LocalPort, RemoteAddress, RemotePort, State, OwningProcess
\`\`\`
**Hasil kasus asli:** \`::1:51455 → ::1:35783\`, \`Established\`, \`OwningProcess = 22704\`.

### 2. Menghubungkan OwningProcess dengan PID
\`\`\`powershell
$targetPid = $conn.OwningProcess

Get-CimInstance Win32_Process -Filter "ProcessId = $targetPid" |
Select-Object Name, ProcessId, ParentProcessId, ExecutablePath, CommandLine
\`\`\`
**Hasil asli:** EpicGamesLauncher.exe, PID 22704, PPID 11220.

### 3. Mengidentifikasi user pemilik proses
\`\`\`powershell
Get-Process -Id $targetPid -IncludeUserName |
Select-Object ProcessName, Id, UserName
\`\`\`
**Hasil asli:** \`DESKTOP-6MVPCMR\elvin\`.

### 4. Query Sysmon Event ID 3 (pernah dicoba, GAGAL karena channel tidak ditemukan)
\`\`\`powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'Microsoft-Windows-Sysmon/Operational'
    Id = 3
} -MaxEvents 3 -ErrorAction Stop |
Format-List TimeCreated, Id, Message
\`\`\`
**Error:** \`NoMatchingLogsFound\`, bukan \`NoMatchingEventsFound\`. Channel \`Microsoft-Windows-Sysmon/Operational\` tidak ditemukan pada host yang diperiksa. Jangan mengklaim Event ID 3 tersedia.

## Tambahan dari materi Day 15 dan koreksi (belum seluruhnya dijalankan)

\`\`\`powershell
Get-NetTCPConnection
Get-NetTCPConnection -State Established
Get-NetTCPConnection -OwningProcess 4320
Get-CimInstance Win32_Process -Filter "ProcessId = 4320" |
Select-Object Name, ProcessId, ParentProcessId, ExecutablePath, CommandLine
hostname
whoami
Get-Service -Name Sysmon64,Sysmon -ErrorAction SilentlyContinue
Get-WinEvent -ListLog '*Sysmon*' -ErrorAction SilentlyContinue |
Select-Object LogName, IsEnabled, RecordCount
\`\`\`

## Commands Day 14 yang harus diingat (active recall)

\`\`\`powershell
Get-CimInstance Win32_Service |
Where-Object {$_.ProcessId -eq 6920} |
Select-Object Name, DisplayName, State, StartName, ProcessId

Test-Path "C:\path\file.exe"
\`\`\`

**Catatan:** \`Get-CimInstance Win32_Process\` melihat proses; \`Win32_Service\` melihat service. \`OwningProcess\` adalah nama field yang berisi PID koneksi TCP.