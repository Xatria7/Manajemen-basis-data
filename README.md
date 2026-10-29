# UTS Manajemen Basis Data: PostgreSQL & pgAdmin4 dengan Docker Compose

Proyek 1 / UTS mata kuliah **Manajemen Basis Data** (Teknik Informatika, Semester V, TA 2026/2027).
Menjalankan PostgreSQL dan pgAdmin4 dengan kontainerisasi, lalu membuat skema, tabel, constraint, serta user dengan hak akses berbeda.

---

## Ringkasan Layanan

| Layanan | Nama Kontainer | Image | Port Host | Port Internal |
|---|---|---|---|---|
| PostgreSQL | `postgres_satria` | `postgres:15` | `22078` | `5432` |
| pgAdmin4 | `pgadmin_satria` | `dpage/pgadmin4:latest` | `44078` | `80` |

> Port mengikuti ketentuan soal: `22XXX` dan `44XXX`, dengan XXX = tiga digit terakhir NIM.

Username dan password **tidak ditulis** di `docker-compose.yml`. Semua nilai dibaca dari file `.env` (tidak ikut di-push ke repository).

---

## Struktur Proyek

```
.
├── docker-compose.yml        # definisi service postgres & pgadmin
├── .env                      # kredensial & port (TIDAK di-push)
├── .env.example              # template variabel lingkungan
├── .gitignore
├── README.md

```

---

## Prasyarat

- Docker Desktop (aktif/running)
- DBeaver (untuk akses eksternal)
- Git

---

## Cara Menjalankan

### 1. Clone repository

```bash
git clone URL_REPOSITORY
cd NAMA_FOLDER_REPOSITORY
```

### 2. Siapkan file `.env`

PowerShell:
```powershell
Copy-Item .env.example .env
```

Linux/macOS/Git Bash:
```bash
cp .env.example .env
```

Lalu isi `.env` sesuai ketentuan soal:

```env
POSTGRES_USER=postgres
POSTGRES_PASSWORD=<sesuai ketentuan soal>
POSTGRES_DB=db_uts

PGADMIN_DEFAULT_EMAIL=<email untuk login pgAdmin>
PGADMIN_DEFAULT_PASSWORD=<password pgAdmin>

PG_EXT_PORT=22078
PGADMIN_EXT_PORT=44078
```

### 3. Jalankan container

```bash
docker compose up -d
docker compose ps
```

Kedua container harus berstatus *running* (PostgreSQL tertulis *healthy*). Untuk memastikan variabel `.env` terbaca:

```bash
docker compose config
```

---

## Akses ke Database

### Internal Docker (pgAdmin4)

1. Buka `http://localhost:44078` lalu login dengan email dan password pgAdmin dari `.env`.
2. **Add New Server**:
   - General, Name: `postgres_NAMA`
   - Connection, Host: `postgres` (nama service)
   - Port: `5432` (port internal)
   - Maintenance database: `db_uts`
   - Username dan Password: sesuai `.env`

### Eksternal Docker (DBeaver)

| Parameter | Nilai |
|---|---|
| Host | `localhost` |
| Port | `22078` |
| Database | `db_uts` |
| Username / Password | sesuai `.env` |

**Perbedaan:** dari dalam jaringan Docker, pgAdmin memanggil PostgreSQL lewat nama service dan port internal `5432`. Dari luar Docker (host), DBeaver memakai `localhost` dengan port yang di-expose, yaitu `22078`.

---

## Skema dan Tabel

Skema **`salam`** berisi tabel **`mahasiswas`** dengan constraint berikut:

| Constraint | Kolom / Aturan |
|---|---|
| PRIMARY KEY | `id` (SERIAL) |
| UNIQUE | `nim` |
| NOT NULL | `nim`, `nama`, `ipk` |
| CHECK `ipk` antara 0.00 dan 4.00 |

---

## User dan Hak Akses
| User | Hak Akses |
|---|---|
| `backend_dev` | CRUD (SELECT, INSERT, UPDATE, DELETE) pada semua tabel |
| `bi_dev` | Hanya SELECT pada semua tabel/view |
| `data_engineer` | CREATE, ALTER, DROP semua objek di schema `salam` + CRUD semua tabel |

---

## Fitur Tambahan
- **Healthcheck PostgreSQL:** pgAdmin baru berjalan setelah PostgreSQL *healthy* (`depends_on` dengan `condition: service_healthy`).
- **Custom network dan volume** dengan nama khusus agar data tetap tersimpan walau container dihapus.

---

## Perintah Berguna

```bash
docker compose ps                 # status container
docker compose logs -f postgres   # log PostgreSQL
docker compose stop               # hentikan container (data tetap)
docker compose down               # hapus container (volume tetap)
docker compose down -v            # hapus container DAN data (reset total)
```

---

## Catatan Keamanan

- File `.env` masuk `.gitignore` dan tidak boleh di-push. Hanya `.env.example` yang dipublikasikan.