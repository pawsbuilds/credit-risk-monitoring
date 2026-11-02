# Dokumentasi Pengolahan Data SBA
## Notebook 01 — Inventarisasi dan Penggabungan Dataset

### 1. Tujuan

Tahap ini bertujuan melakukan inventarisasi, pemeriksaan struktur, dan penggabungan dataset Small Business Administration (SBA) 7(a) dan 504.

Fokus utama tahap ini adalah mempersiapkan dataset gabungan yang memiliki struktur konsisten tanpa menghilangkan informasi dari sumber aslinya.

Pada tahap ini belum dilakukan pembersihan data, penghapusan kolom, imputasi nilai kosong, maupun normalisasi kategori.

### 2. Sumber Data

Dataset berasal dari SBA FOIA Loan Data dengan enam berkas CSV dan satu berkas Excel sebagai kamus data.

| Program | Nama berkas | Jumlah kolom |
|---|---|---:|
| 7(a) | FOIA_7a_FY1991_FY1999_asof_260930.csv | 42 |
| 7(a) | FOIA_7a_FY2000_FY2009_asof_260930.csv | 42 |
| 7(a) | FOIA_7a_FY2010_FY2019_asof_260930.csv | 42 |
| 7(a) | FOIA_7a_FY2020_Present_asof_260930.csv | 42 |
| 504 | FOIA_504_FY1991_FY2009_asof_260930.csv | 39 |
| 504 | FOIA_504_FY2010_Present_asof_260930.csv | 39 |

Seluruh berkas CSV memiliki snapshot tanggal 30 September 2026 berdasarkan penamaan sumber. Keseragaman tanggal snapshot juga diamati pada sampel data yang diperiksa.

### 3. Inventarisasi Dataset

Inventarisasi dilakukan terhadap enam berkas CSV untuk memperoleh informasi mengenai:
- Nama dan lokasi berkas.
- Ukuran setiap berkas.
- Program pinjaman.
- Jumlah kolom.
- Jumlah baris.
- Konsistensi struktur kolom ant berkas dalam program yang sama.

Hasil pemeriksaan menunjukkan:
- Terdapat empat berkas program 7(a).
- Terdapat dua berkas program 504.
- Seluruh berkas 7(a) memiliki 42 kolom dengan struktur yang konsisten.
- Seluruh berkas 504 memiliki 39 kolom dengan struktur yang konsisten.

### 4. Pemeriksaan dan Pemetaan Struktur Kolom

Pemeriksaan lintas program dilakukan untuk mengidentifikasi kolom yang sama dan kolom yang hanya tersedia pada program tertentu.

Hasil pemeriksaan menunjukkan:

30 kolom tersedia pada kedua program.

12 kolom hanya tersedia pada program 7(a).

9 kolom hanya tersedia pada program 504.

Total terdapat 51 kolom unik.

Hasil pemeriksaan ini menjadi dasar pemilihan pendekatan Union Schema pada proses penggabungan dataset.

Catatan: Pemetaan schema telah dilakukan selama eksplorasi, tetapi belum disimpan sebagai berkas CSV terpisah.

### 5. Pemeriksaan Sampel Data

Pemeriksaan sampel dilakukan dengan membaca sebagian kecil record dari masing-masing berkas untuk memperoleh gambaran awal mengenai isi dataset.

Beberapa temuan awal:
- Terdapat variasi representasi kategori `LoanStatus`, misalnya `P I F` dan `PIF`.
- Beberapa variabel memiliki nilai kosong pada sampel.
- Sebagian kolom identifier terbaca sebagai numerik akibat adanya nilai kosong.
- Nilai pada kolom `AsOfDate` dalam sampel menunjukkan tanggal snapshot yang sama.

Temuan tersebut dicatat sebagai indikasi awal untuk pemeriksaan lebih lanjut. Belum dilakukan perubahan atau normalisasi terhadap nilai-nilai tersebut.

### 6. Penghitungan Jumlah Record

Penghitungan jumlah record dilakukan menggunakan pembacaan bertahap (*chunk-based processing*) dengan ukuran 100.000 baris per proses.

Pendekatan ini digunakan untuk menghindari kebutuhan memuat seluruh dataset ke memori sekaligus.

Hasil penghitungan:

| Berkas | Program | Jumlah record |
|---|---|---:|
| FY1991–FY1999 | 7(a) | 337.033 |
| FY2000–FY2009 | 7(a) | 690.333 |
| FY2010–FY2019 | 7(a) | 545.751 |
| FY2020–Present | 7(a) | 403.644 |
| FY1991–FY2009 | 504 | 111.066 |
| FY2010–Present | 504 | 119.599 |
| **Total 7(a)** | | **1.976.761** |
| **Total 504** | | **230.665** |
| **Total keseluruhan** | | **2.207.426** |

### 7. Penggabungan Dataset per Program

Penggabungan dilakukan secara terpisah berdasarkan program.

Proses:
1. Empat berkas 7(a) digabungkan menjadi satu dataset.
2. Dua berkas 504 digabungkan menjadi satu dataset.
3. Pembacaan dan penulisan dilakukan secara bertahap untuk menghemat penggunaan memori.
4. Seluruh kolom dan nilai sumber dipertahankan.

Hasil penggabungan:

| Dataset | Jumlah record |
|---|---:|
| sba_7a_combined.csv | 1.976.761 |
| sba_504_combined.csv | 230.665 |

Validasi menunjukkan jumlah record sebelum dan sesudah penggabungan sama untuk kedua program.

### 8. Keputusan Desain Schema Dataset Gabungan

Karena terdapat perbedaan jumlah kolom antarprogram, digunakan pendekatan **Union Schema**.

Pendekatan ini dipilih untuk mempertahankan sebanyak mungkin informasi dari kedua program.

Dengan pendekatan ini:
- Seluruh 30 kolom bersama dipertahankan.
- Seluruh 12 kolom khusus 7(a) dipertahankan.
- Seluruh 9 kolom khusus 504 dipertahankan.
- Kolom yang tidak tersedia pada suatu program dibiarkan kosong.
- Penyelarasan dilakukan berdasarkan nama kolom, bukan posisi kolom.

Dengan demikian, dataset gabungan memiliki 51 kolom asli.

Keputusan ini juga berarti nilai kosong harus dibedakan berdasarkan penyebabnya. Kolom yang memang tidak tersedia pada suatu program tidak boleh langsung dianggap sebagai data yang gagal dikumpulkan.

### 9. Penggabungan Final dan Penambahan SourceFile

Setelah penggabungan per program, seluruh berkas sumber kembali dibaca untuk membentuk dataset gabungan final.

Kolom tambahan `SourceFile` ditambahkan untuk menyimpan nama berkas sumber setiap record.

Penambahan ini bertujuan:
- Memudahkan pelacakan asal record.
- Mendukung pemeriksaan anomali berdasarkan sumber.
- Mempermudah audit proses pengolahan data.
- Menjaga keterlacakan data sepanjang proses berikutnya.

Dataset final disimpan dengan nama:

`assets/data/staging/sba/sba_full_combined.csv`

Struktur akhirnya terdiri atas:
- 51 kolom asli hasil union schema.
- 1 kolom metadata `SourceFile`.
- Total 52 kolom.

### 10. Validasi Dataset Final

Validasi dilakukan dengan membandingkan jumlah record dari masing-masing berkas sumber dengan jumlah record yang tercatat pada dataset final berdasarkan `SourceFile`.

Hasil validasi:

| Berkas sumber | Expected | Actual | Hasil |
|---|---:|---:|---|
| 7a FY1991–FY1999 | 337.033 | 337.033 | Sesuai |
| 7a FY2000–FY2009 | 690.333 | 690.333 | Sesuai |
| 7a FY2010–FY2019 | 545.751 | 545.751 | Sesuai |
| 7a FY2020–Present | 403.644 | 403.644 | Sesuai |
| 504 FY1991–FY2009 | 111.066 | 111.066 | Sesuai |
| 504 FY2010–Present | 119.599 | 119.599 | Sesuai |

Ringkasan hasil:

| Indikator | Hasil |
|---|---:|
| Total record | 2.207.426 |
| Kolom asli | 51 |
| Kolom metadata | 1 |
| Total kolom final | 52 |
| Jumlah berkas sumber | 6 |
| Seluruh validasi jumlah record | Berhasil |

### 11. Prinsip Pengolahan Data yang Ditetapkan

Selama tahap inventarisasi dan penggabungan, ditetapkan prinsip berikut:

1. Data mentah tidak diubah dan tetap disimpan pada direktori raw.
2. Dataset staging mempertahankan seluruh kolom sumber.
3. Tidak ada kolom yang dihapus pada tahap inventarisasi.
4. Tidak ada record yang dihapus berdasarkan dugaan duplikasi.
5. Tidak dilakukan imputasi atau normalisasi kategori.
6. Setiap perubahan data pada tahap berikutnya harus memiliki alasan dan dokumentasi.
7. Setiap record pada dataset final harus dapat ditelusuri ke berkas sumber melalui `SourceFile`.

### 12. Batasan Tahap Ini

Keberhasilan penggabungan dan validasi jumlah record belum membuktikan bahwa seluruh isi dataset bebas dari kesalahan.

Pemeriksaan berikut masih diperlukan:
- Analisis distribusi dan karakteristik setiap variabel.
- Pemeriksaan tipe data dan konsistensi nilai.
- Investigasi missing values dan penyebabnya.
- Pemeriksaan duplikasi.
- Identifikasi nilai tidak wajar dan anomali.
- Penentuan kolom yang dipertahankan, ditransformasikan, atau dihapus.

Pemeriksaan tersebut akan dilakukan pada tahap pemahaman data dan data quality berikutnya.

### 13. Kesimpulan

Tahap inventarisasi dan penggabungan berhasil menghasilkan satu dataset SBA gabungan yang terdiri atas 2.207.426 record dan 52 kolom, termasuk metadata `SourceFile`.

Seluruh enam berkas sumber berhasil digabungkan dan jumlah record masing-masing berkas sesuai dengan hasil penghitungan awal.

Dataset ini menjadi dasar untuk tahap pemahaman data selanjutnya. Belum ada proses pembersihan maupun penghapusan informasi yang dilakukan.