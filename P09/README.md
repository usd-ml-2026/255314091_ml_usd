# P09 — DBSCAN: dari data sensor mentah hingga keputusan algoritma

Praktikum Pembelajaran Mesin, Pertemuan 9 (DAIS32W), praktikum individual.

**Data:** `p09_sensor_raw_manual.csv`, berisi 15 baris koordinat alat ukur sintetis (x, y dalam meter) + `sensor_aux` + `shift`. Tambahan untuk P7: `make_moons` (240 titik sintetis). Tidak ada label yang dipakai untuk fit maupun memilih model.

## Pertanyaan proyek
Lokasi sensor mana yang membentuk kelompok padat, lokasi mana yang terpencil, dan kapan DBSCAN lebih tepat daripada K-Means atau Ward?

## Audit & cleaning
| Masalah | Baris | Tindakan |
|---|---|---|
| Desimal koma (`"3,8"`) | K | di-parse menjadi 3.8 |
| Duplikat persis | C | dibuang 1 |
| Koordinat hilang | M (y kosong) | dibuang (posisi tidak diimputasi) |
| Di luar domain 0≤x≤10 | N (x = 999) | dibuang |

15 baris → **12 titik (A–L)**. Fitur jarak: **x_m, y_m saja, dalam meter, tanpa scaling**. `sensor_aux` (670–1680, r ≈ 0 dengan posisi) dan `shift` (kategorik) tidak dipakai untuk menghitung jarak.

## DBSCAN manual vs scikit-learn (eps = 1.1 m, min_samples = 3)
- **Core:** A, B, C, D, F, G, H, I · **Border:** E, J · **Noise:** K (tetangga terdekat 3.33 m), L (4.12 m)
- Cluster 0 = {A–E}, cluster 1 = {F–J}. Hasil hitungan manual sama persis dengan `sklearn.cluster.DBSCAN`.

## Sensitivitas
| eps | min_samples | cluster | noise | coverage | silhouette (retained) |
|---|---|---|---|---|---|
| 0.9 | 3 | 0 | 12 | 0% | — |
| **1.1** | **3** | **2** | **2** | **83.3%** | **0.773** |
| 1.5 / 2.5 | 3 | 2 | 2 | 83.3% | 0.773 |
| 1.1 | 4 | 2 | 4 | 66.7% | 0.780 |
| 1.1 | 2 | 2 | 2 | 83.3% | 0.773 |

## Perbandingan adil
| Metode | Coverage | Silhouette (titik yang dinilai) | Silhouette pada 10 titik yang sama |
|---|---|---|---|
| K-Means k=2 | 100% | 0.605 | 0.773 |
| Ward k=2 | 100% | 0.605 | 0.773 |
| DBSCAN | 83.3% | 0.773 | 0.773 |

Skor DBSCAN yang lebih tinggi **seluruhnya berasal dari membuang K dan L**. Di subset yang sama, ketiganya setara.

## Pengaruh preprocessing (eps 1.1 tetap)
- XY + `sensor_aux` mentah: **12/12 noise**. XY + `sensor_aux` terstandar: 6 noise (semuanya Shift A).
- XY terstandar: keanggotaan sama, tapi silhouette 0.686 karena eps tidak lagi bermakna meter.
- Kalau nilai x = 999 tidak dibuang, centroid K-Means menjadi **(999, 0)** dan (4.23, 1.23).

## Dua bulan sabit
DBSCAN (eps 0.23, min_samples 5) mengikuti bentuk lengkung, sedangkan K-Means/Ward memotong lurus. Tapi silhouette justru K-Means 0.486 > Ward 0.436 > DBSCAN 0.328, karena metrik internal bias terhadap cluster cembung.

## Keputusan final
**DBSCAN, eps = 1.1 m, min_samples = 3, fitur x_m dan y_m tanpa scaling.** Hasilnya 2 kelompok lokasi dan 2 lokasi terpencil (K, L) untuk ditinjau. Titik noise **bukan** data salah: K dan L adalah posisi sah dan tidak boleh langsung dihapus.

## Keterbatasan
1. Hanya 12 titik sintetis, jadi kesimpulan tidak bisa digeneralisasi.
2. eps = 1.1 hanya bermakna di ruang XY dalam meter.
3. Silhouette/DBI DBSCAN dihitung pada titik retained, sehingga harus dibaca bersama coverage.

## Isi folder
- `P09_Praktikum_DBSCAN.ipynb`: notebook utama (soal, kode, output, jawaban P1–P7 & L1–L9)
- `p09_sensor_raw_manual.csv`: data sensor mentah

## Reproduksi
Python 3.14.7, NumPy 2.5.4, pandas 3.0.6, scikit-learn 1.9.1, SciPy 1.18.1, Matplotlib 3.11.2, `SEED = 2026`.

```bash
cd P09
jupyter notebook P09_Praktikum_DBSCAN.ipynb   # lalu Restart & Run All
```
