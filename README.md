# VSH Invoice Generator

Screenshot bukti transfer → OCR → konfirmasi → invoice **PDF + PNG** Vita Smart Hub, di-render dengan Playwright (Chromium).

---

## 1. Kebutuhan

- **Node.js 18+**. Cek dengan `node -v`; kalau belum ada, install versi LTS dari [nodejs.org](https://nodejs.org).
- Koneksi internet saat `npm install` (mengunduh dependency dan Chromium ±150 MB, hanya sekali).

## 2. Instalasi

```bash
cd C:\Users\<nama>\Downloads\vsh-invoice      # folder yang berisi package.json
npm install
```

`npm install` otomatis:
1. mengunduh `playwright`, `tesseract.js`, dan data OCR ke `node_modules/`;
2. menjalankan `playwright install chromium` (postinstall).

Cek instalasi:
```bash
npm run invoice -- --help
```

> `--help` hanya menampilkan petunjuk dan **tidak** membuat PDF.

## 3. Membuat invoice

```bash
npm run invoice -- --bukti "C:\Users\<nama>\Downloads\bukti.jpg" --produk canva-pro-6
```

1. Script membaca bukti transfer (OCR).
2. Hasil OCR ditampilkan field per field. Tekan **Enter** untuk memakai nilai di `[kurung]`, atau ketik nilai baru.
3. Jawab **Y** di `Buat invoice?`.
4. Selesai:
   ```
   ✓ INV/VSH/20261007/001
     ...\vsh-invoice\output\INV-VSH-20261007-001.pdf
     ...\vsh-invoice\output\INV-VSH-20261007-001.png
   ```

### Contoh lain

```bash
# Nama pelanggan berbeda dari nama pengirim di bukti
npm run invoice -- --bukti bukti.jpg --produk canva-pro-6 --nama "Jazid"

# Tanpa konfirmasi
npm run invoice -- --bukti bukti.jpg --produk canva-pro-6 --yes

# Tanpa screenshot (input manual)
npm run invoice -- --produk canva-pro-6 --nama "Jazid" --metode BCA --jumlah 33000 --biaya 0 --transaksi ABC123 --tanggal 2026-10-07
```

### Opsi

| Opsi | Keterangan |
|---|---|
| `--bukti <path>` | Screenshot bukti transfer (jpg/png). Path bebas; bungkus dengan `"..."` jika ada spasi. Bisa juga drag file ke terminal. |
| `--produk <kode>` | **Wajib.** Kode produk di `config/products.json` |
| `--nama` | Nama pelanggan (default: nama pengirim hasil OCR) |
| `--metode` | Metode pembayaran (BCA, DANA, ShopeePay, …) |
| `--jumlah` | Jumlah yang dibayar (total) |
| `--biaya` | Biaya transfer/admin |
| `--transaksi` | No. transaksi/referensi |
| `--tanggal` | Tanggal bayar `YYYY-MM-DD` |
| `--jam` | Jam bayar (opsional) |
| `--qty` | Jumlah unit (periode ikut dikalikan), default `1` |
| `--yes` | Lewati konfirmasi |
| `--out <folder>` | Folder output (default `output/`) |
| `--debug` | Tampilkan teks mentah hasil OCR |

## 4. Hasil

```
vsh-invoice/
├─ output/
│  ├─ INV-VSH-20261007-001.pdf   ← A4, siap kirim/cetak
│  └─ INV-VSH-20261007-001.png   ← versi gambar untuk WA/IG
└─ data/
   └─ invoices.json              ← arsip & penomoran
```

Buka cepat dari cmd:
```bash
start output
start output\INV-VSH-20261007-001.pdf
```

- Logo, warna, dan font ter-embed, sehingga PDF bisa dibuka tanpa internet.
- Nomor invoice: `INV/VSH/YYYYMMDD/NNN`, urut per tanggal bayar.
- No. transaksi yang sama menghasilkan nomor invoice yang sama dan file lama **ditimpa** (berguna untuk memperbaiki typo).

## 5. Data di invoice (tidak ada yang hard code)

| Bagian | Sumber |
|---|---|
| Nama pelanggan, metode, jumlah, biaya transfer, no. transaksi, tanggal/jam | OCR bukti → bisa dikoreksi saat konfirmasi / argumen |
| No. invoice | Otomatis dari `data/invoices.json` |
| Produk, harga, durasi, periode | `config/products.json` + tanggal bayar + `--qty` |
| Subtotal / total | `harga × qty` / `subtotal + biaya` |
| Status | **LUNAS** jika jumlah dibayar ≥ total, selain itu **BELUM LUNAS** |
| Nama toko, tagline, Instagram, logo | `config/store.json` + `assets/logo.png` |

Biaya transfer: dibaca dari label *Biaya Transfer / Biaya Admin / Fee*. Tulisan "GRATIS" atau label yang tidak tercantum dibaca sebagai `0`, dan di invoice ditampilkan "Gratis".

## 6. Metode pembayaran yang dikenali

ShopeePay, GoPay, OVO, DANA, LinkAja, SeaBank, Bank Jago, BCA, BRI, BNI, Mandiri, BSI, CIMB Niaga, Permata, blu, Jenius, QRIS.

Field yang dibaca:
- **Pengirim**: *Dari / Pengirim / Sumber Dana / From*
- **Jumlah**: *Jumlah Total / Total / Nominal / Jumlah Transfer / Amount*, format `Rp 33.000`, `Rp33.000,00`, `IDR 33,000.00`
- **No. transaksi**: *No. Transaksi / No. Referensi / ID Transaksi / Ref No / Transaction ID*
- **Tanggal & jam**: `7 Okt 2026`, `07/10/2026`, `2026-10-07`, `14:22 WIB`. Jika tidak ada, diambil dari prefix `YYYYMMDD` no. transaksi, lalu tanggal hari ini.

> Sudah diuji dengan screenshot ShopeePay asli serta contoh teks BCA, DANA, dan BRI. Untuk layout lain, jalankan dengan `--debug`, lalu tambahkan label yang muncul ke `LABELS` / `METHODS` di `src/receipt.js`.

## 7. Testing

```bash
npm run invoice -- --bukti bukti.jpg --produk canva-pro-6 --out test-output --debug
```

- `--out test-output` memisahkan file uji dari invoice asli.
- `--debug` menampilkan teks OCR mentah.
- Pencatatan tetap masuk `data/invoices.json`, sehingga tes ikut memakai nomor urut. Setelah testing, **hapus folder `data/` dan `test-output/`** agar penomoran bersih.

## 8. Kustomisasi

| Ingin mengubah | File |
|---|---|
| Produk & harga | `config/products.json` |
| Nama toko, tagline, Instagram, prefix invoice | `config/store.json` |
| Logo | Timpa `assets/logo.png` (disarankan ≥ 500×500 px, PNG transparan) |
| Desain/teks invoice | `template/invoice.html` (placeholder `{{...}}`) |
| Bank/e-wallet/label baru | `METHODS` / `LABELS` di `src/receipt.js` |

Contoh produk baru:
```json
{
  "canva-pro-6": { "nama": "Canva Pro – Member", "deskripsi": "Sewa akses Canva Pro (member)", "bulan": 6, "harga": 33000 },
  "kode-baru":   { "nama": "...", "deskripsi": "...", "bulan": 1, "harga": 0 }
}
```

## 9. Troubleshooting

| Masalah | Solusi |
|---|---|
| `npm error Missing script: "invoice"` | Anda tidak berada di folder project, atau `package.json` di folder itu bukan milik project ini. Masuk ke folder yang berisi `package.json` + `src/`, atau extract ulang zip ke folder baru lalu `npm install`. |
| PDF tidak muncul | Pastikan menjalankan perintah dengan `--bukti` (bukan hanya `--help`) dan menjawab **Y** saat konfirmasi. Cek folder `output/` di dalam project (`start output`). |
| `Executable doesn't exist` / browser tidak ditemukan | `npx playwright install chromium`, lalu jalankan ulang. |
| PowerShell: *running scripts is disabled* | Pakai Command Prompt, atau `Set-ExecutionPolicy -Scope CurrentUser RemoteSigned` (sekali). |
| Ingin memakai Chrome yang sudah terpasang | `npm install --ignore-scripts`, lalu cmd: `set CHROME_PATH=C:\Program Files\Google\Chrome\Application\chrome.exe` (PowerShell: `$env:CHROME_PATH="..."`). |
| Field hasil OCR salah/kosong | Koreksi saat konfirmasi atau lewat argumen. Gunakan screenshot penuh beresolusi asli. Cek dengan `--debug`. |
| `Produk "..." tidak ada` | Kode `--produk` harus sama persis dengan key di `config/products.json`. |

## 10. Struktur project

```
vsh-invoice/
├─ src/
│  ├─ generate.js      # CLI: OCR → konfirmasi → hitung → render PDF/PNG → arsip
│  └─ receipt.js       # OCR + parser generik bukti transfer
├─ template/
│  └─ invoice.html     # desain invoice (placeholder {{...}})
├─ config/
│  ├─ products.json    # daftar produk & harga
│  └─ store.json       # identitas toko & path logo
├─ assets/
│  └─ logo.png
├─ package.json
└─ README.md
```

`output/`, `data/`, dan `node_modules/` dibuat otomatis dan tidak perlu dibagikan.
