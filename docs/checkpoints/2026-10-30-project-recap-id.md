### AI Agent — Project Checkpoint & Development Checkpoint

Foundation sebagian selesaiWhatsApp belum terhubungDevelopment paused

### 1\. Tujuan proyek

Proyek ini bukan sekadar chatbot WhatsApp. Targetnya adalah personal AI agent yang dapat:

* Memiliki satu persona dan memori yang konsisten.

* Menggunakan beberapa AI provider dengan fallback.

* Belajar dari percakapan, dokumen, riset, dan eksplorasi.

* Mengarsipkan percakapan WhatsApp.

* Mengakses WhatsApp dan terminal sebagai antarmuka.

* Memulai tindakan sendiri secara bertahap dengan sistem permission dan approval.

* Menggunakan koneksi internet untuk AI cloud, tetapi tetap dapat mengelola penyimpanan lokal ketika offline.

Keputusan arsitektur yang sudah ditetapkan:

| Bagian         | Keputusan                  |
| -------------- | -------------------------- |
| Runtime        | Node.js                    |
| Bahasa         | JavaScript ES Modules      |
| WhatsApp       | Baileys                    |
| Database       | SQLite                     |
| AI             | Cloud AI API               |
| Penyimpanan    | Lokal, HDD                 |
| AI lokal       | Tidak direncanakan         |
| Interface      | WhatsApp + terminal        |
| Autonomy       | Bertahap dengan permission |
| Online/offline | Manual toggle              |

### 2\. Kebijakan penyimpanan dan memori

Ada beberapa keputusan penting yang sudah dibuat.

Arsip WhatsApp

Cakupan arsip:

* Pesan personal.

* Pesan masuk dan keluar.

* Percakapan bot.

* Group chat.

* Reply dan quoted message.

* Metadata pengirim, grup, dan timestamp.

Media policy C

* Semua teks dan metadata dapat diarsipkan.

* Media tidak diunduh secara massal.

* Media tertentu dapat dipertahankan jika relevan untuk proses pembelajaran.

* Gambar, dokumen, dan audio diproses berdasarkan kebijakan.

Pemisahan data

Raw archive tidak sama dengan long-term memory.

Alurnya direncanakan seperti ini:

```text
WhatsApp / Documents / Research
              |
              v
        Raw Archive
              |
              v
       Learning Process
              |
              v
       Memory Extraction
              |
              v
        Long-term Memory
```

Memori harus memiliki provenance, confidence, dan status. Fakta, asumsi, dan informasi belum terverifikasi harus tetap dibedakan.

Group ingestion juga tidak boleh diaktifkan tanpa pemberitahuan dan persetujuan peserta terkait.

### 3\. Struktur project yang sudah dibuat

Lokasi project:

```text
/home/hagure/ai-agent/
```

Struktur yang sudah dibuat atau terverifikasi:

```text
ai-agent/
│
├── core/
│   ├── index.js
│   ├── init-database.js
│   └── whatsapp-test.js
│
├── providers/
│
├── memory/
│
├── learning/
│
├── autonomy/
│
├── config/
│
├── logs/
│
├── interfaces/
│   ├── whatsapp/
│   │   └── connection.js
│   │
│   └── terminal/
│
├── storage/
│   ├── database/
│   │   └── agent.db
│   │
│   ├── knowledge/
│   │
│   ├── media/
│   │
│   ├── backups/
│   │
│   └── whatsapp-auth/
│       └── creds.json
│
├── node_modules/
│
├── .git/
├── .gitignore
├── package.json
└── package-lock.json
```

Catatan: struktur di atas merupakan inventaris berdasarkan pekerjaan yang tercatat. Belum merupakan hasil listing filesystem terbaru; isi folder kosong dan file tambahan perlu diverifikasi langsung.

### 4\. File penting dan fungsinya

| File                              | Fungsi                                       |
| --------------------------------- | -------------------------------------------- |
| core/index.js                     | Entry point utama agent, masih placeholder   |
| core/init-database.js             | Inisialisasi database                        |
| core/whatsapp-test.js             | Program pengujian WhatsApp                   |
| interfaces/whatsapp/connection.js | Koneksi dan pairing Baileys                  |
| package.json                      | Konfigurasi npm dan scripts                  |
| package-lock.json                 | Lock versi dependency                        |
| .gitignore                        | Mengecualikan file sensitif dan file runtime |

Scripts pada `package.json`:

```json
{
  "start": "node core/index.js",
  "dev": "node --watch core/index.js",
  "db:init": "node core/init-database.js",
  "wa:test": "node core/whatsapp-test.js"
}
```

`core/index.js` belum menghubungkan AI provider atau WhatsApp. Saat ini hanya placeholder.

### 5\. Database yang sudah dibuat

Lokasi:

```text
storage/database/agent.db
```

Tabel yang sudah diverifikasi:

```text
conversations
knowledge
memories
messages
tasks
events
media_assets
memory_sources
schema_meta
```

Konfigurasi SQLite:

* WAL aktif.

* Foreign keys diaktifkan per koneksi.

* Busy timeout 5000 ms.

* Synchronous NORMAL.

* Database integrity check menghasilkan `ok`.

Perubahan terakhir pada schema `memory_sources` adalah menggunakan `ON DELETE CASCADE` untuk relasi ke `memories`, `messages`, dan `knowledge`.

Pemeriksaan sebelumnya:

```text
integrity_check = ok
```

Database sudah berhasil diinisialisasi dan diuji. Belum ada integrasi AI.

### 6\. Software yang sudah terpasang

Versi berdasarkan pemeriksaan terminal sebelumnya, bukan hasil scan terbaru.

| Software / package | Versi          | Status            |
| ------------------ | -------------- | ----------------- |
| Debian             | 13 Trixie      | Terpasang         |
| Node.js            | 20.19.2        | Terpasang         |
| npm                | 9.2.0          | Terpasang         |
| SQLite CLI         | 3.46.1         | Terpasang         |
| node-sqlite3       | 5.1.5+ds1-1+b5 | Terpasang via APT |
| Baileys            | 6.7.23         | Terpasang         |
| Python             | 3.13.5         | Terpasang         |
| Git                | 2.47.3         | Terpasang         |

Lokasi module SQLite:

```text
/usr/lib/x86_64-linux-gnu/nodejs/sqlite3/lib/sqlite3.js
```

Ada satu hal teknis yang sudah diselesaikan: Debian `node-sqlite3` tidak berhasil diimpor menggunakan ESM import biasa. Solusi yang dipakai:

```js
import { createRequire } from 'node:module';

const require = createRequire(import.meta.url);
const sqlite3 = require('sqlite3');
```

Baileys merupakan dependency npm lokal di dalam project.

`qrcode-terminal` sempat diinstal ketika eksperimen awal, kemudian di-uninstall karena pairing yang dipilih adalah pairing code, bukan QR.

Belum terpasang atau belum dikonfigurasi:

* AI provider/API integration.

* Sistem memory extraction.

* Learning pipeline.

* Autonomy engine.

* Terminal interface.

* WhatsApp event archive integration.

* Scheduler dan systemd service.

### 7\. Riwayat troubleshooting WhatsApp

Ini bagian terakhir yang sedang dikerjakan.

| Tahap                     | Hasil                                            |
| ------------------------- | ------------------------------------------------ |
| Baileys RC14              | Login gagal, 401                                 |
| Pairing API               | requestPairingCode tersedia pada socket instance |
| Baileys 6.7.23            | Berhasil membuka WebSocket, login gagal 405      |
| Versi WA Web default      | Tertinggal                                       |
| fetchLatestWaWebVersion() | Berhasil mengambil versi terbaru                 |
| Explicit WA Web version   | Tetap gagal 401                                  |
| Browser Ubuntu/Chrome     | Gagal 401                                        |
| Browser Mac OS/Chrome     | Gagal 401                                        |

Kondisi terakhir:

```text
Baileys: 6.7.23
WhatsApp Web: latest
Browser: Mac OS / Chrome
WebSocket: connected
Login: failed
Disconnect: 401
Pairing code: belum berhasil diminta
```

Penyelidikan terakhir menemukan:

```text
node_modules/@whiskeysockets/baileys/lib/Utils/validate-connection.js
```

Pada baris 16:

```js
platform: proto.ClientPayload.UserAgent.Platform.WEB,
```

Artinya, mengganti `Browsers.macOS()` hanya mengubah browser fingerprint. Nilai platform protokol tetap `WEB`.

Posisi investigasi terakhir: sedang memeriksa enum `ClientPayload.UserAgent.Platform` di `WAProto.proto` untuk mengetahui apakah `MACOS` tersedia sebelum memutuskan apakah perlu patch.

Belum ada patch terhadap library Baileys.

### 8\. Inventaris aktual laptop

Ada batasan penting: kita sudah mengetahui struktur project dan beberapa software yang terpasang, tetapi belum pernah mengambil daftar lengkap filesystem dan paket APT/npm dari laptop.

Jadi aku tidak akan mengklaim seluruh isi laptop sudah diketahui.

Saat nanti melanjutkan, jalankan rangkaian berikut untuk membuat inventaris yang bisa disimpan sebagai checkpoint.

A. Inventaris file dan direktori project

```bash
cd ~/ai-agent

find . \
  -path './node_modules' -prune -o \
  -path './.git' -prune -o \
  -printf '%y %p\n' | sort
```

B. Daftar dependency npm

```bash
cd ~/ai-agent

npm ls --depth=0
```

C. Daftar paket APT terkait development

```bash
dpkg-query -W \
  -f='${binary:Package}\t${Version}\n' \
  nodejs npm sqlite3 libsqlite3-0 \
  node-sqlite3 git python3 2>/dev/null
```

D. Informasi versi runtime

```bash
node --version
npm --version
sqlite3 --version
python3 --version
git --version
```

E. Status database

```bash
cd ~/ai-agent

sqlite3 storage/database/agent.db <<'EOF'
.tables
PRAGMA integrity_check;
SELECT name, sql
FROM sqlite_master
WHERE type = 'table'
ORDER BY name;
EOF
```

Perintah-perintah tersebut tidak mengubah project. Hasilnya dapat digunakan untuk membuat inventaris final.

### 9\. Posisi checkpoint dan pekerjaan berikutnya

* Project foundation

    Sebagian selesai

    Skeleton, npm configuration, Git, dan struktur penyimpanan sudah tersedia.

* Database foundation

    Selesai untuk tahap awal

    SQLite, schema, WAL, foreign keys, dan integrity check sudah diverifikasi.

* WhatsApp connectivity

    Belum selesai

    WebSocket berhasil, autentikasi ditolak dengan 401.

* AI provider dan memory

    Belum dimulai

    Menunggu fondasi berikutnya.

* Learning dan autonomy

    Belum dimulai

    Belum ada implementasi.

Titik berhenti yang disarankan untuk dicatat:

```text
CHECKPOINT: WhatsApp Connectivity Investigation

Date: 2026-10-30
Project: ~/ai-agent
OS: Debian 13 Trixie
Runtime: Node.js 20.19.2
Baileys: 6.7.23

Completed:
- Project skeleton
- SQLite initialization
- Database schema
- WAL and integrity verification
- Baileys installation
- Pairing API verification
- Latest WhatsApp Web version retrieval
- Browser fingerprint experiment
- ClientPayload platform source inspection

Unresolved:
- WhatsApp login returns 401
- Pairing code not triggered
- ClientPayload Platform remains WEB
- WAProto enum investigation incomplete

Do not:
- Delete WhatsApp auth state without diagnosis
- Patch node_modules without verifying enum
- Proceed to AI integration yet

Status: PAUSED
```
