# P10 — Linear Regression, KNN, dan Logistic Regression

Praktikum Pembelajaran Mesin, Pertemuan 10 (DAIS32W), praktikum individual.

**Data:** `load_diabetes(scaled=False)` (442 pasien, 10 fitur, target numerik) dan `load_iris()` (150 bunga, 4 fitur, 3 spesies), keduanya dari scikit-learn. Split train/validation/test dengan seed 2026. Test hanya dibuka sekali setelah konfigurasi dikunci.

## Perhitungan manual
- **Gradient descent LR** (x = [0,1,2,3], y = [1,3,5,7], α = 0,1): iterasi 0 → grad_w = −17, grad_b = −8, (w, b) = (1,7; 0,8); iterasi 1 → (1,97; 0,93). MSE 21 → 0,535 → 0,014.
- **KNN voting** (petal length/width, 10 referensi): Q1 = ID 52 salah pada k = 1, benar pada k = 3/5. Q2 = ID 129 salah pada k = 1/3, benar pada k = 5.
- **Sigmoid & Log Loss** (w = 2, b = −10,5): P(virginica) = 0,3318 (ID 52) dan 0,7503 (ID 129), BCE rata-rata 0,3453.

## Regresi Diabetes (VALIDATION)
| Model | MAE | RMSE | R² |
|---|---|---|---|
| **LinearRegression 10 fitur** | **46,63** | **56,20** | **0,519** |
| LinearRegression bmi + s5 (ablation) | 48,30 | 56,16 | 0,520 |
| KNNReg k=15 StandardScaler | 50,36 | 60,05 | 0,451 |
| LinearRegression BMI | 54,99 | 64,74 | 0,362 |
| KNNReg k=5 tanpa scaling | 61,32 | 73,12 | 0,186 |
| Dummy median | 69,94 | 83,20 | −0,054 |

Scaling penting untuk KNN, tetapi tidak mengubah prediksi OLS. Multikolinearitas s1–s2 tinggi (r = 0,899).

## Klasifikasi Iris (VALIDATION, Macro-F1)
- KNN all4 + StandardScaler, k = 3–9: **1,000**; petal2: 0,967–1,000; sepal2: 0,576–0,798
- Logistic Regression all4 + StandardScaler, C = 1: 0,9666 (salah 1: ID 77)
- KNN k = 5 + PCA(2) (EVR 95,8%): 0,9327
- Baseline mayoritas: 0,167

## Konfigurasi final & TEST (satu kali)
| Tugas | Konfigurasi | Hasil test |
|---|---|---|
| Regresi | LinearRegression 10 fitur | MAE 41,52 · RMSE 52,34 · R² 0,539 |
| Klasifikasi | KNN all4 + StandardScaler + k = 5 | Accuracy = Macro-F1 = 0,933 (2 salah, versicolor ↔ virginica) |

## Keterbatasan
1. Validation/test Iris hanya 30 baris, jadi 1 bunga = 3,3 poin.
2. Model Diabetes masih underfit (R² ±0,54) dan cenderung underestimate target tinggi. Model ini **bukan** alat keputusan klinis.
3. Koefisien LR tidak bisa dibaca kausal karena multikolinearitas.

## Isi folder
- `P10_Praktikum_LR_KNN_Logistic.ipynb`: notebook utama (soal, kode, output, jawaban)
- `P10_Hasil_Validasi.csv`: tabel eksperimen validation (regresi + klasifikasi)
- `P10_Final_Config.json`: konfigurasi final, seed, dan metrik test
