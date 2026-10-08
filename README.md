# 🔌 WebSocket Real-Time dengan Node.js dan MySQL

Proyek komunikasi **real-time dua arah** menggunakan **WebSocket** (`ws`) pada Node.js yang terhubung dengan basis data **MySQL**. Server mengirim data riwayat dari database, mengirim pesan real-time berkala, dan memberi notifikasi ketika pesan klien mengandung kata kunci tertentu.

---

## ✨ Fitur

- **Koneksi WebSocket** antara server dan beberapa klien sekaligus.
- **Pengiriman data dari MySQL**: saat klien terhubung, server mengirim hingga 10 entri dari tabel `logs`.
- **Pesan real-time berkala**: server mengirim pesan beserta *timestamp* ke setiap klien tiap 3 detik.
- **Notifikasi kata kunci**: jika pesan klien mengandung kata `urgent`, server membalas dengan notifikasi.
- **Dua jenis klien**:
  - Klien **browser** (`index.html`) yang menampilkan data langsung di halaman.
  - Klien **Node.js** (`app.js`) yang menerima data dan menyimpannya ke MySQL.
- **Pembersihan sumber daya**: interval dihentikan otomatis saat klien terputus.

## 🧩 Arsitektur

```
┌──────────────┐   ws://localhost:3000   ┌──────────────┐        ┌─────────┐
│ Klien Browser│ ◄─────────────────────► │              │ ◄────► │         │
│ (index.html) │                         │  server.js   │        │  MySQL  │
└──────────────┘                         │  (WebSocket) │        │ (logs)  │
┌──────────────┐                         │              │        │         │
│ Klien Node.js│ ◄─────────────────────► │              │        │         │
│   (app.js)   │ ───────── INSERT ─────────────────────────────► │         │
└──────────────┘                         └──────────────┘        └─────────┘
```

## 📨 Format Pesan

Semua pesan dikirim dalam format **JSON** dengan properti `type`.

| `type` | Arah | Isi | Keterangan |
|---|---|---|---|
| `database` | Server → Klien | `data`: array baris dari tabel `logs` | Dikirim sekali saat klien terhubung |
| `realtime` | Server → Klien | `message`, `timestamp` | Dikirim tiap 3 detik |
| `notification` | Server → Klien | `notification` | Dikirim jika pesan klien memuat kata `urgent` |
| *(tanpa type)* | Klien → Server | `text` | Pesan dari klien ke server |

Contoh pesan dari klien:

```json
{ "text": "Halo Server! Ini urgent" }
```

## 📁 Struktur Proyek

```
web-socket/
├── server.js      # WebSocket server (port 3000)
├── app.js         # Klien Node.js: menerima data & menyimpan ke MySQL
├── db.js          # Konfigurasi koneksi MySQL
├── index.html     # Klien browser
├── package.json
└── README.md
```

## 🛠️ Teknologi yang Digunakan

- **Node.js**
- **ws**: library WebSocket
- **mysql**: driver MySQL untuk Node.js
- **MySQL** (mis. melalui XAMPP/Laragon)
- **HTML & JavaScript**: klien browser

## 🚀 Cara Menjalankan

### 1. Clone repository

```bash
git clone https://github.com/angfdzr/web-socket.git
cd web-socket
```

### 2. Install dependensi

```bash
npm install ws mysql
```

### 3. Siapkan database MySQL

Buat database `websocket_db` beserta tabel `logs`:

```sql
CREATE DATABASE websocket_db;
USE websocket_db;

CREATE TABLE logs (
    id INT AUTO_INCREMENT PRIMARY KEY,
    message VARCHAR(255),
    timestamp DATETIME
);
```

Konfigurasi koneksi ada di `db.js`. Sesuaikan jika pengaturan MySQL Anda berbeda:

```js
const db = mysql.createConnection({
    host: 'localhost',
    user: 'root',
    password: '',
    database: 'websocket_db'
});
```

### 4. Jalankan server

```bash
node server.js
```

Server berjalan di `ws://localhost:3000`.

### 5. Jalankan klien

- **Klien browser:** buka file `index.html` di browser.
- **Klien Node.js:** jalankan di terminal lain:

```bash
node app.js
```

## 🧪 Contoh Alur

1. Klien terhubung dan langsung menerima hingga 10 data dari tabel `logs` (`type: database`).
2. Klien mengirim pesan JSON ke server, lalu server mencatatnya di konsol.
3. Setiap 3 detik klien menerima pesan `realtime` beserta *timestamp*.
4. Jika klien mengirim pesan yang berisi kata `urgent`, server membalas dengan `notification`.

## ⚠️ Catatan Pengembangan

- Kredensial database masih ditulis langsung di `db.js`. Untuk penggunaan nyata, gunakan *environment variable* (mis. `dotenv`).
- Pada `app.js`, data bertipe `database` yang diterima dari server langsung di-`INSERT` kembali ke tabel `logs`. Setiap kali klien Node.js dijalankan, baris yang sudah ada akan terduplikasi.
- Pesan `realtime` dari server tidak disimpan ke database. Jika ingin menyimpannya, tambahkan proses `INSERT` pada penanganan pesan `realtime`.
- Kueri `SELECT * FROM logs LIMIT 10` belum diurutkan. Tambahkan `ORDER BY` bila ingin menampilkan data terbaru.

## 👤 Penulis

**Angga Fadzar**

---

*Proyek ini dibuat untuk keperluan pembelajaran.*
