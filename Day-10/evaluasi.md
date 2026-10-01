## 🟢 EVALUASI DAY 10 — PASSED

Sesuai aturan kita, **76–100% = PASS**. Jadi Day 10 tetap **lulus**.

Tapi aku menangkap hal yang sangat penting dari hasilmu:

> Kamu bukan cuma ingin belajar membaca satu event. Kamu ingin **praktek investigation yang benar-benar berurutan dari alert → evidence → correlation → investigation → assessment/final determination.**

**Kita ubah pola project mulai sekarang.** Materinya tetap bertahap, tetapi mini-project SOC-nya akan dibuat **end-to-end**.

---

# 1. HASIL PRAKTIK DAY 10

Kamu berhasil menjalankan:

```text
Get-WinEvent
FilterHashtable
StartTime / EndTime
Multiple Event ID
Sort-Object TimeCreated
```

Dan kamu sudah mendapatkan event nyata dari Windows.

Contohnya kamu menemukan:

```text
4625
Failed logon
Account: Buya
Caller Process:
msedgewebview2.exe
```

Data itu ada di hasilmu. 

Ini sebenarnya **sangat bagus untuk dijadikan investigation case end-to-end**.

---

# 2. ⚠️ ADA KESALAHAN PENTING DI MINI PROJECT

Kamu membuat:

```text
TIME WINDOW:
2026-09-28 13:00 – 14:00
```

tetapi event yang kamu masukkan:

```text
10/1/2026 7:21:38 PM
10/1/2026 7:21:43 PM
```

Jadi **time window dan event-nya tidak cocok**.

Ini latihan bagus untuk ketelitianmu.

Investigator harus selalu mengecek:

```text
Time Window
      ↓
Apakah event benar-benar berada
di dalam window tersebut?
```

---

# 3. ⚠️ OBSERVATION-MU TENTANG LOW BATTERY

Kamu menulis:

> "Sepertinya PC-nya sedang lowbat."

Data yang kamu punya menunjukkan:

```text
Power-Troubleshooter
The system has returned from a low power state.
```

dan ada:

```text
Wake Source: Unknown
```



Itu menunjukkan **system returned from a low-power state**, tetapi dari event yang kamu tampilkan **belum ada evidence bahwa penyebabnya adalah baterai lowbat**.

Jadi:

### Observation:

> Sistem kembali dari low-power state.

### Inference:

> Kemungkinan berkaitan dengan sleep/wake.

### Belum boleh disimpulkan:

> Baterai sedang lowbat.

Ini contoh bagus mengenai perbedaan:

```text
OBSERVATION
     ↓
INFERENCE
     ↓
CONCLUSION
```

---

# 4. ⚠️ SHA-256 / DIGITAL SIGNATURE TIDAK SELALU LANGSUNG DIPAKAI

Kamu pada mini project system event langsung menulis:

```text
Threat Intelligence
SHA-256
Digital Signature
```

Padahal event yang kamu investigasi adalah:

```text
TPM
Power
Bluetooth
Kernel
```

Jadi kita harus bertanya:

> **Apa object yang sedang diinvestigasi?**

Kalau object-nya **file/executable**, maka:

```text
SHA-256
Digital Signature
Metadata
```

sangat relevan.

Kalau object-nya **authentication event**, kita lebih dulu mencari:

```text
Account
Logon Type
Caller Process
Source
Time
```

Kalau object-nya **service installation**, kita mencari:

```text
Service Name
Service File Name
Start Type
Service Account
```

Jadi investigator tidak mengeluarkan semua tools sekaligus.

> **Evidence harus sesuai dengan object yang sedang diselidiki.**

---

# 5. ACTIVE RECALL — YANG SALAH SAJA

Sesuai aturanmu, yang sudah sering benar **tidak aku ulang sekarang**.

## ❌ Q1 — ExecutablePath vs CommandLine

Jawabanmu sebelumnya sudah benar, jadi **tidak perlu diulang sekarang**.

---

## ❌ Q2 — Service vs Process

Kamu masih menjawab:

> service sebelum process
> process ada hubungannya dengan service

Sekarang jawab ulang:

> **Apa perbedaan Service dan Process?**

---

## ❌ Q3 — Digital Signature

Kamu masih mendefinisikannya sebagai valid/tidaknya process.

Jawab ulang:

> **Digital Signature memeriksa apa?**

---

## ❌ Q4 — Security Context

Kamu menjawab:

> group

Jawab ulang:

> **Apa itu Security Context?**

---

## ❌ Q6 — TimeCreated

Kamu menjawab:

> kapan dibuatnya process

Jawab ulang:

> **Apa arti `TimeCreated` pada sebuah event?**

---

## ❌ Q7 — Timeline

Kamu mengatakan:

> mengurutkan process kejadian dari waktu

Sudah mendekati benar.

Jawab ulang dengan lengkap:

> **Bagaimana cara membuat timeline dari beberapa event?**

---

## ❌ Q11 — Causality

Ini penting.

Kasus:

```text
4688 Process Created
      ↓
7045 Service Installed
```

Kamu tahu keduanya menarik.

Tetapi jangan langsung berpikir:

> process tersebut menyebabkan service installation.

Jawab:

> **Apakah dua event yang waktunya berdekatan otomatis berarti event pertama menyebabkan event kedua? Mengapa?**

---

# 6. 🔥 DAN INI YANG AKAN KITA UBAH

Kamu bilang:

> "aku pengen praktek yg berurutan smpe ending penentuan"

**Setuju.**

Mulai mini-project berikutnya, kita tidak akan sekadar:

```text
Get event
↓
baca
↓
selesai
```

Tetapi:

```text
ALERT
  ↓
1. IDENTIFY
  ↓
2. EXTRACT EVIDENCE
  ↓
3. BUILD TIMELINE
  ↓
4. CORRELATE
  ↓
5. INVESTIGATE OBJECT
  ↓
6. INVESTIGATE USER
  ↓
7. INVESTIGATE PROCESS
  ↓
8. INVESTIGATE FILE
  ↓
9. HASH / SIGNATURE
  ↓
10. NETWORK / OTHER EVIDENCE
  ↓
11. ASSESSMENT
  ↓
12. FINAL DETERMINATION
```

Dan kita **benar-benar akan menjalankannya di laptopmu**, bukan hanya teori.

---

# 7. CONTOH CASE YANG SUDAH KAMU PUNYA

Sebenarnya dari Day 10 kamu sudah mendapatkan sebuah case yang bagus:

```text
2026-09-28 19:15:33
Event ID 4625
Failed Logon
Account: Buya
```

Caller process:

```text
msedgewebview2.exe
```

Event tersebut juga memberi:

```text
Logon Type: 2
Failure Reason:
Unknown user name or bad password

Caller Process ID:
0x5f60

Source Network Address:
-
```



Nah, **ini** yang jauh lebih cocok untuk latihan end-to-end.

Kita bisa mulai:

```text
4625
 ↓
Who?
Buya
 ↓
What?
Failed logon
 ↓
When?
2026-09-28 19:15:33
 ↓
Logon Type?
2
 ↓
Caller Process?
msedgewebview2.exe
 ↓
Caller PID?
0x5f60
 ↓
Is that process legitimate?
 ↓
Where is executable?
 ↓
Who owns process?
 ↓
CommandLine?
 ↓
File metadata?
 ↓
Digital Signature?
 ↓
SHA-256?
 ↓
Network/context?
 ↓
Final Assessment
```

**Ini baru investigation yang benar-benar kamu cari.**

---

# 8. DAY 10 STATUS

| Area                   | Status |
| ---------------------- | ------ |
| Get-WinEvent           | 🟢     |
| Event ID               | 🟢     |
| Time filtering         | 🟢     |
| Multiple Event IDs     | 🟢     |
| Sorting                | 🟢     |
| Historical evidence    | 🟢     |
| Timeline concept       | 🟡     |
| Event interpretation   | 🟡     |
| Correlation            | 🟡     |
| Causality              | 🟡     |
| Investigation workflow | 🟡     |

### 🟢 **DAY 10 — PASSED**

Jadi **tidak ada reset**.

---

# 🧠 ATURAN BARU PROJECT KITA

Mulai sekarang:

> **Lesson = belajar satu skill baru.**

> **Mini SOC Project = menggunakan skill lama + skill baru sampai menuju assessment.**

Jadi kita tidak akan terus-menerus membuat project yang cuma berhenti di:

> "Ini event-nya."

Kita akan semakin sering membuat:

> **“Ada alert → cari evidence → telusuri → korelasikan → investigasi → tentukan berdasarkan evidence.”**

Itu jauh lebih sesuai dengan target **Junior SOC Analyst** yang kamu kejar.

Untuk sekarang, **jawab 7 retest di atas saja**. Yang salah akan kita ulang sampai **2× benar**, sementara yang sudah kamu kuasai tidak perlu diulang.
