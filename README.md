# UTS Praktik Infrastruktur & Network Attacks — Kasus 6

Repository ini merupakan dokumentasi pengerjaan Ujian Tengah Semester (UTS) Praktik Infrastruktur & Network Attacks untuk **Kasus 6, Pertemuan 1 sampai 7**.

Repository mencakup source code laboratorium, konfigurasi Docker Compose, dokumen panduan praktikum, hasil pengujian, serta screenshot sebagai bukti pengerjaan.

## Identitas Praktikum

| Informasi | Keterangan |
|---|---|
| Nama | [RIO NUGRAHA PUTRA] |
| NIM | 24552011060 |
| Kelas | [TIF RP 24D CNS] |
| Mata Kuliah | Network Security |
| Nomor Kasus | 6 |
| Cakupan Pertemuan | 1–7 |
| Lingkungan Praktikum | [Docker, windows] |

## Perhitungan Kasus

```text
NIM          : 24552011060
Digit        : 2, 4, 5, 5, 2, 0, 1, 1, 0, 6, 0
Jumlah digit : 26
Perhitungan  : 26 mod 10 = 6
Hasil        : Kasus 6
```

## Dokumentasi Utama

**[Buka Jawaban UTS Kasus 6](03-hasil-kasus-6/jawaban.md)**

File `03-hasil-kasus-6/jawaban.md` merupakan dokumentasi utama yang menggabungkan penjelasan, perintah praktikum, hasil pengujian, analisis, dan bukti pengerjaan dari Pertemuan 1 sampai Pertemuan 7.

## Struktur Repository

```text
UTS-Network-Security-Kasus6/
│
├── README.md
│
├── 01-sumber-lab/
│   ├── README.md
│   ├── images/
│   │   ├── uts-openssl/Dockerfile
│   │   ├── uts-kali-tools/Dockerfile
│   │   └── uts-dnsmasq/Dockerfile
│   ├── b/
│   │   ├── b1-compose/docker-compose.yml
│   │   ├── b3-syn-flood/docker-compose.yml
│   │   └── b4-dns-exfil/docker-compose.yml
│   └── kasus/
│       ├── k5-scapy/kirim_icmp.py
│       └── k9-dns-detect/analisis_dns.py
│
├── 02-dokumen/
│   ├── DOKUMEN-UTS-LENGKAP.md
│   └── LEMBAR-PERINTAH-PER-KASUS.md
│
└── 03-hasil-kasus-6/
    ├── jawaban.md
    ├── tugas-1/{hasil.md, screenshot/}
    ├── tugas-2/{hasil.md, screenshot/}
    ├── tugas-3/{hasil.md, screenshot/}
    ├── tugas-4/{hasil.md, screenshot/}
    ├── tugas-5/{hasil.md, screenshot/}
    ├── tugas-6/{hasil.md, screenshot/}
    └── tugas-7/{hasil.md, screenshot/}
```

## Parameter Kasus 6

| Item | Nilai |
|---|---|
| Container target | target-k6 |
| Volume | vol-k6 |
| File laporan | laporan-k6.txt |
| Kata sandi enkripsi | kunci-k6-2026 |
| Domain exfiltrasi | k6.exfil.lab |

## Persiapan Lingkungan Praktikum

```bash
cd 01-sumber-lab
docker build -t uts-openssl images/uts-openssl
docker build -t uts-kali-tools images/uts-kali-tools
docker build -t uts-dnsmasq images/uts-dnsmasq
docker pull alpine
docker pull nginx:alpine
docker pull vulnerables/web-dvwa
docker pull kalilinux/kali-rolling
```

**Catatan:** siapkan semua image sebelum menjalankan praktikum, karena jaringan Bagian B memakai `internal: true` (tidak ada akses internet di dalam container saat praktikum berjalan).

## Cara Menjalankan Praktikum

1. Baca `02-dokumen/DOKUMEN-UTS-LENGKAP.md` dan `02-dokumen/LEMBAR-PERINTAH-PER-KASUS.md`.
2. Jalankan perintah pada bagian **Kasus 6** di `LEMBAR-PERINTAH-PER-KASUS.md`.
3. Dokumentasikan setiap tugas: perintah yang dipakai, output terminal, analisis, dan screenshot.
4. Isi `03-hasil-kasus-6/tugas-N/hasil.md` untuk tiap tugas, lalu rangkum semuanya di `03-hasil-kasus-6/jawaban.md`.

## Catatan Praktikum

1. Seluruh pengujian dilakukan dalam lingkungan laboratorium Docker yang terisolasi (`internal: true`), bukan terhadap host di luar lab.
2. Perintah mengikuti panduan UTS dan disesuaikan dengan parameter Kasus 6.
3. Hasil dalam laporan didasarkan pada keluaran terminal dan bukti screenshot milik sendiri.
4. Perintah yang berhasil dijalankan tidak selalu berarti serangan berhasil mencapai dampak yang diharapkan — analisis harus membedakan proses pengujian dan hasil yang benar-benar terbukti.
