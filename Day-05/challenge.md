Challenge: Windows Services & Process Context

## Challenge Case

Ditemukan sebuah Windows Service dengan informasi berikut:

```text
Service Name : WindowsUpdater
Display Name : Windows Update Helper
State        : Running
StartMode    : Auto
StartName    : LocalSystem
ProcessId    : 4216
PathName     : C:\Users\Buya\AppData\Local\Temp\updater.exe
```

Process yang terkait:

```text
Process Name : updater.exe
PID          : 4216
PPID         : 5500
CommandLine  : updater.exe -silent
ExecutablePath : C:\Users\Buya\AppData\Local\Temp\updater.exe
```

Parent process:

```text
PID          : 5500
ProcessName  : svchost.exe
```

---

## Questions & My Answers

### Q1. Apa yang kamu lihat dari kasus tersebut?

**Jawaban saya:**

Saya melihat service sedang berjalan dan menggunakan updater.exe di folder Temp.

**Koreksi:**

Observasi tersebut benar, tetapi harus lebih lengkap dan dipisahkan dari kesimpulan.

Evidence yang terlihat:

* Service `WindowsUpdater` sedang `Running`.
* `StartMode` adalah `Auto`.
* Service menggunakan account `LocalSystem`.
* Service terkait dengan process PID `4216`.
* `PathName` menunjuk ke `updater.exe` di folder `Temp`.
* Process `updater.exe` memiliki PPID `5500`.
* Command line adalah `updater.exe -silent`.

Folder `Temp` merupakan **indicator yang perlu diperiksa**, tetapi belum cukup untuk menyimpulkan malicious.

---

### Q2. Apa arti ProcessId 4216 pada Service?

**Jawaban saya:**

WindowsUpdater menghasilkan PID 4216.

**Koreksi:**

Lebih tepat:

> Service `WindowsUpdater` memiliki `ProcessId 4216`, sehingga kita dapat mencari process dengan PID 4216.

`4216` adalah PID milik **process**, bukan ID milik service.

---

### Q3. Apa arti StartName = LocalSystem?

**Jawaban saya:**

LocalSystem adalah yang menjalankan service.

**Koreksi:**

> `StartName = LocalSystem` berarti service berjalan menggunakan **account LocalSystem**.

`StartName` menunjukkan service account, bukan nama service dan bukan user yang sedang login.

---

### Q4. Apakah Running + Auto berarti malicious?

**Jawaban saya:**

Tidak tahu.

**Koreksi:**

Benar.

* `Running` → service sedang aktif.
* `Auto` → service dikonfigurasi untuk otomatis dimulai.

Kedua informasi tersebut **bukan verdict keamanan**.

---

### Q5. Apa evidence yang perlu diperiksa?

**Jawaban saya:**

* Network traffic
* Command line
* Executable path
* Metadata
* Digital signature
* Hash

**Koreksi:**

Benar. Evidence tersebut dapat digunakan untuk memperkuat investigasi.

Tambahan yang relevan:

* Parent process
* Service account
* Windows Event Logs
* Threat Intelligence
* File creation/modification time

---

## Lesson Learned

Dalam SOC investigation:

```text
Observation
    ↓
Evidence
    ↓
Analysis
    ↓
Conclusion
```

Jangan langsung:

```text
Suspicious indicator
    ↓
Malware
```

Karena:

> **Unusual ≠ Malicious**
>
> **Indicator ≠ Verdict**
