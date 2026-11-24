# Tugas 3 (Pertemuan 3): ARP Spoofing

## Catatan environment

Image `uts-kali-tools` tidak menyertakan `net-tools` maupun `iproute2` secara default — hanya tool yang disebut eksplisit di Dockerfile (hping3, dnsutils, xxd, dsniff, aircrack-ng, python3-scapy, netcat-openbsd, tcpdump, curl). Akibatnya perintah asli `arp -a` dan `arp -s` di lembar soal tidak bisa dipakai langsung. Sebagai gantinya:
- Tabel ARP dibaca langsung dari `/proc/net/arp` (file kernel, selalu tersedia tanpa paket tambahan)
- Untuk memicu ARP resolution awal (pengganti `ping`, yang juga tidak tersedia), dipakai `nc -zv` ke port korban/gateway
- Untuk memasang static ARP (mitigasi), paket `iproute2` di-install manual di container `korban-k6`, lalu dipakai `ip neigh replace` sebagai pengganti `arp -s`

## Perintah yang dijalankan

```powershell
# Setup
docker network create lab-arp-k6
docker run -d --name korban-k6 --network lab-arp-k6 --cap-add NET_ADMIN uts-kali-tools
docker run -d --name gateway-k6 --network lab-arp-k6 --cap-add NET_ADMIN uts-kali-tools
docker run -d --name penyerang-k6 --network lab-arp-k6 --cap-add NET_ADMIN uts-kali-tools

# Cek IP
docker inspect korban-k6 gateway-k6 | ConvertFrom-Json | ForEach-Object { $_.NetworkSettings.Networks.'lab-arp-k6'.IPAddress }

# Pancing ARP resolution awal (pengganti ping, tidak tersedia di image)
docker exec korban-k6 nc -zv -w 2 172.18.0.3 80

# ARP sebelum spoofing
docker exec korban-k6 cat /proc/net/arp

# Jalankan ARP spoofing (20 detik)
docker exec -d penyerang-k6 timeout 20 arpspoof -i eth0 -t 172.18.0.2 172.18.0.3

# ARP saat spoofing
docker exec korban-k6 cat /proc/net/arp

# Ambil MAC asli gateway
docker exec gateway-k6 cat /sys/class/net/eth0/address

# Pasang mitigasi static ARP
docker exec korban-k6 apt-get update
docker exec korban-k6 apt-get install -y iproute2
docker exec korban-k6 ip neigh replace 172.18.0.3 lladdr 9a:1f:7e:be:d8:26 dev eth0

# Verifikasi ARP setelah mitigasi
docker exec korban-k6 cat /proc/net/arp

# Uji ulang spoofing untuk membuktikan mitigasi bekerja
docker exec -d penyerang-k6 timeout 20 arpspoof -i eth0 -t 172.18.0.2 172.18.0.3
docker exec korban-k6 cat /proc/net/arp
```

## Output terminal

### IP address korban dan gateway
```
korban-k6  : 172.18.0.2
gateway-k6 : 172.18.0.3
```

### MAC address asli gateway
```
9a:1f:7e:be:d8:26
```

### Tabel ARP korban — SEBELUM spoofing
```
IP address       HW type     Flags       HW address            Mask     Device
172.18.0.3       0x1         0x2         9a:1f:7e:be:d8:26     *        eth0
```

### Tabel ARP korban — SAAT spoofing berlangsung
```
IP address       HW type     Flags       HW address            Mask     Device
172.18.0.3       0x1         0x2         da:9e:7b:50:c5:b3     *        eth0
172.18.0.4       0x1         0x2         da:9e:7b:50:c5:b3     *        eth0
```

### Tabel ARP korban — SETELAH mitigasi static ARP dipasang
```
IP address       HW type     Flags       HW address            Mask     Device
172.18.0.3       0x1         0x6         9a:1f:7e:be:d8:26     *        eth0
172.18.0.1       0x1         0x2         16:4b:1e:7d:6c:3d     *        eth0
172.18.0.4       0x1         0x2         da:9e:7b:50:c5:b3     *        eth0
```

### Tabel ARP korban — setelah PERCOBAAN SPOOFING ULANG (mitigasi aktif)
```
IP address       HW type     Flags       HW address            Mask     Device
172.18.0.3       0x1         0x6         9a:1f:7e:be:d8:26     *        eth0
172.18.0.1       0x1         0x2         16:4b:1e:7d:6c:3d     *        eth0
172.18.0.4       0x1         0x2         da:9e:7b:50:c5:b3     *        eth0
```

## Analisis / Penjelasan

**Kenapa ARP spoofing bisa terjadi.** Protokol ARP (layer 2 OSI) menerjemahkan IP address menjadi MAC address di jaringan lokal, tetapi ARP dirancang tanpa mekanisme autentikasi. Host mana pun bisa mengirim balasan ARP (gratuitous ARP) yang mengklaim "saya adalah IP sekian, ini MAC saya", dan host lain akan menerimanya begitu saja tanpa verifikasi. `arpspoof -i eth0 -t 172.18.0.2 172.18.0.3` memanfaatkan celah ini dengan terus-menerus mengirim balasan ARP palsu ke korban (172.18.0.2), mengklaim bahwa IP gateway (172.18.0.3) memiliki MAC milik penyerang.

**Bukti serangan berhasil.** Sebelum spoofing, tabel ARP korban mencatat `172.18.0.3` → MAC `9a:1f:7e:be:d8:26` (MAC asli gateway-k6). Setelah `arpspoof` dijalankan, entry yang sama berubah menjadi `da:9e:7b:50:c5:b3` — identik dengan MAC `172.18.0.4` (penyerang-k6). Artinya korban sekarang percaya bahwa gateway memiliki MAC penyerang. Jika korban mengirim paket ke gateway, pengiriman di layer 2 akan diarahkan berdasarkan MAC, sehingga paket tersebut justru akan sampai ke penyerang terlebih dahulu — inilah fondasi serangan Man-in-the-Middle (MITM): penyerang bisa menyadap atau memanipulasi trafik sebelum (atau tanpa) meneruskannya ke tujuan asli.

**Sifat sementara dari ARP spoofing.** Karena `arpspoof` dijalankan dengan `timeout 20`, begitu proses berhenti setelah 20 detik, racun ARP berhenti dikirim dan cache ARP korban akan expired dengan sendirinya setelah beberapa waktu — entry akan pulih otomatis saat korban melakukan ARP request baru dan kali ini gateway asli yang menjawab. Ini menunjukkan ARP spoofing bukan serangan sekali-jalan yang permanen; di serangan nyata, tool seperti ini biasanya dijalankan looping terus-menerus, bukan dengan timeout.

**Mitigasi: static ARP binding.** Perintah `ip neigh replace 172.18.0.3 lladdr 9a:1f:7e:be:d8:26 dev eth0` memasang entry ARP secara statis dan permanen di kernel korban. Buktinya terlihat dari kolom Flags yang berubah dari `0x2` (entry dinamis, bisa di-overwrite oleh balasan ARP apa pun) menjadi `0x6` (kombinasi COMPLETE + PERM/permanent). Dengan flag ini, kernel menolak meng-update entry tersebut dari balasan ARP mana pun yang diterima dari jaringan. Ini dibuktikan dengan mengulangi serangan `arpspoof` setelah mitigasi terpasang: tabel ARP korban untuk `172.18.0.3` tetap menunjukkan MAC asli gateway dan flag `0x6`, sama sekali tidak terpengaruh — berbeda total dengan percobaan pertama yang langsung berubah dalam hitungan detik.

**Keterbatasan mitigasi ini.** Static ARP efektif untuk jaringan kecil dengan topologi tetap (seperti lab ini), tetapi tidak praktis diterapkan di jaringan besar dengan banyak host yang IP-nya sering berubah (DHCP), karena setiap entry harus di-maintain manual di setiap host.

## Screenshot

Lihat folder `screenshot/` untuk bukti visual.