# Day 19 — Materi

Topic: Windows Process Verification and SOC evidence-based assessment.

Exercises: current process enumeration, parent process identification, Sysmon historical process creation, executable metadata, code signature, file hash, network events, timeline, and simulated case.

Important concepts: a PID may be reused; shared ParentProcessId is not shared ProcessId. Sysmon Event ID 1 means Process Create, ID 3 network activity, ID 22 DNS Query. A SHA-256 hash is a file fingerprint, not a malware scanner. Valid code signing does not guarantee benign behavior. False positive differs from an inconclusive alert. DNS event absence cannot be inferred from an unperformed query.

Day 19 remains in progress: mini-project report has not been submitted. Mentor must teach command usage incrementally and assess output before introducing complex scripts.
