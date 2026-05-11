# 🔬 Edge Detection Citra Medis — Sobel & Canny

> Tugas 3 Opsi D — Praktikum Pengolahan Citra Digital  
> Implementasi deteksi tepi dari scratch pada citra medis (X-ray, MRI, CT-scan)

---

## 📋 Informasi Tugas

| Field | Detail |
|---|---|
| Mata Kuliah | Pengolahan Citra Digital |
| Topik | Edge Detection — Sobel & Canny |
| Opsi | Tugas 3 Opsi D (Dataset Medis) |
| Nama | ___________________ |
| NIM | ___________________ |
| Tanggal | ___________________ |

---

## 🗂️ Struktur File

```
.
├── edge_detection_medis.py   # Program utama
├── README.md                 # Dokumentasi ini
└── hasil_edge_*.png          # Output yang dihasilkan program (auto-generated)
```

---

## ⚙️ Instalasi

### 1. Pastikan Python 3.8+ sudah terinstall

```bash
python --version
```

### 2. Install semua library yang dibutuhkan

```bash
pip install opencv-python numpy matplotlib pillow scipy
```

| Library | Kegunaan |
|---|---|
| `opencv-python` | Pembanding hasil (cv2.Sobel, cv2.Canny) + baca file gambar |
| `numpy` | Operasi array dan konvolusi manual |
| `matplotlib` | Visualisasi GUI interaktif + simpan gambar |
| `pillow` | Fallback baca format gambar |
| `scipy` | Konvolusi efisien (opsional, ada fallback manual) |

---

## 🚀 Cara Menjalankan

### Mode 1 — GUI Interaktif (dengan citra sintetis medis)

Jalankan tanpa argumen, program otomatis memuat 4 citra sintetis:

```bash
python edge_detection_medis.py
```

### Mode 2 — GUI Interaktif dengan gambar sendiri

```bash
python edge_detection_medis.py --image path/ke/xray.jpg
```

Mendukung format: `.jpg`, `.jpeg`, `.png`, `.bmp`, `.tiff`

### Mode 3 — Batch (tanpa GUI, langsung simpan)

```bash
python edge_detection_medis.py --image foto_xray.jpg --batch
```

Output tersimpan otomatis sebagai `batch_<nama_file>.png`

### Argumen lengkap

```
usage: edge_detection_medis.py [-h] [--image IMAGE] [--batch]

optional arguments:
  -h, --help            Tampilkan pesan bantuan
  --image, -i IMAGE     Path ke file gambar (opsional)
  --batch, -b           Mode batch: proses tanpa GUI, simpan langsung
```

---

## 🖥️ Fitur Aplikasi

### Panel Visualisasi (6 panel)

| Panel | Deskripsi |
|---|---|
| Original (Grayscale) | Gambar input setelah konversi grayscale |
| Sobel — Magnitude | Hasil √(Gx² + Gy²) sebelum threshold |
| Sobel — Thresholded | Hasil biner setelah threshold diterapkan |
| Canny — Gaussian Blur | Hasil smoothing tahap pertama Canny |
| Canny — Edges | Hasil akhir deteksi tepi Canny |
| Overlay Perbandingan | 🔴 Merah = Sobel saja \| 🔵 Biru = Canny saja \| ⚪ Putih = Keduanya |

### Kontrol Parameter (Sidebar)

**Sobel:**
- `Threshold` (0–255) — ambang batas binerisasi magnitude
- `Blur σ` (0.0–3.0) — sigma Gaussian pre-blur sebelum Sobel

**Canny:**
- `σ` (0.5–4.0) — sigma Gaussian blur
- `Low` (5–200) — threshold bawah hysteresis
- `High` (10–300) — threshold atas hysteresis

### Citra Medis Sintetis (built-in)

| Nama | Simulasi |
|---|---|
| Sel Darah | Lingkaran berlapis simulasi sel di bawah mikroskop |
| Pembuluh Darah | Kurva sinusoidal simulasi retina (DRIVE dataset) |
| Struktur Tulang | Grid kotak simulasi X-ray tulang |
| Citra Bernoisy | Citra sel + noise acak tinggi (uji ketahanan metode) |

### Tombol Aksi

| Tombol | Fungsi |
|---|---|
| **Upload Foto** | Buka file dialog untuk pilih gambar dari komputer (JPG/PNG/BMP/TIFF) |
| **Simpan Hasil** | Ekspor 6 panel sebagai PNG (150 DPI) |
| **Cetak Laporan** | Tampilkan laporan lengkap di terminal |

> **Catatan:** Tombol Upload Foto membutuhkan `tkinter` (sudah bawaan Python di Windows & macOS). Di Linux jalankan `sudo apt install python3-tk` jika belum ada.

---

## 🧠 Penjelasan Algoritma

### Sobel Edge Detection (Manual)

Implementasi penuh tanpa `cv2.Sobel` atau fungsi konvolusi library manapun.

```
Langkah:
1. Konversi gambar ke float64
2. (Opsional) Gaussian blur manual untuk meredam noise
3. Konvolusi manual dengan kernel Gx dan Gy:

   Gx = [[-1, 0, 1],      Gy = [[-1, -2, -1],
          [-2, 0, 2],             [ 0,  0,  0],
          [-1, 0, 1]]             [ 1,  2,  1]]

4. Hitung magnitude: G = √(Gx² + Gy²)
5. Hitung arah gradien: θ = arctan2(Gy, Gx) × 180/π
6. Thresholding biner
```

Padding mode `reflect` digunakan pada border untuk menghindari artefak tepi.

### Canny Edge Detection (Manual — 5 Tahap)

```
Tahap 1 — Gaussian Blur
   Reduksi noise sebelum deteksi gradien.
   Kernel dibangkitkan dari σ menggunakan fungsi Gaussian 2D.

Tahap 2 — Gradient (Sobel)
   Hitung Gx, Gy, G = √(Gx²+Gy²), dan θ = arctan2(Gy, Gx).
   Magnitude dinormalisasi ke [0, 1].

Tahap 3 — Non-Maximum Suppression (NMS)
   Untuk setiap piksel, bandingkan nilainya dengan dua tetangga
   searah gradien. Hanya piksel lokal maksimum yang dipertahankan
   → tepi menjadi 1 piksel tipis.

Tahap 4 — Double Thresholding
   G ≥ high_thresh  → strong edge  (pasti tepi)
   low ≤ G < high   → weak edge   (kandidat tepi)
   G < low_thresh   → bukan tepi

Tahap 5 — Hysteresis Edge Tracking
   Weak edge dipertahankan hanya jika terhubung (8-connectivity)
   dengan minimal satu strong edge.
```

---

## 📊 Output Statistik

Program mencetak dan menampilkan statistik berikut:

| Metrik | Deskripsi |
|---|---|
| Piksel Tepi | Jumlah piksel yang terdeteksi sebagai tepi |
| Edge Density | Persentase piksel tepi terhadap total piksel |
| Waktu Komputasi | Durasi proses dalam milidetik |
| RMSE vs OpenCV | Selisih hasil manual dibanding implementasi OpenCV |

Contoh output terminal (mode batch / cetak laporan):

```
============================================================
   LAPORAN DETEKSI TEPI CITRA MEDIS
============================================================
  Citra       : xray_chest.jpg
  Resolusi    : 400×320 px

  [PARAMETER]
  Sobel Threshold : 50
  Canny σ         : 1.4
  Canny Low/High  : 50 / 150

  [SOBEL]
  Piksel tepi  : 8,432
  Edge density : 6.59%

  [CANNY]
  Piksel tepi  : 5,217
  Edge density : 4.08%

  [RMSE vs OpenCV]
  Sobel : 0.0031
  Canny : 2.1740
============================================================
```

---

## 🔍 Perbandingan Metode

| Aspek | Sobel | Canny |
|---|---|---|
| Tepi yang dihasilkan | Tebal (multi-piksel) | Tipis (1 piksel) |
| Sensitivitas noise | Sedang | Rendah (ada Gaussian blur) |
| Parameter | Threshold saja | σ, low, high threshold |
| Kecepatan | ⚡ Lebih cepat | 🐢 Lebih lambat |
| Akurasi lokasi tepi | Cukup | Tinggi (ada NMS) |
| Cocok untuk | Deteksi kasar, real-time | Analisis presisi tinggi |

---

## 📁 Contoh Output File

Setelah klik **Simpan Hasil** atau mode batch, file PNG tersimpan:

```
hasil_edge_Sel_Darah.png
hasil_edge_Pembuluh_Darah.png
batch_xray_chest.png
```

---

## 📚 Referensi

1. Canny, J. (1986). *A Computational Approach to Edge Detection*. IEEE Trans. PAMI, 8(6), 679–698.
2. Sobel, I. & Feldman, G. (1968). *A 3×3 isotropic gradient operator for image processing*. Stanford AI Project.
3. Gonzalez, R. C. & Woods, R. E. (2017). *Digital Image Processing* (4th ed.). Pearson.
4. OpenCV Documentation: https://docs.opencv.org/4.x/da/d22/tutorial_py_canny.html
5. DRIVE Dataset (retinal vessels): https://drive.grand-challenge.org/

---

## 🛠️ Troubleshooting

**`ModuleNotFoundError: No module named 'cv2'`**
```bash
pip install opencv-python
```

**GUI tidak muncul di server/headless:**
```bash
# Gunakan mode batch saja
python edge_detection_medis.py --image gambar.jpg --batch
```

**Gambar terlalu lambat diproses:**
> Konvolusi manual bersifat O(n²) per piksel. Gambar besar otomatis di-resize ke maksimal 400×400 px untuk mode GUI, dan 500×500 px untuk mode batch.

**Tombol Upload Foto tidak bisa diklik / error tkinter (Linux):**
```bash
sudo apt install python3-tk
```
> Program tetap berjalan menggunakan konvolusi numpy manual sebagai fallback. Hasil identik, hanya sedikit lebih lambat.

---

*Selamat mengerjakan! — Pengolahan Citra Digital*
