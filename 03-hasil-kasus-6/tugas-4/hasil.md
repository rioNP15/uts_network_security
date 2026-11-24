# Tugas 4 (Pertemuan 4): Analisis Handshake WPA2

## Perintah yang dijalankan

```powershell
docker run --rm -v "${PWD}:/work" uts-kali-tools aircrack-ng -w wordlist-k6.txt handshake-k6.cap
```

## Output terminal (aircrack-ng)

```text
PS D:\UTS-Network-Security-Kasus6\01-sumber-lab> docker run --rm -v "${PWD}:/work" uts-kali-tools aircrack-ng -w wordlist-k6.txt handshake-k6.cap
Reading packets, please wait...
This file is not a regular 802.11 (wireless) capture.
Opening handshake-k6.cap
Read 0 packets.

No networks found, exiting.


Quitting aircrack-ng...
PS D:\UTS-Network-Security-Kasus6\01-sumber-lab>
```

## Verifikasi tambahan dengan Wireshark

Untuk memastikan masalah bukan pada perintah/container, file `handshake-k6.cap` juga dibuka langsung dengan Wireshark di host. Hasilnya:

```
The capture file appears to be damaged or corrupt.
(pcap: File has 3244555293-byte packet, bigger than maximum of 262144)
```

(lihat `screenshot/wireshark-handshake-k6-corrupt.png`)

## Analisis / Penjelasan

**Dua alat independen mengkonfirmasi file corrupt.** `aircrack-ng` (di dalam container) melaporkan `Read 0 packets` / file bukan capture 802.11 yang valid, dan **Wireshark** (dibuka langsung di host, di luar Docker sama sekali) juga menolak file yang sama dengan pesan eksplisit: **file rusak**, dengan detail teknis bahwa struktur pcap mengklaim ada paket berukuran **3.244.555.293 byte (~3,2 GB)**, jauh melebihi batas maksimum pcap sebesar 262.144 byte per paket.

**Penyebab teknis.** Nilai ukuran paket sebesar itu (~3,2 GB) pada satu frame pcap hampir pasti menandakan **header pcap rusak** — kemungkinan beberapa byte di awal file (yang menyimpan panjang paket) berubah/hilang/corrupt saat proses transfer, kompresi, atau penyalinan file, sehingga parser pcap (baik oleh Wireshark maupun library yang dipakai aircrack-ng) membaca nilai length field yang tidak masuk akal dan langsung menolak keseluruhan file sebagai tidak valid.

**Kesimpulan:** masalah ada pada **file `handshake-k6.cap` itu sendiri** (bukan pada command, mounting Docker, maupun langkah yang dijalankan), karena dua tool yang sama sekali berbeda cara kerjanya (`aircrack-ng` via Docker dan Wireshark native di Windows) memberikan kesimpulan yang konsisten: file corrupt dan tidak bisa dibaca sebagai capture 802.11.

**Langkah yang sudah dilakukan:**
1. Menjalankan `aircrack-ng` sesuai perintah kasus — gagal dengan `Read 0 packets`.
2. Memverifikasi file secara independen dengan Wireshark — mengonfirmasi file corrupt dengan pesan error spesifik (ukuran paket melebihi batas maksimum pcap).
3. **Melapor ke penguji** bahwa file `handshake-k6.cap` untuk Kasus 6 corrupt, disertai bukti dari dua tool berbeda, dan meminta file pengganti.

## Penjelasan 4 Langkah WPA2 4-Way Handshake

Meski file capture tidak dapat diproses, berikut penjelasan konseptual proses yang seharusnya dianalisis:

1. **Message 1 (AP → Client):** AP mengirim **ANonce** (nilai acak dari AP). Client kini punya cukup informasi (PMK + ANonce + SNonce yang akan dibuat + MAC address kedua pihak) untuk menghitung PTK (Pairwise Transient Key).
2. **Message 2 (Client → AP):** Client mengirim **SNonce** beserta **MIC** (Message Integrity Code) yang dihitung dari PTK. AP kini juga bisa menghitung PTK-nya sendiri dan memverifikasi MIC ini.
3. **Message 3 (AP → Client):** Setelah MIC terverifikasi, AP mengirim **GTK** (Group Temporal Key, untuk trafik broadcast/multicast) terenkripsi PTK, beserta MIC-nya sendiri.
4. **Message 4 (Client → AP):** Client mengirim **ACK** konfirmasi instalasi kunci berhasil, menandakan handshake selesai dan kedua pihak siap mengenkripsi trafik data dengan PTK/GTK.

Message 2 (SNonce + MIC) adalah bagian yang biasanya ditangkap penyerang untuk serangan dictionary/brute-force offline seperti yang dilakukan `aircrack-ng`, karena MIC diturunkan dari PMK yang berasal dari passphrase — sehingga kandidat passphrase bisa dicoba offline tanpa terus terhubung ke jaringan target. Pada kasus ini, langkah cracking tidak dapat dieksekusi karena file capture yang tersedia corrupt.

## Screenshot

- `screenshot/terminal-aircrack-k6.png` — output aircrack-ng
- `screenshot/wireshark-handshake-k6-corrupt.png` — konfirmasi file corrupt dari Wireshark