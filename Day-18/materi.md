# Day 18 — Materi Extended Lab

## Learning flow
One concept, one explained command, inspect result, investigate, assess. This extended session addresses learner feedback that earlier Day18 was too short.

## Sysmon and network
ID 3 records a network event. ID22 records DNS query; DNS resolution is not proof of a network connection. ID1 is Process Create; ID2 is File Creation Time Changed. ProcessGuid helps identify process instances, while PID can be reused. SourceIp/SourcePort and DestinationIp/DestinationPort describe endpoints. Initiated true means locally initiated; false requires careful interpretation. UDP does not use TCP state Established. Port 443, even a port-service label https, is not proof of HTTPS over TCP, safety or exfiltration.

## Historical vs current
Get-WinEvent returns logged historical evidence. Get-CimInstance Win32_Process and Get-NetUDPEndpoint present current state. A missing historical PID from the current state cannot invalidate Sysmon logs. A process can terminate or the PID can be reused. Multiple process records under one application are possible; same name does not prove same instance.

## PowerShell fundamentals
Get-WinEvent with FilterHashtable and MaxEvents retrieves matching events. Select-Object chooses object properties. Format-List formats output in list form; Format-List TimeCreated,Message does not mean display every property. Format-List * can display all available properties. Where-Object can filter; later steps require understanding simple commands before introducing complex XML loops.

## File verification
Test-Path checks the selected path. Get-Item shows file metadata. Get-AuthenticodeSignature checks Authenticode signature and certificate signer. Get-FileHash -Algorithm SHA256 computes content fingerprint, NOT virus detection. Valid signature is not guarantee of safe behavior. File metadata timestamps do not prove first execution time.

## Real lab observations
A historical Discord.exe network event on October 14, 2026 showed UDP activity initiated locally to destination 162.159.130.233:443, historical PID8808. Later Get-CimInstance found six current Discord.exe processes but not PID8808. Get-NetUDPEndpoint -OwningProcess 8808 found no current UDP endpoint. Test-Path for Discord.exe returned True. Metadata showed Length 220542392, CreationTime 2026-10-08 16:02:01, LastWriteTime 16:02:04. Authenticode signature Status Valid, signer Discord Inc. SHA256 hash was successfully computed. These results concern the file checked later; identical bytes at historical event time were not separately proven.

Another query retrieved three Spotify.exe network events, all UDP. Two shared PID21452 and matching ProcessGuid and destination 35.186.224.24:443. One used PID21148, Initiated false, broadcast-style private LAN endpoint. The same image name does not mean same process instance. Event duplication cannot be diagnosed from three event fields alone.

## SOC investigation workflow
Observation → Evidence Available → Evidence Gap → Hypotheses → Test and Result → Assessment → Next Action.
Assessment and Next Action are distinct. Absence of ID1/22 in provided output is not proof such events do not exist. Do not claim DNS Query in a case that only supplies ID3. Avoid asserting malicious solely from a download path or a numbered port.

## Learning weakness
The learner understands many observation fields but needs more precision in assessment, file hash concepts, timestamp/PID distinction, and report consistency. Long unexplained scripts were counterproductive; maintain full-length sessions with short explained steps.
