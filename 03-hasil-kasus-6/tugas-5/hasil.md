# Tugas 5 (Pertemuan 5): ICMP dengan Scapy

## Perintah yang dijalankan

```powershell
docker inspect korban-k6 | Select-String "IPAddress"
docker run --rm --network lab-arp-k6 -v "${PWD}/kasus/k5-scapy:/work" uts-kali-tools python3 /work/kirim_icmp.py 172.18.0.4
```

## Output terminal

```text
PS D:\UTS-Network-Security-Kasus6\01-sumber-lab> docker inspect korban-k6 | Select-String "IPAddress"

                    "IPAddress": "172.18.0.4",

PS D:\UTS-Network-Security-Kasus6\01-sumber-lab> docker run --rm --network lab-arp-k6 -v "${PWD}/kasus/k5-scapy:/work" uts-kali-tools python3 /work/kirim_icmp.py 172.18.0.4
Mengirim: IP / ICMP 172.18.0.5 > 172.18.0.4 echo-request 0
TTL balasan: 64
Tipe ICMP balasan: 0
PS D:\UTS-Network-Security-Kasus6\01-sumber-lab>
```

## Analisis / Penjelasan

Script `kirim_icmp.py` membuat paket `IP(dst=172.18.0.4)/ICMP()` menggunakan Scapy, dikirim dari container sementara (`172.18.0.5`, IP yang otomatis dialokasikan Docker untuk container `--rm` ini) ke container `korban-k6` (`172.18.0.4`), menggunakan `sr1()` (send-and-receive-1, menunggu satu balasan dengan timeout 2 detik).

Hasilnya, balasan **berhasil diterima**:
- **TTL balasan = 64**: nilai Time To Live standar untuk host Linux yang belum melewati router/hop (TTL awal Linux umumnya 64, berkurang 1 setiap melewati router — nilai 64 yang utuh menunjukkan korban berada tepat satu hop, langsung di jaringan yang sama tanpa router perantara).
- **Tipe ICMP balasan = 0**: tipe `0` adalah **Echo Reply**, balasan normal dari permintaan **Echo Request** (tipe `8`) yang dikirim — membuktikan host `korban-k6` aktif, dapat dijangkau, dan merespons ICMP dengan benar di dalam jaringan `lab-arp-k6`.

## Smurf Attack

**Smurf attack** adalah varian serangan DDoS berbasis ICMP yang memanfaatkan *IP spoofing* dan *broadcast amplification*. Penyerang mengirim paket **ICMP Echo Request** ke alamat **broadcast** suatu jaringan (misalnya `192.168.1.255`), dengan **source IP address yang di-spoof** menjadi alamat IP korban yang sebenarnya. Karena dikirim ke alamat broadcast, **semua host** dalam jaringan tersebut menerima request itu dan masing-masing mengirim **ICMP Echo Reply** — tetapi karena source IP sudah dipalsukan, semua balasan itu justru dikirim ke **korban**, bukan ke penyerang. Jika jaringan broadcast punya banyak host (disebut *smurf amplifier*), korban akan dibanjiri balasan ICMP dalam jumlah jauh lebih besar dari paket asli yang dikirim penyerang — inilah efek *amplifikasi* yang membuat smurf attack efektif sebagai serangan DDoS meski penyerang hanya mengirim sedikit trafik.

**Mitigasi umum:** menonaktifkan *IP-directed broadcast* di router/gateway, serta menerapkan *ingress/egress filtering* (BCP 38) untuk memblokir paket dengan source IP yang di-spoof.

## Screenshot

Lihat folder `screenshot/` untuk bukti visual.