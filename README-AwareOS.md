# Carbon-Aware OS: Green Computing Operating System
**Dikembangkan oleh:** Team Beat Gua di Arab
**Kategori:** Green Computing & System Optimization  
**Basis Sistem:** Ubuntu 22.04 LTS (Remastered via Penguins-eggs)

---

## 1. Latar Belakang & Problem Statement
Pusat data global dan server komputasi menyumbang porsi besar terhadap emisi karbon dunia. Masalah utamanya bukan hanya pada seberapa banyak energi yang digunakan, tetapi **kapan** energi tersebut digunakan. 

Seringkali, tugas-tugas server yang berat (seperti *backup* sistem, pengolahan data numerik, atau *batch processing*) dieksekusi pada jam-jam beban puncak (Peak Hours) di mana grid listrik sangat bergantung pada pembangkit listrik berbahan bakar fosil. Kurangnya sistem operasi yang "sadar" terhadap kondisi lingkungan ini mengakibatkan inefisiensi jejak karbon komputasi.

## 2. Pendekatan Sistem & Justifikasi
**Carbon-Aware OS** hadir dengan pendekatan **Demand-Shifting**. Sistem operasi ini memiliki *Smart Scheduler* bawaan yang akan menahan (pending) eksekusi perintah berat di jam sibuk, dan otomatis menjalankannya (running) saat grid listrik berada pada status emisi rendah (Off-Peak Hours).

**Target Pengguna:**
1. **System Administrator:** Untuk penjadwalan *backup* dan *maintenance*.
2. **Data Scientist/Peneliti:** Untuk menjalankan komputasi numerik dan simulasi berat.
3. **Perusahaan (ESG):** Korporasi yang ingin menekan emisi karbon IT mereka.

---

## 3. Arsitektur Sistem
Sistem ini dibangun dengan mengintegrasikan 3 lapisan utama:
- **OS Layer:** Linux Ubuntu 22.04 yang dimodifikasi (*Remastered*) menggunakan `penguins-eggs` agar bersifat *portable* (Live ISO).
- **Backend (Python & Systemd):** Layanan `carbon-aware.service` yang berjalan di latar belakang untuk membaca status grid dan mengeksekusi antrean dari database MySQL.
- **Frontend (PHP & Apache2):** Dashboard interaktif berbasis web untuk memantau penghematan daya, status CPU/RAM, dan manajemen tugas secara *real-time*.

---

## 4. Langkah-Langkah Pembuatan (Step-by-Step Development)

### Tahap 1: Persiapan Database (MySQL/MariaDB)
Struktur *database* dibuat untuk menampung antrean tugas dan metrik sistem.
```sql
CREATE DATABASE carbon_os;
USE carbon_os;

CREATE TABLE jobs (
    id INT AUTO_INCREMENT PRIMARY KEY, 
    command VARCHAR(255) NOT NULL, 
    status ENUM('pending', 'running', 'completed', 'failed', 'timeout') DEFAULT 'pending', 
    submitted_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP, 
    completed_at TIMESTAMP NULL, 
    pid INT NULL
); 

CREATE TABLE system_metrics (
    id INT AUTO_INCREMENT PRIMARY KEY, 
    timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP, 
    cpu_usage FLOAT, 
    ram_usage FLOAT, 
    is_off_peak BOOLEAN, 
    energy_saved_est FLOAT
);.

---
```
### Tahap 2: Setup Environment & Dashboard (PHP)
Seluruh file antarmuka pengguna (index.php, db.php) ditempatkan di direktori web server /var/www/html/. Hak akses diatur agar Apache dapat membacanya:

```sudo chown -R www-data:www-data /var/www/html/```
```sudo chmod -R 755 /var/www/html/```

### Tahap 3: Personalisasi Desktop
Agar sistem memiliki identitas unik:

- Hostname: Diubah menjadi carbon-os.

- Wallpaper: Menggunakan file awareOS.png yang ditetapkan secara global di skema GNOME.

### Tahap 4: Otomatisasi (Auto-Initialization)
Untuk mengatasi masalah penghapusan database oleh sistem eggs saat pembuatan ISO, ditanamkan skrip Auto-Init ```(/usr/local/bin/carbon-autostart.sh)``` yang berjalan otomatis saat instalasi pertama.

### Tahap 5: Mastering (Build ISO)
Pengemasan OS dilakukan dengan mengkompresi seluruh sistem menjadi file .iso menggunakan ```penguins-eggs:```

```sudo eggs produce --max --prefix "carbon-awareOs"```

---
## 5. Panduan Instalasi & Penggunaan

A. Menjalankan Sistem
1.  Boot menggunakan file Carbon-awareOs.iso (via Flashdisk/VirtualBox).
2. Sistem akan masuk ke Live Mode.
3. Untuk menginstal permanen, buka terminal dan jalankan installer Calamares:
    ```sudo calamares```
4. Setelah instalasi, buka browser Firefox dan akses ```http://localhost``` untuk melihat Web Dashboard.

B. Demonstrasi
Untuk mencoba fitur Demand-Shifting, gunakan utilitas terminal carbon-submit.

1. Memasukkan Tugas Berat ke Antrean:
Sebagai contoh, kita akan melakukan backup sistem:
```carbon-submit 'tar -czf /tmp/backup_sistem.tar.gz /var/www/html' ```
Hasil: Di Web Dashboard, tugas ini akan masuk antrean dengan status "MENUNGGU JAM LENGANG" (Kuning) karena menunda penggunaan CPU.

2. Eksekusi Otomatis (Off-Peak):
Saat waktu sistem memasuki jam ramah lingkungan (Off-Peak), status di Dashboard akan otomatis berubah menjadi "SELESAI" (Hijau).

3. Verifikasi Bukti Eksekusi:
Buktikan bahwa file berhasil dieksekusi oleh background service dengan mengecek ukurannya:
```ls -lh /tmp/backup_sistem.tar.gz```

---
## Langkah 6: Troubleshooting
1. Beberapa isu yang mungkin terjadi saat penggunaan mode Live/Instalasi Baru:
Web Dashboard Error 500: Terjadi jika MySQL belum menyala sempurna. Solusi: ```sudo systemctl restart mysql apache2.```

2. Tidak Bisa Akses Internet (DNS Error): Jika ping 8.8.8.8 berhasil tapi browser gagal, perbaiki DNS dengan:
```echo "nameserver 8.8.8.8" | sudo tee /etc/resolv.conf``` 
```sudo systemctl restart NetworkManager```
