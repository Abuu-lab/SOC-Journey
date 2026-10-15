# Day 19 — Commands

**Status:** Hands-on 1–5 dijalankan; Hands-on 6 berupa reasoning tanpa output korelasi tambahan; mini-project belum dikumpulkan.

## Hands-on 1: Current Discord process list
~~~powershell
Get-CimInstance Win32_Process -Filter "Name = 'Discord.exe'" |
Select-Object Name, ProcessId, ParentProcessId, ExecutablePath, CommandLine
~~~
**Actual result:** enam Discord.exe process instances dengan enam PID berbeda: 2724, 7856, 22476, 9028, 18508, dan 12432. Lima child memiliki ParentProcessId 2724, sedangkan PID 2724 memiliki parent PID 4632. Path dan account lokal disamarkan. Command lines menunjukkan role Electron: crashpad-handler, gpu-process, network utility, renderer, dan audio utility. Role tersebut tidak membuktikan sendiri file aman atau berbahaya.

## Hands-on 2: Query chosen PID
~~~powershell
Get-CimInstance Win32_Process -Filter "ProcessId = 2724" |
Select-Object Name, ProcessId, ParentProcessId, ExecutablePath, CommandLine
~~~
**Actual result:** proses yang dicari ialah Discord.exe dengan PID 2724, **ParentProcessId 4632**. Identitas proses pemilik PID 4632 belum diperiksa. P4 awal salah menjawab 4632 sebagai nama parent: itu angka PPID, bukan hasil query nama parent.

## Hands-on 3: Latest Sysmon Event ID 1
~~~powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'Microsoft-Windows-Sysmon/Operational'
    Id = 1
} -MaxEvents 1 | Format-List TimeCreated, Message
~~~
**Actual result:** 2026-10-15 17:12:32 local, Microsoft Office opushutil.exe (PID 3424), command line /pushregistration, IntegrityLevel Medium, parent svchost.exe (PID 2144), ParentCommandLine includes `-k netsvcs -p -s Schedule`. User/device GUID and private account redacted. Do not assert legitimate or malicious from basename alone.

## Hands-on 4: Verify current Discord executable
~~~powershell
$file = Join-Path $env:LOCALAPPDATA 'Discord\app-1.0.9261\Discord.exe'
Test-Path -LiteralPath $file
Get-Item -LiteralPath $file |
Select-Object Name, Length, CreationTime, LastWriteTime
Get-AuthenticodeSignature -LiteralPath $file |
Select-Object Status, StatusMessage
Get-FileHash -LiteralPath $file -Algorithm SHA256
~~~
**Actual result:** Test-Path True; Discord.exe 220542392 bytes; CreationTime 2026-10-08 16:02:01, LastWriteTime 16:02:04 (local). Authenticode status Valid, Signature verified. SHA256 was computed; public record does not publish full fingerprint. SHA256 is a content fingerprint, **not** virus scanning. File data are a current file check, not proof of the exact bytes associated with every historical process event.

## Hands-on 5: Three Sysmon Network Events
~~~powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'Microsoft-Windows-Sysmon/Operational'
    Id = 3
} -MaxEvents 3 | Format-List TimeCreated, Message
~~~
**Actual results:** three Spotify.exe ID 3 records, on 2026-10-15 local:
- 17:24:40: TCP, PID 22300, ProcessGuid [REDACTED_GUID_A], outbound to [REDACTED_EXTERNAL_IP]:443.
- 17:24:25: UDP, PID 6680, ProcessGuid [REDACTED_GUID_B], Initiated false, source multicast 239.255.255.250:1900 toward local UDP endpoint [REDACTED_PRIVATE_IP]:56104.
- 17:24:24: UDP, same PID 6680 and same ProcessGuid [REDACTED_GUID_B], Initiated true, local endpoint to multicast 239.255.255.250:1900.

Port 1900 is labeled ssdp in log. Multicast SSDP-related traffic is distinct from internet traffic on port 443. The samples contain **2 process instances** (two distinct GUIDs), even though there are 3 events.

## Hands-on 6: Timeline
No additional command output submitted. Learner tried to order Event ID 1 → 3 → 22, but the supplied actual records were ID 1 from opushutil.exe and three ID 3 from Spotify.exe. **No Event ID 22 was supplied for this lab.** Correct chronological order of observed records:
1. 17:12:32 ID 1 opushutil.exe.
2. 17:24:24 ID 3 Spotify.exe (UDP).
3. 17:24:25 ID 3 Spotify.exe (UDP).
4. 17:24:40 ID 3 Spotify.exe (TCP).
Do not imply the Office event caused or relates to the Spotify events; GUIDs differ.

## Next reporting commands — TAUGHT, NOT YET EXECUTED FOR MINI PROJECT
Read-only options already learned for single-case follow-up:
~~~powershell
Get-CimInstance Win32_Process -Filter "ProcessId = 6680" |
Select-Object Name, ProcessId, ParentProcessId, ExecutablePath, CommandLine

Get-WinEvent -FilterHashtable @{
    LogName = 'Microsoft-Windows-Sysmon/Operational'
    Id = 3
    StartTime = (Get-Date '2026-10-15 17:24:00')
    EndTime = (Get-Date '2026-10-15 17:25:00')
} | Format-List TimeCreated, Message
~~~
No assumption that historical PID 6680 is still in use. Neither command above has been executed for mini-project as of this submission.

## Privacy
All local user names, raw hostname, unique Sysmon GUIDs, user-profile identifying paths and private IP addresses are redacted from this portfolio.
