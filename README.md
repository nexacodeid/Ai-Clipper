# Reelcut

Auto-clipper untuk video panjang. Masukkan file atau link YouTube, Reelcut mencari momen terbaik, membuat subtitle per kata, menambahkan efek, lalu mengekspor klip siap upload. Semua proses edit berjalan di browser kamu.

## Menjalankan

Butuh **Node.js 18+**. Tidak ada `npm install`, tidak ada dependency.

```bash
npm start
```

Buka <http://localhost:3000> di **Chrome atau Edge**.

Port lain: `PORT=4000 npm start`.

### Input URL / YouTube

Server memakai [yt-dlp](https://github.com/yt-dlp/yt-dlp). Kalau belum ada di komputer, server mengunduhnya sendiri ke `./bin` saat pertama kali jalan (butuh internet). Kalau kamu sudah punya `yt-dlp` di PATH, itu yang dipakai. Format yang diambil adalah MP4 progresif, jadi `ffmpeg` tidak diperlukan. Gunakan hanya untuk video milikmu atau yang kamu punya izinnya.

## Cara kerjanya

1. **Audio**: audio didekode ke 16 kHz mono, lalu dihitung kurva loudness per 0,25 detik.
2. **Transkrip**: Whisper (transformers.js, WASM) jalan di Web Worker dan menghasilkan timestamp per kata. Model diunduh sekali dari CDN lalu di-cache browser.
3. **Pilih klip**: transkrip dipecah jadi kalimat. Setiap jendela kalimat dinilai dari:

   | Komponen | Bobot |
   | --- | --- |
   | Energi suara | 42% |
   | Tempo bicara | 22% |
   | Kata hook / tanda tanya di pembuka | 18% |
   | Berakhir di akhir kalimat | 10% |
   | Kedekatan dengan durasi target | 8% |

   Klip dipilih greedy berdasarkan skor, tanpa overlap.
4. **Render**: tiap frame digambar ke canvas (background blur, zoom, color grade, vignette, judul hook, subtitle, progress bar, transisi) lalu direkam dengan `MediaRecorder` bersama audionya.

Skor adalah heuristik, bukan prediksi viral. Judul hook bisa diedit langsung di kartu klip.

## Struktur

```
server.js                    server statis + API download (zero-dependency)
public/
  index.html
  css/styles.css
  js/
    main.js · app.js         bootstrap dan orkestrasi
    config.js · utils.js · settings.js
    services/                audio, highlights (scoring), source, transcriber
    workers/whisper.worker.js
    render/                  camera, captions, caption-chunks, renderer, player
    ui/                      timeline, clip-list, progress, toast
test/                        unit test logika murni (npm test)
```

## Batasan yang perlu diketahui

- **Export berjalan real-time.** Klip 40 detik butuh 40 detik render. Biarkan tab tetap terbuka di depan; browser menghentikan animasi di tab tersembunyi.
- **Memori.** Video sangat panjang (lebih dari ±1 jam) bisa berat karena audio didekode penuh di memori.
- **Transkrip di CPU.** Whisper jalan lewat WASM. Model `base` cukup untuk kebanyakan kasus; pakai `tiny` kalau mau cepat, `small` kalau mau akurat.
- **Format output** MP4 jika browser mendukung perekaman MP4 (Chrome terbaru), selain itu WebM.

## Tes

```bash
npm test
```
