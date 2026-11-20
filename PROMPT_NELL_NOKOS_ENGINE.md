# PROMPT MASTER — NELL NOKOS ENGINE
## Python CLI untuk integrasi provider nomor virtual dengan akses gratis resmi

> **Instruksi utama untuk Codex:** Kerjakan langsung pada repository yang sedang terbuka di GitHub Codespaces. Jangan hanya memberikan contoh atau rencana. Audit repository, implementasikan project secara bertahap, jalankan seluruh test, perbaiki kegagalan, dan dokumentasikan hasilnya. Jangan mengklaim integrasi provider berhasil jika belum diuji terhadap dokumentasi dan akses nyata.

---

# 1. Misi dan batasan produk

Bangun **NELL NOKOS ENGINE**, aplikasi terminal/CLI berbasis Python untuk mengelola integrasi provider nomor virtual yang secara resmi menyediakan akses gratis atau free-tier dan secara eksplisit mengizinkan penerimaan SMS melalui API atau mekanisme resmi.

Produk ini:
- CLI sepenuhnya; **tidak membuat dashboard web, frontend web, atau panel browser**.
- Bukan simulasi. Integrasi yang dinyatakan aktif harus menggunakan API/provider sungguhan dan kredensial sah.
- Tidak mengharuskan API berbayar. Prioritaskan free-tier resmi, open-source, dan provider yang menyediakan akses gratis secara sah. Jelaskan jika nomor, aktivasi, atau penggunaan nyata ternyata tetap berbayar.
- Tidak boleh mengakali OTP, CAPTCHA, rate limit, verifikasi identitas, pembatasan negara, atau ketentuan provider.
- Tidak boleh mengambil nomor publik bersama/temporary-SMS untuk menerima OTP akun pihak ketiga, mengotomatisasi pembuatan akun, atau melakukan verifikasi massal.
- Gunakan hanya nomor yang dialokasikan secara sah kepada pengguna dan layanan yang mengizinkan alur penggunaan yang dimaksud.
- Fokus pada pengelolaan nomor yang sah, penerimaan pesan masuk, diagnostik, dan pengujian integrasi.

**Jangan menjanjikan “nomor gratis dari semua platform/dunia”.** Ketersediaan nomor, negara, kemampuan menerima SMS, dan free-tier bergantung pada provider serta dapat berubah.

# 2. Cara kerja Codex — wajib

Sebelum mengedit:
1. Jalankan `pwd`, `ls -la`, dan inspeksi struktur repository.
2. Periksa file README, konfigurasi, dependency, test, dan Git status.
3. Jangan menghapus atau menimpa pekerjaan pengguna yang sudah ada.
4. Jika repository sudah berisi project, pertahankan kode yang relevan dan buat rencana patch. Jangan melakukan rewrite total tanpa alasan.
5. Buat `IMPLEMENTATION_PLAN.md` singkat berisi tahapan dan risiko, lalu lanjut implementasi tanpa menunggu persetujuan kecuali ada keputusan yang benar-benar memerlukan kredensial atau pilihan pengguna.
6. Gunakan dokumentasi resmi provider sebagai sumber utama. Catat URL dokumentasi, tanggal pemeriksaan, wilayah, syarat free-tier, dan batasan.
7. Jangan mengarang endpoint, parameter, response schema, harga, kuota, maupun status aktif.
8. Bila provider tidak dapat diverifikasi, masukkan sebagai `UNVERIFIED` dan jangan aktifkan adapter produksinya.
9. Jangan mencetak token, nomor penuh, isi SMS sensitif, atau secret ke log.
10. Jangan pernah meminta pengguna menaruh secret ke source code atau commit ke Git.

# 3. Riset provider yang dapat dipertanggungjawabkan

Lakukan riset menyeluruh tetapi terukur terhadap provider nomor virtual/telekomunikasi yang:
- Memiliki dokumentasi resmi yang dapat diakses.
- Memiliki free-tier, trial gratis, kredit gratis, atau program gratis yang masih berlaku.
- Secara eksplisit menyediakan nomor yang dapat menerima SMS melalui integrasi resmi.
- Mengizinkan penggunaan untuk tujuan aplikasi ini berdasarkan terms dan acceptable-use policy.
- Tidak membutuhkan bypass, scraping, endpoint privat, akun hasil beli, atau penyalahgunaan nomor publik.

Untuk setiap kandidat, verifikasi:
- Nama dan situs resmi provider.
- Dokumentasi API resmi.
- Terms of Service dan Acceptable Use Policy.
- Negara/region yang didukung.
- Apakah nomor benar-benar dapat menerima SMS.
- Apakah API inbound SMS tersedia pada free-tier.
- Persyaratan registrasi, identitas, metode pembayaran, dan batasan akun.
- Kuota, masa berlaku kredit/trial, biaya setelah kuota, dan batas penggunaan.
- Cara autentikasi dan mekanisme webhook/polling resmi.
- Apakah layanan masih aktif dan dokumentasinya masih relevan.
- Tanggal terakhir diverifikasi.

Buat `docs/PROVIDER_RESEARCH.md` dengan tabel:

| Provider | Region | Free-tier evidence | Inbound SMS | API docs | Terms checked | Status | Last checked |
|---|---|---|---|---|---|---|---|

Status yang diperbolehkan:
- `VERIFIED_FREE`: free-tier dan inbound SMS resmi terbukti tersedia.
- `VERIFIED_TRIAL`: hanya trial/kredit terbatas; bukan gratis permanen.
- `PAID_ONLY`: layanan relevan tetapi penggunaan nyata berbayar.
- `UNVERIFIED`: bukti tidak cukup.
- `UNAVAILABLE`: layanan/API sudah tidak tersedia.
- `NOT_ALLOWED`: terms melarang alur penggunaan yang dimaksud.

**Jangan masukkan provider ke `VERIFIED_FREE` hanya karena memiliki free account.** Harus ada bukti bahwa nomor dan penerimaan SMS memang termasuk akses gratisnya. Jika tidak ditemukan provider yang memenuhi semua syarat, tulis hasil negatif secara jujur dan siapkan adapter interface serta mode konfigurasi yang menunggu provider sah. Jangan mengisi kekosongan dengan layanan SMS publik yang berisiko.

# 4. Arsitektur aplikasi

Gunakan Python 3.11+ (usahakan kompatibel dengan versi Python Codespaces yang tersedia), dengan struktur modular sederhana dan mudah dirawat.

Struktur target:

```text
nell-nokos-engine/
├── pyproject.toml
├── README.md
├── LICENSE
├── .gitignore
├── .env.example
├── IMPLEMENTATION_PLAN.md
├── src/
│   └── nell_nokos/
│       ├── __init__.py
│       ├── cli.py
│       ├── config.py
│       ├── database.py
│       ├── models.py
│       ├── logging_config.py
│       ├── security.py
│       ├── services/
│       │   ├── provider_registry.py
│       │   ├── number_service.py
│       │   ├── message_service.py
│       │   └── health_service.py
│       └── providers/
│           ├── base.py
│           ├── registry.py
│           └── [adapter yang benar-benar terverifikasi]
├── tests/
│   ├── conftest.py
│   ├── unit/
│   ├── integration/
│   ├── contract/
│   └── security/
└── docs/
    ├── ARCHITECTURE.md
    ├── PROVIDER_RESEARCH.md
    ├── SECURITY.md
    ├── TESTING.md
    └── TROUBLESHOOTING.md
```

Jangan membuat file/folder yang tidak diperlukan. Jika repository sudah memiliki struktur yang baik, adaptasikan tanpa memaksakan struktur di atas.

## Stack

- Python 3.11+
- Typer untuk CLI.
- Rich untuk tampilan terminal.
- HTTPX untuk HTTP API.
- Pydantic untuk validasi konfigurasi/model jika memang bermanfaat.
- SQLite untuk penyimpanan lokal.
- SQLAlchemy tidak wajib; gunakan `sqlite3` standar bila lebih sederhana.
- pytest untuk testing.
- pytest-cov untuk coverage.
- Ruff untuk lint dan format.
- mypy atau pyright bila sesuai dengan kompleksitas project.
- respx atau mock transport HTTPX untuk mock API dalam test.

Hindari dependency berlebihan. Jangan memakai Docker, PostgreSQL, Redis, atau layanan eksternal yang tidak dibutuhkan.

# 5. Provider adapter contract

Buat interface/abstract base class yang konsisten untuk seluruh provider.

Setiap adapter hanya boleh mengimplementasikan endpoint dan kemampuan yang tercantum di dokumentasi resmi provider.

Kontrak konseptual:
- `provider_id`
- `display_name`
- `capabilities`
- `validate_configuration()`
- `health_check()`
- `list_owned_numbers()` — hanya nomor yang memang dialokasikan ke akun pengguna.
- `get_number_details(number_id)`
- `list_inbound_messages(number_id, cursor=None)` — hanya pesan yang dapat diakses secara sah oleh akun.
- `normalize_message(raw_payload)` — normalisasi ke model internal.

Ketentuan:
- Jangan membuat fungsi `rent_number`, `activate_number`, atau `request_otp` untuk mengotomatisasi verifikasi akun pihak ketiga.
- Jangan membuat fungsi untuk memanen nomor, mencari OTP platform lain, atau mengulang permintaan kode.
- Jangan mengasumsikan semua provider mendukung fitur yang sama.
- Capability harus eksplisit, misalnya `inbound_sms`, `webhook`, `polling`, `owned_numbers`.
- Jika provider tidak mendukung fitur, kembalikan error capability yang jelas; jangan berpura-pura berhasil.
- Terapkan timeout, retry terbatas hanya untuk error sementara yang aman, exponential backoff, dan idempotency pada pemrosesan pesan.
- Hormati rate limit dan `Retry-After` sesuai dokumentasi.
- Jangan retry operasi yang berpotensi menimbulkan efek samping tanpa jaminan idempotensi.

# 6. Fitur CLI

Gunakan command root:

```bash
nell-nokos
```

Sediakan command yang jelas dan punya `--help`:

```bash
nell-nokos --help
nell-nokos version
nell-nokos init
nell-nokos providers list
nell-nokos providers inspect <provider>
nell-nokos providers check <provider>
nell-nokos numbers list
nell-nokos numbers show <number-id>
nell-nokos messages list
nell-nokos messages show <message-id>
nell-nokos system health
nell-nokos config show
nell-nokos db status
```

Jangan membuat command yang melakukan pembelian/penyewaan nomor atau meminta OTP untuk layanan pihak ketiga.

## Perilaku command

### `init`
- Buat direktori data aplikasi di lokasi user yang sesuai OS.
- Buat database SQLite dan tabel yang dibutuhkan.
- Buat file konfigurasi awal tanpa secret.
- Tampilkan petunjuk konfigurasi provider resmi.
- Tidak meminta pengguna memasukkan nomor atau token ke command line yang dapat tersimpan di shell history.

### `providers list`
- Tampilkan nama, status riset, capability, dan status konfigurasi.
- Bedakan status bukti free-tier dari status koneksi akun pengguna.
- Jangan menyebut provider `VERIFIED_FREE` bila riset tidak mendukungnya.

### `providers inspect`
- Tampilkan ringkasan dokumentasi, region, batasan, dan tautan resmi.
- Jangan tampilkan credential.

### `providers check`
- Jalankan health check yang aman dan read-only.
- Masking identifier dan credential.
- Berikan error yang actionable.

### `numbers list/show`
- Hanya tampilkan nomor milik/yang dialokasikan kepada akun sah pengguna.
- Masking nomor secara default. Full number hanya dengan flag eksplisit dan peringatan privasi jika memang diperlukan.
- Jangan melakukan pencarian nomor publik.

### `messages list/show`
- Tampilkan metadata pesan dan isi hanya untuk pesan yang memang diterima melalui akun sah.
- Redaksi kode OTP/token autentikasi secara default.
- Jangan menyediakan filter atau parser yang ditujukan untuk mengotomatisasi penggunaan OTP di platform pihak ketiga.
- Jangan menampilkan isi SMS sensitif di log.

### `system health`
- Periksa Python, dependency penting, database, konfigurasi, dan konektivitas provider yang telah diatur.
- Jangan mengirim request ke provider yang belum dikonfigurasi.

### `config show`
- Tampilkan konfigurasi non-rahasia saja.
- Nilai secret selalu `********`.

# 7. Database

Gunakan SQLite lokal dengan migration/versioning sederhana.

Tabel minimum:
- `schema_migrations`
- `providers`
- `owned_numbers`
- `inbound_messages`
- `sync_state`
- `audit_events`

Aturan data:
- Simpan provider ID, external resource ID, nomor yang dimasking bila memungkinkan, negara, status, timestamp, dan metadata minimum.
- Simpan isi pesan hanya jika diperlukan untuk fungsi pengguna, dan jelaskan retensi data.
- Hindari menyimpan OTP sebagai field khusus atau mengekstrak kode OTP.
- Buat unique constraint pada provider + external message ID untuk mencegah duplikasi.
- Gunakan transaksi dan parameterized SQL.
- Sediakan `db status` dan mekanisme backup sederhana.
- Jangan menyimpan credential provider di database dalam bentuk plaintext.

# 8. Konfigurasi dan secret management

Dukung environment variables dan file `.env` lokal yang tidak masuk Git.

Contoh `.env.example`:

```dotenv
NELL_DATA_DIR=
NELL_LOG_LEVEL=INFO
NELL_DATABASE_PATH=
# Tambahkan variabel provider hanya jika adapter resmi telah diverifikasi.
# PROVIDER_EXAMPLE_API_TOKEN=
```

Ketentuan:
- `.env` masuk `.gitignore`.
- Jangan membuat token palsu yang terlihat seperti credential valid.
- Jangan hardcode API key.
- Jangan pernah mencetak environment variable secret.
- Validasi konfigurasi saat provider digunakan.
- Berikan error yang menjelaskan nama variabel yang kurang tanpa membocorkan nilainya.
- Gunakan permission file yang wajar pada sistem yang mendukungnya.

# 9. Keamanan dan privasi

Buat `docs/SECURITY.md`.

Wajib:
- Masking nomor telepon dan identifier.
- Redaksi pola kode autentikasi/OTP pada output dan log.
- Sanitasi log dan exception HTTP.
- TLS verification tetap aktif; jangan menyediakan opsi default untuk menonaktifkannya.
- Jangan menyimpan request/response body provider secara utuh pada log.
- Batasi ukuran payload masuk.
- Validasi signature webhook jika provider mendukungnya, sesuai dokumentasi.
- Cegah replay webhook menggunakan timestamp/ID dan penyimpanan idempotensi.
- Jangan membuka server webhook ke internet secara otomatis.
- Jika webhook diperlukan, jelaskan setup tunnel/domain HTTPS resmi dan risiko endpoint publik; jangan menganggap Codespaces URL permanen.
- Terapkan retention policy dan perintah penghapusan data lokal.
- Jangan menyimpan atau mengumpulkan data pengguna lain.
- Jangan bypass provider rate limit, KYC, geoblocking, atau kebijakan anti-abuse.

# 10. Testing — wajib, bukan opsional

Codex harus menulis test nyata dan menjalankannya. Jangan mengklaim test lulus tanpa menjalankan command dan membaca hasilnya.

## Unit tests
Uji:
- parsing konfigurasi;
- validasi model;
- masking nomor;
- redaksi pesan sensitif;
- provider registry;
- capability checks;
- normalisasi pesan;
- timestamp/timezone;
- database CRUD;
- transaksi dan rollback;
- deduplikasi external message ID;
- pagination/cursor;
- penanganan error dan timeout.

## Integration tests
- CLI `init` pada temporary directory.
- SQLite dibuat dan schema terpasang.
- Provider check menggunakan HTTP mock.
- Normalisasi response provider yang sudah didokumentasikan.
- Sinkronisasi pesan mock dan idempotensi.
- Perintah list/show memakai database sementara.
- Aplikasi tetap berfungsi tanpa provider yang dikonfigurasi.

## Contract tests
Untuk setiap adapter:
- Cocokkan path, method, header, parameter, dan response schema dengan dokumentasi resmi yang dicatat.
- Uji response sukses dan error.
- Uji 401/403, 404, 429, 5xx, timeout, malformed JSON, dan payload tak terduga.
- Semua test default harus memakai mock; jangan menghabiskan free-tier atau mengirim SMS sungguhan.
- Live smoke test harus opt-in, read-only, dan membutuhkan flag eksplisit serta credential yang disediakan pengguna. Jangan jalankan live test otomatis di CI.

## Security tests
- Pastikan secret tidak muncul di output CLI atau log.
- Pastikan `.env` tidak dilacak Git.
- Pastikan nomor dimasking secara default.
- Pastikan konten autentikasi direduksi.
- Pastikan SQL memakai parameter binding.
- Pastikan webhook menolak signature invalid jika signature diwajibkan.
- Pastikan duplicate webhook/message tidak diproses dua kali.
- Pastikan payload terlalu besar ditolak.

## CLI tests
Uji:
- `--help`
- `version`
- `init`
- `providers list`
- `system health`
- `db status`
- error untuk provider tidak dikenal
- error untuk provider belum dikonfigurasi

## Coverage
Target coverage minimal 85% untuk core logic dan seluruh adapter yang diimplementasikan. Jangan menulis test yang hanya memeriksa bahwa fungsi ada. Test perilaku dan edge case.

# 11. Quality gates dan command

Konfigurasikan command yang konsisten di `pyproject.toml` atau dokumentasi.

Minimal jalankan:

```bash
python -m pip install -e ".[dev]"
ruff check .
ruff format --check .
pytest -q
pytest --cov=nell_nokos --cov-report=term-missing
```

Jika mypy dipasang:

```bash
mypy src
```

Tambahkan GitHub Actions workflow `.github/workflows/ci.yml`:
- Trigger push dan pull request.
- Python versions yang relevan dan didukung.
- Install dependency.
- Ruff lint dan format check.
- Unit/integration/contract mock tests.
- Coverage report.
- Tidak memakai credential provider.
- Tidak mengirim SMS nyata.
- Tidak menjalankan live smoke test.

CI harus aman untuk fork dan pull request dari luar. Jangan expose secrets.

# 12. Dokumentasi pengguna

`README.md` wajib menjelaskan:
- Apa itu NELL NOKOS ENGINE.
- Batasan dan model biaya: software gratis tidak menjamin nomor gratis.
- Syarat penggunaan yang sah.
- Dukungan OS dan Python.
- Instalasi di GitHub Codespaces dan Linux.
- Cara menjalankan `pip install -e ".[dev]"`.
- Cara `nell-nokos init`.
- Cara menambahkan konfigurasi provider yang benar-benar diverifikasi.
- Contoh command CLI.
- Cara menjalankan test.
- Troubleshooting.
- Daftar provider dan tanggal verifikasi.
- Fitur yang belum tersedia.

Sertakan contoh output yang jelas diberi label **contoh**, bukan klaim bahwa provider telah aktif.

# 13. Definition of Done

Project dianggap selesai hanya jika:
- [ ] Repository diaudit sebelum perubahan.
- [ ] `IMPLEMENTATION_PLAN.md` dibuat.
- [ ] CLI bisa di-install dan `--help` berfungsi.
- [ ] `init` membuat database/config lokal.
- [ ] Provider registry bekerja.
- [ ] Riset provider memiliki sumber resmi dan status jujur.
- [ ] Adapter nyata hanya dibuat bila API resmi dan free-tier inbound SMS benar-benar terverifikasi.
- [ ] Tidak ada endpoint/API fiktif.
- [ ] Database dan deduplikasi berfungsi.
- [ ] Masking dan redaksi aktif secara default.
- [ ] Unit, integration, contract-mock, dan security tests lulus.
- [ ] Coverage core logic minimal 85%, atau jelaskan bagian yang belum mencapai target.
- [ ] Ruff lulus.
- [ ] GitHub Actions CI dibuat dan valid.
- [ ] README dan dokumentasi lengkap.
- [ ] Tidak ada secret, credential, atau data SMS pribadi di Git.
- [ ] Tidak ada fitur dashboard web atau otomasi OTP pihak ketiga.

# 14. Laporan akhir Codex

Setelah implementasi, tampilkan laporan dengan format:

1. **Repository audit** — kondisi awal dan file yang dipertahankan/diubah.
2. **Implemented features** — fitur yang benar-benar selesai.
3. **Provider research** — provider, status, bukti free-tier, region, dan sumber resmi.
4. **Real integration status** — bedakan adapter yang benar-benar terhubung dari adapter mock.
5. **Files changed** — daftar file penting.
6. **Tests executed** — command dan hasil aktual.
7. **Coverage** — persentase aktual.
8. **CI status** — workflow dan status validasi.
9. **Security review** — secret handling, masking, redaksi, dan keterbatasan.
10. **Manual steps** — hanya langkah yang benar-benar memerlukan tindakan pengguna, seperti membuat akun provider atau memasukkan credential.
11. **Known limitations** — biaya nomor, wilayah, free-tier, webhook, atau provider yang belum terverifikasi.

Jangan menutupi kegagalan. Jika tidak ada provider gratis yang memenuhi kriteria, katakan terus terang. Tetap selesaikan CLI, registry, database, dokumentasi, dan test suite agar provider resmi dapat ditambahkan kemudian.

---

## Instruksi penutup untuk Codex

Mulai dengan audit repository dan riset dokumentasi resmi. Setelah itu implementasikan project secara bertahap. Prioritaskan correctness, keamanan, transparansi biaya, dan testability daripada jumlah provider.

**Jangan berhenti setelah membuat struktur file.** Implementasikan, jalankan test, perbaiki error, dan berikan laporan akhir berdasarkan hasil command yang benar-benar dijalankan.
