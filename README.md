Tentu, mari kita bedah topik ini sedalam-dalamnya. Karena kamu memilih fokus **"Perijinan/Presensi Ketidakhadiran"** (Sakit, Izin, Alpha, Dispensasi, Pulang Awal), kita akan merancang sistem yang kuat (robust) agar tidak ada celah logika.

Berikut adalah **Rancangan Mini Riset** lengkap dengan analisis mendalam, skenario konsekuensi, dan implementasi teknisnya.

---

### 1. Judul Riset
**"Analisis dan Perancangan Basis Data Relasional untuk Sistem Manajemen Presensi, Dispensasi, dan Ketidakhadiran Siswa Terintegrasi (SIPADIS)"**

### 2. Pembagian Tugas Kelompok (Role-Play)
*   **Analist Sistem:** Menganalisis alur "Surat masuk" menjadi "Data Digital". Menentukan aturan bisnis (contoh: Apakah sakit 3 hari butuh 1 input atau 3 input? Apakah Dispen menghapus status Alpha?).
*   **Database Designer:** Mendesain ERD untuk menangani hubungan *Many-to-Many* (Satu siswa bisa banyak izin, satu jenis izin bisa dipakai banyak siswa). Menjamin integritas referensial.
*   **Backend Implementer:** Menulis *Stored Procedure* atau *Query* kompleks untuk menghitung rekapitulasi bulanan otomatis.
*   **Documentation Officer:** Menyusun skenario pengujian "Edge Case" (Kasus ekstrim, misal: Izin sakit tapi di hari libur).

### 3. Identifikasi Struktur Data (Entitas & Atribut)
Untuk memenuhi syarat minimal 5 tabel dan berpikir mendalam tentang konsekuensi data, berikut tabel yang kita butuhkan:

1.  **`Siswa`**: Data master siswa.
    *   *Atribut:* `nis` (PK), `nama_lengkap`, `jenis_kelamin`, `id_kelas` (FK), `no_hp_ortu`.
2.  **`Kelas`**: Pengelompokan siswa agar laporan bisa per kelas.
    *   *Atribut:* `id_kelas` (PK), `nama_kelas` (X-RPL-1, dll), `nip_wali_kelas` (FK).
3.  **`Guru_Staf`**: Siapa yang berhak menyetujui izin/mencatat data (Guru Piket/Wali Kelas).
    *   *Atribut:* `nip` (PK), `nama_guru`, `jabatan`.
4.  **`Jenis_Ketidakhadiran`**: Tabel referensi (Lookup Table) untuk standarisasi dan pembobotan poin.
    *   *Atribut:* `kode_jenis` (PK), `nama_jenis` (Sakit, Izin, Alpha, Dispen, Pulang Paksa), `poin_pelanggaran` (Int), `status_akademik` (Hadir/Tidak Hadir).
    *   *Analisis:* Dispen poinnya 0 dan status akademik "Hadir". Alpha poinnya misal 5 dan status "Tidak Hadir".
5.  **`Riwayat_Perizinan`**: Tabel Transaksi utama.
    *   *Atribut:* `id_izin` (PK), `nis` (FK), `kode_jenis` (FK), `nip_pencatat` (FK), `tanggal_mulai`, `tanggal_selesai`, `waktu_input` (Timestamp), `keterangan_detail`, `file_bukti` (path foto surat).

---

### 4. Tahapan Pelaksanaan Riset

#### Tahap 1: Tahap Analisis (Deep Analysis & Consequences)

Di sini kita membedah "Kemungkinan dari Kemungkinan":

*   **Masalah Ambiguitas:** Bagaimana jika siswa izin "Pulang Awal" jam 10 pagi?
    *   *Konsekuensi:* Jika sistem hanya mencatat "Hadir" atau "Absen", data ini hilang.
    *   *Solusi:* Kita butuh jenis izin khusus "Izin Pulang Awal" (IPA). Secara absensi pagi dia hadir, tapi secara KBM jam terakhir dia absen.
*   **Masalah Rentang Waktu:** Siswa sakit Demam Berdarah selama 7 hari.
    *   *Konsekuensi:* Input data 7 kali sangat melelahkan bagi Guru Piket.
    *   *Solusi:* Desain tabel harus menggunakan `tanggal_mulai` dan `tanggal_selesai`. Query nanti yang akan menghitung durasinya (`DATEDIFF`).
*   **Masalah Dispensasi (Dispen):** Siswa ikut lomba Futsal mewakili sekolah.
    *   *Konsekuensi:* Guru mapel mungkin mengira siswa bolos (Alpha). Jika tercatat Alpha, nilai sikap turun.
    *   *Solusi:* Status Dispen harus dihitung sebagai "Hadir" dalam rekap akademik, tapi ada catatan "Dinas Luar".
*   **Masalah Integritas:** Siswa mengaku sakit tapi tidak ada surat.
    *   *Konsekuensi:* Penyalahgunaan izin.
    *   *Solusi:* Kolom `file_bukti` bersifat *nullable* tapi sistem memberi peringatan jika kosong lebih dari 2 hari (status *pending*).

#### Tahap 2: Tahap Desain (Modeling & Normalisasi)

*   **ERD Logic:**
    *   `Kelas` 1 -- N `Siswa` (Satu kelas banyak siswa).
    *   `Siswa` 1 -- N `Riwayat_Perizinan` (Satu siswa bisa punya banyak riwayat izin).
    *   `Jenis_Ketidakhadiran` 1 -- N `Riwayat_Perizinan` (Satu jenis izin dipakai di banyak riwayat).
    *   `Guru_Staf` 1 -- N `Riwayat_Perizinan` (Satu guru piket mencatat banyak izin).

*   **Normalisasi (3NF):**
    *   Jangan simpan `nama_kelas` di tabel `Siswa`, simpan `id_kelas` saja.
    *   Jangan simpan teks "Sakit" atau "Alpha" berulang-ulang di tabel `Riwayat`, gunakan `kode_jenis` (misal: 'S', 'I', 'A', 'D'). Ini menghemat penyimpanan dan memudahkan perubahan nama kategori.

#### Tahap 3: Tahap Implementasi (Coding SQL Preview)

*(Akan didetailkan di bagian Laporan Hasil Riset)*

#### Tahap 4: Tahap Pengujian (Query Testing - The Deep Questions)

1.  **Deteksi Pola Bolos:** *"Tampilkan siswa yang Alpha atau Izin tepat pada hari 'Senin' lebih dari 2 kali dalam sebulan."* (Mendeteksi siswa yang malas upacara).
2.  **Rekap Poin:** *"Hitung total poin pelanggaran siswa X, di mana Dispen tidak menambah poin, tapi Alpha menambah 5 poin."*
3.  **Cross-Check Dispen:** *"Siapa saja siswa yang statusnya Dispen hari ini? (Untuk guru mapel mengecek agar tidak dialphakan)."*

---

### 5. Sistematika Laporan Mini Riset

#### 1. Pendahuluan
Membahas masalah administrasi sekolah manual. Surat izin sering hilang, rekapitulasi kehadiran memakan waktu, dan sering terjadi kesalahan komunikasi antara Guru Piket dan Wali Kelas mengenai status "Dispen" vs "Bolos".

#### 2. Landasan Teori
*   **RDBMS (MySQL):** Mengapa cocok untuk data terstruktur sekolah.
*   **Data Integrity:** Pentingnya Foreign Key agar tidak ada data izin untuk siswa yang sudah keluar/tidak terdaftar.

#### 3. Hasil Riset

**a. Skema Tabel (Data Types & Constraints)**

```sql
-- 1. Tabel Kelas
CREATE TABLE Kelas (
    id_kelas VARCHAR(10) PRIMARY KEY,
    nama_kelas VARCHAR(50) NOT NULL,
    wali_kelas VARCHAR(50)
);

-- 2. Tabel Siswa
CREATE TABLE Siswa (
    nis VARCHAR(10) PRIMARY KEY,
    nama_lengkap VARCHAR(100) NOT NULL,
    id_kelas VARCHAR(10),
    no_hp_ortu VARCHAR(15),
    FOREIGN KEY (id_kelas) REFERENCES Kelas(id_kelas)
);

-- 3. Tabel Guru/Staf
CREATE TABLE Guru_Staf (
    nip VARCHAR(20) PRIMARY KEY,
    nama_guru VARCHAR(100) NOT NULL,
    jabatan VARCHAR(50)
);

-- 4. Tabel Jenis Ketidakhadiran (Otak dari logika bisnis)
CREATE TABLE Jenis_Ketidakhadiran (
    kode_jenis CHAR(3) PRIMARY KEY, -- 'S', 'I', 'A', 'D', 'IPA'
    keterangan VARCHAR(50) NOT NULL,
    poin_pelanggaran INT DEFAULT 0, -- Alpha=5, Sakit=0
    is_counted_as_present BOOLEAN -- Dispen = True, Alpha = False
);

-- 5. Tabel Riwayat Perizinan (Transaksi)
CREATE TABLE Riwayat_Perizinan (
    id_izin INT AUTO_INCREMENT PRIMARY KEY,
    nis VARCHAR(10),
    kode_jenis CHAR(3),
    nip_pencatat VARCHAR(20),
    tanggal_mulai DATE NOT NULL,
    tanggal_selesai DATE NOT NULL,
    alasan_detail TEXT,
    FOREIGN KEY (nis) REFERENCES Siswa(nis),
    FOREIGN KEY (kode_jenis) REFERENCES Jenis_Ketidakhadiran(kode_jenis),
    FOREIGN KEY (nip_pencatat) REFERENCES Guru_Staf(nip)
);
```

**b. Gambar ERD:**
*(Deskripsikan panah relasi sesuai kode di atas)*

**c. Flowchart:**
Mulai -> Siswa membawa surat/bukti -> Guru Piket Cek Validitas -> (Jika Valid) -> Pilih Jenis (S/I/A/D) -> Input Tanggal Mulai & Selesai -> Simpan ke Database -> Selesai.

#### 4. Analisis Data: Hasil Eksekusi Query

**a. Insert Data Contoh (Skenario Nyata)**

```sql
-- Setup Data Dasar
INSERT INTO Kelas VALUES ('X-RPL', 'X Rekayasa Perangkat Lunak', 'Pak Budi');
INSERT INTO Siswa VALUES ('1001', 'Ahmad Dani', 'X-RPL', '08123456789');
INSERT INTO Siswa VALUES ('1002', 'Budi Santoso', 'X-RPL', '08129876543');
INSERT INTO Guru_Staf VALUES ('198001', 'Bu Siti', 'Guru Piket');

-- Setup Jenis Izin (PENTING: Logika Poin & Kehadiran)
INSERT INTO Jenis_Ketidakhadiran VALUES 
('S', 'Sakit', 0, 0), -- Poin 0, Tidak Hadir
('I', 'Izin Keluarga', 0, 0), -- Poin 0, Tidak Hadir
('A', 'Alpha/Tanpa Ket', 10, 0), -- Poin 10, Tidak Hadir
('D', 'Dispensasi Sekolah', 0, 1), -- Poin 0, DIHITUNG HADIR
('IPA', 'Izin Pulang Awal', 2, 1); -- Poin 2, DIHITUNG HADIR (tapi pulang)

-- Transaksi (Skenario)
-- 1. Ahmad Dani Sakit 3 Hari
INSERT INTO Riwayat_Perizinan (nis, kode_jenis, nip_pencatat, tanggal_mulai, tanggal_selesai, alasan_detail) 
VALUES ('1001', 'S', '198001', '2023-10-01', '2023-10-03', 'Sakit Tifus ada surat dokter');

-- 2. Budi Santoso Alpha 1 Hari
INSERT INTO Riwayat_Perizinan (nis, kode_jenis, nip_pencatat, tanggal_mulai, tanggal_selesai, alasan_detail) 
VALUES ('1002', 'A', '198001', '2023-10-02', '2023-10-02', 'Tidak ada kabar');

-- 3. Ahmad Dani Dispensasi Lomba (Overlap tanggal beda bulan)
INSERT INTO Riwayat_Perizinan (nis, kode_jenis, nip_pencatat, tanggal_mulai, tanggal_selesai, alasan_detail) 
VALUES ('1001', 'D', '198001', '2023-10-20', '2023-10-20', 'Lomba LKS Web Tech');
```

**b. Query Pengujian (Jawaban Pertanyaan Riset)**

*   **Pertanyaan 1:** "Siapa siswa yang statusnya Alpha minggu ini dan berapa poin dendanya?"
    ```sql
    SELECT s.nama_lengkap, k.nama_kelas, j.poin_pelanggaran
    FROM Riwayat_Perizinan rp
    JOIN Siswa s ON rp.nis = s.nis
    JOIN Kelas k ON s.id_kelas = k.id_kelas
    JOIN Jenis_Ketidakhadiran j ON rp.kode_jenis = j.kode_jenis
    WHERE rp.kode_jenis = 'A';
    ```

*   **Pertanyaan 2:** "Rekapitulasi Absensi: Siapa siswa yang sering 'Pulang Awal' (Modus cabut pelajaran)?"
    ```sql
    SELECT s.nama_lengkap, COUNT(*) as frekuensi_pulang_awal
    FROM Riwayat_Perizinan rp
    JOIN Siswa s ON rp.nis = s.nis
    WHERE rp.kode_jenis = 'IPA' -- Izin Pulang Awal
    GROUP BY s.nama_lengkap
    HAVING COUNT(*) > 0;
    ```

*   **Pertanyaan 3:** "Daftar siswa yang Dispensasi hari ini (Agar tidak di-Alpha-kan oleh guru mapel)"
    ```sql
    SELECT s.nama_lengkap, k.nama_kelas, rp.alasan_detail
    FROM Riwayat_Perizinan rp
    JOIN Siswa s ON rp.nis = s.nis
    JOIN Kelas k ON s.id_kelas = k.id_kelas
    WHERE rp.kode_jenis = 'D' AND CURDATE() BETWEEN rp.tanggal_mulai AND rp.tanggal_selesai;
    ```

#### 5. Kesimpulan
Database ini memecahkan masalah ambiguitas antara "Tidak Hadir karena Bolos" dan "Tidak Hadir karena Tugas Sekolah (Dispen)". Dengan memisahkan tabel referensi `Jenis_Ketidakhadiran` yang memiliki atribut `poin` dan `status_akademik`, sistem dapat secara otomatis menghitung mana siswa yang bermasalah (perlu dipanggil BK) dan mana siswa yang berprestasi (sering Dispen lomba), tanpa intervensi manual yang rentan kesalahan.
