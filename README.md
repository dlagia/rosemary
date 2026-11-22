# rosemary

Kernel Xiaomi rosemary (Redmi Note 10S, Linux 4.14) dengan xxKSU
(backslashxx/KernelSU) + SUSFS v2.4.0 ala Najip (glue parent-kernel) + NoMount
+ fgetpriority, `uname -r` = `4.14.186-perf` (sama dengan kernel stok),
dibangun otomatis oleh GitHub Actions. Ini cabang uji `susfs-najip`: rilisnya
prerelease, belum diflash; `main` tetap tanpa susunan ini.

- Manager: APK dari rilis [backslashxx/KernelSU](https://github.com/backslashxx/KernelSU/releases).
  Manager BakaSU tidak dikenali kernel ini.

- Zip siap flash: halaman **Releases**. Yang berjudul `TERBUKTI BOOT - ...`
  sudah diuji di HP.
- Cara memakai, menaikkan versi, flash, dan pulih dari bootloop:
  **[PANDUAN.md](PANDUAN.md)**.

Lisensi: GPL-2.0 (lihat `LICENSE`), sama dengan kernel Linux.
