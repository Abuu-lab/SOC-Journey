# Day 16 — Materi
## Sysmon Telemetry Troubleshooting & Evidence Validation

**Objective:** Determine whether Sysmon logging exists, what event types are available, distinguish current vs historical evidence, and document observation, hypothesis, test, result, evidence gap, and assessment.

## Terminology and mental model

- Sysmon is a Windows system service and event provider. Service Running does **not** prove every Sysmon Event ID is logged.
- Event ID 1: Process Create; records Image, ProcessId, ProcessGuid, parent information, command line, user context, hash (depending on configuration).
- Event ID 3: Network Connection; records process-associated TCP/UDP networking when enabled and matched by rules.
- Event ID 22: DNS Query; a useful complement to network connection evidence.
- Event ID 2: File Creation Time Change; 4: Sysmon Service State Change; 5: Process Terminate; 11: File Create; 13: Registry Value Set; 16: Sysmon Configuration Change.
- ProcessId (PID) can be reused by Windows after a process terminates.
- ProcessGuid refers to a Sysmon process instance and helps correlate Event ID 1 and other event types; it is **not** simply the PID plus network information.
- Event ID 1 = historical process creation; Event ID 3 = historical network connection; Get-NetTCPConnection = current TCP connection view; Get-CimInstance Win32_Process = current process information.
- Both ::1 endpoints mean IPv6 loopback on the same host, not an external internet destination.
- RemotePort 443 does not by itself prove HTTPS, nor that activity is legitimate.
- IntegrityLevel High is an elevated integrity level, but does not grant all Windows privileges.
- services.exe is the Service Control Manager process; svchost.exe is a host for relevant Windows services; -k CameraMonitor names a service host group, not evidence of camera recording.

## Three telemetry states

1. Log channel unavailable: the requested channel cannot be located by the tool on the queried system. Check host, service, registered channel; do not automatically reinstall.
2. Channel available, requested Event ID absent: event might not have happened, logging may be disabled/filtered, events may have rolled over.
3. Event ID present: evidence exists that records of the type were collected, though specific behavior and full coverage need additional verification.

## Confirmed outputs from Day 16

**Actual practice**: Sysmon service Running/Auto, operational channel used successfully; 108 Event ID 3 and 352 Event ID 22 records. Latest Event ID 1 sample: EpicGamesUpdater.exe executed with queryinstallation, parent EpicGamesLauncher.exe. Process path appears consistent with Epic Games installation, but signature, behavior, and full reputation were not verified. Do not assert 100% benign.

**Important distinction**: earlier example with 43 records was *total record count at an earlier check*, NOT '43 Event ID 1'. Day 16 snapshot has 71 Event ID 1 and 108 Event ID 3. The mini SOC case was hypothetical and assumed Event ID 3 was absent; do not confuse it with the actual host data.

## Investigation report template

~~~text
Observation: Which events/fields were actually observed?
Hypothesis 1 / 2: Alternative possible explanations.
Test: Commands or validations planned/performed.
Expected Result: What would a chosen test clarify?
Actual Result: Only if the test was performed.
Evidence Available: Sources already seen (no inventions).
Evidence Gap: What's missing or unavailable?
Assessment: Evidence-grounded judgment.
Next Action: Best next step.
~~~

A blank Get-CimInstance for a past PID does not prove false positive. If ProcessId same but ProcessGuid differs, test whether they are different process instances / PID reuse. A current established connection does not require a corresponding Event ID 3 to be present, since logging may be disabled, filtered, or incomplete.

## Technical reference (outside lesson)
- Microsoft Sysmon: https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon
- Sysmon events: https://learn.microsoft.com/en-us/windows/security/operating-system-security/sysmon/sysmon-events

## Documentation privacy
Log excerpts and machine-specific account/host identifiers have been anonymized. Do not commit raw username, hostname, or personal profile paths.
