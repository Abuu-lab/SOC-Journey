# 📌 Day 18 — Extended Hands-on Milestone

## Status
Extended practical work completed/submitted for sections 4–8, simulated challenge and partial report. Overall Day18 status remains PASS from prior 90/100 formative evaluation; supplemental work reveals gaps to carry into Day19. No new numerical score yet for extended work.

## Actually practiced
- Read five historical Sysmon ID3 event timestamps.
- Read three complete network event messages: all Spotify.exe and UDP; two share one ProcessGuid, one is from a different process instance.
- Differentiate local source ports from destination ports and Initiated true.
- Check historical PID8808 against current process list; absent.
- Enumerate six current Discord.exe process entries.
- Check live UDP endpoints and confirm no match for historical PID8808.
- Check Discord executable exists with Test-Path.
- Retrieve file metadata: 220542392 bytes and timestamps.
- Verify Authenticode Status Valid / Discord Inc. signer.
- Compute a SHA256 fingerprint with Get-FileHash.
- Answer simulated investigation questions and attempt a structured SOC investigation report.

## Learning and gaps
- Correctly: destination port alone not malware, current state != historical, multiple process names/instances, valid signature is evidence but not definitive safety.
- Needs correction: SHA256 does not scan viruses; port443 does not prove legitimate; ID3 does not imply DNS Query; matching ProcessGuid must be observed, not assumed.
- User's final report incorrectly says metadata/signature/SHA256 Not checked despite them being tested.
- Timeline P17–P19 lacks actual Command11/12 output, and statements about PID8808 are inconsistent. No additional historical Discord event verified from these answers.
- Parent of historical Discord PID8808 not observed; current different Discord process parent info is not its historical parent.
- Reinforce: assessment = classification plus evidence-grounded reason; next action = specific test. End report must keep event timestamps and destination IP tied to actual inspected event, not mix observations.
- Revisit false-positive definition, Format-List, and UDP vs TCP field meanings.

## Privacy
Only sanitized examples can appear in public GitHub: hide hostname, username, LAN source addresses, full local profile paths and host-specific ProcessGuid.
