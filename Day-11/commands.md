COMMANDS DAY 11
1. Cari 4625
Get-WinEvent -FilterHashtable @{
    LogName='Security'
    Id=4625
} -MaxEvents 10 |
Select-Object TimeCreated, Id, ProviderName, LevelDisplayName, Message
2. Konversi HEX PID → Decimal
[Convert]::ToInt32("5f60",16)

Ganti "5f60" dengan hexadecimal PID yang kamu dapat.

3. Cari Process + User
Get-Process -Id <PID> -IncludeUserName |
Select-Object ProcessName, Id, UserName
4. Cari Full Process Context
Get-CimInstance Win32_Process -Filter "ProcessId = <PID>" |
Select-Object Name, ProcessId, ParentProcessId, ExecutablePath, CommandLine
5. Cari Parent Process
Get-CimInstance Win32_Process -Filter "ProcessId = <PPID>" |
Select-Object Name, ProcessId, ParentProcessId, ExecutablePath, CommandLine
6. File Metadata
Get-Item "<ExecutablePath>" |
Select-Object Name, Length, CreationTime, LastWriteTime, LastAccessTime
7. Digital Signature
Get-AuthenticodeSignature "<ExecutablePath>"
8. SHA-256
Get-FileHash "<ExecutablePath>" -Algorithm SHA256
