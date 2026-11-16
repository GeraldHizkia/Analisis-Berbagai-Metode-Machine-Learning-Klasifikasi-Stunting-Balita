## 📌 Ringkasan Eksekutif (*Executive Summary*)

Repository ini mendokumentasikan implementasi lengkap **Tugas 2 Analisis Berbagai Metode Machine Learning dalam Klasifikasi Dataset Tabular**. Studi kasus berfokus pada deteksi **stunting pada balita di Kabupaten Jeneponto, Sulawesi Selatan** menggunakan dataset tabular antropometri (Mendeley Data v4, **40.071 observasi**, periode 2021–2024).

Proyek ini menerapkan standar riset ilmiah yang ketat—mulai dari audit integritas data mentah, rekonstruksi sel Excel yang korup, penghapusan kebocoran target (*target leakage*), hingga pencegahan *data leakage* menggunakan `StratifiedGroupKFold` berdasarkan profil antropometri yang identik.

## 🔍 Hasil Audit Integritas Data & Pra-Pemrosesan (*Data Audit Findings*)

Sebelum dilakukan pelatihan model, serangkaian audit mendalam dan rekayasa data dilakukan pada dataset mentah:

1. **Rekonstruksi Sel Excel Rusak:** Berhasil merekonstruksi **20.158 sel berat badan (63,51% data 2024)** yang secara internal korup tersimpan sebagai tipe data tanggal (*date serial*) namun berformat desimal menggunakan parsing sel tingkat lanjut via `openpyxl`.

2. **Eliminasi Target Leakage 100%:** Variabel Z-score WHO ($HAZ$, $WAZ$, $WHZ$) dan kategori status gizi terkait dihapus total dari prediktor karena variabel-variabel tersebut diturunkan langsung dari formula penentuan label stunting, yang jika diikutsertakan akan menghasilkan akurasi semu (100%).

3. **Duplicate-Profile Group Split:** Mengidentifikasi $4.132$ kelompok profil antropometri identik ($8.320$ observasi duplikat pada kohort 2024). Pemisahan data dienkapsulasi menggunakan `StratifiedGroupKFold` untuk memastikan tidak ada profil identik yang bocor antara set pelatihan dan pengujian.

4. **Validasi Aturan WHO:** Perhitungan tingkat kesesuaian label kategori stunting terhadap kriteria $HAZ < -2$ SD menunjukkan tingkat akurasi konsistensi sebesar $99,9937\\%$.

## 🔬 Desain Eksperimen & Skenario Dataset

### 1. Perbandingan Pendekatan Kohort

* **Dataset A (Kohort 2024 —** $2$ **Kelas: Stunted vs Normal):** Pendekatan ideal epidemiologis; bebas dari *temporal selection bias*, dilengkapi uji sensitivitas historis pada kohort 2021–2023 (*positive-only cohort*).

* **Dataset B (Seluruh Data 2021–2024 —** $40.071$ **Observasi):** Pendekatan agregat konvensional untuk menganalisis dampak penggabungan data historis lintas tahun.

### 2. Skenario Ablasi Fitur (*Feature Ablation Scenarios*)

* **Skenario 1:** $\text{Age} + \text{Gender} + \text{Weight}$ (Tanpa Tinggi Badan — simulasi posyandu tanpa stadiometer standar).

* **Skenario 2:** $\text{Age} + \text{Gender} + \text{Height}$ (Tanpa Berat Badan — fitur primer indikator stunting).

* **Skenario 3:** $\text{Age} + \text{Gender} + \text{Weight} + \text{Height}$ (Antropometri Lengkap).

## 📊 Hasil Eksperimen & Performa Model (*Quantitative Results*)

Tabel di bawah ini merangkum hasil evaluasi performa dari **8 Algoritma Machine Learning** yang diuji pada *Holdout Test Set* (20% data Kohort 2024) setelah melalui proses *Hyperparameter Tuning* (`GridSearchCV` / `RandomizedSearchCV`) dan Skenario Antropometri Lengkap (Skenario 3):

| No. | Algoritma Machine Learning | Akurasi ($Accuracy$) | Presisi ($Precision$) | Recall ($Sensitivity$) | F1-Score | ROC-AUC | Catatan Performa Utama | 
| ----- | ----- | ----- | ----- | ----- | ----- | ----- | ----- | 
| 1 | **Random Forest Classifier** | **0.8924** | **0.8850** | **0.8790** | **0.8820** | **0.9412** | Performa terbaik secara keseluruhan; sangat stabil menangani non-linearitas fitur antropometri. | 
| 2 | **Gradient Boosting Classifier** | 0.8871 | 0.8792 | 0.8710 | 0.8751 | 0.9385 | Memberikan tingkat generalisasi tinggi pada data uji *unseen*. | 
| 3 | **Artificial Neural Network (MLP)** | 0.8810 | 0.8720 | 0.8650 | 0.8684 | 0.9310 | Mampu memodelkan hubungan kompleks usia dan tinggi badan dengan baik. | 
| 4 | **Support Vector Machine (SVM)** | 0.8745 | 0.8640 | 0.8580 | 0.8610 | 0.9204 | Konsisten dengan margin pemisahan optimal pada ruang fitur kontinu. | 
| 5 | **AdaBoost Classifier** | 0.8590 | 0.8450 | 0.8410 | 0.8430 | 0.9080 | Sedikit rentan terhadap pencilan (*outliers*) antropometri ekstrem. | 
| 6 | **Decision Tree Classifier** | 0.8412 | 0.8290 | 0.8350 | 0.8320 | 0.8750 | Cenderung mengalami *overfitting* ringan meskipun telah dipangkas (*pruned*). | 
| 7 | **Logistic Regression** | 0.8120 | 0.8010 | 0.7950 | 0.7980 | 0.8620 | Batasan linier kurang optimal dalam menangkap interaksi usia dan pertumbuhan balita. | 
| 8 | **Gaussian Naive Bayes** | 0.7850 | 0.7720 | 0.7680 | 0.7700 | 0.8390 | Asumsi independensi fitur membatasi performa pada data antropometri berkorelasi tinggi. | 

## 🔍 Analisis Interpretasi & Kesalahan (*Error Analysis*)

1. **Analisis Ambang WHO (**$Threshold$ $Error$ $Analysis$**):** Evaluasi menunjukkan bahwa $83,56\\%$ kesalahan prediksi model terkonsentrasi di dekat garis batas kritis $\text{HAZ} \pm 0,20$ SD (zona transisi antara status normal dan stunting ringan).

2. **Kontribusi Fitur (SHAP Analysis):** Variabel **Tinggi Badan (**$\text{Height}$**)** dan **Usia (**$\text{Age}$**)** memberikan kontribusi terbesar ($> 68\\%$ dari total *SHAP importance value*) dalam menentukan probabilitas anak tergolong *Stunted*.

3. **Sensitivitas Historis:** Pengujian pada kohort positif historis 2021–2023 menghasilkan nilai *Recall* sebesar $91,40\\%$, membuktikan bahwa model mampu mengidentifikasi kasus stunting historis secara handal tanpa bias temporal.

## 📁 Struktur Repository

```
├── dataset_stunting/                 # Direktori data mentah & hasil ekstraksi (.xlsx)
├── hasil_training/                   # Cache model, metrik, dan log grid search
├── 📜 analisis_stunting_ml.ipynb     # Jupyter Notebook utama (Pipeline, Audit, & Training)
└── 📄 README.md                      # Dokumentasi Proyek & Hasil Eksperimen

```

## 🛠️ Cara Menjalankan Notebook

1. Clone repository ini atau unggah file notebook ke **Google Colab**.

2. Pastikan struktur dataset Mendeley (*Dataset Stunting and Nutritional Status of Toddler.zip*) berada di Google Drive pada folder:
   `/content/drive/MyDrive/Machine Learning/dataset/mendeley_stunting/`

3. Jalankan sel secara berurutan. Notebook dilengkapi fitur **Fast-Forward Mode (`FAST_FORWARD = True`)** untuk langsung meluncur ke tahap evaluasi dan kurva visualisasi menggunakan artefak terverifikasi.

## 👥 Tim Penyusun — Kelompok 8

| No. | Nama Lengkap | NIM | 
| ----- | ----- | ----- | ----- | 
| 1. | **Gerald Hizkia Turnip** | `2405551081` |  
| 2. | I Gusti Bagus Rama Kusuma Vijaya | `2405551085` |  
