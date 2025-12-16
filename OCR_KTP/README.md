# OCR KTP Digit Recognition

## 1. Ringkasan
Dokumen ini menjelaskan pipeline OCR (Optical Character Recognition) numerik/karakter pada KTP Indonesia menggunakan pendekatan **classical computer vision** berbasis OpenCV dan model ML ringan (SVM / k-NN). README ini hanya fokus pada OCR sesuai permintaan (bagian fraud dihapus).

Notebook utama: `ocr_ktp.ipynb`  
Model tersimpan: `digit_svm_best_ml.xml` + konfigurasi fitur `digit_feature_config.json`.

## 2. Keterkaitan Dengan Artikel
Artikel rujukan: *"Histogram of Oriented Gradients Based Off-Line Handwritten Devanagari Characters Recognition Using SVM, K-NN and NN Classifiers"*.

| Konsep Artikel | Implementasi di Proyek | Adaptasi untuk Digit KTP |
|----------------|-----------------------|--------------------------|
| HOG sebagai fitur utama | Fungsi `extract_feature` (mode HOG) | HOG menangkap pola gradien tepi digit NIK |
| Evaluasi SVM & K-NN | SVM RBF grid search, opsi k-NN | Memilih SVM terbaik untuk akurasi tinggi |
| Pra-proses (binarisasi, normalisasi) | Otsu + invert + morphology + deskew | Menstabilkan bentuk digit sebelum fitur |
| Variasi bentuk tulisan/karakter | Deskew & resize konsisten | Mengurangi efek kemiringan/cropping |
| Optimasi hyperparameter | Grid search C & gamma | Menyesuaikan parameter dengan dataset lokal |

Perbedaan: Digit KTP bersifat printed bukan handwritten kompleks; namun prinsip penggunaan HOG + SVM tetap relevan dan transferable.

## 3. Pipeline OCR Digit
1. Preprocessing: grayscale → blur → Otsu → invert → morphological close.  
2. Deteksi ROI teks: MSER atau contour + filtering area/aspect ratio + NMS.  
3. Segmentasi karakter: proyeksi vertikal & pemisahan pada gap rendah piksel.  
4. Fitur: HOG (opsional deskew) atau flatten raw piksel.  
5. Klasifikasi: SVM (linear/RBF) atau k-NN.  
6. Inferensi end-to-end: bounding boxes + prediksi per ROI + strip karakter.  

## 4. Struktur Dataset Digit
```
dataset/
  train/
    0/ 1/ 2/ 3/ 4/ 5/ 6/ 7/ 8/ 9/   (gambar digit)
  test/ (opsional gambar uji)
```
Format gambar: `.png/.jpg/.jpeg/.bmp` grayscale atau RGB (akan dikonversi). Pastikan tiap subfolder berisi cukup sampel.

## 5. Persiapan Lingkungan
Minimal paket:
```
py -3.11 -m venv .venv
.venv\Scripts\activate
pip install --upgrade pip
pip install numpy opencv-contrib-python matplotlib
```
`opencv-contrib-python` dibutuhkan untuk HOGDescriptor dan MSER.

## 6. Cara Menjalankan
1. Buka `ocr_ktp.ipynb` dan jalankan sel environment repair (opsional).  
2. Pastikan struktur `dataset/train/0..9` tersedia.  
3. Jalankan sel pelatihan grid search (`train_digit_classifier_best`).  
4. Simpan model otomatis ke `digit_svm_best_ml.xml`.  
5. Set `TEST_IMAGE_PATH` ke gambar KTP penuh → jalankan pipeline inferensi.  
6. Tinjau visualisasi hasil (kotak + prediksi karakter).  

## 7. Evaluasi & Metrik
- Akurasi per digit (persentase benar).  
- Sequence accuracy (seluruh NIK tanpa kesalahan).  
- Confusion matrix 0–9 untuk analisis kesalahan.  
- Robustness manual: uji blur, noise, pencahayaan.  
Tambahkan hasil numerik setelah eksperimen.

## 8. Kontribusi & Kebaruan
- Adaptasi HOG + SVM dari domain karakter handwritten ke digit printed KTP.  
- Deskew otomatis menggunakan momen untuk konsistensi bentuk.  
- Deteksi ROI fleksibel (MSER / contour) dengan NMS sederhana.  
- Pipeline ringan tanpa deep learning → cocok resource terbatas / edge.  

## 9. Keterbatasan
- Kesalahan segmentasi jika jarak antar digit tidak konsisten.  
- MSER sensitif terhadap noise latar kompleks.  
- Tidak mendukung huruf (hanya digit).  
- Belum ada koreksi otomatis sequence (mis. validasi pola NIK).  

## 10. Arah Pengembangan
- Peningkatan segmentasi dengan connected components & grouping adaptif.  
- Tambah model CNN ringan untuk pembandingan.  
- Augmentasi sintetis (blur, perspektif, noise) untuk generalisasi.  
- Language model ringan untuk validasi/prediksi ulang NIK.  
- Ekspor model ke ONNX/WASM untuk deployment browser.  

## 11. Citing & Lisensi
Contoh sitasi artikel:
```
@article{DevanagariHOGYear,
  title={Histogram of Oriented Gradients Based Off-Line Handwritten Devanagari Characters Recognition Using SVM, K-NN and NN Classifiers},
  author={Nama Author},
  journal={Jurnal/Proceedings},
  year={Tahun},
  doi={DOI}
}
```
Lisensi kode: Tambahkan (mis. MIT) bila diperlukan. Saat ini tidak ada header lisensi per file.

## 12. Reproducibility
- Seed ditetapkan (42).  
- Versi paket: catat output sel environment.  
- Simpan model + `digit_feature_config.json`.  
- Opsional buat `requirements.txt` minimal.  

## 13. Ringkas Komponen
| Komponen | Fungsi | Tujuan |
|----------|--------|--------|
| Preprocessing | `preprocess_for_text` | Stabilkan citra teks & konversi biner |
| Deteksi ROI | `detect_text_regions` | Menemukan area angka untuk segmentasi |
| Segmentasi | `segment_characters` | Memecah ROI menjadi digit individual |
| Fitur | `extract_feature` (HOG) | Representasi gradien digit |
| Pelatihan | `train_digit_classifier_best` | SVM RBF optimum (grid search) |
| Inferensi | `OCRKTPPipeline.run_on_image` | Alur lengkap deteksi→prediksi |
| Visualisasi | `visualize_results` | Tampilkan kotak & hasil per karakter |

---
README ini telah dipadatkan agar fokus hanya pada OCR digit KTP.
