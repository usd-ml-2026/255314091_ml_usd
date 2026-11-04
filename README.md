# P08 — Memilih Hierarchical (Ward) atau K-Means berdasarkan EDA

Praktikum Pembelajaran Mesin, Pertemuan 8 (DAIS32W), praktikum individual.

**Dataset:** `sklearn.datasets.load_wine()`, 178 sampel × 13 fitur kimia numerik. Label cultivar (`wine.target`) **tidak** dipakai untuk fit maupun memilih model.

## Pertanyaan proyek
Bagaimana hasil EDA, scaling, feature selection, PCA, dan konteks pemakaian memengaruhi pilihan antara Ward dan K-Means?

## Alur eksperimen
Audit data (EDA) → baseline yang adil (Ward vs K-Means, X dan k sama) → ablation satu perubahan per run → simulasi kualitas data (missing/duplikat/outlier) → dua kondisi data (Wine vs 3.000 titik sintetis) → keputusan final → profil cluster.

## Temuan EDA utama
| Temuan | Angka |
|---|---|
| Ukuran & tipe | 178 × 13, semua `float64` |
| Missing / duplikat | 0 / 0 |
| Ketimpangan skala | rasio rentang terbesar/terkecil **2645×** (`proline` 1402 vs `nonflavanoid_phenols` 0.53) |
| Outlier 1.5×IQR | maksimal 4 per fitur (21 sel total) |
| Korelasi tertinggi | \|r\| `total_phenols`–`flavanoids` = 0.865 |

Keputusan preprocessing: **StandardScaler wajib**; outlier tidak dihapus; fitur redundan hanya diuji lewat ablation.

## Hasil perbandingan (k = 3, dievaluasi pada ruang StandardScaler yang sama)
| Run | Silhouette | DBI | Ukuran cluster |
|---|---|---|---|
| Ward, StandardScaler | 0.277 | 1.419 | 56 / 58 / 64 |
| **K-Means, StandardScaler** | **0.285** | **1.389** | 51 / 62 / 65 |
| K-Means, tanpa scaling | 0.108 | 3.364 | 47 / 62 / 69 |
| K-Means, PCA 90% (8 komponen) | 0.285 | 1.389 | 51 / 62 / 65 |
| Ward, outlier simulasi `proline`×8 | 0.260 | 1.439 | — |
| K-Means, outlier simulasi `proline`×8 | 0.278 | 1.400 | — |

## Keputusan final
- **Metode:** K-Means, **k = 3**, StandardScaler, 13 fitur (tanpa seleksi/PCA), tanpa clipping IQR, `n_init=20`, `random_state=2026`
- **Silhouette:** 0.2849, **DBI:** 1.3892 (ruang StandardScaler)
- **Alasan:** k = 3 memberi silhouette tertinggi untuk kedua metode, dan dendrogram Ward menunjukkan lompatan tinggi merge (±12.6 → ±27.7) yang mendukung tiga cabang. K-Means sedikit lebih baik dan paling stabil di semua ablation (0.280–0.285) serta terhadap outlier simulasi. Centroidnya juga memberi profil yang mudah ditafsirkan. Ward k = 3 tetap alternatif yang wajar (selisih silhouette hanya 0.008) bila tujuan utamanya eksplorasi hierarki.

**Profil cluster (satuan asli):**
| Cluster | n | alcohol | color_intensity | proline |
|---|---|---|---|---|
| proline tinggi | 62 | 13.68 | 5.45 | 1100.2 |
| alkohol & warna rendah | 65 | 12.25 | 2.97 | 510.2 |
| warna pekat | 51 | 13.13 | 7.24 | 619.1 |

Cluster ini adalah pola statistik, **bukan** kelas cultivar atau penilaian mutu.

## Dua kondisi data
Pada 3.000 titik sintetis (`make_blobs`), Ward dan K-Means memberi kualitas identik pada subsampel adil n = 450 (silhouette 0.715). K-Means dipilih karena tujuannya segmentasi (centroid dapat dipakai untuk `predict`), hierarki tidak dibutuhkan, dan matriks jarak tumbuh O(n²) (≈68.7 MB pada n = 3000). Pada data bulan sabit (`make_moons`), **keduanya** gagal mengikuti bentuk melengkung, yang menjadi alasan mempelajari DBSCAN.

## Keterbatasan
1. Hasil sangat sensitif terhadap scaling (silhouette 0.285 vs 0.108 tanpa scaling).
2. Struktur cluster hanya sedang (silhouette ≈0.28), jadi k = 3 adalah pilihan yang paling didukung bukti, bukan kebenaran mutlak.
3. Makna domain belum divalidasi; nomor cluster arbitrer dan bergantung pada seed/versi library.
4. Analisis ini eksplorasi deskriptif: scaler di-fit pada seluruh data, tanpa evaluasi out-of-sample.
5. Missing/duplikat/outlier di bagian P5 adalah **simulasi**, bukan sifat Wine asli.

## Isi repo
- `P08_Ward_vs_KMeans.ipynb`: notebook utama (soal, kode, output, jawaban)
- `P08_output/P08_experiment_table.csv`: semua run eksperimen
- `P08_output/P08_cluster_profile.csv`: profil cluster final
- `P08_output/P08_final_config.json`: konfigurasi final
- `P08_output/README_PORTOFOLIO_P08.md`: ringkasan yang dihasilkan otomatis oleh notebook

## Reproduksi
Python 3 (diuji pada 3.14.7), NumPy, Pandas, Matplotlib, SciPy 1.18.1, scikit-learn 1.9.1.

```bash
pip install numpy pandas matplotlib scipy scikit-learn jupyter
jupyter notebook P08_Ward_vs_KMeans.ipynb   # lalu Restart & Run All
```
