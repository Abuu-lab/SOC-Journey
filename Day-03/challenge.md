CHALLENGE

Sekarang saya beri kamu kasus.

Kamu sedang melakukan investigation terhadap:

unknown.exe
PID: 4216
CPU: 92%
Memory: 180 MB

Kamu menjalankan PowerShell dan mendapatkan:

Name: unknown.exe
ProcessId: 4216
ParentProcessId: 5500

ExecutablePath:
C:\Users\Buya\AppData\Local\Temp\unknown.exe

CommandLine:
unknown.exe -update

Kemudian:

PID 5500
Name: firefox.exe
Pertanyaan:

1. Apa yang kamu ketahui dari data tersebut?
kemungkinan berbahaya, ini hanya kecurigaan awal. soalnya kalau PPID nya firefox harusnya PIDnya firefox juga , tapi ini unknown.exe. okelah anggap kita positif think. tapi yg jadi masalah itu executablenya kok di "temp" yang notabene temp itu kan folder sampah yang umumnya ke hidden dan jarang dibuka user karena emg isinya sampah dan bawaan dasarnya hidden. ini jadi muncul pertanyaan kenapa berada disana
2. Apa yang masih belum kamu ketahui?
mungkin aku belum tau evidence lain yang bisa menjadi bukti untuk meyakinkanku bahwa itu file berbahaya atau bukan. mungkin bisa aku coba check melalui resmon untuk melihat traffic data
3. Apakah kamu boleh langsung menyimpulkan unknown.exe malicious? Jelaskan.
kalau dalam pembelajaranku di day 2, karena belum meluas cara pandangku terhadap banyaknya evidence. aku menyimpulkan 70% itu malicious, karena terlihat dari executable pathnya yg gk masuk akal berada di "temp" sedangkan parent id nya aja firefox
4. Apa hubungan antara Firefox dan unknown.exe berdasarkan data tersebut?
kyknya unknown.exe itu file diluar firefox executable firefox umumnya bukan di "temp" temp itu tempat sampah tapi firefox punya PID unknown.exe jadi timbul pertanyaan. 
5. Evidence apa yang ingin kamu cari berikutnya?
Kemungkinan sya mau liat traffic data menggunakan Resmon, trs CommandLine, atau kyk license ya? privasi policy? intinya kyk ngecheck keasliannya. gk tau nama metodenya apa belum smpai sana, bisa lihat commandline juga kita bisa tau perintah apa saja yg dijalankan si unknown.exe itu
Minimal 3 evidence tambahan.

