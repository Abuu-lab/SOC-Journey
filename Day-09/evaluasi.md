# Day 09 — Evaluasi

Catatan ini mempertahankan jawaban, koreksi, dan penilaian yang tercatat. Sebagian jawaban hanya tersedia sebagai ringkasan mentor; pertanyaan atau jawaban lengkap yang tidak tercatat tidak direkonstruksi. Skenario latihan tidak dianggap sebagai incident pada endpoint aktual.

## Challenge

### 8. CHALLENGE DAY 9

##### Q1 ✅

4624 = successful logon.

Benar berdasarkan challenge yang diberikan.

##### Q2 ✅

4688 = process creation.

##### Q3 ✅

7045 = service installation.

##### Q4 ✅

Kamu menjawab:

> belum cukup

✅ Benar.

##### Q5 🟡

Kamu:

> lihat process yang dibuat apa

Benar sebagai inti.

Tapi kita ingin lebih detail:

```text
Process name
PID
Parent process
CommandLine
```

Ini menghubungkan Event Log dengan Day 7.

##### Q6 ✅

> service apa yang di-install

Benar.

Nanti kita juga akan perhatikan:

```text
Service Name
Service File Name
Start Type
Service Account
```

Contohnya terlihat pada event 7045 milik labmu. 

##### Q7 🟡

Kamu mengatakan:

> harus tahu kapan dibuatnya kapan diaksesnya

Ide dasarnya benar, tetapi **timestamp event berarti kapan event terjadi/tercatat**, bukan otomatis kapan suatu file "dibuat".

Ini penting dibedakan:

```text
File CreationTime
≠
Event Time
```

##### Q8 ❌

Kamu lupa membuat timeline.

Ini justru akan menjadi fokus latihan kita.

Dari challenge:

```text
20:01:10
4624 — User Logon

20:03:22
4688 — PowerShell Process Created

20:03:25
4688 — Another Process Created

20:04:10
7045 — Service Installed
```

Jadi cara membuat timeline sebenarnya sederhana:

> **Susun event berdasarkan TimeCreated dari paling awal → paling akhir.**

Kita akan ulang ini nanti.

##### Q9 ✅

Benar:

> Tidak ditemukannya 4688 tidak membuktikan process creation tidak pernah terjadi.

##### Q10 ✅

Benar:

> Satu 4625 bisa saja karena user salah password.

---

## Mini-project / Investigasi

### 9. MINI SOC PROJECT

Bagian ini sebenarnya cukup bagus.

Kamu mendapatkan event System seperti:

```text
1041
Microsoft-Windows-TPM-WMI
```

dan:

```text
Event 1
Microsoft-Windows-Power-Troubleshooter
```



Application:

```text
edgeupdate
Security-SPP
```



Security:

```text
4672
SYSTEM
```



Jadi kamu **berhasil mengumpulkan event nyata dari Windows**.

---
### 10. ANALYSIS MINI PROJECT

Kamu menjawab:

> waktu dibuatnya, PID, providername, leveldisplayname, message, subject, privilege

🟡 Sebagian benar.

Yang perlu diperbaiki:

`TimeCreated`:

> kapan event terjadi/tercatat

bukan:

> kapan dibuatnya

Karena "created" bisa menimbulkan kebingungan dengan `CreationTime` file.

Kamu juga sudah melihat:

```text
ProviderName
LevelDisplayName
Message
Subject
Privileges
```

tetapi belum memahami semuanya.

Itu **normal**, karena memang baru pertama kali kita melihat field-field tersebut.

---

## Active Recall / Self-test

### 7. ACTIVE RECALL

Sekarang kita tidak perlu mengulang semua pertanyaan yang sudah benar. Aku hanya fokus pada yang masih salah/belum terjawab.
### 🔁 YANG AKAN DIULANG DI DAY 10

Sesuai aturanmu, **yang sudah benar tidak akan aku spam ulang besok**.

Yang perlu diperbaiki:

```text
1. ExecutablePath vs CommandLine
2. Service vs Process
3. Digital Signature
4. Security Context
5. Current State vs Historical Evidence
6. Arti Timestamp
7. Cara membuat Timeline
```

Untuk yang salah, aku akan ulang sampai kamu bisa menjawab **2× dengan benar**, baru kita lepas dari daily recall.

Yang sudah kamu kuasai seperti PID, PPID, SHA-256, basic User Context, dan 4624/4625/4688/7045 **tidak perlu kita tanyakan setiap hari**. Kita simpan untuk recall lagi sekitar **2–3 minggu**.

## Penilaian dan Hasil Praktik


### 🟢 EVALUASI DAY 9 — PASSED

Aku sudah cek **seluruh hasil Day 9** yang kamu kirim. Sesuai aturanmu:

> **76–100% = PASS**
> Kesalahan yang belum benar akan diulang **2× sampai benar**.
> Yang sudah benar tidak perlu diulang setiap hari; nanti bisa muncul lagi setelah **2–3 minggu**.

Hasilmu **PASS**. Kamu sudah mulai memahami konsep **historical evidence**, walaupun beberapa istilah Event Log masih belum kuat.

---
### 1. PRAKTIK 1 — SYSTEM LOG ✅

Kamu mendapatkan Event ID `7040`:

```text
Background Intelligent Transfer Service
start type changed
auto start → demand start
```

dan sebaliknya:

```text
demand start → auto start
```

Kamu juga menemukan Event ID `105` dari `Microsoft-Windows-Kernel-Power`. 

Bagus. Kamu berhasil membaca:

```text
TimeCreated
Event ID
Provider
Level
Message
```

##### Catatan penting

`7040` di hasilmu **bukan otomatis suspicious**. Itu hanya menunjukkan perubahan start type suatu service. Untuk menentukan apakah perubahan itu penting bagi security investigation, kita perlu context tambahan.

---
### 2. PRAKTIK 2 — APPLICATION LOG ✅

Kamu mendapatkan:

```text
16384
16394
```

dengan provider:

```text
Microsoft-Windows-Security-SPP
```

dan semuanya `Information`. 

✅ Benar.

Kamu sudah bisa membaca struktur event.

---
### 3. PRAKTIK 3 — SECURITY LOG ✅✅

Ini justru bagian paling bagus.

Kamu menemukan:

##### Event 4672

```text
Account Name: SYSTEM
Privileges:
SeDebugPrivilege
SeLoadDriverPrivilege
...
```

##### Event 4624

```text
Account Name: SYSTEM
Logon Type: 5
Elevated Token: Yes
Process:
C:\Windows\System32\services.exe
```

Semua itu memang ada dalam output yang kamu kirim.  

Ini sudah mulai menarik karena sekarang kamu melihat:

```text
Event
 ↓
Account
 ↓
Privilege
 ↓
Logon
 ↓
Process
```

Itulah **correlation** yang nantinya akan sangat penting.

---
### 4. EVENT 4625 ✅

Kamu menemukan:

```text
Event ID: 4625
Account: Buya
Failure Reason:
Unknown user name or bad password
```

dan:

```text
Logon Type: 2
Source Address: 127.0.0.1
Caller Process:
C:\Windows\System32\svchost.exe
```



Ini bagus sekali untuk latihan investigation.

Yang dapat kita katakan **berdasarkan event ini**:

> Ada **failed logon** untuk account `Buya`, dengan `Logon Type 2`, dan source address `127.0.0.1` pada event tersebut.

Jangan langsung:

```text
4625 = attacker
```

Karena satu failed authentication juga bisa terjadi karena kesalahan password. Kamu menjawab ini dengan benar di Challenge.

---
### 5. EVENT 4688 ✅

Kamu tidak mendapatkan event:

```text
No events were found
```



✅ Ini jawaban valid.

Dan ini penting:

> **No result ≠ No process creation ever happened.**

Bisa karena auditing/configuration/log retention dan faktor lainnya.

---
### 6. EVENT 7045 ✅

Kamu menemukan:

```text
SOC Analyst Test Service
Service File Name:
C:\Windows\System32\notepad.exe

Start Type:
auto start

Service Account:
LocalSystem
```



Dan event yang sama muncul lagi pada 22 September. 

Ini sangat bagus untuk menghubungkan:

```text
DAY 5
Service

      ↓

DAY 9
Historical Event
```

Tapi satu hal penting:

Karena ini adalah **SOC Analyst Test Service** yang ada di labmu, kita tidak boleh memperlakukannya sebagai bukti malware sungguhan.

---
#### ❌ Q5 — ExecutablePath vs CommandLine

Kamu sebelumnya sudah beberapa kali mendekati benar, tetapi masih mencampurkannya.

Ingat:

```text
ExecutablePath → WHERE?
CommandLine    → HOW?
```

Contoh:

```text
ExecutablePath:
C:\Windows\System32\powershell.exe
```

CommandLine:

```text
powershell.exe -File C:\Temp\test.ps1
```

`CommandLine` bisa mengandung path executable, **tetapi definisinya bukan "ExecutablePath + argument"**.

---
#### ❌ Q6 — Service vs Process

Masih salah.

Kamu menjawab sebelumnya:

> service itu windows manager OS
> process itu terhubungan dengan service

Yang benar:

> **Service adalah background component managed by Windows Service Control Manager.**

> **Process adalah instance program yang sedang berjalan.**

Contoh:

```text
Dhcp Service
     ↓
ProcessId 2300
     ↓
svchost.exe
```

Jangan pakai lagi:

```text
Service menghasilkan Process
```

---
#### ❌ Q9 — Digital Signature

Kamu masih menyebut:

> valid atau tidaknya process

Yang benar:

> **Digital Signature memeriksa signature pada file/executable**, bukan menentukan process "valid".

Dan:

```text
Valid Signature
≠
Safe
```

---
#### ❌ Q14 — Security Context

Kamu menjawab:

> lupa

Tidak apa-apa.

Memory sederhana:

> **Security Context = identity + groups + privileges yang digunakan ketika suatu action/process berjalan.**

Contoh:

```text
User:
NT AUTHORITY\SYSTEM
```

memiliki security context yang berbeda dengan:

```text
DESKTOP-C7BHMKL\Buya
```

---
#### ❌ Q18 — Current State vs Historical Evidence

Kamu menjawab lupa.

Ini harus mulai jelas:

```text
Get-Process
↓
Current State
"What is running now?"
```

sedangkan:

```text
Get-WinEvent
↓
Historical Evidence
"What happened?"
```

---
### 11. Get-Process vs Event Log 🟡

Jawabanmu:

> process berisi informasi yang minim, event log bisa banyak

Ini perlu diperbaiki.

Bukan sekadar **banyak vs sedikit**.

Lebih penting:

```text
Get-Process
→ current state

Get-WinEvent
→ historical records
```

Ini harus kamu pegang.

---
### 12. TIMESTAMP 🟡

Kamu:

> bisa mengetahui kapan dibuatnya

Perbaikan:

> **Timestamp membantu menentukan kapan event terjadi/tercatat dan menyusun urutan aktivitas.**

Ini jauh lebih penting untuk SOC karena kita bisa membuat:

```text
Event A
 ↓
Event B
 ↓
Event C
```

dan mulai melihat relationship.

---
### 13. FINAL ASSESSMENT ✅

Kamu menjawab:

> harus ada investigation lebih lanjut

✅ Benar.

Dan:

> network analysis, threat intelligence

✅ Bagus sebagai next evidence.

Nanti kita akan tambahkan **process correlation, user/account, command line, parent process, dan timeline**.

---
### 📊 HASIL DAY 9

| Area                      | Status |
| ------------------------- | ------ |
| System Log                | 🟢     |
| Application Log           | 🟢     |
| Security Log              | 🟢     |
| Event ID concept          | 🟢     |
| 4625                      | 🟢     |
| 4688                      | 🟢     |
| 7045                      | 🟢     |
| Historical evidence       | 🟡     |
| Timestamp                 | 🟡     |
| Timeline                  | 🔴     |
| Event fields              | 🟡     |
| SOC investigation mindset | 🟢     |
#### 🟢 FINAL: **PASSED**

Kamu masih berada di atas batas **76%**.

---

