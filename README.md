# TUGAS-1-NRG-DWBI# Panduan Eksekusi ETL Data Warehouse (Pentaho Data Integration)

Repositori ini berisi kumpulan file transformasi (`.ktr`) dan job (`.kjb`) Pentaho Data Integration (Spoon) untuk memuat data dari sumber operasional (OLTP) ke dalam Data Warehouse (OLAP).

Ikuti langkah-langkah di bawah ini untuk menjalankan proses ETL dari kondisi awal (database kosong).

## 1. Persiapan Awal (Prerequisites)
Pastikan perangkat lunak berikut telah terinstal dan berjalan:
1. **Pentaho Data Integration (PDI / Spoon)** versi stabil terbaru.
2. **Database Management System (DBMS)** (misal: MySQL, PostgreSQL, atau SQL Server) yang sudah berjalan.
3. Driver JDBC (misal: `mysql-connector-java.jar` atau `postgresql.jar`) sudah diletakkan di dalam folder `\data-integration\lib` pada direktori instalasi Pentaho Anda.

## 2. Inisialisasi Skema Database (DDL)
Sebelum menjalankan Pentaho, tabel-tabel di Data Warehouse harus dibuat terlebih dahulu.
1. Buka database client/IDE Anda (DBeaver, phpMyAdmin, pgAdmin, dll).
2. Buat database baru, misalnya dengan nama `dw_nrg`.
3. Jalankan script SQL (DDL) untuk membuat tabel dimensi dan tabel fakta (pastikan struktur tabel sesuai dengan skema yang digunakan di transformasi Pentaho).

## 3. Konfigurasi Koneksi Database di Pentaho
Agar file transformasi (`.ktr`) bisa membaca dan menulis data, Anda harus membuat koneksi database di Pentaho.

1. Buka Pentaho Spoon.
2. Buka salah satu file transformasi (misal: `tr_load_dim_produk.ktr`).
3. Di tab **View** (sebelah kiri), klik kanan pada **Database connections** lalu pilih **New**.
4. Buat dua koneksi utama berikut:

   **A. Koneksi Database Sumber (OLTP)**
   * **Connection Name**: `conn_dw_nrg` *(Catatan: Nama ini harus sama persis dengan yang dipilih pada step Table Input)*
   * **Connection Type**: Pilih sesuai DBMS sumber Anda (misal: MySQL)
   * **Host Name**: `localhost` (atau IP server sumber)
   * **Database Name**: `dw_nrg` (nama database sumber)
   * **Username/Password**: *Sesuai kredensial Anda*

   **B. Koneksi Database Tujuan (Data Warehouse / OLAP)**
   * **Connection Name**: `conn_dw_nrg` *(Catatan: Nama ini harus sama persis dengan yang dipilih pada step Table Output/Dimension Lookup/Update)*
   * **Connection Type**: Pilih sesuai DBMS tujuan Anda
   * **Host Name**: `localhost` 
   * **Database Name**: `dw_nrg`
   * **Username/Password**: *Sesuai kredensial Anda*

5. Klik tombol **Test** untuk memastikan koneksi berhasil, lalu klik **OK**. 
*(Anda mungkin perlu membagikan/Share koneksi ini dengan klik kanan pada nama koneksi -> Share, agar bisa digunakan di semua file .ktr)*.

## 4. Urutan Eksekusi Transformasi (.ktr)
Proses pengisian tabel fakta **sangat bergantung** pada tabel dimensi. Oleh karena itu, tabel dimensi wajib diisi (di-load) terlebih dahulu. Jika Anda menjalankan secara manual tanpa *Job*, jalankan file `.ktr` dengan urutan mutlak berikut:

### Tahap 1: Load Semua Dimensi (Wajib Pertama)
1. `tr_load_dim_waktu.ktr` (Jika tidak generate otomatis via Pentaho, ambil dari tabel sumber)
2. `tr_load_dim_kanal.ktr`
3. `tr_load_dim_produk.ktr`
4. `tr_load_dim_promo.ktr`
5. `tr_load_dim_pelanggan.ktr` (Load pelanggan reguler)
6. `tr_load_dim_pelanggan_mpl.ktr` (Load pelanggan marketplace)

*Tips: Pastikan baris default (seperti 'Tidak Diketahui' atau 'Non-Member') berhasil masuk ke `dim_pelanggan` dan `dim_produk` pada tahap ini.*

### Tahap 2: Load Tabel Fakta (Wajib Terakhir)
Setelah **semua** transformasi dimensi sukses dijalankan, baru Anda boleh menjalankan:
7. `tr_load_fact_penjualan.ktr`
8. `tr_load_fact_s3.ktr`

## 5. Cara Menjalankan Menggunakan File Job (.kjb) (Direkomendasikan)
Untuk menghindari kesalahan urutan eksekusi, sangat disarankan menggunakan file **Job** (`jb_main_etl.kjb`). Job ini sudah mengatur urutan (orchestration) dari seluruh file `.ktr` di atas.

1. Buka Pentaho Spoon.
2. Pilih **File** -> **Open** -> Cari dan pilih `jb_main_etl.kjb`.
3. Tekan tombol **Play / Run** (ikon segitiga hijau) di toolbar atas.
4. Pada jendela *Execute a job*, biarkan pengaturan default, lalu klik **Run**.
5. Perhatikan tab **Execution Results** di bagian bawah layar:
   * Ikon centang hijau menandakan proses sukses.
   * Ikon silang merah menandakan ada error (cek tab *Logging* untuk melihat detail error).

## 6. Troubleshooting Umum
* **Error "Driver class not found"**: Anda lupa memasukkan file `.jar` JDBC ke folder `lib` Pentaho. Tutup Pentaho, masukkan file `.jar`, lalu buka kembali.
* **Error "Table or view does not exist"**: Anda belum menjalankan script DDL pembuatan tabel di database, atau nama tabel di Pentaho tidak sesuai dengan di database.
* **Tabel Fakta banyak bernilai NULL di kolom ID Dimensi (Surrogate Key)**: Pastikan transformasi dimensi sudah dijalankan *sebelum* fakta, dan pastikan tipe data pencarian (Lookup) cocok antara tabel sumber dan tabel dimensi.
