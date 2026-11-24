# Tugas 1 (Pertemuan 1): Docker dan Volume

## Perintah yang dijalankan

```bash
docker volume create vol-k6
docker run --name w-k6 -v vol-k6:/data alpine sh -c 'echo 24552011060 > /data/laporan-k6.txt'
docker rm w-k6
docker run --rm -v vol-k6:/data alpine cat /data/laporan-k6.txt
docker compose -p k6 -f b/b1-compose/docker-compose.yml up -d
docker compose -p k6 -f b/b1-compose/docker-compose.yml exec attacker wget -qO- http://target
docker compose -p k6 -f b/b1-compose/docker-compose.yml down -v
```

## Output terminal

```text
PS D:\UTS-Network-Security-Kasus6\01-sumber-lab> docker volume create vol-k6
vol-k6
PS D:\UTS-Network-Security-Kasus6\01-sumber-lab> docker run --name w-k6 -v vol-k6:/data alpine sh -c 'echo 24552011060 > /data/laporan-k6.txt'
PS D:\UTS-Network-Security-Kasus6\01-sumber-lab> docker rm w-k6
w-k6
PS D:\UTS-Network-Security-Kasus6\01-sumber-lab> docker run --rm -v vol-k6:/data alpine cat /data/laporan-k6.txt
24552011060
PS D:\UTS-Network-Security-Kasus6\01-sumber-lab> docker compose -p k6 -f b/b1-compose/docker-compose.yml up -d
[+] up 3/3
 ✔ Network k6_lab          Created                                                                 0.1s
 ✔ Container k6-target-1   Started                                                                 0.9s
 ✔ Container k6-attacker-1 Started                                                                 0.6s
PS D:\UTS-Network-Security-Kasus6\01-sumber-lab> docker compose -p k6 -f b/b1-compose/docker-compose.yml exec attacker wget -qO- http://target
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
html { color-scheme: light dark; }
body { width: 35em; margin: 0 auto;
font-family: Tahoma, Verdana, Arial, sans-serif; }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, nginx is successfully installed and working.
Further configuration is required for the web server, reverse proxy,
API gateway, load balancer, content cache, or other features.</p>

<p>For online documentation and support please refer to
<a href="https://nginx.org/">nginx.org</a>.<br/>
To engage with the community please visit
<a href="https://community.nginx.org/">community.nginx.org</a>.<br/>
For enterprise grade support, professional services, additional
security features and capabilities please refer to
<a href="https://f5.com/nginx">f5.com/nginx</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>
```

## Analisis / Penjelasan

**Persistensi volume (baris 1–4).** File `laporan-k6.txt` ditulis oleh container `w-k6`, lalu container tersebut dihapus sepenuhnya dengan `docker rm w-k6`. Meski begitu, saat sebuah container baru (sekali pakai, lewat `--rm`) dipasang ke volume `vol-k6` yang sama, isi file `24552011060` tetap terbaca. Ini membuktikan bahwa volume Docker disimpan oleh Docker Engine di lokasi terpisah dari *writable layer* container (biasanya di `/var/lib/docker/volumes/` pada host). Volume punya siklus hidup sendiri yang tidak terikat pada container yang memakainya — menghapus container hanya menghapus container-nya, bukan data yang sudah "dipasang" (mount) dari volume.

**Jaringan terisolasi dengan Compose (baris 5–7).** Service `attacker` berhasil mengakses `target` memakai nama service (`http://target`), bukan alamat IP, dan mendapat halaman default nginx. Ini terjadi karena Docker Compose otomatis membuat jaringan bridge khusus (`k6_lab`) dengan DNS resolution internal: setiap service bisa saling menemukan lewat namanya. Jaringan ini diset `internal: true`, artinya tidak ada gateway ke luar (tidak ada akses internet dari dalam container), tapi komunikasi *antar* container dalam jaringan yang sama tetap berjalan normal — sesuai yang diminta soal B1 (isolasi dari luar, tapi tetap terhubung di dalam).

## Screenshot

Lihat folder `screenshot/` untuk bukti visual.