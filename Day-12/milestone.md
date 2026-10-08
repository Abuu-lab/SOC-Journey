# Day 12 — 📌 Catatan Penting / Milestone Hari Ini

- Sysmon
- Sysmon Service
- Sysmon Event ID 1 — Process Creation
- Historical Process Telemetry
- ProcessGuid
- ProcessId
- Image
- CommandLine
- User
- IntegrityLevel
- ParentProcessId
- ParentImage
- ParentCommandLine
- Sysmon Hash telemetry
- Endpoint visibility
- Historical process investigation
- Correlation: Process → User → Parent → CommandLine → File → Hash
- Process yang sudah mati tidak selalu mengakhiri investigation



Catatan kecil yang akan kita simpan untuk milestone-mu:



Data praktismu sendiri menunjukkan Sysmon Event ID 1 dapat menyediakan `ProcessId`, `Image`, `CommandLine`, `User`, hash, `ParentProcessId`, `ParentImage`, dan `ParentCommandLine` dalam satu event. 

---
