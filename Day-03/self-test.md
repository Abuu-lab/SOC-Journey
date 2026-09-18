SELF TEST

Jawab tanpa melihat materi di atas sebisa mungkin.

1. Apa perbedaan Application dan Process?
aplikasi adalah program itu sendiri, process adalah informasi data program yang berjalan

2. Apa fungsi PID? untuk memudahkan kita mengidentifikasi, karena contoh firefox memiliki childproses yg bnyak itulah fungsi PID karena angkanya berbeda dan memudahkan kita mengidentifikasi

3. Apakah PID bisa menentukan sebuah process malicious atau tidak? tidak, itu hanya untuk memudahkan kita mengidentifikasi

4. Apa yang dimaksud Executable? file yang bisa dijalankan

5. Mengapa Executable Path penting dalam investigation? untuk mengetahui apakah aplikasi itu masuk akal jika disimpan disana .

6. Apa yang dimaksud Parent Process? proses Utama dan dia memmiliki childproses

7. Apa hubungan PID dan PPID? PPID adalah mainproses PID adalah anak proses

8. Mengapa Command Line dapat lebih berguna daripada hanya melihat Process Name? ya karena kita bisa melihat aktivitas didalam aplikasi tersebut atau aktivitas PID yang kita periksa. apakah normal atau tidak

9. Jika:

unknown.exe
PID 4216
PPID 5500

apa yang ingin kamu cari dari PID 5500?
Executable path apakah berhubungan dengan PID 4216 atau tidak biasanya Parentproses dan childproses executablepathnya hamper sama
lalu Commandline 

10. Jika menemukan:

firefox.exe
   ↓
unknown.exe

apakah otomatis malicious? Mengapa?
patut dicurigai, tapi tetap kita harus check 
dari sisi commanline,executable path
agar lebih meyakinkan
