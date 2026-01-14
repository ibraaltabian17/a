
# LAPORAN MINI RISET: SISTEM DATABASE

### 1. Judul Riset
**"Analisis dan Perancangan Basis Data Relasional untuk Manajemen Kedisiplinan Siswa di Sekolah Berbasis Data (SIPADIS)"**

### 2. Pembagian Tugas Kelompok (Role-Play)
*   **Analist Sistem:** Menganalisis alur izin, membedakan antara "Izin Harian" (Sakit/Alpha) dengan "Izin Keluar Jam Pelajaran" (Dispen Sementara/Pulang Awal). Menentukan aturan poin.
*   **Database Designer:** Merancang ERD dan melakukan normalisasi 3NF. Menentukan relasi agar status "Dispensasi" tidak dianggap "Bolos".
*   **Backend Implementer:** Menulis kode SQL (DDL & DML) untuk tabel Siswa, Guru, dan Transaksi Izin, serta memastikan *constraint* tanggal valid.
*   **Documentation Officer:** Menyusun skenario pengujian untuk menjawab pertanyaan guru tentang frekuensi izin dan siswa yang belum kembali.

### 3. Identifikasi Struktur Data (Entitas & Atribut)
Untuk memenuhi syarat minimal 5 tabel dan logika yang dalam, berikut rancangannya:
1.  **`Kelas`**: Pengelompokan siswa.
2.  **`Siswa`**: Data utama siswa.
3.  **`Guru_Staf`**: Pihak yang mengotorisasi izin (Audit Trail).
4.  **`Jenis_Pelanggaran_Izin`**: Tabel referensi (Master) yang menyimpan bobot poin dan status akademik.
5.  **`Riwayat_Aktivitas`**: Tabel transaksi utama (Gabungan Absensi, Pelanggaran, dan Izin Keluar).

---

### 4. Tahapan Pelaksanaan Riset

#### Tahap 1: Tahap Analisis (Requirements)
*   **Masalah:** Sekolah kesulitan membedakan siswa yang bolos (Alpha) dengan siswa tugas sekolah (Dispen).
*   **Solusi:** Dibutuhkan sistem yang mencatat **Poin Pelanggaran** dan **Status Kembali**.
*   **Jenis Data:**
    *   *Alpha/Bolos:* Poin 10, Status Tidak Hadir.
    *   *Sakit:* Poin 0, Status Tidak Hadir (Valid).
    *   *Izin Keluar (Dispen Sementara):* Poin 0, Status Hadir (Tapi perlu dicatat jam kembali).

#### Tahap 2: Tahap Desain (Modeling)
*   **ERD Concept:** `Siswa` melakukan `Riwayat_Aktivitas` yang disetujui `Guru_Staf` berdasarkan kategori `Jenis_Pelanggaran_Izin`.
*   **Normalisasi:** Memisahkan data master (Jenis Izin) agar jika bobot poin berubah, tidak perlu ubah data transaksi.

#### Tahap 3: Tahap Implementasi (Coding)
*(Detail kode SQL ada di Bab Analisis Data)*

#### Tahap 4: Tahap Pengujian (Query Testing)
Fokus menjawab 3 pertanyaan wajib di soal:
1.  Mendeteksi siswa yang sering izin keluar.
2.  Menghitung total poin (Otomasi).
3.  Melacak siswa yang "kabur" (Belum kembali).

---

### 5. Sistematika Laporan Mini Riset

#### 1. Pendahuluan: Latar Belakang Masalah
Sekolah sering kehilangan jejak siswa di jam pelajaran. Pencatatan manual di buku piket tidak bisa memberi rekap otomatis ("Siapa siswa yang poinnya sudah 100?"). Selain itu, sering terjadi kerancuan data antara siswa yang benar-benar sakit dengan siswa yang membolos. Dibutuhkan database yang menegakkan integritas data dan mampu melacak status "Izin Keluar" secara *real-time*.

#### 2. Landasan Teori
Menggunakan **RDBMS MySQL** dengan prinsip **Integritas Referensial** (Foreign Key). Sistem ini menerapkan **Logika Boolean** untuk membedakan kehadiran fisik vs kehadiran akademik (Dispensasi).

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

-- 3. Tabel Guru (Otorisator)
CREATE TABLE Guru_Staf (
    nip VARCHAR(20) PRIMARY KEY,
    nama_guru VARCHAR(100)
);

-- 4. Tabel Jenis (Otak Logika Poin & Kehadiran)
CREATE TABLE Jenis_Pelanggaran_Izin (
    kode_jenis CHAR(3) PRIMARY KEY,
    nama_aktivitas VARCHAR(50), -- Contoh: Alpha, Sakit, Izin Keluar
    poin INT DEFAULT 0,
    wajib_kembali BOOLEAN DEFAULT 0 -- 1=Izin Keluar (Harus balik), 0=Sakit/Alpha (Tidak balik)
);

-- 5. Tabel Riwayat Aktivitas (Transaksi Gabungan)
CREATE TABLE Riwayat_Aktivitas (
    id BIGINT AUTO_INCREMENT PRIMARY KEY,
    nis VARCHAR(10),
    kode_jenis CHAR(3),
    nip_pencatat VARCHAR(20),
    tgl_kejadian DATE,
    jam_keluar TIME,           -- Untuk Izin Keluar/Bolos
    jam_kembali_realisasi TIME DEFAULT NULL, -- NULL artinya BELUM KEMBALI
    keterangan TEXT,
    FOREIGN KEY (nis) REFERENCES Siswa(nis),
    FOREIGN KEY (kode_jenis) REFERENCES Jenis_Pelanggaran_Izin(kode_jenis),
    FOREIGN KEY (nip_pencatat) REFERENCES Guru_Staf(nip)
);
```

**b. Kardinalitas:**
*   Siswa *One-to-Many* Riwayat (Satu siswa, banyak pelanggaran/izin).
*   Guru *One-to-Many* Riwayat (Satu guru mencatat banyak kejadian).

**c. Flowchart:**
Mulai -> Siswa Lapor -> Guru Pilih Jenis (Pelanggaran/Izin) -> Input Database -> (Jika Izin Keluar: Input Jam Keluar) -> Siswa Kembali -> Guru Update Jam Kembali -> Selesai.

#### 4. Analisis Data: Hasil Eksekusi Query SQL

**a. Insert Data Contoh (Skenario)**
```sql
-- Setup Data
INSERT INTO Kelas VALUES ('XRPL', 'X RPL 1');
INSERT INTO Siswa VALUES ('101', 'Budi', 'XRPL'), ('102', 'Siti', 'XRPL');
INSERT INTO Guru_Staf VALUES ('G01', 'Pak Guru');

-- Setup Jenis Logika
INSERT INTO Jenis_Pelanggaran_Izin VALUES 
('A', 'Alpha', 10, 0),          -- Pelanggaran Berat
('S', 'Sakit', 0, 0),           -- Izin Harian
('IK', 'Izin Keluar', 0, 1);    -- Izin Sebentar (Wajib Kembali)

-- Transaksi (Skenario Jawaban Soal)
-- 1. Budi Alpha
INSERT INTO Riwayat_Aktivitas (nis, kode_jenis, nip_pencatat, tgl_kejadian, keterangan)
VALUES ('101', 'A', 'G01', '2023-10-01', 'Bolos');

-- 2. Siti Izin Keluar (Dispen Osis) - Sering (4x seminggu)
INSERT INTO Riwayat_Aktivitas (nis, kode_jenis, nip_pencatat, tgl_kejadian, jam_keluar, jam_kembali_realisasi) VALUES 
('102', 'IK', 'G01', '2023-10-02', '08:00', '09:00'),
('102', 'IK', 'G01', '2023-10-03', '08:00', '09:00'),
('102', 'IK', 'G01', '2023-10-04', '08:00', '09:00'),
('102', 'IK', 'G01', '2023-10-05', '08:00', NULL); -- NULL = Belum Kembali!
```

**b. Jawaban Query Testing (Sesuai Soal Poin 4)**

**1. "Siapa saja siswa yang pernah izin keluar lebih dari 3 kali dalam seminggu?"**
```sql
SELECT s.nama_lengkap, COUNT(*) as frekuensi_izin
FROM Riwayat_Aktivitas r
JOIN Siswa s ON r.nis = s.nis
WHERE r.kode_jenis = 'IK' -- Filter: Izin Keluar
  AND WEEK(r.tgl_kejadian) = WEEK('2023-10-05') -- Minggu tertentu
GROUP BY s.nama_lengkap
HAVING frekuensi_izin > 3;
```
*(Hasil: Siti muncul karena izin 4 kali).*

**2. "Berapa total poin pelanggaran yang dimiliki siswa tertentu?"**
```sql
SELECT s.nama_lengkap, SUM(j.poin) as total_poin
FROM Riwayat_Aktivitas r
JOIN Siswa s ON r.nis = s.nis
JOIN Jenis_Pelanggaran_Izin j ON r.kode_jenis = j.kode_jenis
WHERE s.nama_lengkap = 'Budi'
GROUP BY s.nama_lengkap;
```
*(Hasil: Budi memiliki total poin 10 dari Alpha).*

**3. "Tampilkan daftar siswa yang belum kembali ke sekolah setelah izin keluar."**
Query ini sangat penting untuk keamanan siswa. Logikanya: Cari yang jenisnya 'Izin Keluar' DAN jam kembalinya masih KOSONG (NULL).
```sql
SELECT s.nama_lengkap, r.jam_keluar, r.keterangan
FROM Riwayat_Aktivitas r
JOIN Siswa s ON r.nis = s.nis
JOIN Jenis_Pelanggaran_Izin j ON r.kode_jenis = j.kode_jenis
WHERE j.wajib_kembali = 1    -- Hanya jenis izin yang harusnya balik
  AND r.jam_kembali_realisasi IS NULL -- Tapi datanya masih kosong
  AND r.tgl_kejadian = CURDATE(); -- Kejadian Hari Ini
```
*(Hasil: Siti muncul karena data terakhirnya jam kembalinya NULL).*

#### 5. Kesimpulan
Perancangan database ini berhasil memecahkan masalah pemantauan siswa.
1.  **Guru BK** terbantu karena **Total Poin** dihitung otomatis oleh sistem, menghindari kesalahan hitung manual.
2.  **Guru Piket/Wali Kelas** terbantu dengan fitur deteksi **"Belum Kembali"**, sehingga keamanan siswa yang izin keluar (Dispen/Beli bahan) dapat dipantau secara *real-time*.
3.  Pemisahan tabel `Jenis_Pelanggaran_Izin` membuat sistem fleksibel; membedakan antara Alpha (Hukuman) dan Izin Keluar (Administrasi) dalam satu database terintegrasi.
