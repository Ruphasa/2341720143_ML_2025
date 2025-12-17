## Project PBL: Fraud Detection & OCR pada KTP

Proyek ini merupakan tugas **Project Based Learning (PBL)** yang berfokus pada pembangunan sistem untuk:

1. **Fraud Detection KTP** – mendeteksi apakah citra KTP yang diunggah merupakan **asli** atau **hasil pemalsuan/tampering** menggunakan model **Convolutional Neural Network (CNN)**.
2. **OCR KTP** – mengekstrak teks penting pada KTP (nama, NIK, alamat, dsb.) menggunakan teknik **Optical Character Recognition (OCR)** berbasis pemrosesan citra **dengan model OCR yang dilatih sendiri (bukan library OCR siap pakai)**.

Sistem ini ditujukan sebagai prototipe pendukung verifikasi identitas pada proses KYC (Know Your Customer), pembukaan rekening, dan layanan lain yang membutuhkan validasi KTP secara otomatis.

---

## 1. Tujuan Proyek

- Mengembangkan model **deteksi pemalsuan KTP** berbasis citra.
- Membangun pipeline **OCR** untuk membaca informasi penting dari KTP.
- Mengintegrasikan kedua komponen menjadi alur verifikasi:
  1. Input: citra KTP.
  2. Output 1: prediksi *asli* / *palsu*.
  3. Output 2: hasil ekstraksi teks (field penting KTP).
- Menyusun dokumentasi dan eksperimen sebagai bahan laporan PBL.

---

## 2. Lingkup & Batasan

- Dataset KTP hanya digunakan untuk keperluan akademik, tidak dibagikan secara publik (isu privasi data).
- Model fraud detection difokuskan pada **tampering visual** (edit digital sederhana/menengah), bukan verifikasi keaslian fisik dokumen.
- Hasil OCR tidak dijamin 100% akurat pada semua jenis KTP (perbedaan kualitas scan, resolusi, dan desain KTP).

---


## 3. Metodologi

### 3.1. Fraud Detection KTP (CNN)

1. **Preprocessing Citra**
	- Resize citra KTP ke ukuran tetap (misal 224×224 atau 256×256 piksel).
	- Normalisasi nilai piksel ke [0, 1] atau [-1, 1].
	- (Opsional) Augmentasi data: rotasi kecil, zoom, flip, perubahan brightness/contrast.

2. **Arsitektur Model** (gambaran umum)
	- Beberapa lapisan **Convolution + ReLU + MaxPooling**.
	- Lapisan **Flatten / GlobalAveragePooling**.
	- Beberapa lapisan **Dense** dengan Dropout untuk regularisasi.
	- Lapisan output: 1 neuron (sigmoid) atau 2 neuron (softmax) untuk klasifikasi **asli/palsu**.

3. **Training**
	- Loss: `binary_crossentropy` atau `categorical_crossentropy`.
	- Optimizer: misalnya `Adam`.
	- Metrik utama: `accuracy`, plus `precision`, `recall`, dan `F1-score` pada evaluasi.
	- Early stopping & model checkpoint untuk mencegah overfitting.

4. **Evaluasi**
	- Train/validation/test split (misal 70/15/15).
	- Confusion matrix untuk melihat distribusi prediksi benar/salah.
	- Analisis contoh: beberapa citra yang salah klasifikasi untuk memahami kelemahan model.

5. **Export Model**
	- Menyimpan model ke format `.h5` untuk penggunaan di Python.
	- Konversi ke `.tflite` untuk deployment di perangkat dengan resource terbatas.

### 3.2. OCR KTP (Model Kustom Berbasis SVM, Tanpa Deep Learning)

Pada bagian OCR ini, sistem **tidak menggunakan library OCR siap pakai** seperti Tesseract/EasyOCR dan **tidak menggunakan deep learning**. Model yang digunakan adalah **model klasik berbasis fitur HOG + SVM** yang dilatih sendiri dari dataset karakter KTP.

1. **Preprocessing Citra untuk OCR**
	- Konversi ke grayscale.
	- Filtering noise (Gaussian blur/bilateral filter).
	- Thresholding (Otsu/adaptive) untuk memperjelas teks.
	- (Opsional) Deskew dan crop area teks.
	- Normalisasi ukuran patch karakter/segmen teks.

2. **Segmentasi & Representasi Karakter/Teks**
	- Segmentasi area teks dari background KTP (misalnya menggunakan thresholding + contour detection, MSER, dan operasi morfologi).
	- Pemotongan citra menjadi patch karakter atau kata.
	- Transformasi citra ke bentuk yang sesuai dengan ekstraktor fitur (ukuran tetap, teks menjadi foreground putih di atas background hitam, dsb.).

3. **Ekstraksi Fitur & Training Model OCR (HOG + SVM)**
	- Setiap patch karakter diubah menjadi citra biner berukuran tetap, lalu diekstraksi fiturnya menggunakan **Histogram of Oriented Gradients (HOG)**.
	- Fitur HOG tersebut menjadi vektor input untuk **SVM (Support Vector Machine)** dengan kernel **RBF**.
	- Dilakukan **grid search** terhadap hyperparameter C dan gamma, dikombinasikan dengan **k-fold cross validation**, untuk mencari konfigurasi SVM terbaik.
	- Model SVM terbaik kemudian dilatih ulang menggunakan seluruh data training.

4. **Ekstraksi & Penyusunan Teks**
	- Menggunakan model SVM terlatih untuk memprediksi label dari tiap patch karakter/segmen.
	- Mengurutkan kembali hasil prediksi berdasarkan posisi di citra sehingga membentuk kata/kalimat.
	- Post-processing: pembersihan teks (hilangkan karakter aneh, spasi berlebih, dsb.).
	- Regex atau rule-based parsing untuk mengekstrak field:
	  - NIK
	  - Nama
	  - Tempat/Tanggal Lahir
	  - Jenis Kelamin
	  - Alamat
	  - RT/RW, Kelurahan, Kecamatan, dsb.

5. **Validasi Hasil OCR**
	- Membandingkan hasil OCR dengan ground truth (jika tersedia) untuk menghitung akurasi per karakter/per kata.
	- Mengamati error umum, misalnya salah baca angka 0 dan huruf O, 1 dan I, 1 dan l, dsb.

---

## 4. Alur Sistem

1. Pengguna mengunggah citra KTP.
2. Sistem melakukan **fraud detection** menggunakan model CNN:
	- Output: skor probabilitas dan label **Asli** / **Palsu**.
3. Sistem menjalankan **OCR** pada citra yang sama:
	- Output: hasil teks mentah dan field-field penting yang sudah diparsing.
4. Hasil dikembalikan ke pengguna dalam bentuk:
	- Status KTP (asli/palsu).
	- Informasi teks terstruktur (NIK, nama, alamat, dll.).

---

## 5. Kebutuhan Lingkungan & Dependensi

Disarankan menggunakan Python 3.9+ dan virtual environment.

Dependensi utama (kurang lebih):

- TensorFlow / Keras – training & inferensi model CNN untuk fraud detection KTP.
- NumPy, Pandas – manipulasi data.
- OpenCV – pemrosesan citra dan seluruh pipeline OCR (preprocessing, segmentasi, HOG, SVM, dsb.).
- Matplotlib/Seaborn – visualisasi hasil training & evaluasi.

Sesuaikan pemasangan paket dengan kebutuhan aktual pada notebook.

---

## 6. Cara Menjalankan

1. **Persiapan Environment**
	- Buat dan aktifkan virtual environment (opsional tapi direkomendasikan).
	- Install dependensi yang diperlukan (lihat bagian sebelumnya / file requirements jika dibuat).

2. **Menjalankan Notebook Fraud Detection**
	- Buka [PBL/ML/coba.ipynb](PBL/ML/coba.ipynb) untuk melihat proses training.
	- Gunakan [PBL/ML/tes.ipynb](PBL/ML/tes.ipynb) untuk melakukan inferensi/testing terhadap citra baru dengan model tersimpan di [PBL/ML/saved_models](PBL/ML/saved_models).

3. **Menjalankan Notebook OCR**
	- Buka [PBL/PCVK/ocr_ktp.ipynb](PBL/PCVK/ocr_ktp.ipynb).
	- Jalankan sel secara berurutan: preprocessing → OCR → post-processing.
	- Sesuaikan path citra KTP pada cell input.

4. **Pengujian Akhir**
	- Uji beberapa citra KTP: asli dan contoh yang dimodifikasi.
	- Amati konsistensi prediksi fraud detection dan kualitas hasil OCR.

---

## 7. Hasil & Evaluasi

### 7.1. Hasil Pelatihan OCR KTP (Model Kustom)

- **Skema training**
	- Dataset karakter/segmen teks KTP disusun dalam folder kelas (misal digit 0–9 dan huruf A–Z).
	- Setiap citra karakter dipreproses (grayscale, threshold, invert, normalisasi ukuran) lalu diekstraksi fiturnya menggunakan **HOG (Histogram of Oriented Gradients)**.
	- Dilakukan **grid search** untuk model **SVM kernel RBF** dengan variasi nilai **C** dan **gamma**, dikombinasikan dengan **k-fold cross validation (k=5)** untuk mencari kombinasi hyperparameter terbaik.
	- Model terbaik kemudian dilatih ulang menggunakan seluruh data training dengan konfigurasi C dan gamma terbaik tersebut.

- **Evaluasi karakter (digit/karakter KTP)**
	- Dilakukan pemisahan data menjadi data training dan testing (hold-out split) untuk mengecek generalisasi model.
	- Hasil evaluasi ditampilkan dalam bentuk **confusion matrix** untuk melihat distribusi prediksi benar/salah per kelas karakter.
	- Nilai **Eval Accuracy** (akurasI pada data uji) yang diperoleh adalah sekitar **98,37%**.

Secara umum, angka akurasi 98,37% pada level karakter menunjukkan bahwa model OCR kustom mampu mengenali sebagian besar digit/karakter KTP dengan sangat baik. Sisa kesalahan biasanya muncul pada karakter yang bentuknya mirip (misalnya 0–O, 1–I–l) atau citra yang blur/noisy.

### 7.2. Ringkasan Hasil Lain yang Dapat Dicantumkan

- Akurasi training/validation/test model CNN untuk fraud detection.
- Nilai precision, recall, F1-score untuk kelas **Asli** dan **Palsu**.
- Contoh visualisasi: kurva loss & accuracy per epoch, confusion matrix untuk fraud detection.
- Contoh hasil OCR (tampilan teks asli di KTP vs hasil OCR dari model kustom).
- Analisis error dan faktor yang mempengaruhi performa (kualitas scan, blur, noise, resolusi rendah, dll.).

---

## 8. Kesimpulan & Pekerjaan Lanjutan

**Kesimpulan umum yang dapat diambil:**

- Model CNN mampu membedakan citra KTP asli dan palsu dengan akurasi tertentu (lengkapi sesuai hasil).
- Pipeline OCR dapat mengekstrak sebagian besar informasi penting dari KTP, meskipun masih terdapat kesalahan pada beberapa karakter/kata.
- Integrasi fraud detection dan OCR berpotensi menjadi sistem verifikasi KTP semi-otomatis untuk proses KYC.

**Pekerjaan lanjutan yang memungkinkan:**

- Menambah dan memperkaya dataset (variasi scanner/kamera, kondisi cahaya, kualitas cetak, dll.).
- Fine-tuning model CNN dengan arsitektur yang lebih kuat (misal transfer learning dari ResNet, EfficientNet, dsb.).
- Peningkatan pipeline OCR (segmentasi area teks per field, fine-tuning bahasa, atau training OCR khusus KTP).

---
