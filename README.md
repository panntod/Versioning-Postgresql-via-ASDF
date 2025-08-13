# PostgreSQL Version Management dengan ASDF di macOS

Tutorial ini membahas cara install PostgreSQL multi versi menggunakan `asdf` di macOS, troubleshooting error saat build, dan setup alias untuk versioning PostgreSQL dengan mudah.

---

## Prasyarat

- macOS terbaru (disarankan macOS 15+)
- Homebrew sudah terinstall
- ASDF sudah terinstall (`brew install asdf`)

---

## 1. Install PostgreSQL Versi 17.2 dengan asdf

```bash
asdf plugin-add postgres
asdf install postgres 17.2
asdf global postgres 17.2
````

---

## 2. Error Saat Build: ICU Library Not Found

Saat install muncul error:

```
configure: error: ICU library not found
If you have ICU already installed, see config.log for details on the failure.
It is possible the compiler isn't looking in the proper directory.
Use --without-icu to disable ICU support.
```

### Penyebab:

ICU (International Components for Unicode) yang dibutuhkan PostgreSQL tidak ditemukan karena Homebrew menginstall ICU dengan path khusus (keg-only).

### Solusi:

Set environment variable di terminal sebelum install:

```bash
export LDFLAGS="-L/opt/homebrew/opt/readline/lib -L/opt/homebrew/opt/icu4c@77/lib -L/opt/homebrew/opt/openssl@3/lib"
export CPPFLAGS="-I/opt/homebrew/opt/readline/include -I/opt/homebrew/opt/icu4c@77/include -I/opt/homebrew/opt/openssl@3/include"
export PKG_CONFIG_PATH="/opt/homebrew/opt/readline/lib/pkgconfig:/opt/homebrew/opt/icu4c@77/lib/pkgconfig:/opt/homebrew/opt/openssl@3/lib/pkgconfig"
```

Lalu install ulang PostgreSQL:

```bash
asdf install postgres 17.2
```

---

## 3. Error Lainnya: `strchrnul` Only Available on macOS 15.4 or Newer

Saat compile muncul error:

```
error: 'strchrnul' is only available on macOS 15.4 or newer [-Werror,-Wunguarded-availability-new]
```

### Penyebab:

Kode PostgreSQL menggunakan fungsi yang hanya tersedia di macOS 15.4+, tapi target build macOS kamu 15.0.

### Solusi:

Upgrade macOS ke versi terbaru (15.4+) atau menunggu patch dari PostgreSQL/asdf. Alternatif lain, pakai versi PostgreSQL yang lebih lama.

---

## 4. Setelah Berhasil Install

Inisialisasi database untuk setiap versi PostgreSQL yang kamu install:

```bash
~/.asdf/installs/postgres/17.2/bin/initdb -D ~/.asdf/installs/postgres/17.2/data
~/.asdf/installs/postgres/15.5/bin/initdb -D ~/.asdf/installs/postgres/15.5/data
```

---

## 5. Menjalankan PostgreSQL dengan Port Berbeda

Agar bisa jalan bersamaan, edit file `postgresql.conf` di masing-masing data directory dan ubah port default:

* PostgreSQL 17.2 → port 5432
* PostgreSQL 15.5 → port 5433

---

## 6. Alias PostgreSQL untuk Versioning di `.zshrc`

Tambahkan script berikut di file `~/.zshrc`:

```bash
pg-start() {
  local ver=${1:-17.2}  
  ~/.asdf/installs/postgres/$ver/bin/pg_ctl -D ~/.asdf/installs/postgres/$ver/data -l logfile_$ver start
}

pg-stop() {
  local ver=${1:-17.2}
  ~/.asdf/installs/postgres/$ver/bin/pg_ctl -D ~/.asdf/installs/postgres/$ver/data stop
}

psql() {
  local ver=${1:-17.2}
  shift
  ~/.asdf/installs/postgres/$ver/bin/psql "$@"
}
```

Reload shell:

```bash
source ~/.zshrc
```

---

## 7. Cara Pakai Alias

```bash
pg-start 15.5     # Start PostgreSQL 15.5
pg-stop 15.5      # Stop PostgreSQL 15.5
pg-start          # Start PostgreSQL default 17.2
psql 15.5 -d mydb # Connect ke PostgreSQL 15.5 database mydb
psql -d mydb      # Connect ke PostgreSQL default 17.2 database mydb
```

---

## Catatan

* Pastikan `asdf` sudah dimuat di shell kamu (`. $(brew --prefix asdf)/libexec/asdf.sh`).
* Setiap versi PostgreSQL harus punya direktori data sendiri (`~/.asdf/installs/postgres/<version>/data`).
* Gunakan port berbeda agar tidak bentrok jika menjalankan beberapa versi bersamaan.

---

Kalau ada error atau kendala, cek dulu dependencies `icu4c`, `readline`, dan `openssl` yang terinstall via Homebrew.
