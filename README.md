# Sistem Penalaran Berbasis Kasus untuk Putusan Mahkamah Agung

Proyek ini adalah implementasi sistem Penalaran Berbasis Kasus (Case-Based Reasoning - CBR) untuk menganalisis dan mengambil putusan hukum dari direktori Mahkamah Agung Republik Indonesia. Sistem ini mencakup seluruh alur kerja CBR, mulai dari pengumpulan data (case base), representasi kasus, pengambilan kasus (retrieval), hingga penggunaan kembali solusi (solution reuse) dan evaluasi model.

Proyek ini dikembangkan sebagai bagian dari Ujian Akhir Semester (UAS) mata kuliah Penalaran Komputer oleh:
- **Haidar Dimas Heryanto** - 202210370311088
- **Zeedan Mustami Argani** - 202210370311104

**Topik Kasus:** Perdagangan Orang

## Deskripsi Proyek

Sistem ini dibangun dalam satu file Jupyter Notebook (`ALL_Penalaran_Komputer_UAS.ipynb`) yang terbagi menjadi lima tahap utama:

1.  **Tahap 1: Membangun Case Base (Scraping & Cleaning)**
    - Melakukan scraping data putusan dari situs Mahkamah Agung berdasarkan kata kunci.
    - Mengunduh file PDF putusan yang relevan.
    - Mengekstrak teks dari file PDF menggunakan `pdfminer`.
    - Membersihkan teks mentah dari disclaimer dan noise lainnya.
    - Menyimpan metadata ke dalam file CSV dan teks mentah ke dalam file `.txt`.

2.  **Tahap 2: Representasi Kasus (Case Representation)**
    - Memuat data yang telah di-scrape.
    - Mengekstrak fitur-fitur penting dari teks lengkap putusan menggunakan teknik heuristik dan regex, seperti:
      - `ringkasan_fakta`
      - `argumen_hukum_utama`
      - `pasal_digunakan`
    - Melakukan feature engineering sederhana seperti menghitung jumlah kata.
    - Menyimpan hasil pengolahan ke dalam file `cases_processed.csv`.

3.  **Tahap 3: Pengambilan Kasus (Case Retrieval)**
    - Membangun model representasi vektor untuk setiap kasus menggunakan dua metode:
      - **TF-IDF**: Model statistik berbasis frekuensi kata.
      - **BERT**: Model Transformer pre-trained (`indobenchmark/indobert-base-p1`) untuk mendapatkan embedding semantik.
    - Mengimplementasikan fungsi `retrieve` untuk mencari kasus-kasus yang paling mirip dengan query baru berdasarkan *cosine similarity*.
    - Membuat file `queries.json` berisi contoh query dan *ground truth* untuk evaluasi.

4.  **Tahap 4: Penggunaan Kembali Solusi (Solution Reuse)**
    - Mengimplementasikan fungsi `predict_outcome` yang mengambil kasus-kasus hasil retrieval.
    - Memprediksi solusi (kategori amar putusan) untuk query baru menggunakan dua algoritma:
      - **Majority Vote**: Memilih solusi yang paling sering muncul dari kasus-kasus teratas.
      - **Weighted Similarity**: Serupa dengan majority vote, namun setiap suara diberi bobot berdasarkan skor kemiripan.
    - Menyimpan hasil prediksi ke `predictions.csv`.

5.  **Tahap 5: Evaluasi & Revisi**
    - Mengekstrak dan mengkategorikan amar putusan (`amar_kategori`) dari teks lengkap dan menyimpannya kembali ke `cases_processed.csv`.
    - Mengevaluasi performa model *retrieval* menggunakan metrik **Precision@k, Recall@k, dan F1-score@k**.
    - Mengevaluasi performa model *prediksi solusi* menggunakan **Accuracy, Precision, Recall, dan F1-score**.
    - Membuat visualisasi untuk membandingkan performa model.
    - Memberikan panduan untuk analisis kegagalan (error analysis).

## Struktur Proyek

Proyek ini dirancang untuk dijalankan di Google Colab dan menggunakan Google Drive untuk penyimpanan data. Pastikan Anda memiliki struktur folder berikut di dalam folder proyek utama Anda di Google Drive.

/content/drive/MyDrive/Penalaran Komputer UAS/
├── ALL_Penalaran_Komputer_UAS.ipynb
├── data/
│   ├── raw/
│   │   └── (Berisi file case_XXX.txt hasil scraping Tahap 1)
│   ├── processed/
│   │   └── cases_processed.csv
│   ├── eval/
│   │   ├── queries.json
│   │   ├── retrieval_metrics.csv
│   │   └── prediction_metrics.csv
│   └── results/
│       └── predictions.csv
├── PDFs_Putusan/
│   └── (Berisi file PDF putusan hasil unduhan Tahap 1)
├── Scraper_CSVs/
│   └── (Berisi file CSV metadata hasil scraping Tahap 1)
└── logs/
└── cleaning.log

## Penyiapan & Instalasi

1.  **Prasyarat**:
    - Akun Google dengan akses ke Google Colab dan Google Drive.
    - Browser web.

2.  **Instalasi**:
    - Buka file `ALL_Penalaran_Komputer_UAS.ipynb` di Google Colab.
    - Buat folder proyek utama di Google Drive Anda (misalnya, `Penalaran Komputer UAS`).
    - Instal semua pustaka yang dibutuhkan dengan menjalankan sel kode yang berisi perintah `pip install`. Atau, Anda dapat membuat *virtual environment* lokal dan menginstal menggunakan `requirements.txt` yang disediakan.
      ```bash
      pip install -r requirements.txt
      ```
    - Pustaka NLTK akan mengunduh data `punkt` dan `stopwords` secara otomatis saat sel yang relevan dijalankan.

## Cara Menjalankan

Notebook ini dirancang untuk dijalankan secara berurutan dari sel pertama hingga terakhir. Setiap tahap bergantung pada output dari tahap sebelumnya.

1.  **Mount Google Drive**: Jalankan sel untuk me-mount Google Drive Anda dan berikan otorisasi yang diperlukan.

2.  **Konfigurasi Tahap 1**:
    - Buka sel "Configuration Section" di awal notebook.
    - **Wajib diubah**:
      - `BASE_DRIVE_PATH`: Sesuaikan path ini dengan lokasi folder proyek Anda di Google Drive. Contoh: `"/content/drive/MyDrive/UAS_PK"`
      - `MA_SEARCH_RESULT_URL`: Masukkan URL halaman hasil pencarian dari situs Mahkamah Agung untuk topik yang Anda inginkan. Contohnya sudah ada di kode (`q=Perdaganganorang`).
      - `KEYWORD_FOR_FILENAMING`: Berikan kata kunci singkat untuk penamaan file output.
      - `MIN_DOCUMENTS_TO_SCRAPE`: Tentukan jumlah minimum dokumen yang ingin di-scrape (misal: 100).

3.  **Jalankan Sel per Tahap**:
    - **Tahap 1**: Akan memulai proses scraping. Proses ini mungkin memakan waktu lama tergantung jumlah dokumen dan koneksi internet.
    - **Tahap 2**: Akan memproses file teks mentah dan mengekstrak fitur.
    - **Tahap 3**: Akan membangun model TF-IDF dan BERT, lalu menguji fungsi retrieval.
    - **Tahap 4**: Akan menjalankan prediksi solusi berdasarkan hasil retrieval.
    - **Tahap 5**: Pertama, sel ini akan meng-update `cases_processed.csv` dengan `amar_kategori`. Setelah itu, sel-sel berikutnya akan menjalankan evaluasi penuh dan menghasilkan file metrik serta visualisasi.

4.  **Evaluasi (Penting)**:
    - Untuk mendapatkan hasil evaluasi yang akurat di **Tahap 5**, Anda **wajib** mengedit file `data/eval/queries.json` secara manual.
    - File ini dibuat di **Tahap 3** dengan *ground truth* kosong.
    - Buka file tersebut dan isilah `ground_truth_ids` (daftar `case_id` yang relevan) dan `ground_truth_solution` (kategori amar yang benar) untuk setiap query.

## Deskripsi Output

-   **`data/raw/`**: Kumpulan file teks (`.txt`) berisi teks lengkap dari setiap putusan yang telah dibersihkan.
-   **`data/processed/cases_processed.csv`**: File CSV utama yang berisi data terstruktur untuk setiap kasus, termasuk metadata, fitur yang diekstrak (`ringkasan_fakta`, `argumen_hukum_utama`), dan kategori amar.
-   **`data/results/predictions.csv`**: Log hasil prediksi dari Tahap 4. Berisi ID query, metode yang digunakan, solusi yang diprediksi, dan ID kasus yang relevan.
-   **`data/eval/`**: Folder berisi file-file untuk evaluasi.
    - `queries.json`: Daftar query yang digunakan untuk pengujian.
    - `retrieval_metrics.csv`: Hasil metrik performa (Precision, Recall, F1) untuk sistem retrieval.
    - `prediction_metrics.csv`: Hasil metrik performa (Accuracy, dll.) untuk sistem prediksi solusi.
-   **`PDFs_Putusan/`**: Arsip file PDF asli yang diunduh dari situs Mahkamah Agung.
