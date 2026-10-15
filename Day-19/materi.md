# Day 19 — Process Verification

**Completed: PASS 80/100 (mentor formative).** Final report was mentor-assisted.

Topics practiced: current processes versus historical Sysmon events; PID and parent PID; process-instance identity; file metadata, Authenticode signature and SHA-256; Windows event timelines and incident assessment.

Actual exercise: six current Discord.exe processes were observed. A separate Office process creation event was examined. Three Spotify network events were inspected: two UDP records for historical PID 6680 and a TCP record from another process instance. A later time-scoped search showed five Spotify child process creation events with parent PID 6680. Their shared parent instance identifier matched the historical UDP records.

Limitations: the parent's own creation event was not located in the five child records. Spotify file validation and correlated DNS records were not verified. No malicious or false-positive verdict was established.

Review for Day 20: PID versus PPID, child versus parent instance identifiers, SHA-256 fingerprint, false positive versus inconclusive, and accurate time-based correlation.
