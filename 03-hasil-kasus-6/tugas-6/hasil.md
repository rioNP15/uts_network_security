# Tugas 6 (Pertemuan 6): SYN Flood & Three-Way Handshake — Kasus 6

## Perintah yang dijalankan

```powershell
docker compose -p k6 -f b/b3-syn-flood/docker-compose.yml up -d

# Baseline SYN_RECV (sebelum serangan)
docker compose -p k6 -f b/b3-syn-flood/docker-compose.yml exec target netstat -ant | Select-String "SYN_RECV" | Measure-Object | Select-Object -ExpandProperty Count

# Lancarkan SYN flood (background, 15 detik)
docker compose -p k6 -f b/b3-syn-flood/docker-compose.yml exec -d attacker timeout 15 hping3 -S -p 80 --flood target

# SYN_RECV saat serangan
docker compose -p k6 -f b/b3-syn-flood/docker-compose.yml exec target netstat -ant | Select-String "SYN_RECV" | Measure-Object | Select-Object -ExpandProperty Count

# Capture traffic dengan tcpdump
docker compose -p k6 -f b/b3-syn-flood/docker-compose.yml exec attacker sh -c 'timeout 10 tcpdump -i eth0 -nn tcp port 80 & sleep 1; curl -s http://target > /dev/null; sleep 10'

docker compose -p k6 -f b/b3-syn-flood/docker-compose.yml down -v
```

> Catatan teknis: `grep` tidak tersedia secara native di PowerShell, sehingga diganti dengan `Select-String` yang dikombinasikan dengan `Measure-Object` untuk menghitung jumlah baris yang cocok (setara dengan `grep -c`).

## Output terminal

**Baseline (sebelum serangan):**
```
0
```

**Saat serangan (setelah flood dilepas):**
```
1
```

**Capture tcpdump (saat curl dijalankan bersamaan):**
```
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on eth0, link-type EN10MB (Ethernet), snapshot length 262144 bytes
13:37:31.822105 IP 172.19.0.2.59884 > 172.19.0.3.80: Flags [S], seq 2492389743, win 64240, options [mss 1460,sackOK,TS val 1408519519 ecr 0,nop,wscale 10], length 0
13:37:31.822154 IP 172.19.0.3.80 > 172.19.0.2.59884: Flags [S.], seq 518990461, ack 2492389744, win 65160, options [mss 1460,sackOK,TS val 1625370650 ecr 1408519519,nop,wscale 10], length 0
13:37:31.822164 IP 172.19.0.2.59884 > 172.19.0.3.80: Flags [.], ack 1, win 63, options [nop,nop,TS val 1408519519 ecr 1625370650], length 0
13:37:31.822288 IP 172.19.0.2.59884 > 172.19.0.3.80: Flags [P.], seq 1:71, ack 1, win 63, options [nop,nop,TS val 1408519519 ecr 1625370650], length 70: HTTP: GET / HTTP/1.1
13:37:31.822319 IP 172.19.0.3.80 > 172.19.0.2.59884: Flags [.], ack 71, win 64, options [nop,nop,TS val 1625370650 ecr 1408519519], length 0
13:37:31.824597 IP 172.19.0.3.80 > 172.19.0.2.59884: Flags [P.], seq 1:239, ack 71, win 64, options [nop,nop,TS val 1625370653 ecr 1408519519], length 238: HTTP: HTTP/1.1 200 OK
13:37:31.824650 IP 172.19.0.2.59884 > 172.19.0.3.80: Flags [.], ack 239, win 63, options [nop,nop,TS val 1408519522 ecr 1625370653], length 0
13:37:31.825053 IP 172.19.0.3.80 > 172.19.0.2.59884: Flags [P.], seq 239:1135, ack 71, win 64, options [nop,nop,TS val 1625370653 ecr 1408519522], length 896: HTTP
13:37:31.825079 IP 172.19.0.2.59884 > 172.19.0.3.80: Flags [.], ack 1135, win 63, options [nop,nop,TS val 1408519522 ecr 1625370653], length 0
13:37:31.825143 IP 172.19.0.2.59884 > 172.19.0.3.80: Flags [F.], seq 71, ack 1135, win 63, options [nop,nop,TS val 1408519522 ecr 1625370653], length 0
13:37:31.825241 IP 172.19.0.3.80 > 172.19.0.2.59884: Flags [F.], seq 1135, ack 72, win 64, options [nop,nop,TS val 1625370653 ecr 1408519522], length 0
13:37:31.825253 IP 172.19.0.2.59884 > 172.19.0.3.80: Flags [.], ack 1136, win 63, options [nop,nop,TS val 1408519522 ecr 1625370653], length 0

12 packets captured
12 packets received by filter
0 packets dropped by kernel
```

## Analisis / Penjelasan

### 1. Perubahan jumlah koneksi `SYN_RECV`

Sebelum serangan, jumlah koneksi dengan status `SYN_RECV` pada container `target` adalah **0** — kondisi normal, tidak ada *half-open connection* yang tertahan. Setelah `hping3 -S -p 80 --flood` dilepaskan dari container `attacker`, jumlahnya naik menjadi **1**.

Kenaikan ini memang kecil dibanding efek SYN flood pada jaringan nyata (yang bisa mencapai ratusan/ribuan). Ada beberapa alasan teknis yang masuk akal untuk lab Docker lokal seperti ini:

- **Throughput internal Docker bridge network sangat tinggi** — paket SYN palsu diproses dan di-"reset"/dibuang oleh stack TCP target jauh lebih cepat dibanding pada jaringan WAN nyata, sehingga backlog tidak sempat menumpuk banyak dalam window pengecekan yang singkat.
- **Window pengecekan sangat sempit** — perintah `netstat` dijalankan sesaat setelah flood dilepas, hanya menangkap satu *snapshot* kondisi backlog pada momen tertentu, bukan rata-rata selama 15 detik serangan berlangsung.
- **Nilai default `net.ipv4.tcp_max_syn_backlog`** pada container target kemungkinan masih memungkinkan backlog diproses/dibersihkan dengan cepat karena tidak ada beban traffic lain yang bersaing.

Meski kenaikannya kecil, hasil ini **tetap merupakan bukti valid** bahwa SYN flood berhasil menimbulkan minimal satu koneksi *half-open* (`SYN_RECV`) yang sebelumnya tidak ada — menunjukkan prinsip serangan bekerja, walau skala dampaknya terbatas oleh karakteristik lab lokal.

### 2. Hasil capture `tcpdump`

Capture `tcpdump` yang dijalankan bersamaan dengan `curl http://target` menangkap **12 paket** yang justru menunjukkan *three-way handshake yang lengkap dan normal*, bukan paket dari serangan flood:

```
SYN       → 172.19.0.2:59884 → 172.19.0.3:80   (langkah 1: SYN)
SYN-ACK   ← 172.19.0.3:80    → 172.19.0.2:59884 (langkah 2: SYN-ACK)
ACK       → 172.19.0.2:59884 → 172.19.0.3:80    (langkah 3: ACK)
```

diikuti dengan pertukaran data HTTP (`GET /` → `HTTP/1.1 200 OK`) dan penutupan koneksi normal (`FIN`/`ACK`).

Ini sebenarnya adalah contoh **three-way handshake yang berhasil sempurna** — bagus sebagai pembanding konsep dengan kondisi serangan. Namun capture ini **tidak menangkap paket SYN flood dari `hping3`**. Kemungkinan penyebabnya:

- `hping3 -S -p 80 --flood target` dijalankan dengan `timeout 15`, dan ada jeda waktu (karena mengetik ulang perintah secara manual di PowerShell) sebelum perintah `tcpdump` benar-benar dieksekusi — kemungkinan window 15 detik sudah habis sebelum capture dimulai.
- `tcpdump` berjalan di sisi **attacker**, bukan di sisi **target**; jika paket flood dikirim lebih cepat dari *buffer* capture, atau antarmuka yang dipantau tidak sinkron dengan traffic flood yang sudah selesai, traffic tersebut tidak akan muncul di capture ini.

**Catatan kejujuran laporan:** capture ini valid sebagai bukti mekanisme three-way handshake normal, tetapi tidak membuktikan langsung paket flood. Bukti paket flood sudah terwakili secara tidak langsung oleh perubahan `SYN_RECV` (0 → 1) pada langkah sebelumnya.

## Penjelasan Konsep: Three-Way Handshake vs SYN Flood

**Three-way handshake normal** (SYN → SYN-ACK → ACK) terlihat jelas pada capture `tcpdump` di atas: client mengirim SYN, server membalas SYN-ACK sambil mengalokasikan entri di *backlog queue* (status `SYN_RECV`), lalu client menyelesaikan dengan ACK — barulah koneksi dianggap `ESTABLISHED` dan backlog dibersihkan.

**SYN flood** mengeksploitasi langkah kedua: penyerang mengirim banyak paket SYN (biasanya dengan source IP acak/spoofed) tanpa pernah mengirim ACK penutup. Server tetap mengalokasikan entri backlog untuk setiap SYN yang diterima dan menunggu ACK yang tidak akan pernah datang, hingga backlog penuh dan server menolak koneksi baru yang sah — inilah bentuk *Denial of Service*.

## Mitigasi: SYN Cookies

Mitigasi standar untuk SYN flood adalah **SYN cookies** (`net.ipv4.tcp_syncookies = 1` di Linux). Alih-alih menyimpan status koneksi di backlog saat menerima SYN, server mengenkripsi informasi koneksi (sequence number, MSS, dsb.) ke dalam *sequence number* balasan SYN-ACK itu sendiri, tanpa mengalokasikan memori backlog. Jika client membalas dengan ACK yang valid, server bisa merekonstruksi state koneksi dari ACK number tersebut tanpa pernah menyimpan state sebelumnya — sehingga serangan SYN flood (yang tidak pernah mengirim ACK balik) tidak lagi menghabiskan resource backlog server.

Kelemahan SYN cookies: beberapa opsi TCP (seperti window scaling penuh) tidak bisa disimpan dalam cookie, sehingga performa koneksi sedikit berkurang dibanding koneksi normal — namun ini trade-off yang wajar dibanding downtime akibat DoS.

## Screenshot
Lihat folder `screenshot/` untuk bukti visual.