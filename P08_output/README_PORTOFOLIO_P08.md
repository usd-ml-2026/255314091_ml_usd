# P08 — Memilih Hierarchical atau K-Means berdasarkan EDA

**Dataset utama:** scikit-learn Wine, 178 sampel × 13 fitur numerik. Label target tidak digunakan.

## Pertanyaan proyek
Bagaimana pilihan scaling, feature selection, PCA, dan konteks pemakaian memengaruhi pemilihan Ward atau K-Means?

## Alur eksperimen
Audit data → baseline fair comparison → ablation → simulasi kualitas data → keputusan akhir → profil cluster.

## Keputusan final (perlu Anda jelaskan kembali dengan angka)
- Metode: K-Means
- k: 3
- Scaling: StandardScaler
- Representasi: ALL
- Clipping IQR: False
- Silhouette pada ruang StandardScaler: 0.2849
- DBI pada ruang StandardScaler: 1.3892
- Alasan: Wine bersih (0 missing, 0 duplikat) tetapi skalanya timpang (rasio rentang 2645×), sehingga wajib StandardScaler. Pada X_std yang sama, k=3 memberi silhouette tertinggi (K-Means 0.285 vs Ward 0.277) dan DBI K-Means 1.389. Dendrogram Ward menunjukkan lompatan merge ±12.6 → ±27.7 yang mendukung tiga cabang. K-Means dipilih karena paling stabil di semua ablation (0.280–0.285) dan terhadap outlier simulasi (0.278 vs Ward 0.260), serta centroidnya mudah ditafsirkan. Ward tetap alternatif wajar (ukuran cluster k=3 serupa) bila tujuan utamanya eksplorasi hierarki.

## Bukti
- Notebook utama `.ipynb` (soal, kode, output, interpretasi)
- `P08_experiment_table.csv`
- `P08_cluster_profile.csv`
- `P08_final_config.json`

## Keterbatasan
1. **Sensitif terhadap skala:** tanpa scaling, silhouette_common jatuh ke ≈0.11 karena `proline` mendominasi; hasil bergantung pada pilihan scaler.
2. **Pilihan k tidak mutlak:** silhouette k=3 hanya sedang (≈0.28) dan beda Ward–K-Means sangat kecil; k lain masih bisa dibela untuk tujuan berbeda.
3. **Makna domain belum divalidasi:** cluster adalah pola statistik, bukan kelas cultivar atau mutu; label target tidak dipakai, dan interpretasi kimia perlu ahli.
4. **Generalisasi & biaya:** scaler di-fit pada seluruh 178 sampel (eksplorasi deskriptif, tanpa evaluasi out-of-sample); Ward memerlukan matriks jarak O(n²) bila diterapkan pada data besar.

## Reproduksi
Python 3; NumPy; Pandas; SciPy; scikit-learn; Matplotlib. Jalankan notebook dari atas sampai bawah.
