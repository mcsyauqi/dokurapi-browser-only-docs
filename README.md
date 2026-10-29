# How Dokurapi works: browser-only document tools

Notes on how [Dokurapi](https://dokurapi.com/) compresses, merges and converts application documents without sending them to a server. Every step described here runs inside your browser tab, and each number below is taken from the public site code (`assets/app.js` on dokurapi.com).

Bahasa Indonesia: lihat bagian [Ringkasan bahasa Indonesia](#ringkasan-bahasa-indonesia) di bawah.

## Why browser-only matters

The files people shrink most often for job, scholarship and university portals are certificates, transcripts, ID cards and passport scans. They carry ID numbers, addresses and signatures. A server-based compressor needs that file to travel to someone else's machine. A browser-only tool downloads program code instead, and your document stays on your device.

| Aspect | Server-based tool | Browser-only (Dokurapi) |
|---|---|---|
| Where the file goes | To the provider's server | Stays in the browser tab |
| How you can check | Read the privacy policy | Watch the DevTools Network panel |
| Speed depends on | Upload and download speed | Your device's processor and memory |
| Text in the output PDF | Some tools keep real text | Pages are re-rendered as images |

## What happens when you press compress

1. **You pick a file.** The browser hands the page a file object through the File API. The page sees only the file you chose.
2. **The file is read into memory** inside the tab.
3. **Pages are rendered.** Mozilla PDF.js draws each page onto a canvas, with parsing done in a Web Worker.
4. **Each page is encoded as JPG** at a chosen quality.
5. **A new PDF is assembled** with pdf-lib.
6. **You get a download.** The result becomes a Blob with a temporary `blob:` address, released 1.5 seconds after the download starts.

Both PDF libraries (`pdf-lib.min.js`, `pdf.min.mjs`, `pdf.worker.min.mjs`) are served from dokurapi.com itself, not from a third-party CDN.

## How a target size is reached

- **1 KB = 1,000 bytes**, following SI prefixes, not 1,024.
- **6% safety margin.** A 200 KB target is treated as 188,000 bytes, so a portal that rounds differently will not reject the file.
- **PDF search grid.** The compressor tries four render scales (1.2, 1, 0.82, 0.68) and three JPG qualities (0.78, 0.64, 0.5), at most 12 attempts from sharpest to smallest. The first result under the safe target is kept. If none fits, you get the smallest result, labelled as missing the target.
- **Photos.** The image compressor caps the longest side at 2,400 px, lowers JPG quality step by step down to a floor of 18%, and only then shrinks dimensions, stopping at 640 px on the longest side.
- **Photo to PDF.** JPG and PNG are embedded as they are. Other formats are redrawn on a white canvas with the longest side at most 1,800 px.

Tools that use this pipeline:

- [Compress PDF](https://dokurapi.com/compress-pdf/) and [Kompres PDF](https://dokurapi.com/id/kompres-pdf/)
- [Merge PDF](https://dokurapi.com/merge-pdf/) and [Gabung PDF](https://dokurapi.com/id/gabung-pdf/)
- [Photo to PDF](https://dokurapi.com/photo-to-pdf/)
- [Check PDF size](https://dokurapi.com/check-pdf-size/) and [Cek ukuran PDF](https://dokurapi.com/id/cek-ukuran-pdf/)

## Trade-offs to know before compressing

- **Text is no longer selectable** after rasterising, so the output cannot be searched or copied.
- **Certificate-based digital signatures stop validating**, because the file contents change. Do not compress a document whose signature will be checked.
- **Small print can blur** when a tight target is spread over many pages. Remove pages you do not need or use a looser target.
- **Scanned PDFs lose almost nothing**, since they were page images already.

## Verify it yourself in two minutes

1. Open a Dokurapi tool in desktop Chrome or Edge, press F12 and open the **Network** panel.
2. Clear the log.
3. Pick a PDF and run the compression.
4. Read the request list. You will see library files load from dokurapi.com and possibly small Google Analytics requests. No request carries a payload the size of your PDF.

A stronger test: run a tool once so the libraries load, switch on airplane mode, then compress another file. It still works.

The response headers add a third safeguard. Dokurapi sends a `Content-Security-Policy` whose `connect-src` limits page scripts to dokurapi.com and Google Analytics.

## Ringkasan bahasa Indonesia

[Dokurapi](https://dokurapi.com/id/) adalah kumpulan alat dokumen untuk keperluan pendaftaran (CPNS, beasiswa, kampus) yang bekerja sepenuhnya di browser. Berkas yang kamu pilih dibaca di memori tab, diproses di perangkatmu, lalu dikembalikan sebagai unduhan. Tidak ada langkah upload karena situsnya tidak punya endpoint penerima berkas.

- Kompres PDF memakai PDF.js untuk merender halaman dan pdf-lib untuk menyusun PDF baru, maksimal 12 percobaan dari yang paling tajam ke yang paling kecil.
- Target ukuran dihitung 1 KB = 1.000 byte, dengan margin aman 6% (target 200 KB menjadi 188.000 byte).
- Hasil kompres berupa gambar halaman, jadi teks tidak bisa disalin dan tanda tangan digital berbasis sertifikat tidak lagi valid.
- Cara mengecek sendiri: buka panel Network (F12), jalankan alat, dan pastikan tidak ada request seukuran berkasmu.

Penjelasan lengkap: [Kompres PDF tanpa upload, cara kerjanya](https://dokurapi.com/id/articles/how-browser-tools-work.html). English version: [How browser tools work](https://dokurapi.com/articles/how-browser-tools-work.html).

## Libraries credited

- [PDF.js](https://github.com/mozilla/pdf.js) by Mozilla (Apache-2.0)
- [pdf-lib](https://github.com/Hopding/pdf-lib) (MIT)
- [MediaPipe Selfie Segmenter](https://ai.google.dev/edge/mediapipe/solutions/vision/image_segmenter) by Google (Apache-2.0), used by the ID photo background tools

This repository holds documentation only. It is maintained alongside [dokurapi.com](https://dokurapi.com/).
