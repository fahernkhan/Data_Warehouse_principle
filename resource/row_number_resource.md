# Materi Pembelajaran SQL: Mengatasi Duplikat Data dengan `ROW_NUMBER()`

## Daftar Isi
1. [Kasus Kesalahan Awal](#kasus-kesalahan-awal)
2. [Analisis Masalah](#analisis-masalah)
3. [Solusi dengan ROW_NUMBER()](#solusi-dengan-row_number)
4. [Fundamental Window Functions](#fundamental-window-functions)
5. [Studi Kasus Lain](#studi-kasus-lain)
6. [Best Practices](#best-practices)
7. [Latihan](#latihan)

## Kasus Kesalahan Awal

### Query Awal yang Bermasalah
```sql
SELECT 
  h.KodeBarang,
  h.StokAkhir
FROM 
  HistoriStokBarang h
WHERE 
  h.Tanggal = (
    SELECT MAX(h2.Tanggal)
    FROM HistoriStokBarang h2
    WHERE h2.KodeBarang = h.KodeBarang
  );
```

### Data Contoh
```
| KodeStok | Tanggal   | KodeBarang | StokAkhir |
|----------|-----------|------------|-----------|
| 1        | 01/01/09  | A          | 2         |
| 2        | 02/01/09  | A          | 5         |
| 3        | 02/01/09  | B          | 1         |
| 4        | 03/01/09  | A          | 3         |
| 5        | 03/01/09  | B          | 3         |
| 6        | 04/01/09  | A          | 5         |
| 7        | 04/01/09  | A          | 4         |
```

### Hasil yang Salah
```
| KodeBarang | StokAkhir |
|------------|-----------|
| A          | 5         | ← Tidak diinginkan
| A          | 4         | ← Seharusnya hanya ini
| B          | 3         |
```

## Analisis Masalah

### Penyebab Kesalahan
1. **Tanggal Tidak Unik**: Ada dua record dengan tanggal maksimal untuk barang A
2. **Tidak Ada Kriteria Penentu**: Query tidak memiliki cara untuk memilih antara dua record dengan tanggal sama
3. **GROUP BY Tidak Cukup**: GROUP BY biasa tidak bisa menangani kasus ini karena kita butuh record lengkap

### Konsep yang Terlibat
1. **Window Functions**: Fungsi yang beroperasi pada sekumpulan baris terkait
2. **PARTITION BY**: Membagi data menjadi grup-grup
3. **ORDER BY**: Mengurutkan data dalam setiap partisi
4. **ROW_NUMBER()**: Memberi nomor urut pada setiap baris dalam partisi

## Solusi dengan ROW_NUMBER()

### Query yang Diperbaiki
```sql
WITH RankedStok AS (
  SELECT
    KodeBarang,
    StokAkhir,
    ROW_NUMBER() OVER (
      PARTITION BY KodeBarang 
      ORDER BY Tanggal DESC, KodeStok DESC
    ) AS rn
  FROM HistoriStokBarang
)
SELECT
  KodeBarang,
  StokAkhir
FROM RankedStok
WHERE rn = 1;
```

### Penjelasan Langkah Demi Langkah
1. **Common Table Expression (CTE)**:
   - Membuat tabel sementara bernama `RankedStok`
   
2. **PARTITION BY KodeBarang**:
   - Membagi data menjadi grup berdasarkan KodeBarang
   - Setiap grup berisi semua record untuk satu barang

3. **ORDER BY Tanggal DESC, KodeStok DESC**:
   - Mengurutkan record dalam setiap grup:
     - Pertama berdasarkan Tanggal terbaru (DESC)
     - Jika tanggal sama, diurutkan berdasarkan KodeStok terbesar

4. **ROW_NUMBER()**:
   - Memberi nomor urut (rn) mulai dari 1 untuk setiap record dalam partisi
   - Record terbaru dapat rn=1

5. **WHERE rn = 1**:
   - Hanya mengambil record pertama dari setiap partisi

### Visualisasi Proses
```
Data Awal (Partisi A):
| KodeStok | Tanggal   | StokAkhir |
|----------|-----------|-----------|
| 1        | 01/01/09  | 2         |
| 2        | 02/01/09  | 5         |
| 4        | 03/01/09  | 3         |
| 6        | 04/01/09  | 5         |
| 7        | 04/01/09  | 4         |

Setelah ORDER BY:
| KodeStok | Tanggal   | StokAkhir | Urutan |
|----------|-----------|-----------|--------|
| 6        | 04/01/09  | 5         | 1      |
| 7        | 04/01/09  | 4         | 2      |
| 4        | 03/01/09  | 3         | 3      |
| 2        | 02/01/09  | 5         | 4      |
| 1        | 01/01/09  | 2         | 5      |

Hasil Akhir (WHERE rn=1):
| KodeStok | Tanggal   | StokAkhir |
|----------|-----------|-----------|
| 6        | 04/01/09  | 5         |
```

## Fundamental Window Functions

### Jenis-Jenis Window Functions
1. **ROW_NUMBER()**: Nomor urut (1, 2, 3, ...)
2. **RANK()**: Ranking dengan loncatan (1, 1, 3, ...)
3. **DENSE_RANK()**: Ranking tanpa loncatan (1, 1, 2, ...)
4. **NTILE(n)**: Membagi data menjadi n grup
5. **Aggregasi**: SUM(), AVG(), COUNT(), dll dengan OVER()

### Sintaks Dasar
```sql
FUNCTION_NAME() OVER (
  [PARTITION BY kolom1, kolom2, ...]
  [ORDER BY kolom1 [ASC|DESC], kolom2 [ASC|DESC], ...]
  [frame_clause]
)
```

### Perbedaan ROW_NUMBER(), RANK(), DENSE_RANK()
```
Data:
| Nilai |
|-------|
| 100   |
| 100   |
| 90    |
| 80    |

Hasil:
| Nilai | ROW_NUMBER | RANK | DENSE_RANK |
|-------|------------|------|------------|
| 100   | 1          | 1    | 1          |
| 100   | 2          | 1    | 1          |
| 90    | 3          | 3    | 2          |
| 80    | 4          | 4    | 3          |
```

## Studi Kasus Lain

### Kasus 1: Top 3 Produk Terlaris per Kategori
```sql
WITH ProdukRanking AS (
  SELECT
    Kategori,
    Produk,
    JumlahTerjual,
    ROW_NUMBER() OVER (PARTITION BY Kategori ORDER BY JumlahTerjual DESC) AS ranking
  FROM Penjualan
)
SELECT *
FROM ProdukRanking
WHERE ranking <= 3;
```

### Kasus 2: Gap Analysis (Perbedaan dengan Record Sebelumnya)
```sql
SELECT
  Tanggal,
  Nilai,
  Nilai - LAG(Nilai, 1) OVER (ORDER BY Tanggal) AS Selisih
FROM Metrik;
```

### Kasus 3: Running Total
```sql
SELECT
  Tanggal,
  Pendapatan,
  SUM(Pendapatan) OVER (ORDER BY Tanggal) AS RunningTotal
FROM Transaksi;
```

## Best Practices

1. **Gunakan PARTITION BY** untuk analisis per grup
2. **ORDER BY yang tepat** untuk urutan yang benar
3. **CTE untuk keterbacaan** ketika query kompleks
4. **Hindari ROW_NUMBER()** jika ada kemungkinan nilai sama (gunakan RANK() atau DENSE_RANK())
5. **Index kolom** yang digunakan di PARTITION BY dan ORDER BY untuk performa

## Latihan

### Latihan 1
Tabel: `Karyawan`
```
| ID | Nama   | Departemen | Gaji  |
|----|--------|------------|-------|
| 1  | Andi   | IT         | 5000  |
| 2  | Budi   | IT         | 6000  |
| 3  | Cici   | HR         | 4500  |
| 4  | Dedi   | HR         | 5500  |
| 5  | Eka    | IT         | 7000  |
```

Tugas: Tampilkan karyawan dengan gaji tertinggi di setiap departemen

### Jawaban Latihan 1
```sql
WITH GajiRanking AS (
  SELECT
    Nama,
    Departemen,
    Gaji,
    ROW_NUMBER() OVER (PARTITION BY Departemen ORDER BY Gaji DESC) AS ranking
  FROM Karyawan
)
SELECT
  Nama,
  Departemen,
  Gaji
FROM GajiRanking
WHERE ranking = 1;
```

### Latihan 2
Tabel: `Transaksi`
```
| ID | Tanggal    | CustomerID | Jumlah |
|----|------------|------------|--------|
| 1  | 2023-01-01 | 100        | 150    |
| 2  | 2023-01-01 | 101        | 200    |
| 3  | 2023-01-02 | 100        | 75     |
| 4  | 2023-01-03 | 102        | 300    |
| 5  | 2023-01-03 | 101        | 125    |
```

Tugas: Hitung total kumulatif (running total) transaksi per customer

### Jawaban Latihan 2
```sql
SELECT
  Tanggal,
  CustomerID,
  Jumlah,
  SUM(Jumlah) OVER (
    PARTITION BY CustomerID 
    ORDER BY Tanggal
    ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
  ) AS RunningTotal
FROM Transaksi
ORDER BY CustomerID, Tanggal;
```