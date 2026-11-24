# Sumber Lab — UTS Network Security

Source lab untuk UTS praktik berbasis Docker: Dockerfile image khusus, file docker-compose, dan skrip pendukung.

**Gunakan hanya di lingkungan lab lokal. Jangan memindai atau menyerang host di luar lab.**

## Struktur

```
images/uts-openssl/       Dockerfile image openssl (Tugas 2)
images/uts-kali-tools/    Dockerfile image hping3, dnsutils, xxd, dsniff, aircrack-ng (Tugas 3,4,5,6,7)
images/uts-dnsmasq/       Dockerfile image dnsmasq (Tugas 7)
b/b1-compose/             docker-compose.yml jaringan internal (Tugas 1)
b/b3-syn-flood/           docker-compose.yml SYN flood (Tugas 6)
b/b4-dns-exfil/           docker-compose.yml DNS exfiltration (Tugas 7)
kasus/k5-scapy/           Skrip Scapy untuk Tugas 5
kasus/k9-dns-detect/      Skrip deteksi DNS exfiltration untuk Tugas 7
```

## Persiapan (sebelum ujian)

Jaringan Bagian B memakai `internal: true`, sehingga container di dalamnya tidak punya akses internet. Semua paket harus sudah terpasang di image.

```bash
docker build -t uts-openssl images/uts-openssl
docker build -t uts-kali-tools images/uts-kali-tools
docker build -t uts-dnsmasq images/uts-dnsmasq
docker pull alpine
docker pull nginx:alpine
docker pull vulnerables/web-dvwa
docker pull kalilinux/kali-rolling
```

Untuk perintah lengkap per tugas Kasus 6, lihat `../02-dokumen/LEMBAR-PERINTAH-PER-KASUS.md` bagian Kasus 6.
