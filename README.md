# Open WebUI Docker Migration Guide

Panduan ini menjelaskan langkah-langkah untuk memigrasikan sistem Open WebUI dari VM lama ke VM baru, termasuk container Docker, data mount, dan konfigurasi user.

---

## A. Persiapan di VM Lama

### Daftar File yang Harus Dipindahkan ke VM Baru

Sebelum melanjutkan setup di VM baru, pastikan file-file berikut telah dipindahkan dari VM lama:

- Semua file image hasil `docker save` (format `.tar`), misalnya:
  - `openwebui_backup.tar`
  - `litellm_backup.tar`
  - `tika_backup.tar`
- File mount data yang telah dikompres, misalnya: `data_backup.tar.gz`
- Kompresan file konfigurasi user dari `/home`, misalnya: `home_backup.tar.gz`

Referensi video migrasi Docker secara umum:\
👉 [Docker Migration YouTube Guide](https://youtu.be/QskmB4fb-uo?si=sWTQPROMhJvWvaGq)

---

### 1. Docker Container

**a. Stop container terlebih dahulu:**

```bash
sudo docker stop <container-name>
```

**b. Commit container menjadi image baru:**

```bash
sudo docker commit <container-name> <nama-image>
```

**c. Lihat semua image yang tersedia:**

```bash
sudo docker images
```

**d. Simpan image ke file **`.tar`**:**

```bash
sudo docker save <image-name> > <backup-name>.tar
```

Ulangi langkah ini untuk semua image yang ingin dipindahkan (misalnya: `open-webui`, `litellm`, `tika`, dsb).

---

### 2. Mount File

**a. Cek lokasi mount file dengan **`docker inspect`**:**

```bash
sudo docker inspect <container-name> | grep Mounts -A 20
```

**b. Setelah mengetahui lokasi (misalnya **`/mnt/sdb1/data`**), kompres datanya:**

```bash
cd /mnt/sdb1
tar -czvf data_backup.tar.gz data/
```

---

### 3. Kompres Semua File di `/home`

Jika konfigurasi seperti file `.yaml` atau mount berada di direktori user, kompres seluruh isi direktori `/home`:

```bash
cd /home
sudo tar -czvf home_backup.tar.gz <user>
```

---

## B. Setup di VM Baru

### 1. Restore Docker Images

**a. Transfer semua file **`.tar`** ke VM baru, lalu load image:**

```bash
sudo docker load -i <backup-name>.tar
```

Ulangi untuk semua image yang telah disimpan.

---

### 2. Restore Mount File dan Folder `/home`

**a. Ekstrak file data mount:**

```bash
cd /mnt/sdb1
sudo tar -xzvf data_backup.tar.gz
```

**b. Ekstrak folder **`/home`**:**

```bash
cd /home
sudo tar -xzvf home_backup.tar.gz
```

---

### 3. Buat Docker Network (jika container sebelumnya berada di jaringan yang sama)

```bash
sudo docker network create \
  --driver bridge \
  --subnet <ip>/24 \
  --gateway <ip> \
  my-network
```

Gantilah `<ip>` sesuai konfigurasi subnet dan gateway dari VM lama (misalnya: `172.18.0.0/24` dan `172.18.0.1`).

---

### 4. Jalankan Container

```bash
# Jalankan litellm
sudo docker run -d \
  --name litellm \
  --network my-network \
  --ip <ip> \
  -p 4000:4000 \
  -v /home/<user>/litellm_config.yaml:/app/config.yaml \
  litellmbackup:latest

# Jalankan open-webui
sudo docker run -d \
  --name open-webui \
  --network my-network \
  --ip <ip> \
  -p 3001:8080 \
  -v /mnt/sdb1/data/anythingllm1:/app/backend/data \
  ouibackup:latest

# Jalankan tika
sudo docker run -d \
  --name tika \
  --network my-network \
  --ip <ip> \
  -p 9998:9998 \
  tikabackup:latest
```

---

## Selesai 🎉

Sekarang sistem Open WebUI telah berhasil dipindahkan ke VM baru, lengkap dengan konfigurasi container, data, dan environment-nya.

---

> **Catatan:** Pastikan semua IP dan path sesuai dengan konfigurasi masing-masing dan tidak terjadi konflik port atau IP saat menjalankan container.

---

## Migrasi Hanya Chat atau Konfigurasi

Jika Anda hanya ingin memigrasi riwayat chat atau konfigurasi tanpa memindahkan seluruh sistem, Anda dapat mengekspor dan mengimpor database secara langsung.

Lihat dokumentasi resmi Open WebUI:  
👉 [Exporting Database - Open WebUI](https://docs.openwebui.com/tutorials/database#exporting-database)

