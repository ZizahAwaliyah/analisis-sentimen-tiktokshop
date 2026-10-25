# Analisis Sentimen dan Jejaring Sosial: Percakapan TikTok Shop di Platform X

Pipeline analisis end-to-end untuk memahami percakapan publik tentang **TikTok Shop** di platform X (Twitter) selama bulan Oktober 2025. Proyek ini menggabungkan tiga pendekatan: analisis sentimen berbasis *lexicon*, *topic clustering* dengan K-Means, dan *Social Network Analysis* (SNA) untuk memetakan struktur komunitas percakapan.

> Proyek ini merupakan tugas individu dalam konteks akademik, dengan positioning sebagai **baseline analysis**. Metode-metode yang digunakan (lexicon sentiment, K-Means) dipilih untuk interpretabilitas dan efisiensi; model yang lebih canggih seperti IndoBERT berpotensi memberikan akurasi lebih tinggi dan menjadi arah pengembangan lanjutan.

---

## Konteks

| | |
|---|---|
| **Mata Kuliah** | Analisis Jejaring Sosial (SNA) |
| **Semester** | 5 |
| **Jenis** | Proyek Individu |
| **Periode** | 2025 |

---

## Tujuan Penelitian

1. Memfilter dan memisahkan tweet berisi persepsi pelanggan asli dari tweet promosi/jualan.
2. Mengklasifikasikan persepsi pelanggan ke dalam tiga kategori sentimen: positif, negatif, netral.
3. Mengidentifikasi topik-topik utama yang dibicarakan pelanggan melalui *topic clustering*.
4. Melakukan analisis jejaring untuk mengidentifikasi pengguna berpengaruh dan struktur komunitas.
5. Memberikan rekomendasi strategis berdasarkan temuan analisis.

---

## Dataset

**Sumber:** Platform X (Twitter) via [tweet-harvest v2.6.1](https://www.npmjs.com/package/tweet-harvest)
**Keyword:** `tiktok shop`
**Periode:** 1–31 Oktober 2025
**Bahasa:** Indonesia (`lang:id`)
**Volume mentah:** 1.281 tweet (akumulasi ~500 tweet/hari)

Dataset tidak di-*commit* ke repo ini karena berisi data publik yang dinamis dan untuk menghindari isu privasi *user handle*.

---

## Metodologi

### 1. Scraping Data (`data_scraping.ipynb`)
- *Tool:* `tweet-harvest` v2.6.1 via Node.js
- Scraping harian selama 31 hari, hasil per hari disimpan sebagai CSV terpisah
- File harian digabung menjadi satu dataset master menggunakan `pandas.concat`

### 2. Preprocessing (`visualisasi.ipynb`)
- **Filter bahasa** `lang == 'in'`
- **Klasifikasi "Jualan vs Persepsi"** berbasis aturan:
  - Tweet yang merupakan *reply* → otomatis dikategorikan sebagai Persepsi
  - Tweet non-reply yang mengandung keyword promosi (`promo`, `diskon`, `shopee`, dsb) → Jualan
  - Sisanya → Persepsi
- **Text cleaning:** lowercase, hapus URL/mention/hashtag/tanda baca/angka
- **Tokenisasi** dengan NLTK
- **Normalisasi slang** menggunakan kamus kustom
- **Stopword removal** gabungan: NLTK Bahasa Indonesia + Inggris + stopword kustom
- Filter token dengan panjang > 2 karakter

### 3. Analisis Sentimen (Lexicon-Based)
- Kamus sentimen manual: ~20 kata positif dan ~20 kata negatif berbahasa Indonesia
- Skor sentimen = jumlah kata positif − jumlah kata negatif
- Klasifikasi: Positif (skor > 0), Negatif (skor < 0), Netral (skor = 0)
- **Validasi:** 100 tweet sampel dilabeli manual sebagai *ground truth*, lalu dibandingkan dengan prediksi lexicon menggunakan *classification report* dan *confusion matrix*

### 4. Topic Clustering
- Vektorisasi: **TF-IDF** (max 1.000 fitur)
- Algoritma: **K-Means** dengan `k=5`, `n_init=10`
- Interpretasi topik dilakukan secara manual berdasarkan Top 10 kata kunci per cluster
- Visualisasi dengan reduksi dimensi **t-SNE** (2D scatter plot)

### 5. Social Network Analysis
- Konstruksi graf berarah menggunakan `networkx`:
  - *Source* = pembuat tweet (`user_id_str`)
  - *Target* = akun yang dibalas (`in_reply_to_screen_name`)
- Metrik: **degree centrality** untuk identifikasi pengguna paling berpengaruh
- Deteksi komunitas: **greedy modularity** (algoritma pada graf *undirected*)
- *Edge list* diekspor ke format CSV untuk visualisasi lanjutan di **Gephi**

### 6. Analisis Gabungan
- *Cross-tabulation* Sentimen × Topik (jumlah tweet dan persentase per topik)
- Visualisasi *stacked horizontal bar chart*

---

## Hasil & Visualisasi

Notebook menghasilkan tiga visualisasi utama:
1. **Confusion Matrix** — evaluasi lexicon sentiment vs 100 label manual
2. **t-SNE Scatter Plot** — representasi 2D dari 5 cluster topik
3. **Stacked Bar Chart** — distribusi sentimen per topik (dalam persentase)

Interpretasi lengkap tersedia di `laporan/Laporan_Analisis_Sentimen.pdf`.

---

## Keterbatasan dan Arah Pengembangan

Transparansi atas batasan metodologis:

1. **Lexicon sentimen bersifat sederhana** — hanya ~40 kata total. Pendekatan ini memiliki *trade-off* antara interpretabilitas (jelas kata mana yang memicu klasifikasi) dan akurasi. Model berbasis *transformer* (misalnya IndoBERT) berpotensi memberi akurasi lebih tinggi.
2. **Kamus normalisasi slang terbatas** — hanya ~20 entri; banyak slang Twitter spesifik yang mungkin terlewat.
3. **Jumlah cluster k=5 ditentukan secara eksploratif** — belum divalidasi dengan metode kuantitatif seperti *Elbow Method* atau *Silhouette Score*.
4. **Interpretasi label topik subjektif** — nama cluster ("Live Shopping", "UX/Checkout", dsb.) diberikan secara manual berdasarkan Top 10 kata kunci.
5. **Filter aturan "Jualan vs Persepsi"** rentan terhadap *edge case* seperti sarkasme atau promo tersembunyi yang tidak menggunakan kata kunci standar.

Arah pengembangan yang disarankan: penggunaan model sentimen berbasis *transformer* (IndoBERT, mBERT), penerapan Elbow/Silhouette untuk validasi *k*, dan evaluasi filter *jualan* dengan sampel manual yang lebih besar.

---

## Teknologi

- **Bahasa:** Python 3
- **Lingkungan:** Google Colab
- **Scraping:** Node.js, tweet-harvest v2.6.1
- **Pustaka utama:**
  - Data: Pandas, NumPy
  - NLP: NLTK (stopwords, tokenisasi)
  - ML: Scikit-learn (TF-IDF, K-Means, t-SNE, classification metrics)
  - Jejaring: NetworkX (deteksi komunitas, centrality)
  - Visualisasi: Matplotlib, Seaborn
- **Visualisasi jejaring eksternal:** Gephi

---

## Struktur Repository

```
├── notebook/
│   ├── data_scraping.ipynb        # Pipeline scraping harian
│   └── visualisasi.ipynb          # Preprocessing, analisis, visualisasi
├── laporan/
│   └── Laporan_Analisis_Sentimen.pdf
├── visualisasi/
│   ├── confusion_matrix.png
│   ├── tsne_scatter_plot.png
│   └── stacked_bar_sentimen_topik.png
└── README.md
```

---

## Catatan Keamanan

Notebook `data_scraping.ipynb` menggunakan *auth token* X/Twitter. Token asli telah **dihapus dan diganti placeholder** sebelum repository ini dipublikasikan. Pengguna yang ingin menjalankan ulang pipeline scraping perlu menyediakan token mereka sendiri melalui dokumentasi [tweet-harvest](https://www.npmjs.com/package/tweet-harvest).

---

## Kontak

**Azizah Awaliyah**
Mahasiswa Teknik Informatika — Universitas Hasanuddin
Email: azizahawaliyahjijah@gmail.com
GitHub: [@ZizahAwaliyah](https://github.com/ZizahAwaliyah)
