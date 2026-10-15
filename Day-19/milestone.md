# Day 19 — Milestone Hari Ini

**PASS — 80/100 (formative mentor evaluation).**

Completed: current Windows process enumeration, historical Sysmon process creation, file metadata/signature/hash checks on Discord, and three Spotify network event observations.

Key result: a time-window search revealed five Spotify child process creation events sharing parent PID 6680. Their shared parent process-instance identifier matched previously collected UDP network events belonging to PID 6680. This supports a real parent-child and network evidence correlation.

Evidence gap: the parent's own creation record, Spotify file validation and related DNS telemetry have not been confirmed. The final report and the outstanding simulated challenge answer were mentor-assisted, not independently completed.

Day 20 recall topics: PID/PPID, process-instance identity, SHA-256 versus malware scanning, false positive versus insufficient evidence, and careful evidence-based timeline construction.

Policy: 76–100 PASS and advance. Below 76: short focused explanation and 2–5 remedial questions/practices, then reassess without requiring the whole day to be repeated.

Privacy: device/account identifiers are not published.
