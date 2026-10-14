# Day 17 — Commands

**Topic:** Sysmon Event ID 22 (DNS Query), event correlation, process-instance timeline.

## 1. Commands yang benar-benar dijalankan pengguna

### Mengambil satu historical DNS event
~~~powershell
$log = 'Microsoft-Windows-Sysmon/Operational'
$dns = Get-WinEvent -FilterHashtable @{
    LogName = $log
    Id = 22
} -MaxEvents 1
~~~
Ketika memakai assignment seperti $dns = ..., hasil disimpan ke variabel dan **tidak otomatis dicetak ke layar**. Pengguna sempat bingung karena tidak melihat output; kemudian berhasil menampilkan data:

~~~powershell
$dns | Format-List TimeCreated, Id, Message
~~~

### Mengambil field terstruktur dari Event ID 22
~~~powershell
[xml]$dnsXml = $dns.ToXml()
$fields = @{}
foreach ($item in $dnsXml.Event.EventData.Data) {
    $fields[$item.Name] = $item.'#text'
}
[pscustomobject]@{
    TimeCreated = $dns.TimeCreated
    ProcessName = Split-Path $fields['Image'] -Leaf
    ProcessId = $fields['ProcessId']
    ProcessGuid = $fields['ProcessGuid']
    QueryName = $fields['QueryName']
    QueryStatus = $fields['QueryStatus']
    QueryResults = $fields['QueryResults']
} | Format-List
~~~
**Status pemahaman:** command berhasil dijalankan dengan arahan, tetapi [xml], foreach, @{}, PSCustomObject dan parsing struktur data **belum dikuasai**. Tidak boleh mengklaim bisa menulis script ini sendiri.

### Correlation berdasarkan ProcessGuid (script mentor yang dijalankan pengguna)
~~~powershell
$guid = $fields['ProcessGuid']
$events = Get-WinEvent -FilterHashtable @{
    LogName = $log
    Id = 1,3,22
} -MaxEvents 1000

$related = foreach ($event in $events) {
    [xml]$xml = $event.ToXml()
    $f = @{}
    foreach ($d in $xml.Event.EventData.Data) {
        $f[$d.Name] = $d.'#text'
    }
    if ($f['ProcessGuid'] -eq $guid) {
        [pscustomobject]@{
            Time = $event.TimeCreated
            EventID = $event.Id
            ProcessName = Split-Path $f['Image'] -Leaf
            QueryName = $f['QueryName']
            DestinationIp = $f['DestinationIp']
            DestinationPort = $f['DestinationPort']
        }
    }
}
$related | Sort-Object Time | Format-Table -AutoSize
~~~

Hasil: **47 Event ID 22**, seluruhnya dari MuMuNxMain.exe dengan QueryName www.163.com pada rentang 2026-10-13 **16:03:11–16:49:53** waktu log lokal. Tidak ditemukan ID 1 atau ID 3 untuk ProcessGuid target **di dalam 1.000 event yang dikumpulkan**. ProcessGuid asli tidak dipublikasikan.

Catatan penting: output ini tidak menampilkan ProcessGuid sebagai kolom, meski filter ProcessGuid telah diterapkan. Pencarian terbatas tidak boleh diperlakukan sebagai bukti seluruh riwayat logging.

## 2. Command sederhana yang diajarkan sebagai referensi — BELUM dieksekusi oleh pengguna
~~~powershell
Get-WinEvent -FilterHashtable @{
    LogName = 'Microsoft-Windows-Sysmon/Operational'
    Id = 1,3
} -MaxEvents 10 | Format-List TimeCreated, Id, Message
~~~
Perintah ini hanya mengambil 10 Event ID 1/3 terbaru tanpa memfilter MuMuNxMain.exe. Cocok sebagai latihan membaca event, **bukan** bukti correlation pada proses target.

## 3. Milestone command dan batas penguasaan

- Mempraktikkan Get-WinEvent untuk ID 22 dan menampilkan output dengan Format-List.
- Mempraktikkan, dengan panduan mentor, filter ProcessGuid serta sorting timeline.
- Memahami bahwa assignment variabel dapat menghasilkan output kosong di terminal meski berhasil.
- **Belum menghafal atau menguasai syntax command**, dan ini bukan penghalang PASS pada tahap konsep; latihan sintaks sederhana akan diberikan bertahap.

## Privasi
Tidak menyimpan raw hostname, user lokal, exact machine-specific ProcessGuid atau full raw log yang memuat identitas endpoint.
