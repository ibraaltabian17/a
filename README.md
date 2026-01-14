
1.  **Izin Pulang Awal (IPA):** Siswa sakit parah atau ada urusan keluarga mendadak di tengah jam sekolah. -> **Tidak Wajib Kembali.**
2.  **Izin Keluar Lingkungan (IKL):** Keluar gerbang sekolah sebentar (fotokopi di luar, beli bahan praktek, tugas OSIS beli konsumsi). -> **Wajib Kembali.**
3.  **Dispensasi (DIS):** Kegiatan Organisasi/Ekskul/Lomba yang menyebabkan tidak ikut KBM di kelas, tapi siswa masih dianggap hadir secara akademik.
4.  **UKS/Toilet:** Tidak dicatat di database ini (kecuali kalau dari UKS terus diputuskan pulang).

---

# LAPORAN MINI RISET: SISTEM DATABASE

### 1. Judul Riset
**"Analisis dan Perancangan Basis Data Relasional untuk Manajemen Kedisiplinan dan Perizinan Siswa (SIPADIS)"**

### 2. Pembagian Tugas Kelompok (Role-Play)
*   **Analist Sistem:** Memilah kategori izin: mana yang "Pulang Seterusnya" (Sakit/Izin Pulang) dan mana yang "Keluar Sementara" (Tugas Luar/Beli Bahan). Mengabaikan aktivitas trivial seperti toilet/UKS sebentar.
*   **Database Designer:** Merancang tabel referensi (`Jenis_Izin`) yang memiliki atribut kontrol `wajib_kembali` agar sistem bisa membedakan logika query "Siswa Hilang".
*   **Backend Implementer:** Coding SQL dengan validasi: jika jenis izinnya "Pulang", kolom jam kembali tidak perlu diisi.
*   **Documentation Officer:** Menyusun laporan sesuai sistematika dan memastikan query menjawab pertanyaan tentang siswa yang belum kembali dari izin sementara.

### 3. Identifikasi Struktur Data
5 Tabel Utama:
1.  `Kelas`
2.  `Siswa`
3.  `Guru_Piket` (Pemberi izin gerbang)
4.  `Kategori_Izin` (Master Data: Menyimpan aturan main/logika)
5.  `Jurnal_Aktivitas` (Transaksi harian)

---

### 4. Tahapan Pelaksanaan Riset

#### Tahap 1: Tahap Analisis (Requirements & Logic)
*   **Logika Izin Pulang (IPA):** Jika siswa sakit di jam ke-2 dan pulang, statusnya "Izin Pulang". Tidak perlu dicari jam kembalinya.
*   **Logika Izin Keluar (IKL):** Jika siswa OSIS izin beli paku untuk mading di luar sekolah. Statusnya "Izin Keluar". **Wajib Kembali**. Jika tidak kembali, dianggap membolos/kabur.
*   **Logika Dispen (DIS):** Siswa lomba/rapat OSIS. Tidak mengurangi kehadiran, Poin 0.

#### Tahap 2: Tahap Desain (Modeling)
*   Menambahkan kolom `flag_wajib_kembali` (Boolean) di tabel Kategori.
*   ERD menghubungkan Siswa -> Jurnal <- Guru.

#### Tahap 3 & 4: Implementasi & Testing
*(Dijabarkan di poin 5 bawah)*

---

### 5. Sistematika Laporan Mini Riset

#### 1. Pendahuluan: Latar Belakang Masalah
Sekolah sering kecolongan dengan modus siswa yang izin keluar gerbang (alasan fotokopi/beli bahan) namun tidak kembali lagi ke sekolah. Sistem manual sulit melacak mana siswa yang "Izin Pulang Awal" (resmi selesai) dan mana yang "Izin Keluar Sementara" (harus balik). Selain itu, perlu pemisahan data antara pelanggaran (Alpha) dengan ketidakhadiran yang sah (Dispensasi Organisasi), agar nilai siswa adil.

#### 2. Landasan Teori
Menggunakan **MySQL** sebagai DBMS untuk mengelola relasi antar tabel. Fitur utama yang digunakan adalah `JOIN` untuk menggabungkan data transaksi dengan aturan di tabel master, serta logika `IS NULL` untuk mendeteksi siswa yang belum kembali.

#### 3. Hasil Riset

**a. Daftar Tabel & Skema (Data Types & Constraints)**

```sql
-- 1. Tabel Kelas
CREATE TABLE Kelas (
    id_kelas CHAR(5) PRIMARY KEY,
    nama_kelas VARCHAR(30)
);

-- 2. Tabel Siswa
CREATE TABLE Siswa (
    nis VARCHAR(10) PRIMARY KEY,
    nama_lengkap VARCHAR(100),
    id_kelas CHAR(5),
    FOREIGN KEY (id_kelas) REFERENCES Kelas(id_kelas)
);

-- 3. Tabel Guru Piket (Gatekeeper)
CREATE TABLE Guru_Piket (
    nip VARCHAR(20) PRIMARY KEY,
    nama_guru VARCHAR(100)
);

-- 4. Tabel Kategori Izin (LOGIC ENGINE)
-- Disini letak kecerdasan sistem membedakan Pulang vs Keluar Sebentar
CREATE TABLE Kategori_Izin (
    kode_kat CHAR(3) PRIMARY KEY,
    nama_kategori VARCHAR(50), 
    poin INT DEFAULT 0,
    -- 1 = Harus Balik (Beli bahan/Fotokopi), 0 = Tidak Balik (Sakit/Pulang Awal/Alpha)
    wajib_kembali BOOLEAN DEFAULT 0 
);

-- 5. Tabel Jurnal Aktivitas (Transaksi)
CREATE TABLE Jurnal_Aktivitas (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    nis VARCHAR(10),
    kode_kat CHAR(3),
    nip_pencatat VARCHAR(20),
    tgl_kejadian DATE,
    jam_keluar TIME,
    jam_kembali TIME DEFAULT NULL, -- Kosong berarti belum balik
    keterangan TEXT,
    FOREIGN KEY (nis) REFERENCES Siswa(nis),
    FOREIGN KEY (kode_kat) REFERENCES Kategori_Izin(kode_kat),
    FOREIGN KEY (nip_pencatat) REFERENCES Guru_Piket(nip)
);
```

**b. Flowchart:**
Siswa lapor Guru Piket -> Guru cek alasan:
*   Jika **Sakit/Urgent** -> Pilih kode 'IPA' -> Input Jam -> Siswa Pulang (Selesai).
*   Jika **Tugas Luar/Fotokopi** -> Pilih kode 'IKL' -> Input Jam Keluar -> Siswa Keluar -> (Nanti) Siswa Balik -> Guru Update Jam Kembali.
*   Jika **Bolos** -> Pilih kode 'A' -> Poin bertambah.

#### 4. Analisis Data: Hasil Eksekusi Query SQL

**a. Create Database & Table**
*(Sesuai script SQL di atas)*

**b. Insert Into (Input Contoh Data Skenario Nyata)**

```sql
-- Setup Data Dasar
INSERT INTO Kelas VALUES ('XRPL', 'X RPL');
INSERT INTO Siswa VALUES ('101', 'Budi', 'XRPL'), ('102', 'Siti', 'XRPL'), ('103', 'Edo', 'XRPL');
INSERT INTO Guru_Piket VALUES ('G01', 'Pak Satpam/Piket');

-- Setup Kategori (KUNCI LOGIKA)
INSERT INTO Kategori_Izin VALUES 
('A',   'Alpha / Bolos', 10, 0),         -- Poin 10, Tidak Wajib Kembali (memang tidak masuk)
('IPA', 'Izin Pulang Awal (Sakit)', 0, 0),-- Poin 0, Tidak Wajib Kembali (memang pulang)
('IKL', 'Izin Keluar Lingkungan', 0, 1),  -- Poin 0, WAJIB KEMBALI (Beli bahan/Tugas)
('DIS', 'Dispensasi Organisasi', 0, 0);   -- Poin 0, Dianggap hadir (Internal)

-- Input Transaksi
-- 1. Budi: Sakit di jam ke-3, lalu Pulang. (Tidak perlu dicari jam baliknya)
INSERT INTO Jurnal_Aktivitas (nis, kode_kat, nip_pencatat, tgl_kejadian, jam_keluar, keterangan)
VALUES ('101', 'IPA', 'G01', CURRENT_DATE, '10:00', 'Sakit Maag');

-- 2. Edo: Bolos Sekolah
INSERT INTO Jurnal_Aktivitas (nis, kode_kat, nip_pencatat, tgl_kejadian, keterangan)
VALUES ('103', 'A', 'G01', CURRENT_DATE, 'Tanpa Keterangan');

-- 3. Siti: Izin keluar beli kertas manila untuk mading (Sering keluar)
INSERT INTO Jurnal_Aktivitas (nis, kode_kat, nip_pencatat, tgl_kejadian, jam_keluar, jam_kembali) VALUES 
('102', 'IKL', 'G01', '2023-10-01', '09:00', '09:30'), -- Balik
('102', 'IKL', 'G01', '2023-10-02', '09:00', '09:30'), -- Balik
('102', 'IKL', 'G01', '2023-10-03', '09:00', '09:30'), -- Balik
('102', 'IKL', 'G01', CURRENT_DATE, '08:00', NULL);    -- HARI INI BELUM BALIK!
```

**c. Demonstrasi Query Menjawab Pertanyaan Tugas**

**(1) "Siapa saja siswa yang pernah izin keluar lebih dari 3 kali dalam seminggu?"**
*(Query ini mendeteksi siswa yang sering wara-wiri keluar gerbang)*
```sql
SELECT s.nama_lengkap, COUNT(*) as frekuensi_keluar
FROM Jurnal_Aktivitas j
JOIN Siswa s ON j.nis = s.nis
WHERE j.kode_kat = 'IKL' -- Hanya Izin Keluar Lingkungan (Bukan sakit/bolos)
GROUP BY s.nama_lengkap
HAVING frekuensi_keluar > 3;
```
*Hasil: Siti (4 kali).*

**(2) "Berapa total poin pelanggaran yang dimiliki siswa tertentu?"**
*(Query ini menghitung poin Edo yang bolos)*
```sql
SELECT s.nama_lengkap, SUM(k.poin) as total_poin
FROM Jurnal_Aktivitas j
JOIN Siswa s ON j.nis = s.nis
JOIN Kategori_Izin k ON j.kode_kat = k.kode_kat
WHERE s.nama_lengkap = 'Edo';
```
*Hasil: Edo (10 Poin).*

**(3) "Tampilkan daftar siswa yang belum kembali ke sekolah setelah izin keluar."**
*(Query ini PENTING. Budi (IPA/Sakit) tidak akan muncul, tapi Siti (IKL/Beli bahan) akan muncul karena dia wajib balik tapi datanya NULL)*
```sql
SELECT s.nama_lengkap, j.jam_keluar, k.nama_kategori
FROM Jurnal_Aktivitas j
JOIN Siswa s ON j.nis = s.nis
JOIN Kategori_Izin k ON j.kode_kat = k.kode_kat
WHERE k.wajib_kembali = 1      -- Filter: Hanya jenis izin yang harus balik
  AND j.jam_kembali IS NULL    -- Filter: Data kembali masih kosong
  AND j.tgl_kejadian = CURRENT_DATE;
```
*Hasil: Siti (Jam keluar 08:00, Status: Izin Keluar Lingkungan).*

#### 5. Kesimpulan
Database ini memberikan solusi cerdas untuk membedakan jenis ketidakhadiran:
1.  **Akurasi Keamanan:** Sistem dapat membedakan mana siswa yang "Pulang Resmi" (Sakit/IPA) dan mana siswa yang "Berpotensi Kabur" (Izin Keluar/IKL). Query nomor 3 membuktikan bahwa Budi yang pulang sakit tidak dianggap "belum kembali", sedangkan Siti yang keluar beli bahan dan belum kembali langsung terdeteksi.
2.  **Pemantauan Organisasi:** Siswa yang memiliki aktivitas organisasi (Dispensasi) tercatat rapi tanpa menambah poin pelanggaran, melindungi nilai akademik siswa aktif.
3.  **Otomasi BK:** Perhitungan poin pelanggaran (Alpha) dilakukan otomatis, memudahkan BK mengambil tindakan disiplin.
