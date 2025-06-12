# Fundamental SQL Lengkap: Panduan Komprehensif untuk Persiapan Kerja

```markdown
# 🚀 Fundamental SQL Komprehensif: Dari Dasar hingga Siap Kerja

## Daftar Isi
1. [Pengenalan SQL](#1-pengenalan-sql)
2. [Struktur Database](#2-struktur-database)
3. [DDL (Data Definition Language)](#3-ddl-data-definition-language)
4. [DML (Data Manipulation Language)](#4-dml-data-manipulation-language)
5. [Query Dasar](#5-query-dasar)
6. [Fungsi Agregasi](#6-fungsi-agregasi)
7. [Join dan Relasi Tabel](#7-join-dan-relasi-tabel)
8. [Subquery](#8-subquery)
9. [Window Functions](#9-window-functions)
10. [Optimasi Query](#10-optimasi-query)
11. [Best Practices](#11-best-practices)
12. [Studi Kasus Real-World](#12-studi-kasus-real-world)
13. [Latihan dan Solusi](#13-latihan-dan-solusi)

---

## 1. Pengenalan SQL

### 1.1 Apa itu SQL?
SQL (Structured Query Language) adalah bahasa standar untuk mengelola dan memanipulasi database relasional.

### 1.2 Jenis Perintah SQL
- **DDL** (Data Definition Language): Membuat/mengubah struktur database
- **DML** (Data Manipulation Language): Memanipulasi data
- **DCL** (Data Control Language): Mengontrol akses data
- **TCL** (Transaction Control Language): Mengelola transaksi

### 1.3 RDBMS Populer
- MySQL
- PostgreSQL
- Oracle
- SQL Server
- SQLite

---

## 2. Struktur Database

### 2.1 Komponen Database
```sql
-- Contoh struktur tabel
CREATE TABLE karyawan (
    id INT PRIMARY KEY,
    nama VARCHAR(100) NOT NULL,
    departemen VARCHAR(50),
    gaji DECIMAL(10,2),
    tanggal_masuk DATE DEFAULT CURRENT_DATE
);
```

### 2.2 Tipe Data SQL
- **String**: CHAR, VARCHAR, TEXT
- **Numerik**: INT, DECIMAL, FLOAT
- **Tanggal**: DATE, TIME, DATETIME, TIMESTAMP
- **Boolean**: BOOLEAN
- **Binary**: BLOB, JSON

---

## 3. DDL (Data Definition Language)

### 3.1 CREATE
```sql
CREATE DATABASE perusahaan;
CREATE TABLE departemen (
    id SERIAL PRIMARY KEY,
    nama VARCHAR(100) NOT NULL UNIQUE,
    budget DECIMAL(15,2) CHECK (budget > 0)
);
```

### 3.2 ALTER
```sql
ALTER TABLE karyawan ADD COLUMN email VARCHAR(100);
ALTER TABLE departemen RENAME COLUMN budget TO anggaran;
```

### 3.3 DROP
```sql
DROP TABLE karyawan;
DROP DATABASE perusahaan;
```

---

## 4. DML (Data Manipulation Language)

### 4.1 INSERT
```sql
INSERT INTO karyawan (id, nama, departemen, gaji)
VALUES (1, 'Budi', 'IT', 7500000);
```

### 4.2 UPDATE
```sql
UPDATE karyawan 
SET gaji = gaji * 1.1 
WHERE departemen = 'IT';
```

### 4.3 DELETE
```sql
DELETE FROM karyawan 
WHERE tanggal_masuk < '2020-01-01';
```

---

## 5. Query Dasar

### 5.1 SELECT
```sql
SELECT nama, gaji FROM karyawan;
```

### 5.2 WHERE
```sql
SELECT * FROM karyawan 
WHERE gaji > 5000000 AND departemen = 'IT';
```

### 5.3 ORDER BY
```sql
SELECT * FROM karyawan 
ORDER BY gaji DESC, nama ASC;
```

### 5.4 LIMIT
```sql
SELECT * FROM karyawan 
ORDER BY gaji DESC 
LIMIT 5;
```

---

## 6. Fungsi Agregasi

### 6.1 Fungsi Dasar
```sql
SELECT 
    COUNT(*) AS total_karyawan,
    AVG(gaji) AS rata_gaji,
    MAX(gaji) AS gaji_tertinggi,
    MIN(gaji) AS gaji_terendah,
    SUM(gaji) AS total_gaji
FROM karyawan;
```

### 6.2 GROUP BY
```sql
SELECT 
    departemen,
    COUNT(*) AS jumlah_karyawan,
    AVG(gaji) AS rata_gaji
FROM karyawan
GROUP BY departemen;
```

### 6.3 HAVING
```sql
SELECT 
    departemen,
    AVG(gaji) AS rata_gaji
FROM karyawan
GROUP BY departemen
HAVING AVG(gaji) > 5000000;
```

---

## 7. Join dan Relasi Tabel

### 7.1 Jenis-Jenis Join
```sql
-- INNER JOIN
SELECT k.nama, d.nama AS departemen
FROM karyawan k
INNER JOIN departemen d ON k.departemen_id = d.id;

-- LEFT JOIN
SELECT k.nama, d.nama AS departemen
FROM karyawan k
LEFT JOIN departemen d ON k.departemen_id = d.id;

-- FULL OUTER JOIN
SELECT k.nama, d.nama AS departemen
FROM karyawan k
FULL OUTER JOIN departemen d ON k.departemen_id = d.id;
```

### 7.2 Multiple Join
```sql
SELECT 
    k.nama AS karyawan,
    p.nama AS proyek,
    d.nama AS departemen
FROM karyawan k
JOIN proyek_karyawan pk ON k.id = pk.karyawan_id
JOIN proyek p ON pk.proyek_id = p.id
JOIN departemen d ON k.departemen_id = d.id;
```

---

## 8. Subquery

### 8.1 Subquery di WHERE
```sql
SELECT nama, gaji
FROM karyawan
WHERE gaji > (SELECT AVG(gaji) FROM karyawan);
```

### 8.2 Subquery di FROM
```sql
SELECT d.nama, k.rata_gaji
FROM departemen d
JOIN (
    SELECT departemen_id, AVG(gaji) AS rata_gaji
    FROM karyawan
    GROUP BY departemen_id
) k ON d.id = k.departemen_id;
```

### 8.3 EXISTS
```sql
SELECT nama
FROM karyawan k
WHERE EXISTS (
    SELECT 1 FROM proyek_karyawan pk
    WHERE pk.karyawan_id = k.id
);
```

---

## 9. Window Functions

### 9.1 ROW_NUMBER()
```sql
SELECT 
    nama,
    gaji,
    departemen,
    ROW_NUMBER() OVER (PARTITION BY departemen ORDER BY gaji DESC) AS ranking
FROM karyawan;
```

### 9.2 RANK() dan DENSE_RANK()
```sql
SELECT 
    nama,
    gaji,
    RANK() OVER (ORDER BY gaji DESC) AS rank_gaji,
    DENSE_RANK() OVER (ORDER BY gaji DESC) AS dense_rank_gaji
FROM karyawan;
```

### 9.3 LAG dan LEAD
```sql
SELECT 
    bulan,
    pendapatan,
    LAG(pendapatan, 1) OVER (ORDER BY bulan) AS bulan_sebelumnya,
    LEAD(pendapatan, 1) OVER (ORDER BY bulan) AS bulan_berikutnya
FROM laporan_keuangan;
```

---

## 10. Optimasi Query

### 10.1 Indexing
```sql
CREATE INDEX idx_karyawan_departemen ON karyawan(departemen_id);
```

### 10.2 EXPLAIN ANALYZE
```sql
EXPLAIN ANALYZE SELECT * FROM karyawan WHERE departemen = 'IT';
```

### 10.3 Tips Optimasi
1. Gunakan SELECT kolom spesifik, bukan SELECT *
2. Batasi data dengan WHERE
3. Gunakan LIMIT untuk data besar
4. Hindari subquery yang tidak perlu

---

## 11. Best Practices

### 11.1 Penamaan
- Gunakan nama deskriptif (user_account bukan usr_acc)
- Konsisten dalam penulisan (camelCase atau snake_case)

### 11.2 Formatting
```sql
-- Baik
SELECT 
    k.nama,
    d.nama AS departemen,
    COUNT(p.id) AS jumlah_proyek
FROM karyawan k
JOIN departemen d ON k.departemen_id = d.id
LEFT JOIN proyek_karyawan pk ON k.id = pk.karyawan_id
LEFT JOIN proyek p ON pk.proyek_id = p.id
WHERE k.status = 'Aktif'
GROUP BY k.nama, d.nama
HAVING COUNT(p.id) > 2
ORDER BY jumlah_proyek DESC;
```

### 11.3 Keamanan
- Gunakan parameterized query untuk hindari SQL injection
- Batasi hak akses user
- Enkripsi data sensitif

---

## 12. Studi Kasus Real-World

### 12.1 Laporan Karyawan
```sql
SELECT 
    d.nama AS departemen,
    COUNT(k.id) AS jumlah_karyawan,
    AVG(k.gaji)::NUMERIC(10,2) AS rata_gaji,
    SUM(CASE WHEN k.status = 'Aktif' THEN 1 ELSE 0 END) AS aktif,
    SUM(CASE WHEN k.status = 'Resign' THEN 1 ELSE 0 END) AS resign
FROM departemen d
LEFT JOIN karyawan k ON d.id = k.departemen_id
GROUP BY d.nama
ORDER BY rata_gaji DESC;
```

### 12.2 Analisis Penjualan
```sql
WITH penjualan_bulanan AS (
    SELECT 
        EXTRACT(MONTH FROM tanggal) AS bulan,
        produk_id,
        SUM(jumlah) AS total_penjualan,
        SUM(harga * jumlah) AS total_pendapatan
    FROM penjualan
    WHERE EXTRACT(YEAR FROM tanggal) = 2023
    GROUP BY bulan, produk_id
)
SELECT 
    p.nama AS produk,
    pb.bulan,
    pb.total_penjualan,
    pb.total_pendapatan,
    RANK() OVER (PARTITION BY pb.bulan ORDER BY pb.total_pendapatan DESC) AS ranking
FROM penjualan_bulanan pb
JOIN produk p ON pb.produk_id = p.id
ORDER BY pb.bulan, ranking;
```

---

## 13. Latihan dan Solusi

### Latihan 1: Query Karyawan
**Tugas**: Buat query untuk menampilkan 5 karyawan dengan gaji tertinggi per departemen

**Solusi**:
```sql
WITH ranked_karyawan AS (
    SELECT 
        nama,
        gaji,
        departemen,
        DENSE_RANK() OVER (PARTITION BY departemen ORDER BY gaji DESC) AS ranking
    FROM karyawan
)
SELECT nama, gaji, departemen
FROM ranked_karyawan
WHERE ranking <= 5
ORDER BY departemen, ranking;
```

### Latihan 2: Analisis Produk
**Tugas**: Hitung pertumbuhan penjualan bulanan untuk setiap produk

**Solusi**:
```sql
WITH penjualan_bulanan AS (
    SELECT 
        produk_id,
        EXTRACT(YEAR FROM tanggal) AS tahun,
        EXTRACT(MONTH FROM tanggal) AS bulan,
        SUM(jumlah) AS total_penjualan
    FROM penjualan
    GROUP BY produk_id, tahun, bulan
)
SELECT 
    p.nama AS produk,
    pb.tahun,
    pb.bulan,
    pb.total_penjualan,
    LAG(pb.total_penjualan, 1) OVER (PARTITION BY p.id ORDER BY pb.tahun, pb.bulan) AS penjualan_bulan_sebelumnya,
    ROUND((pb.total_penjualan - LAG(pb.total_penjualan, 1) OVER (PARTITION BY p.id ORDER BY pb.tahun, pb.bulan)) / 
    LAG(pb.total_penjualan, 1) OVER (PARTITION BY p.id ORDER BY pb.tahun, pb.bulan) * 100, 2) AS pertumbuhan_persen
FROM penjualan_bulanan pb
JOIN produk p ON pb.produk_id = p.id
ORDER BY p.nama, pb.tahun, pb.bulan;
```

---

# 🎯 Kesimpulan
Dokumen ini mencakup fundamental SQL secara komprehensif dari dasar hingga konsep lanjutan yang dibutuhkan di dunia kerja. Untuk menjadi ahli SQL:
1. **Pahami konsep dasar** dengan baik
2. **Latihan secara teratur** dengan berbagai kasus
3. **Pelajari optimasi query** untuk database besar
4. **Terapkan best practices** dalam pekerjaan sehari-hari
5. **Ikuti perkembangan** fitur-fitur baru di RDBMS

Selamat belajar dan semoga sukses dalam karir SQL Anda! 🚀
```