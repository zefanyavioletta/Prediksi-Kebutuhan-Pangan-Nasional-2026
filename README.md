# Prediksi Kebutuhan Pangan Nasional 2026 (PatchTST)

Sistem peramalan konsumsi pangan nasional per provinsi-komoditas menggunakan
PatchTST (Patch Time Series Transformer), dengan dashboard Decision Support
System (DSS) untuk menerjemahkan hasil prediksi menjadi rekomendasi alokasi
logistik berbasis standar gizi nasional.

## Masalah

Data konsumsi pangan publik Indonesia tersedia per provinsi dan komoditas,
tetapi keputusan alokasi stok masih banyak mengandalkan data historis mentah
tanpa proyeksi ke depan. Proyek ini menguji apakah model deep learning
time-series (PatchTST) bisa memberi proyeksi yang lebih berguna dibanding
sekadar memakai angka tahun terakhir.

## Pipeline

| Tahap | Isi | File/fungsi |
|---|---|---|
| 1. Load & clean | Unduh CSV publik, standarisasi kolom, agregasi ke level (Provinsi, Komoditas, Tahun) | `load_and_clean()` |
| 2. EDA | Cek struktur data, sebaran tahun, tren nasional, top-10 komoditas | — |
| 3. Evaluasi | Hold-out 1 tahun terakhir per seri, bandingkan PatchTST vs baseline naive | `nf_eval.cross_validation()` |
| 4. Model produksi | Latih ulang di seluruh data historis untuk prediksi 2026 | `nf_prod.fit()` |
| 5. Prediksi | Generate forecast 2026 per kombinasi wilayah-komoditas | `hasil_2026` |
| 6. Deploy | Dashboard Gradio: prediksi vs standar gizi → status & rekomendasi stok | `UI_prediksi()` |

## Evaluasi model

Metrik dihitung dari hold-out per seri (bukan angka yang ditulis manual):
tahun terakhir tiap seri disembunyikan saat training, lalu dibandingkan
dengan prediksi model.

- **MAE PatchTST**: dicetak saat runtime, lihat output cell evaluasi
- **MAE baseline (naive last-value)**: pembanding wajib

Baseline naive disertakan karena rata-rata panjang deret waktu per seri
hanya ~3 titik tahunan — pada data sependek ini, model transformer tidak
otomatis mengungguli pendekatan naif, dan itu perlu dibuktikan lewat
perbandingan, bukan diasumsikan.

## Batasan

- **Panjang deret waktu pendek** (rata-rata ~3 tahun/seri) adalah batasan
  data, bukan batasan model — hasil PatchTST harus dibaca dengan
  mempertimbangkan hal ini.
- Kombinasi wilayah-komoditas dengan data <2 titik tahun tidak bisa dilatih
  dan tidak akan punya prediksi 2026; dashboard menangani ini dengan
  fallback eksplisit ke data historis terakhir (diberi label jelas, bukan
  ditampilkan seolah prediksi).
- Jika URL sumber data gagal diakses, sistem beralih ke data contoh kecil
  dan menandainya lewat `USED_FALLBACK` — status ini ditampilkan di UI,
  bukan disembunyikan.

## Tech stack

Python, Pandas, NumPy, NeuralForecast (PatchTST), scikit-learn (metrik
evaluasi), Matplotlib/Seaborn (EDA), Gradio (dashboard).

## Menjalankan

```bash
pip install neuralforecast gradio scikit-learn pandas numpy matplotlib seaborn
python forecasting_kebutuhan_pangan_nasional_2026.py
```

Dashboard akan berjalan lokal lewat Gradio setelah `demo.launch()` dipanggil.
