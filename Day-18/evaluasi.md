# Day 18 — Evaluasi Extended Hands-on

**Status:** Day18 PASS (prior core lab 90/100 formative); extended lab completed with corrections and unresolved evidence. No fabricated new verified outputs.

## Section 4: Network Fundamentals
- N1 user's answer: "sourceport itu port local dr pc kita ... port tujuan." Correct. Preferred terms SourcePort and DestinationPort.
- N2: user does not know port4444 but refuses to equate it with malware. Correct: port alone does not prove intent.
- N3: "inisiasi local." Correct for Initiated true.

## Section 5: Historical Network Events
Command1: Get-WinEvent FilterHashtable Id3 -MaxEvent 5 | Select-Object TimeCreated,Id. User executed successfully; five records around 20:23 on Oct14. Parameter abbreviation -MaxEvent was accepted; prefer -MaxEvents.
Command2: -MaxEvents 3 | Format-List TimeCreated,Message.
User: "format-list itu untuk memunculkan semua informasi ya?" Correction: it controls list formatting for selected properties. Message includes detailed fields; Format-List * shows all available object properties.
P4 user "Spotify.exe semua; ada 2 PID yg sama 1 berbeda" correct; two PID21452 records same ProcessGuid, PID21148 separate instance.
P5 "udp semua" correct.
P6 matching PID events same IP/port; separate PID event differs correct. Note repeated records are not proof of separate connections; Initiated false record includes broadcast-style local address, not automatically external malicious traffic.

## Section 6: Current process vs historical
P7 user reports PID8808 not currently found. Correct reported result.
P8 output lists 6 current Discord.exe processes (16220, 17708, 19312, 9144, 20744, 21412). Correct.
P9 "aku tidak mengerti false positive, tapi ... jangan memberi kesimpulan terburu2." Good caution but definition missed. False positive = an alert/detection of alleged undesired behavior not supported after verification. A historical event is not false simply because originating PID now absent. ParentProcessId values for current Discord instances do not establish parent of historical PID8808.

## Section 7: UDP endpoints
P10: OwningProcess stores PID — correct.
P11: Get-NetUDPEndpoint -OwningProcess 8808 returned ObjectNotFound; no CURRENT matching UDP endpoint — correct.
P12: current UDP endpoint view different from historical Sysmon ID3 — correct. This does not invalidate previous network event.

## Section 8: File investigation
P13: Test-Path True — correct for tested file.
P14: File metadata recorded Discord.exe 220542392 bytes, CreationTime 2026-10-08 16:02:01, LastWriteTime 16:02:04 — correct.
P15: Signature Status Valid, verified; certificate subject Discord Inc. — correct, not absolute safety proof.
P16 user "SHA256 bisa sebagai evidence untuk memeriksa apakah file mengandung konten virus". Incorrect: SHA256 fingerprint cannot itself scan or detect malware. Compare fingerprint against known hashes/other file samples, plus contextual evidence. User successfully calculated SHA256; repository omits full hash as unnecessary unique file datum.

## Section 9: Timeline
P17 user "ya masih ada PID 8808". Not verified: no Command 11/12 historical time-window output supplied. Inconsistent with P7 no current PID; historical event presence does not prove process currently running.
P18 "tidak ada, semua PID 8808". Unverified without corresponding timeline event output.
P19 "ya benar 1 processguid". Not established; same PID/image alone does not prove ProcessGuid match. Need inspect GUID of each relevant historical event.

## Section 10: Simulated Challenge #018
Scenario: audiohelper.exe from Public Downloads, Sysmon ID3 UDP locally initiated, destination 198.51.100.44:443, parent/signature/hash unknown. No DNS event supplied.
C1 "Audiohelper.exe" correct.
C2 "UDP" correct.
C3 Source vs destination IP correct concept, although scenario source is documentation-range address, not their real private network.
C4 file in Downloads suspicious or unknown: legitimate reason to investigate, not a malware verdict.
C5 executable name may mislead: correct.
C6 cannot call untested signature Valid: correct evidence discipline.
C7 legitimate because port443: incorrect. Port443 is not proof of legitimate behavior. Benign possibility could be authorized application operation; malicious possibility could be masquerading executable.
C8 described "DNS Query" to destination IP, incorrectly; simulated evidence is network Event ID3 only. Assessment ideal: Needs Investigation due to insufficient parent/file identity verification and network context.
C9 "check Sysmon ID1 dan 22 lalu SHA256, DigitalSignature": direction appropriate but not a clear ordered 3-step investigation; suggest retrieve ID1 parent/commandline, inspect file path/signature/hash, review related network/DNS context.

## Section 11: User's mini report and precise corrections
User's report listed Discord observation at 15:59:50 and destination 162.159.128.232. Previously documented Discord event was at 15:55:07 with destination 162.159.130.233. The different time/IP may refer to an additional event, but matching original output for this later observation was NOT provided; cannot assume same event. Anchor the report to the verified 15:55:07 event or supply a matching event log.
Reported historical PID8808 absent: consistent with prior query.
ParentProcess blank: historical parent not verified.
ExecutablePath profile sensitive; sanitized as C:\Users\[REDACTED_USER]\AppData\Local\Discord\app-1.0.9261\Discord.exe.
File Exists True, metadata checked, signature Valid, SHA256 calculated; report said Not checked for those three fields — contradiction. Fix by filling actual collected evidence.
Historical Sysmon ID3 available. Current UDP endpoint for PID8808 not found. Other events: 3 Spotify historical records were examined, not correlated with Discord event.
Evidence gap should NOT include metadata/signature/hash as Not Checked after execution. IDs1/22 may be unfound or simply not searched for Discord ProcessGuid — note NOT CHECKED, not nonexistent.
Legitimate hypothesis "path looks normal" supports possibility, does not prove legitimacy.
Malicious hypothesis "because UDP" invalid: UDP itself not malicious.
Assessment should be "Insufficient Evidence / Needs Investigation" with reason and context, not just "need further investigation."
Next Action should specify investigation of historical process creation, parent and Discord network correlation by ProcessGuid/time.

## Example mentor-assisted corrected report
Case: Discord.exe historical Sysmon UDP network event.
Observation: 2026-10-14 15:55:07 local, Sysmon Event ID3, historical PID8808, UDP Initiated true, destination 162.159.130.233:443.
Process: current PID8808 absent when queried; six other Discord.exe processes present at check; historical parent not verified.
Executable: current Discord.exe file at redacted user profile path exists; file size 220542392 bytes, metadata timestamps Oct8 2026, Authenticode Valid signed Discord Inc., SHA256 computed.
Network: historical event ID3 available; no current UDP endpoint belonging to PID8808; Event ID1/22 not yet correlated for same historical instance.
Evidence gap: parent/commandline of historical process, application-layer protocol, corroborating event correlations, definitive behavior assessment.
Hypothesis benign: ordinary Discord network behavior. Hypothesis malicious: misuse/masquerading or anomalous activity may require testing; UDP alone is not reason.
Assessment: Insufficient Evidence for malicious verdict. Evidence supports plausible legitimate executable but not a complete activity verdict.
Next Action: investigate same ProcessGuid historical Process Create and nearby Sysmon networking; check event/time identity and any relevant DNS logs before claiming causal relation.

## Milestone / carry to Day19
Practice completed: historical Sysmon search, reading Image/PID/Guid/source-destination/protocol, live process and UDP endpoint checks, file existence/metadata/Authenticode/SHA256, preliminary investigator reasoning.
Active recall needs: SHA256 function, 'false positive', Format-List vs Select-Object, current/historical, PID reuse/ProcessGuid identity, Event ID3 != DNS, port 443 != safe, observation vs hypothesis vs assessment; report evidence consistency.
