# P11 — Gaussian Naive Bayes dan SVM

Praktikum Pembelajaran Mesin, Pertemuan 11 (DAIS32W), praktikum individual.

**Data:** `load_iris()` dari scikit-learn (150 bunga, 3 spesies). Model memakai dua fitur petal (length, width). Split 90 / 30 / 30, stratified, seed 2026. Delapan ID demonstrasi manual (50, 51, 52, 53, 100, 101, 102, 123) dimasukkan ke training supaya validation dan test tidak bocor.

## Pembuktian manual
- **GaussianNB, dua batch** (versicolor vs virginica, δ = 0,05): prior 0,5 → 0,5; ID 53 → P(V) = 0,99934 (versicolor). Untuk ID 123, prediksi berbalik dari versicolor (δ = 0,05) ke virginica (δ = 0,005).
- **Kesetaraan dengan sklearn:** `var_smoothing = 0,05 / 0,328056 = 0,152413` menghasilkan `epsilon_` = 0,05 dan posterior yang identik dengan perhitungan manual. `partial_fit` dua batch sama dengan `fit` sekaligus.
- **SVM SGD manual** (λ = 0,1, η = 0,5, 5 update): satu fitur w = 1,3152, b = 0,5; dua fitur w = (0,8152; 0,6412), b = 0. Model dua fitur benar 3 dari 4 kueri (ID 123 salah).

## Perbandingan VALIDATION (fitur petal)
Semua model seri di validation dengan Macro-F1 = 1,000: GaussianNB, SVM Linear (22 SV), RBF (28 SV), Polynomial d2 (18 SV), KNN k5, dan Logistic Regression.
- `var_smoothing` 1e-9 s.d. 1e-1: semua 1,000
- SVM RBF C × gamma: 8 dari 9 kombinasi 1,000; hanya C = 0,1 & γ = 0,05 yang underfit (0,548)
- Polynomial: coef0 = 0 menurunkan skor (d2 0,901; d3 0,965)

## Konfigurasi final & TEST (satu kali, refit di train + val)
| Model | Konfigurasi | Macro-F1 test | Accuracy test |
|---|---|---|---|
| GaussianNB | var_smoothing = 1e-9 | 0,9333 | 0,9333 |
| **SVM RBF** | C = 0,1, γ = 0,5, StandardScaler | **0,9666** | **0,9667** |

Semua kesalahan terjadi antara versicolor dan virginica (misalnya ID 133). Selisih kedua model hanya 1 bunga, jadi tidak signifikan.

## Keterbatasan
1. Validation dan test masing-masing hanya 30 baris, dan banyak konfigurasi seri.
2. Asumsi independensi GaussianNB dilanggar: korelasi petal di dalam kelas versicolor 0,77.
3. Hanya satu dataset dan satu split. Langkah lanjut yang diusulkan: StratifiedKFold berulang dengan 4 fitur.

## Isi folder
- `P11_255314091_Bartolomeus_Simanjuntak_NaiveBayes_SVM.ipynb`: notebook utama (soal, kode, output, jawaban P11.1–P11.9)
