# rosemary

Kernel Xiaomi rosemary (Redmi Note 10S, Linux 4.14.357 LTS dari OpenELA) dengan xxKSU
(dlagia/backupxksu) + SUSFS v2.4.0 ala Najip (glue parent-kernel, opsi
SUSFS = JackA1ltman) + NoMount, `uname -r` = `4.14.186-perf` (sama dengan kernel stok),
dibangun otomatis oleh GitHub Actions. `uname` sengaja tetap 4.14.186 agar
cocok dengan fsck.f2fs dan modul vendor stok.

- Manager: APK `arm64-v8a` non-GKI dari rilis [dlagia/backupxksu](https://github.com/dlagia/backupxksu/releases)
  (paket `me.weishu.kernelsu`, tanda tangan v2-only `0x363` / `4359c171...` sinkron
  dengan kernel ini). Manager BakaSU tidak dikenali kernel ini.

- Zip siap flash: halaman **Releases**. Yang berjudul `TERBUKTI BOOT - ...`
  sudah diuji di HP.
- Cara memakai, menaikkan versi, flash, dan pulih dari bootloop:
  **[PANDUAN.md](PANDUAN.md)**.

## Kredit

- **[Najip](https://github.com/Najip)** - metode SUSFS di atas xxKSU yang dipakai
  kernel ini, dari repo
  [Najip/KernelSU_SUSFS_Samsung_SM-A325X](https://github.com/Najip/KernelSU_SUSFS_Samsung_SM-A325X)
  (GPL-2.0): susunan glue parent-kernel (dispatcher perintah SUSFS lewat
  reboot magic, jembatan lifecycle `xxksu_parent_*`, `Kconfig.susfs`) dan
  patch `patches/kernelsu/0002-susfs-lifecycle-calls.patch`, yang diterapkan
  apa adanya ke xxKSU (dipin `NAJIP_PIN`). Terima kasih sudah membuktikan
  xxKSU + SUSFS bisa jalan di kernel 4.14 non-GKI.

Lisensi: GPL-2.0 (lihat `LICENSE`), sama dengan kernel Linux.
