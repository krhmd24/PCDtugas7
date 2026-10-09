# PCD7 — Verifikasi Nomor Ijazah dan Deteksi Tanda Tangan

## 1. Deskripsi Proyek

Proyek ini merupakan implementasi pengolahan citra digital untuk membantu proses verifikasi dokumen ijazah. Sistem mengolah citra ijazah melalui beberapa tahapan, mulai dari konversi grayscale, peningkatan kualitas citra, pemotongan area tertentu (*cropping*), pembacaan nomor ijazah menggunakan Optical Character Recognition (OCR), hingga deteksi keberadaan tanda tangan.

Proyek ini menggunakan dua kategori dataset, yaitu **Present** dan **Absent**. Kategori Present berisi citra ijazah yang memiliki tanda tangan, sedangkan kategori Absent berisi citra ijazah yang tidak memiliki tanda tangan.

Nomor ijazah diproses menggunakan OCR untuk kategori Present. Sesuai dengan ketentuan dataset proyek, kolom nomor ijazah untuk kategori Absent dikosongkan.

## 2. Tujuan

Tujuan dari proyek ini adalah:

* Menerapkan teknik pengolahan citra digital pada dokumen ijazah.
* Mengonversi citra berwarna menjadi grayscale.
* Meningkatkan kualitas citra menggunakan metode Contrast Limited Adaptive Histogram Equalization (CLAHE) dan sharpening.
* Membandingkan hasil pembacaan nomor ijazah menggunakan beberapa metode prapemrosesan OCR.
* Mengevaluasi hasil OCR menggunakan Character Error Rate (CER).
* Mendeteksi keberadaan tanda tangan melalui thresholding, operasi morfologi, dan klasifikasi citra.
* Menggabungkan hasil pembacaan nomor ijazah dan deteksi tanda tangan ke dalam tabel hasil akhir.

## 3. Dataset

Dataset terdiri dari **18 citra ijazah**, yang dibagi menjadi dua kategori.

| Kategori |   Jumlah | Keterangan                 |
| -------- | -------: | -------------------------- |
| Present  |  9 citra | Ijazah dengan tanda tangan |
| Absent   |  9 citra | Ijazah tanpa tanda tangan  |
| Total    | 18 citra | Dataset pengujian          |

Dataset Present mencakup beberapa kondisi kualitas citra, antara lain:

* High Quality Enhanced
* Low Contrast
* Blurred
* High Noise
* Low Resolution Upsampled
* Faded Underexposed
* Color Shift / Warm Tint
* JPEG Compression Artifacts
* Combined Degradation

Dataset Absent berisi sembilan citra ijazah tanpa tanda tangan yang digunakan untuk menguji proses deteksi tanda tangan.

**Struktur dataset:**

```text
PCD7/
├── Absent/
│   ├── gambar 1 pcd.jpg
│   ├── gambar 2 pcd.jpg
│   └── ...
└── Present/
    ├── 01_HighQuality_Enhanced.jpg
    ├── 02_LowContrast.jpg
    ├── 03_Blurred.jpg
    └── ...
```

## 4. Teknologi dan Library

Proyek ini dikembangkan menggunakan Python di Google Colab dengan library berikut:

| Library / Teknologi | Kegunaan                                                |
| ------------------- | ------------------------------------------------------- |
| Python              | Bahasa pemrograman utama                                |
| Google Colab        | Lingkungan pengembangan dan eksekusi notebook           |
| OpenCV              | Pengolahan citra, cropping, thresholding, dan morfologi |
| NumPy               | Pengolahan array dan data piksel                        |
| Matplotlib          | Visualisasi citra dan hasil pemrosesan                  |
| Pandas              | Pengolahan tabel dan ekspor hasil ke CSV                |
| Pillow (PIL)        | Dukungan pembacaan dan pengolahan citra                 |
| Tesseract OCR       | Ekstraksi teks nomor ijazah                             |
| scikit-learn        | Klasifikasi dan evaluasi deteksi tanda tangan           |

## 5. Alur Pengolahan Citra

Secara umum, alur sistem adalah sebagai berikut:

```text
Citra Ijazah
     |
     v
Konversi Grayscale
     |
     v
Peningkatan Kualitas Citra
     |
     +------------------------+
     |                        |
     v                        v
Area Nomor Ijazah       Area Tanda Tangan
     |                        |
     v                        v
Prapemrosesan OCR       Otsu Thresholding
     |                        |
     v                        v
Tesseract OCR           Operasi Morfologi
     |                        |
     v                        v
Nomor Ijazah            Ekstraksi Fitur /
                        Klasifikasi
                              |
                              v
                     Status Tanda Tangan
     |                        |
     +-----------+------------+
                 |
                 v
           Hasil Verifikasi
```

## 6. Tahapan Implementasi

### 6.1. Konversi Grayscale

Citra ijazah dikonversi dari format berwarna menjadi grayscale. Tahap ini mengubah citra menjadi representasi intensitas keabuan sehingga pengolahan selanjutnya dapat berfokus pada perbedaan intensitas piksel.

Hasil grayscale juga menjadi dasar untuk proses peningkatan kualitas, thresholding, dan ekstraksi teks.

### 6.2. Cropping Area

Tahap cropping digunakan untuk mengambil bagian tertentu dari citra ijazah agar pemrosesan lebih terfokus.

Dua area utama yang digunakan adalah:

* **Area nomor ijazah:** digunakan sebagai masukan untuk OCR.
* **Area tanda tangan:** digunakan untuk menentukan ada atau tidaknya tanda tangan.

Ketepatan koordinat cropping memengaruhi kualitas hasil OCR dan deteksi tanda tangan.

### 6.3. Peningkatan Kualitas Citra (*Image Enhancement*)

Dua metode enhancement digunakan dan dibandingkan.

**A. Contrast Limited Adaptive Histogram Equalization (CLAHE)**

CLAHE meningkatkan kontras lokal citra. Metode ini membagi citra ke dalam beberapa bagian dan menyesuaikan kontras pada setiap bagian, dengan pembatasan untuk mengurangi penguatan noise secara berlebihan.

Metode ini digunakan untuk membantu memperjelas teks atau pola pada citra yang memiliki kontras rendah.

Parameter yang digunakan:

* `clipLimit = 2.0`
* `tileGridSize = (4, 4)`

**B. Sharpening**

Sharpening digunakan untuk mempertegas detail dan tepi objek dalam citra menggunakan kernel konvolusi.

Kernel yang digunakan:

```python
kernel = np.array([
    [0, -1, 0],
    [-1, 5, -1],
    [0, -1, 0]
])
```

Metode ini dapat membantu memperjelas tepi karakter, tetapi juga berpotensi memperkuat noise atau artefak pada citra yang kualitasnya buruk.

### 6.4. Pembacaan Nomor Ijazah dengan OCR

OCR digunakan untuk mengekstraksi karakter pada area nomor ijazah. Mesin OCR yang digunakan adalah Tesseract.

Tahapan pemrosesan meliputi:

1. Mengambil area nomor ijazah melalui cropping.
2. Memutar atau menyesuaikan orientasi citra sesuai kebutuhan.
3. Memperbesar citra agar karakter lebih mudah dibaca.
4. Membandingkan beberapa metode prapemrosesan.
5. Menyimpan hasil ekstraksi nomor ijazah.

Tiga pendekatan yang dibandingkan adalah:

* Grayscale.
* Otsu Thresholding.
* CLAHE dilanjutkan dengan Otsu Thresholding.

OCR menggunakan konfigurasi yang membatasi karakter keluaran pada angka untuk membantu pembacaan nomor ijazah.

Hasil setiap metode disimpan agar perbedaan pembacaan karakter dapat dianalisis.

**Catatan:** hasil OCR merupakan keluaran otomatis dan tetap perlu dibandingkan dengan nomor referensi yang benar.

### 6.5. Evaluasi OCR dengan Character Error Rate (CER)

Character Error Rate (CER) digunakan untuk mengukur tingkat kesalahan hasil OCR berdasarkan perbandingan karakter dengan teks referensi.

Rumus CER:

$$
CER = \frac{S + D + I}{N}
$$

Keterangan:

* \(S\): jumlah karakter yang salah diganti (*substitution*).
* \(D\): jumlah karakter yang hilang (*deletion*).
* \(I\): jumlah karakter tambahan (*insertion*).
* \(N\): jumlah karakter pada teks referensi.

Semakin kecil nilai CER, semakin dekat hasil OCR dengan teks referensi. Nilai CER sebesar 0 menunjukkan bahwa hasil OCR sama persis dengan teks referensi.

Agar CER dapat dihitung secara valid, setiap citra yang dievaluasi harus memiliki nomor referensi yang benar. Jika referensi belum dimasukkan, nilai CER belum dapat dijadikan dasar untuk menyimpulkan metode OCR terbaik.

Nomor ijazah pada kategori Absent dikosongkan sesuai ketentuan proyek dan tidak diperlakukan sebagai hasil OCR yang harus dibandingkan dengan nomor referensi.

### 6.6. Deteksi Tanda Tangan dengan Thresholding

Area tanda tangan dikonversi menjadi citra biner menggunakan metode Otsu Thresholding.

Metode Otsu menentukan nilai ambang secara otomatis untuk memisahkan piksel berdasarkan intensitasnya. Pada implementasi ini, digunakan inversi biner agar pola tinta tampak sebagai piksel putih pada latar belakang hitam.

Thresholding membantu memisahkan pola tanda tangan dari latar belakang dokumen, tetapi hasilnya dapat dipengaruhi oleh noise, kualitas cetak, kontras, dan isi lain di dalam area cropping.

### 6.7. Operasi Morfologi

Setelah thresholding, operasi morfologi digunakan untuk memperbaiki struktur pola biner.

Operasi yang diterapkan meliputi:

* **Opening:** membantu mengurangi titik-titik kecil atau noise.
* **Closing:** membantu menghubungkan bagian yang terputus dan menutup celah kecil.

Kernel berbentuk elips berukuran 3 × 3 digunakan dalam proses tersebut.

Hasil morfologi kemudian digunakan untuk membantu ekstraksi fitur dan proses deteksi tanda tangan.

### 6.8. Klasifikasi dan Evaluasi Tanda Tangan

Selain analisis persentase piksel putih, proyek ini menggunakan fitur citra untuk mendukung klasifikasi tanda tangan.

Fitur yang digunakan meliputi:

* Kepadatan piksel putih.
* Jumlah komponen terhubung.
* Proporsi komponen terbesar.
* Rasio aspek objek.

Fitur tersebut digunakan dalam model klasifikasi berbasis Support Vector Machine (SVM) melalui pipeline scikit-learn.

Evaluasi dilakukan menggunakan Stratified K-Fold Cross-Validation dengan tiga fold agar kedua kelas tetap terwakili pada pembagian data.

Karena dataset hanya terdiri dari 18 citra, hasil klasifikasi perlu ditafsirkan secara hati-hati. Hasil pada dataset kecil belum tentu menunjukkan kemampuan generalisasi pada dokumen ijazah lain.

## 7. Hasil dan Evaluasi

Hasil pengolahan disimpan dalam tabel agar setiap citra dapat dianalisis dan dibandingkan.

Keluaran yang dihasilkan meliputi:

* Citra grayscale.
* Citra hasil CLAHE dan sharpening.
* Hasil OCR dari beberapa metode prapemrosesan.
* Hasil evaluasi OCR menggunakan CER, jika nomor referensi tersedia.
* Hasil thresholding dan morfologi untuk area tanda tangan.
* Hasil evaluasi deteksi tanda tangan.
* Tabel verifikasi akhir.

### Perbandingan Metode Enhancement dan OCR

CLAHE dan sharpening memiliki tujuan yang berbeda. CLAHE berfokus pada peningkatan kontras lokal, sedangkan sharpening berfokus pada penegasan detail dan tepi citra.

Untuk menentukan pendekatan yang memberikan hasil OCR terbaik, pembacaan karakter harus dibandingkan dengan teks referensi melalui CER. Citra yang terlihat lebih tajam belum tentu menghasilkan pembacaan OCR yang lebih akurat.

### Evaluasi Deteksi Tanda Tangan

Evaluasi deteksi tanda tangan dilakukan dengan membandingkan prediksi sistem terhadap label dataset, yaitu Present dan Absent.

Akurasi dihitung menggunakan rumus:

$$
Accuracy = \frac{TP + TN}{TP + TN + FP + FN}
$$

Keterangan:

* TP (*True Positive*): tanda tangan ada dan berhasil dideteksi.
* TN (*True Negative*): tanda tangan tidak ada dan berhasil dikenali.
* FP (*False Positive*): sistem mendeteksi tanda tangan padahal tidak ada.
* FN (*False Negative*): sistem tidak mendeteksi tanda tangan yang sebenarnya ada.

Nilai evaluasi akhir sebaiknya diambil dari keluaran notebook yang telah dijalankan, bukan diperkirakan. Perlu diperhatikan bahwa pemilihan ambang berdasarkan dataset yang sama dapat membuat hasil terlihat lebih baik daripada kinerja pada data baru.

## 8. Hasil Verifikasi Akhir

Sistem menggabungkan hasil OCR dan deteksi tanda tangan ke dalam satu tabel.

Contoh struktur tabel:

| Nama Citra             | Nomor Ijazah | Status Tanda Tangan |
| ---------------------- | ------------ | ------------------- |
| Citra kategori Present | Hasil OCR    | Present             |
| Citra kategori Absent  | Kosong       | Absent              |

Tabel tersebut merupakan ringkasan hasil pemrosesan dan dapat diekspor ke format CSV untuk dokumentasi serta analisis lebih lanjut.

Kolom nomor ijazah pada kategori Absent sengaja dikosongkan sesuai ketentuan dataset proyek.

## 9. Struktur File Hasil

Struktur direktori hasil pengolahan pada Google Drive:

```text
PCD7_Enhancement/
├── Present/
│   ├── CLAHE/
│   └── Sharpening/
├── Absent/
│   ├── CLAHE/
│   └── Sharpening/
├── hasil_ocr_nomor.csv
├── perbandingan_ocr.csv
├── evaluasi_ttd.csv
└── hasil_verifikasi_akhir.csv
```

Nama dan lokasi berkas keluaran mengikuti proses yang dijalankan di notebook. Beberapa berkas evaluasi hanya tersedia setelah sel terkait berhasil dieksekusi.

## 10. Cara Menjalankan Proyek

### Persyaratan

* Akun Google.
* Google Colab.
* Dataset citra ijazah.
* Akses Google Drive.
* Library Python yang digunakan dalam notebook.
* Tesseract OCR.

### Langkah menjalankan

1. Unggah file `PCD7.ipynb` ke Google Colab.
2. Siapkan dataset di Google Drive dengan struktur folder yang sesuai.
3. Hubungkan Google Colab ke Google Drive.
4. Pastikan lokasi dataset pada kode sudah sesuai dengan lokasi penyimpanan.
5. Jalankan sel notebook secara berurutan dari bagian awal sampai akhir.
6. Periksa visualisasi hasil grayscale, enhancement, cropping, thresholding, dan morfologi.
7. Periksa tabel perbandingan OCR serta hasil evaluasi tanda tangan.
8. Pastikan tabel hasil verifikasi akhir berhasil dibuat.
9. Unduh atau gunakan berkas CSV yang dihasilkan untuk dokumentasi proyek.

**Penting:** jalankan sel secara berurutan agar variabel dan hasil pemrosesan tersedia ketika digunakan pada tahapan berikutnya.

## 11. Keterbatasan

Proyek ini memiliki beberapa keterbatasan:

* Jumlah dataset relatif kecil, yaitu 18 citra.
* Akurasi OCR dipengaruhi oleh orientasi, kontras, blur, noise, resolusi, dan kualitas area cropping.
* Koordinat cropping disesuaikan dengan struktur citra yang digunakan pada dataset.
* CLAHE dan sharpening tidak selalu meningkatkan hasil OCR pada semua kondisi.
* Thresholding dan operasi morfologi dapat menghasilkan pola yang berbeda pada citra dengan kualitas beragam.
* Evaluasi CER membutuhkan teks referensi yang benar.
* Hasil klasifikasi tanda tangan pada dataset kecil belum cukup untuk menjamin kinerja pada dokumen lain.
* Keberadaan pola tinta pada area tertentu tidak selalu membuktikan bahwa pola tersebut merupakan tanda tangan yang sah.

Oleh karena itu, sistem ini ditujukan sebagai prototipe pembelajaran pengolahan citra digital dan bukan sebagai pengganti verifikasi dokumen resmi secara manual.

## 12. Kesimpulan

Proyek ini menerapkan beberapa teknik pengolahan citra digital untuk mendukung verifikasi dokumen ijazah. Grayscale digunakan sebagai tahap awal, CLAHE dan sharpening diterapkan untuk peningkatan kualitas citra, sedangkan Tesseract OCR digunakan untuk membaca nomor ijazah. Akurasi pembacaan dapat dievaluasi menggunakan CER apabila teks referensi tersedia.

Pada jalur deteksi tanda tangan, Otsu Thresholding dan operasi morfologi digunakan untuk memisahkan serta memperbaiki pola citra sebelum ekstraksi fitur dan klasifikasi. Seluruh hasil kemudian digabungkan ke dalam tabel verifikasi akhir.

Melalui proyek ini, pengaruh kualitas citra dan metode prapemrosesan terhadap OCR serta deteksi tanda tangan dapat dipelajari secara langsung.

## 13. Kontributor

**Nama kelompok:** [Isi nama kelompok]

**Anggota kelompok:**

* [Nama anggota 1]
* [Nama anggota 2]
* [Nama anggota 3]

*Sesuaikan bagian kontributor dengan identitas kelompok sebelum repositori dipublikasikan.*

## 14. Lisensi dan Penggunaan

Proyek ini dibuat untuk keperluan pembelajaran dan tugas akademik Pengolahan Citra Digital. Penggunaan dataset, hasil pemrosesan, serta kode perlu memperhatikan izin dan privasi pemilik dokumen.
