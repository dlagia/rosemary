# Panduan dlagia/rosemary

Kernel Xiaomi rosemary (Redmi Note 10S, MT6785, Linux 4.14.357 LTS hasil
merge OpenELA kernel-lts ke source vendor) dengan
xxKSU (backslashxx/KernelSU) + SUSFS v2.4.0 ala Najip + NoMount,
dibangun otomatis
oleh GitHub Actions. `uname -r` = `4.14.186-perf`, sama dengan kernel stok
(step "Samakan SUBLEVEL dengan versi stok").
Panduan ini ditulis supaya semua pekerjaan rutin bisa dikerjakan sendiri,
tanpa bantuan AI.

Semua tombol ada di tab **Actions**. Pilih workflow di kiri, tekan
**Run workflow**, isi kolomnya, tekan **Run workflow** lagi.

## Isi repo

| berkas | gunanya |
|---|---|
| `pins.env` | 7 baris `NAMA_PIN=<SHA 40 huruf>`: versi persis tiap komponen. Ini satu-satunya yang biasanya diubah. |
| `.github/workflows/build-kernel-ksu-v2.yml` | resep build kernel |
| `.github/workflows/pin-watch.yml` | robot pemantau upstream (tiap hari) |
| `.github/workflows/naikkan-pin.yml` | tombol menaikkan pin |
| `.github/workflows/tandai-boot.yml` | tombol menandai rilis terbukti boot |
| `.github/workflows/kembali-ke-titik.yml` | tombol rollback |
| `.github/workflows/bersihkan.yml` | pembersih rilis uji lama + cache (tiap Minggu) |

Komponen yang di-pin:

| pin | repo upstream | branch |
|---|---|---|
| `KERNEL_PIN` | dlagia/android_kernel_xiaomi_rosemary | rosemary-13 |
| `KSU_PIN` | backslashxx/KernelSU (xxKSU) | master |
| `SUSFS_PIN` | JackA1ltman/NonGKI_Kernel_Build_2nd (SUSFS 4.14) | mainline |
| `NAJIP_PIN` | Najip/KernelSU_SUSFS_Samsung_SM-A325X (lifecycle 0002) | main |
| `NOMOUNT_PIN` | maxsteeel/nomount | dev |
| `NOMOUNT_LOADER_PIN` | maxsteeel/nomount | master |
| `AK3_PIN` | vlxlxlv/AnyKernel3 | mt6781 |
| `CLANG_PIN` | kdrag0n/proton-clang | master |

SUSFS aktif lewat patch 4.14 JackA1ltman + glue parent-kernel di atas xxKSU,
disusun seperti Najip (tanpa BakaSU, tanpa hook manual). Hook lifecycle di
KernelSU diambil dari patch `0002-susfs-lifecycle-calls.patch` milik Najip.
Penandaan `TIF_PROC_UMOUNTED` (dipakai `SUS_MOUNT`, `SUS_MAP`,
`OPEN_REDIRECT`, `SUS_KSTAT`, `SUS_PATH_LOOP`) tetap jalan pada aplikasi
non-root turunan zygote walau `kernel_umount` mati atau hanya memakai
NoMount. Opsi SUSFS mengikuti JackA1ltman: 11 opsi `=y`, termasuk
`PRIORITY_UID_FILTER` (pengganti fgetpriority, yang dicabut) dan `ENABLE_LOG`.

Hanya build dari `main` yang terbit sebagai rilis biasa ("Latest"). Build
dari cabang lain (kalau suatu saat dibuat untuk uji) selalu *prerelease* dan
tidak pernah jadi "Latest", jadi gerbang build `main` dan tombol "Tandai
terbukti boot" tidak terganggu.

## Manager

Pasang APK manager dari halaman rilis
[backslashxx/KernelSU](https://github.com/backslashxx/KernelSU/releases)
(paket `me.weishu.kernelsu`). Manager BakaSU atau KernelSU resmi tidak
dikenali kernel ini. Pasang APK-nya SEBELUM flash zip pertama dari repo ini,
supaya sesudah boot manager langsung jalan.

## Istilah

- **Rilis** = hasil build hijau di halaman Releases, berisi zip siap flash.
  Namanya `build-YYYYMMDD-HHMMSS` (waktu UTC).
- **Titik** = rilis yang sudah Anda flash dan terbukti boot normal. Ditandai
  dengan tombol **Tandai terbukti boot**, yang membuat tag `titik-...`.
  Titik terbaru = pegangan utama. Titik kedua = **jaring kedua** (cadangan
  kalau titik terbaru ternyata bermasalah belakangan).
- **Vonis pin-watch**:
  - **di HEAD**: pin sudah versi terbaru, tidak ada yang perlu dilakukan.
  - **AMAN dinaikkan**: upstream bergerak, tapi tidak menyentuh berkas sensitif.
  - **PERLU AUDIT**: upstream mengubah berkas sensitif (kode yang masuk ke
    kernel). Perlu dinilai dulu, lihat bagian di bawah.
  - **ORPHAN**: upstream me-rebase/force-push, commit yang di-pin lepas dari
    branch. Build masih bisa jalan (ada self-heal), tapi sebaiknya pindah ke
    commit baru.
  - **GAGAL AMBIL**: API GitHub sedang error. Tunggu laporan besok.

## Alur rutin (dari issue sampai HP)

1. **Baca issue pin-watch.** Tiap hari robot memperbarui satu issue berisi
   tabel pin + vonis. Notifikasinya masuk ke email/app GitHub.
2. **Vonis AMAN**: Actions -> **Naikkan pin** -> pilih komponen, kolom SHA
   kosongkan (= HEAD upstream), centang *bangun*. Robot menulis pins.env,
   commit, lalu memanggil build.
3. **Tunggu build** (biasanya 10-20 menit; build pertama tanpa cache lebih
   lama). Hijau = rilis baru muncul di
   Releases. Merah = tidak ada rilis; buka run-nya, lihat step yang merah
   (lihat "Build gagal" di bawah).
4. **Baca bagian "Banding log dengan titik terbukti boot"** di badan rilis.
   - **SAMA**: log compile identik dengan titik. Risiko kecil.
   - Ada baris **NAIK**: baca dulu. `warning` naik sedikit biasanya tidak
     apa-apa. `error` atau `rej` naik = ada patch yang tidak masuk bersih;
     buka log run dan cari kata `error:` atau `.rej` sebelum flash.
5. **Siapkan jalan pulang sebelum flash.** Salin zip titik terbaru ke memori
   HP (atau SD card). Di TWRP, menu Backup -> centang *Boot* -> swipe,
   supaya ada cadangan boot sekarang.
6. **Flash**: unduh zip rilis baru (tautan di badan rilis bisa dibuka dari
   browser HP) -> reboot ke TWRP -> Install -> pilih zip -> swipe -> Reboot
   System.
7. **Uji**: HP boot, manager KernelSU menunjukkan versi kernel baru, modul
   jalan, app yang biasa Anda pakai normal. Uji sehari-dua kalau mau yakin.
8. **Tandai**: Actions -> **Tandai terbukti boot** -> isi tag rilis yang
   diflash (atau kosongkan kalau itu rilis Latest), catatan opsional.
   Rilis itu kini jadi titik terbaru; titik sebelumnya otomatis jadi jaring
   kedua.

## Vonis PERLU AUDIT: cara menilai sendiri

Issue pin-watch mencantumkan tautan `diff:` dan daftar berkas sensitif yang
berubah. Buka tautannya, lalu periksa:

1. **Apakah berkas itu benar-benar dikompilasi di kernel 4.14 kita?**
   Perubahan di dalam `#if LINUX_VERSION_CODE >= KERNEL_VERSION(5,...)`,
   cabang `#ifdef CONFIG_KSU_MODULE` / `MODULE`, atau berkas untuk arsitektur
   lain (x86, arm32) tidak berpengaruh pada kita.
2. **Apakah hanya komentar, teks log (`pr_info`/`pr_warn`), nama, atau teks
   bantuan Kconfig?** Itu aman.
3. **Apakah menyentuh jalur boot**: hook LSM/syscall, mount, `init`,
   selinux, sumount/umount? Ini yang berisiko bootloop. Baca pesan commit
   dan issue upstream terkait; kalau ada laporan bootloop di sana, tunggu.
4. Kalau masih ragu: naikkan dengan **paksa** dicentang, biarkan build
   jalan, baca **banding log**, lalu flash **hanya dengan zip titik sudah
   ada di HP**. Kalau bootloop, ikuti "Pulih dari bootloop" - kerugiannya
   hanya beberapa menit.

Jangan naikkan beberapa komponen PERLU AUDIT sekaligus: kalau bootloop,
Anda tidak tahu mana penyebabnya. Satu komponen, satu build, satu uji.

## Pulih dari bootloop

HP tidak lewat logo, atau restart terus:

1. Tahan **Power + Volume Atas** sampai masuk TWRP.
2. Flash zip titik terbaru (yang sudah Anda salin ke HP di langkah 5).
   Kalau zip tidak ada di HP: di PC unduh dari Releases (rilis berjudul
   `TERBUKTI BOOT - ...`), sambungkan kabel, lalu di TWRP pilih
   Advanced -> ADB Sideload, dan di PC jalankan `adb sideload <zip>`.
   Alternatif: TWRP Restore cadangan *Boot* dari langkah 5.
3. Reboot System. HP kembali ke kernel titik.
4. Di GitHub: Actions -> **Kembali ke titik** (kolom titik kosongkan).
   pins.env kembali ke isi titik, supaya kenaikan berikutnya mulai dari
   keadaan yang terbukti boot. Ringkasan run-nya juga memuat tautan zip
   titik.
5. Kalau titik terbaru pun ternyata bermasalah: jalankan lagi dengan kolom
   titik = `2` (jaring kedua), `3`, dan seterusnya.

## Build gagal (merah)

Tidak ada rilis yang terbit, jadi HP tidak terancam. Buka run yang merah,
klik step yang bertanda X:

- **Muat pins.env**: format pins.env rusak. Harus 7 baris
  `NAMA_PIN=<40 huruf 0-9a-f>`, tanpa spasi.
- **Clone / fetch gagal, `not our ref`, `couldn't find remote ref`**: commit
  hilang di upstream (lihat "Upstream hilang"), atau GitHub sedang gangguan
  (tekan *Re-run jobs*).
- **`HTTP 403: Resource not accessible by integration` di step "Terbitkan
  Release" atau tombol "Tandai terbukti boot"**: isi `.github/workflows/`
  di commit yang dibangun berbeda dari `main` saat itu, misalnya ada push
  workflow lain selagi build jalan, atau rilis berasal dari cabang uji yang
  sudah dihapus. Token Actions tidak boleh membuat tag di commit seperti itu.
  Kernel tidak rusak. Untuk build: tekan *Run workflow* lagi di `main`, dan
  jangan push perubahan workflow selama build berjalan. Untuk tag titik:
  ikuti perintah `gh api ... git/refs` yang dicetak tombol itu, lalu
  jalankan tombolnya lagi.
- **Assert / `ERROR: ...` di step integrasi (xxKSU, SUSFS, NoMount, SUBLEVEL)**: upstream
  mengubah bentuk kode yang ditambal workflow ini. Paling aman:
  **Kembali ke titik**, lalu tunggu upstream stabil, atau naikkan komponen
  lain dulu.
- **Compile kernel**: cari baris `error:` pertama di log; nama berkas di
  depannya menunjukkan komponen mana yang bermasalah.
- Gangguan sesaat (timeout unduh, runner mati): *Re-run jobs*.

## Upstream hilang (repo dihapus atau commit lenyap)

Setiap build hijau mengarsipkan source semua pin ke rilis **arsip-sumber**
(nama aset = komponen + 12 huruf awal SHA). Rilis itu jangan dihapus.

Kalau upstream sebuah komponen hilang:

1. Buat repo baru di akun Anda (mis. `dlagia/KernelSU`).
2. Unduh asetnya dari rilis arsip-sumber, lalu dorong ke repo baru:
   - KSU (`ksu-<sha>.bundle`):
     ```
     git clone ksu-<sha>.bundle KernelSU
     cd KernelSU
     git push https://github.com/dlagia/KernelSU.git pin:master
     ```
   - Komponen lain (`*.tar.gz`): ekstrak, `git init`, `git add -A`,
     `git commit`, push. SHA-nya akan baru; pakai SHA baru itu di pins.env.
3. Di PC, ganti nama repo lama dengan repo baru di
   `.github/workflows/build-kernel-ksu-v2.yml` (cari nama repo lamanya) dan
   di daftar `pin-watch.yml` (baris yang berisi `;nama/repo;branch;`).
   Nama repo itu juga ada di `naikkan-pin.yml` dan di daftar job `arsip`
   workflow build: cari nama repo lama di SEMUA berkas `.github/workflows/`
   supaya tidak ada yang terlewat.
4. Commit dan push (lihat "Kerja dari PC").

## Kerja dari PC

```
git clone https://github.com/dlagia/rosemary.git
cd rosemary
# ubah berkas
git add -A
git commit -m "pesan singkat"
git push
```

- Push yang mengubah `pins.env` atau workflow build memicu build otomatis.
  Gerbang build melewati build kalau 7 SHA dan workflow-nya sama persis
  dengan rilis terakhir.
- Mengubah pin dengan tangan: ganti SHA di baris `NAMA_PIN=` dengan SHA
  40 huruf dari halaman commit upstream. Tombol **Naikkan pin** melakukan
  hal yang sama plus pengecekan berkas sensitif, jadi lebih aman.
- Berkas di bawah `.github/workflows/` hanya bisa diubah dari PC atau web
  GitHub (tombol-tombol Actions dilarang GitHub menulisnya).

## Bersihkan otomatis

Tiap Minggu 09:41 WIB, workflow **Bersihkan** menghapus rilis `build-*`
yang belum ditandai titik, bukan 3 rilis terbaru, dan lebih tua dari
14 hari, beserta cache ccache lama. Titik dan arsip-sumber tidak pernah
dihapus. Mau lihat dulu tanpa menghapus: jalankan tangan dengan **coba**
dicentang.

## Yang tetap keputusan Anda

Robot tidak pernah menaikkan pin sendiri dan tidak pernah tahu apakah HP
Anda boot. Tiga hal ini selalu di tangan Anda: menekan **Naikkan pin**,
menilai **PERLU AUDIT**, dan menekan **Tandai terbukti boot** sesudah uji
di HP.
