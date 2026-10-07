# Day 02 — Materi

**Performance Baseline**

### Konsep utama

- **Performance baseline** adalah catatan kondisi penggunaan resources untuk dibandingkan dengan kondisi lain.
- Bandingkan CPU, memory, dan disk saat **idle** dan **active**, dengan mempertimbangkan workload aplikasi.
- Satu application dapat memiliki beberapa processes; nama sama tidak otomatis berarti malware.
- **PID** menghubungkan process dengan data lain, termasuk network connection.
- Resource Monitor dan netstat membantu melihat hubungan process ↔ PID ↔ connection.

### Praktik

Bandingkan penggunaan resources Discord dan Spotify, lalu catat perubahan dan evidence tambahan yang diperlukan. Hasil mini-project, challenge, self-test, dan koreksi tersedia di [Evaluasi](evaluasi.md).

### Alur investigasi

Observation → hypothesis → evidence → analysis → conclusion. Executable path, parent, command line, user, dan network context membantu menguji hypothesis.
