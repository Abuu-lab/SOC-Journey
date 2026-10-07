# Day 12 — Milestone Hari Itu

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

### Catatan milestone dari materi/evaluasi

### 📌 MILESTONE 1 TAHUN — DAY 12

Catatan kecil yang akan kita simpan untuk milestone-mu:

##### DAY 12 — Sysmon & Endpoint Telemetry

* Sysmon
* Sysmon Service
* Sysmon Event ID 1 — Process Creation
* Historical Process Telemetry
* ProcessGuid
* ProcessId
* Image
* CommandLine
* User
* IntegrityLevel
* ParentProcessId
* ParentImage
* ParentCommandLine
* Sysmon Hash telemetry
* Endpoint visibility
* Historical process investigation
* Correlation: Process → User → Parent → CommandLine → File → Hash
* **Process yang sudah mati tidak selalu mengakhiri investigation**

Data praktismu sendiri menunjukkan Sysmon Event ID 1 dapat menyediakan `ProcessId`, `Image`, `CommandLine`, `User`, hash, `ParentProcessId`, `ParentImage`, dan `ParentCommandLine` dalam satu event. 

---
