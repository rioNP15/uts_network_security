# Tugas 2 (Pertemuan 2): Kriptografi (OpenSSL)

## Perintah yang dijalankan

```powershell
docker run --rm -v "${PWD}:/work" uts-openssl sh -c 'echo isi laporan k6 > laporan-k6.txt && openssl enc -aes-256-cbc -pbkdf2 -salt -in laporan-k6.txt -out laporan-k6.enc -pass pass:kunci-k6-2026 && sha256sum laporan-k6.txt && openssl enc -d -aes-256-cbc -pbkdf2 -in laporan-k6.enc -out hasil-k6.txt -pass pass:kunci-k6-2026 && sha256sum hasil-k6.txt'

docker run --rm -v "${PWD}:/work" uts-openssl sh -c 'openssl req -x509 -newkey rsa:2048 -nodes -keyout key-k6.pem -out cert-k6.pem -days 30 -subj /CN=k6.uts.local && openssl x509 -in cert-k6.pem -noout -subject -issuer -dates'
```

## Output terminal

```text
PS D:\UTS-Network-Security-Kasus6\01-sumber-lab> docker run --rm -v "${PWD}:/work" uts-openssl sh -c 'echo isi laporan k6 > laporan-k6.txt && openssl enc -aes-256-cbc -pbkdf2 -salt -in laporan-k6.txt -out laporan-k6.enc -pass pass:kunci-k6-2026 && sha256sum laporan-k6.txt && openssl enc -d -aes-256-cbc -pbkdf2 -in laporan-k6.enc -out hasil-k6.txt -pass pass:kunci-k6-2026 && sha256sum hasil-k6.txt'
ba00dcca033b5d1bcd46a143d134172554ea37b3a4849e4e98297e9846aaef54  laporan-k6.txt
ba00dcca033b5d1bcd46a143d134172554ea37b3a4849e4e98297e9846aaef54  hasil-k6.txt
PS D:\UTS-Network-Security-Kasus6\01-sumber-lab> docker run --rm -v "${PWD}:/work" uts-openssl sh -c 'openssl req -x509 -newkey rsa:2048 -nodes -keyout key-k6.pem -out cert-k6.pem -days 30 -subj /CN=k6.uts.local && openssl x509 -in cert-k6.pem -noout -subject -issuer -dates'
subject=CN=k6.uts.local
issuer=CN=k6.uts.local
notBefore=Nov 21 12:45:47 2026 GMT
notAfter=Dec 21 12:45:47 2026 GMT
```

> Catatan: baris titik/plus panjang sebelum `subject=...` adalah indikator progres OpenSSL saat mencari bilangan prima untuk kunci RSA 2048-bit, bukan pesan error — sengaja tidak disertakan di sini karena tidak membawa informasi, tapi tetap tampak di screenshot asli.

## Analisis / Penjelasan

**Enkripsi-dekripsi AES-256 (perintah pertama).** File `laporan-k6.txt` dienkripsi dengan `openssl enc -aes-256-cbc -pbkdf2` memakai password `kunci-k6-2026`, menghasilkan `laporan-k6.enc`. File itu lalu didekripsi kembali menjadi `hasil-k6.txt` dengan password yang sama. Hash SHA-256 dari `laporan-k6.txt` (sebelum dienkripsi) dan `hasil-k6.txt` (setelah didekripsi) **identik**: `ba00dcca033b5d1bcd46a143d134172554ea37b3a4849e4e98297e9846aaef54`. Ini membuktikan proses enkripsi-dekripsi AES bersifat *lossless* — selama kunci/password yang dipakai untuk dekripsi sama dengan yang dipakai saat enkripsi, data kembali identik bit demi bit. SHA-256 dipakai sebagai pembanding karena hash sangat sensitif: perubahan satu bit saja pada input akan menghasilkan hash yang sama sekali berbeda (avalanche effect), sehingga hash yang sama adalah bukti kuat bahwa isi file benar-benar sama persis.

**Sertifikat self-signed (perintah kedua).** Perintah `openssl req -x509 -newkey rsa:2048 ... -subj /CN=k6.uts.local` membuat pasangan kunci RSA 2048-bit sekaligus sertifikat X.509 dalam satu langkah (flag `-x509`). Hasilnya:
```
subject=CN=k6.uts.local
issuer=CN=k6.uts.local
```
Subject (pihak yang diklaim identitasnya oleh sertifikat) dan Issuer (pihak yang menandatangani sertifikat) **sama persis**. Inilah definisi sertifikat *self-signed*: entitas yang membuat sertifikat menandatangani sertifikatnya sendiri, tanpa melibatkan pihak ketiga.

**Perbedaan sertifikat self-signed vs CA-signed:**
Sertifikat self-signed memiliki subject dan issuer yang sama karena diterbitkan sendiri oleh pemiliknya, sehingga tidak ada pihak independen yang memverifikasi identitas tersebut — klien yang mengaksesnya tidak bisa memastikan keasliannya tanpa secara eksplisit mempercayai sertifikat itu (biasanya muncul peringatan "not trusted" di browser). Sertifikat yang ditandatangani CA (Certificate Authority) diterbitkan oleh pihak ketiga tepercaya yang root certificate-nya sudah tertanam di sistem operasi/browser, sehingga klien bisa memverifikasi rantai kepercayaan (chain of trust) dari sertifikat tersebut sampai ke root CA, tanpa perlu mempercayai pemilik sertifikat secara langsung.

## Screenshot

Lihat folder `screenshot/` untuk bukti visual.