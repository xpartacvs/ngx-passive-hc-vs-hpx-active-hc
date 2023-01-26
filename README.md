# NGINX Passive Healthcheck VS HAProxy Active Healthcheck

Repositori ini berisi contoh simulasi healthcheck pada backend service menggunakan NGINX dan HAProxy. Pada NGINX, healthcheck dilakukan secara pasif, sedangkan pada HAProxy dilakukan secara aktif. Lalu apa perbedaan antara healthcheck pasif dan aktif? Berikut penjelasannya:

## Tentang Healthcheck

Sebelum membahas perbedaan healthcheck pasif dan aktif, mari kita bahas dulu apa itu healthcheck.

Pada sistem berbasis `client-server`, healthcheck adalah proses untuk memeriksa apakah backend service yang digunakan masih dalam kondisi hidup atau mati. Healthcheck ini dilakukan oleh load balancer, misalnya NGINX atau HAProxy. Jika backend service mati, maka load balancer akan menghapus backend service tersebut dari daftar backend service yang aktif. Jika backend service sudah hidup kembali, maka load balancer akan menambahkan backend service tersebut kembali ke daftar backend service yang aktif.

Lalu apa benefit dari healthcheck?

## Manfaat Healthcheck

Healthcheck berguna untuk menghindari kejadian dimana request yang datang terkirim ke backend service yang mati. Jika request terkirim ke backend service yang mati, maka request tersebut akan gagal dan akan menghasilkan error. Jika terjadi error, maka user akan mendapatkan respon error. Jika terjadi error secara berulang, maka user akan merasa tidak nyaman dengan aplikasi yang digunakan.

## Mode Healthcheck

Ada dua mode healthcheck, yaitu `pasif` dan `aktif`. Perbedaan antara healthcheck pasif dan aktif adalah pada cara kerjanya:

### Mode Pasif

Pada mode `pasif`, load balancer akan memeriksa status health backend service yang hanya akan terpicu oleh kedatangan request. Jika backend service merespon dengan status code `200`, maka load balancer akan menganggap backend service tersebut masih hidup. Jika backend service tidak merespon, maka load balancer akan menganggap backend service tersebut sudah mati.

### Mode Aktif

Pada mode `aktif`, load balancer akan memeriksa status halth backend service secara berkala tanpa perlu menunggu datangnya request terlebih dahulu. Jika backend service merespon dengan status code `200`, maka load balancer akan menganggap backend service tersebut masih hidup. Jika backend service tidak merespon, maka load balancer akan menganggap backend service tersebut sudah mati.

## Mode Healthcheck pada NGINX dan HAProxy

Kenapa menggunakan mode pasif pada NGINX sedangkan mode aktif diterapkan pada HAProxy? Jawabannya adalah karena mode aktif pada NGINX tidak tersedia pada versi gratisan. Mode aktif hanya tersedia pada versi pro. Jadi, jika ingin menggunakan mode aktif pada NGINX, maka harus membeli lisensi pro.

## Praktik Lapangan

Pada praktik lapangan ini, kita akan melakukan simulasi healthcheck yang masing-masing dilakukan oleh NGINX dan HAProxy. Kita akan melihat perbedaan cara kerja dari keduanya.

Dengan asumsi bahwa kita sudah meng-_clone_ repositori ini, mari kita mulai dengan menghidupkan semua service yang dibutuhkan dengan menjalankan perintah berikut:

```bash
sudo docker-compose up -d
```

Setelah semua service hidup, coba akses aplikasi yang tersedia melalui masing-masing load balancer menggunakan browser:

- Untuk mengakses aplikasi melalui NGINX, kunjungi [http://localhost:8181](http://localhost:8181)
- Untuk mengakses aplikasi melalui HAProxy, kunjungi [http://localhost:8282](http://localhost:8282)

Jika seluruh service berjalan dengan baik, maka kita akan mendapati halaman berlatar putih dengan hanya bertuliskan `Server A` di tengah-tengahnya, baik pada aplikasi yang diakses melalui NGINX maupun HAProxy.

Jika kita `refresh` halaman tersebut, maka kita akan mendapatkan halaman yang berbeda, yaitu halaman bertuliskan`Server B`. Hal ini terjadi karena aplikasi yang diakses melalui NGINX dan HAProxy akan memilih backend service secara bergantian.

Sekaranng kita akan melakukan simulasi healthcheck. Pertama-tama, kita akan mematikan salah satu backend service (misal: `serverb`) dengan menjalankan perintah berikut :

```bash
sudo docker-compose stop serverb
```

Setelah backend service dimatikan, tunggu sekitar 10 detik dan coba akses kembali aplikasi yang tersedia melalui masing-masing load balancer dan lihat apa yang terjadi:

### Perilaku Passive Healthcheck pada NGINX

Setelah dikunjungi kembali, dapat terlihat bahwa browser menunggu beberapa saat hingga akhirnya menampilkan halaman `Server A`. Jika kita `refesh`, maka kita akan mendapatkan halaman yang sama, dengan perilaku yang sama pula (menunggu beberapa saat).

Jika kita `refersh` terus menerus, kita masih akan mendapatkan halaman yang sama dengan perilaku yang (mungkin) masih sama (yaitu menunggu beberapa saat), hingga akhirnya halaman tersebut muncul tanpa perlu ada fase menunggu lagi. Ini artinya NGINX sudah menghapus backend service yang mati dari daftar backend service yang tersedia.

### Perilaku Active Healthcheck pada HAProxy

Setelah dikunjungi kembali, dapat terlihat bahwa browser langsung menampilkan halaman `Server A` tanpa ada proses tunggu. Hal ini dikarenakan HAProxy secara aktif memeriksa status health backend service secara berkala dan menghapus backend service yang mati tanpa perlu menunggu datangnya request terlebih dahulu.

Jika kita lihat log service haproxy dengan perintah berikut:

```bash
sudo docker-compose logs hpx
```

Maka akan terlihat bahwa HAProxy memeriksa status health backend service secara berkala. Perilaku `active healthcheck` ini memeberikan benefit user experience yang lebih baik dibandingkan dengan `passive healthcheck`, karena user tidak perlu menunggu beberapa saat untuk mendapatkan halaman yang diinginkan.
