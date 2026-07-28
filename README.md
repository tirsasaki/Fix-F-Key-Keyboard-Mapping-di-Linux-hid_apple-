# Fix F-Key Keyboard Mapping di Linux (hid_apple)

Beberapa keyboard pihak ketiga (seperti Rexus M84X dan keyboard 75% lainnya) menggunakan firmware yang mengikuti layout Apple, sehingga tombol F1–F12 ter-mapping ke fungsi media (brightness, launch browser, dll) alih-alih menjadi function keys standar. Di Linux, hal ini ditangani oleh modul `hid_apple` yang perlu dikonfigurasi ulang.

---

## Gejala

- Menekan `F4` malah membuka browser (launch browser)
- Tombol F1–F12 menghasilkan aksi media/fungsi khusus, bukan F1–F12 biasa
- Terjadi di keyboard non-Apple yang menggunakan firmware kompatibel Apple

---

## Langkah Perbaikan

### 1. Verifikasi modul hid_apple aktif

```bash
lsmod | grep hid_apple
dmesg | grep -i apple
```

Jika modul aktif, lanjut ke langkah berikutnya.

---

### 2. Buat file konfigurasi

```bash
sudo nano /etc/modprobe.d/hid_apple.conf
```

Isi file dengan:

```
options hid_apple fnmode=2
```

> **Keterangan nilai `fnmode`:**
> - `0` = Disable F-keys sepenuhnya (hanya fungsi media)
> - `1` = F-keys aktif hanya jika tombol `Fn` ditekan
> - `2` = F-keys aktif secara default (yang kita inginkan)

---

### 3. Update initramfs (berbeda tiap distro)

#### Debian / Ubuntu / Linux Mint

```bash
sudo update-initramfs -u
sudo reboot
```

#### Fedora / RHEL / CentOS Stream

```bash
sudo dracut --force
sudo reboot
```

#### Arch Linux / Manjaro / EndeavourOS

```bash
sudo mkinitcpio -P
sudo reboot
```

---

### 4. (Opsional) Reload modul tanpa reboot

Jika ingin mencoba efeknya segera tanpa reboot:

```bash
sudo rmmod hid_apple
sudo modprobe hid_apple fnmode=2
```

> Catatan: Cara ini bersifat sementara. Setelah reboot, konfigurasi dari file `/etc/modprobe.d/hid_apple.conf` yang akan berlaku.

---

## Verifikasi

Setelah reboot, tekan tombol F4 dan pastikan menghasilkan input F4, bukan aksi browser. Bisa juga dicek via:

```bash
cat /sys/module/hid_apple/parameters/fnmode
```

Outputnya harus `2`.

---

## Catatan

- File `/etc/modprobe.d/hid_apple.conf` akan hilang jika install ulang sistem. Sebaiknya backup file ini.
- Konfigurasi ini berlaku global untuk semua keyboard yang menggunakan modul `hid_apple`.
- Diuji pada: Debian 13, Fedora 44, Arch Linux.
