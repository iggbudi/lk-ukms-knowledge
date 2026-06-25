# Project: Sistem Disposisi Surat UNISBANK (aidis.my.id)

> **Server**: aidis.my.id  
> **Domain**: https://aidis.my.id  
> **Status**: Active Development  
> **Last Updated**: 2026-06-25

---

## 📋 Overview

Aplikasi web untuk manajemen disposisi surat masuk di **Universitas Stikubank (UNISBANK)**. Sistem ini menangani routing surat hierarkis dari level Rektor hingga Kaprodi, dengan siklus review, usulan bottom-up, delegasi kewenangan, notifikasi, dan pelaporan komprehensif.

### Tujuan Utama
- Registrasi surat masuk dengan nomor agenda otomatis
- Routing disposisi melalui hierarki organisasi (Rektor → Wakil Rektor → Dekan → Kaprodi)
- Tracking tanggapan, review, dan instruksi lanjutan
- Mendukung delegasi kewenangan
- Dashboard dan reporting real-time
- Usulan bottom-up dari admin fakultas

---

## 🛠 Technology Stack

| Layer | Technology | Version |
|-------|-----------|---------|
| **Backend** | PHP Native (tanpa framework) | 8.2+ |
| **Database** | PostgreSQL | 15+ |
| **DB Access** | PDO | - |
| **Dependency** | Composer | - |
| **Email** | PHPMailer | 7.1 |
| **Frontend** | HTML/CSS/JS Vanilla | - |
| **Testing** | PHPUnit | 10 |
| **Test DB** | SQLite in-memory | - |

### ⚠️ Constraints (WAJIB DIINGAT)
- ❌ **TIDAK BOLEH** pakai framework (Laravel, Symfony, dll)
- ❌ **TIDAK BOLEH** pakai frontend framework (React, Vue, Angular)
- ❌ **TIDAK BOLEH** tambah dependency tanpa approval
- ✅ **WAJIB** pakai prepared statements untuk SQL
- ✅ **WAJIB** escape HTML output dengan `htmlspecialchars()`
- ✅ **WAJIB** validasi CSRF token untuk state-changing operations

---

## 🏗 Arsitektur

### Pola 3-Layer
```
Controller → Service → Repository → Database
```

### Request Flow
```
public/index.php (Front Controller)
  ↓
Router (route matching)
  ↓
Middleware (Auth, RBAC, RateLimit)
  ↓
Controller (request handling)
  ↓
Service (business logic)
  ↓
Repository (database queries)
  ↓
Database (PostgreSQL)
  ↓
Template Rendering (layouts + pages)
  ↓
HTTP Response
```

### Layer Responsibilities

| Layer | Tanggung Jawab |
|-------|---------------|
| **Controller** | Handle HTTP request/response, validasi input, session, flash messages, render view |
| **Service** | Business logic, validasi domain, koordinasi antar repository, return standardized response |
| **Repository** | SQL queries via PDO, CRUD operations, prepared statements |
| **Helper** | Database, Router, Validator, Response, RolePermissions, CsrfToken |
| **Middleware** | AuthMiddleware, RBACMiddleware, RateLimitMiddleware |

### Service Response Format
```php
[
  'success' => bool,
  'message' => string,
  'data'    => array,    // optional
  'errors'  => array,    // optional
]
```

### Dependency Injection Pattern
```php
public function __construct(?AuthService $authService = null)
{
    $this->authService = $authService ?? new AuthService(new PenggunaRepository());
}
```

---

## 📁 Struktur Direktori

```
/var/www/aidis.my.id/
├── config/              # Konfigurasi aplikasi & database
│   ├── app.php          # App config (timezone, upload, SMTP, WhatsApp)
│   └── database.php     # Database connection config
├── database/
│   ├── migrations/      # SQL migration files
│   │   ├── 001_create_all_tables.sql
│   │   └── 002_alur_disposisi_unit.sql
│   ├── seeds/           # Seed data
│   ├── schema.sql       # Schema export
│   └── full_dump.sql    # Full DB dump
├── public/
│   ├── index.php        # Front controller & routing (500+ lines)
│   ├── router.php       # PHP built-in server router
│   └── .htaccess        # Apache rewrite
├── src/
│   ├── Controllers/     # 8 controllers
│   ├── Services/        # 14 services
│   ├── Repositories/    # 17 repositories
│   ├── Helpers/         # 6 helpers
│   └── Middleware/      # 3 middleware
├── templates/
│   ├── layouts/         # main.php, header.php, sidebar.php
│   ├── pages/           # Page templates (32 files)
│   └── partials/        # Reusable components
├── tests/               # PHPUnit tests (22 files)
├── uploads/             # File uploads
│   ├── lampiran/        # Mail attachments
│   └── foto_profil/     # Profile photos
├── docs/                # Wiki documentation (16 files)
├── AGENTS.md            # Agent guidance
└── composer.json
```

### Codebase Stats
- **50** PHP files di `src/`
- **32** PHP template files di `templates/`
- **22** PHP test files di `tests/`

---

## 👥 Role-Based Access Control (RBAC)

### 11 Roles
| Role | Keterangan |
|------|-----------|
| **Rektor** | Otoritas tertinggi, buat disposisi, review usulan |
| **Sekretaris_Rektor** | Bisa delegasi kewenangan Rektor |
| **Wakil_Rektor** | Buat disposisi untuk domainnya |
| **Admin_Wakil_Rektor** | Support admin untuk Wakil Rektor |
| **Admin_Fakultas** | Buat usulan bottom-up, terima disposisi |
| **Direktur** | Terima dan respon disposisi |
| **Dekan** | Penerima disposisi level fakultas |
| **Kepala_Divisi** | Penerima disposisi akhir (baru) |
| **Kaprodi** | Penerima disposisi level prodi |
| **Staf** | Staff umum, terima disposisi |
| **Admin_Tata_Usaha** | Kelola surat masuk, master data, pengguna |

### Permission Matrix

| Resource | Action | Roles |
|----------|--------|-------|
| surat | daftar | Admin_Tata_Usaha |
| surat | lihat | Admin_Tata_Usaha, Rektor, Sekretaris_Rektor, Wakil_Rektor, Admin_Wakil_Rektor |
| surat | review_awal | Sekretaris_Rektor |
| surat | delegasi_pemberi | Sekretaris_Rektor |
| disposisi | buat | Rektor, Sekretaris_Rektor, Wakil_Rektor |
| disposisi | lihat | Semua role (kecuali Admin_Tata_Usaha) |
| disposisi | review | Rektor, Sekretaris_Rektor, Wakil_Rektor |
| disposisi | proses_distribusi | Sekretaris_Rektor, Admin_Wakil_Rektor |
| tanggapan | beri | Admin_Wakil_Rektor, Admin_Fakultas, Direktur, Dekan, Kepala_Divisi, Kaprodi, Staf |
| usulan | buat | Admin_Fakultas |
| usulan | review | Rektor, Sekretaris_Rektor |
| pengguna | kelola | Admin_Tata_Usaha, Sekretaris_Rektor |
| master_data | kelola | Admin_Tata_Usaha |

### Authorization Pattern
- **Route-level**: RBACMiddleware checks role/action
- **Object-level**: Service checks ownership/permissions (WAJIB, jangan rely on route-level saja)

---

## 🗄 Database Schema

### Enum Types
- `peran_pengguna` - User roles (11 values)
- `status_pengguna` - Aktif/Non-aktif
- `status_disposisi` - Pending/Dikerjakan/Selesai/Ditolak
- `tingkat_urgensi` - Urgency levels
- `status_usulan` - Proposal status
- `tipe_notifikasi` - Notification types (includes new unit flow types)
- `entity_type_lampiran` - Attachment entity types
- `mode_delegasi` - Delegation modes

### Core Tables

| Tabel | Fungsi |
|-------|--------|
| `pengguna` | Akun, role, unit kerja |
| `surat_masuk` | Data surat masuk, status arsip |
| `disposisi` | Header disposisi, parent follow-up |
| `disposisi_penerima` | Penerima dan status pekerjaan |
| `disposisi_penerima_unit` | Unit penerima sebelum distribusi (baru) |
| `disposisi_instruksi` | Pivot disposisi-instruksi |
| `tanggapan` | Respons penerima disposisi |
| `instruksi_lanjutan` | Instruksi reviewer setelah tanggapan |
| `lampiran` | Metadata file (polymorphic: surat/tanggapan/usulan) |
| `notifikasi` | Notifikasi in-app |
| `usulan` | Usulan bottom-up |
| `usulan_riwayat` | Riwayat aksi usulan |
| `status_history` | Histori status disposisi |
| `delegasi_kewenangan` | Delegasi Rektor → Sekretaris_Rektor |
| `audit_log` | Audit trail |
| `unit_kerja` | Master unit organisasi |
| `jenis_unit_kerja` | Master jenis unit |
| `kategori_surat` | Master kategori surat |
| `instruksi_disposisi` | Master instruksi disposisi |

### Unit Kerja Kanonis
`REKTOR`, `SEKREK`, `WR1`, `WR2`, `ADM_WR1`, `ADM_WR2`

### Migration Files
1. `001_create_all_tables.sql` - Initial schema
2. `002_alur_disposisi_unit.sql` - Unit flow additions

---

## 🛣 Routes

### Auth & Dashboard
| Method | Path | Handler |
|--------|------|---------|
| GET | `/` | Redirect |
| GET | `/login` | AuthController::showLogin |
| POST | `/login` | AuthController::processLogin |
| GET | `/logout` | AuthController::logout |
| GET | `/dashboard` | DashboardController::index |
| GET | `/dashboard/report` | DashboardController::report |
| GET | `/dashboard/export` | DashboardController::exportReport |

### Surat
| Method | Path | Handler | RBAC |
|--------|------|---------|------|
| GET | `/surat` | SuratController::index | surat:lihat |
| GET | `/surat/create` | SuratController::create | surat:daftar |
| POST | `/surat` | SuratController::store | surat:daftar |
| GET | `/surat/{id}` | SuratController::show | surat:lihat |
| POST | `/surat/{id}/archive` | SuratController::archiveSurat | surat:daftar |
| GET | `/surat/archived` | SuratController::archived | surat:daftar |
| GET | `/surat/review-sekretaris` | SuratController::reviewSekretarisList | surat:review_awal |
| GET | `/surat/{id}/review-sekretaris` | SuratController::reviewSekretaris | surat:review_awal |
| POST | `/surat/{id}/update-pemberi-disposisi` | SuratController::updatePemberiDisposisi | surat:delegasi_pemberi |
| POST | `/surat/{id}/delegasi-pemberi` | SuratController::delegasiPemberi | surat:delegasi_pemberi |
| GET | `/surat/disposisi-masuk` | SuratController::disposisiMasuk | disposisi:buat |

### Disposisi
| Method | Path | Handler | RBAC |
|--------|------|---------|------|
| GET | `/disposisi/create/{suratId}` | DisposisiController::create | disposisi:buat |
| POST | `/disposisi/store/{suratId}` | DisposisiController::store | disposisi:buat |
| GET | `/disposisi/{id}` | DisposisiController::show | disposisi:lihat |
| GET/POST | `/disposisi/{id}/follow-up` | DisposisiController::followUp | disposisi:buat |
| GET | `/disposisi/proses` | DisposisiController::prosesList | disposisi:proses_distribusi |
| GET | `/disposisi/{id}/proses` | DisposisiController::prosesShow | disposisi:proses_distribusi |
| POST | `/disposisi/{id}/proses` | DisposisiController::prosesSubmit | disposisi:proses_distribusi |

### Master Data (Auth + RBAC master_data:kelola)
- `/master/jenis-unit-kerja` - CRUD
- `/master/unit-kerja` - CRUD
- `/master/kategori-surat` - CRUD
- `/master/instruksi-disposisi` - CRUD

### Pengguna & Profil
- `/pengguna` - CRUD (RBAC pengguna:kelola)
- `/pengguna/delegasi/*` - Delegasi management
- `/api/unit-kerja/{id}/jabatan` - API endpoint
- `/profil` - Profile management (auth only)

### Usulan (Bottom-up Proposal)
- `/usulan` - List & create
- `/usulan/{id}` - Detail
- `/usulan/{id}/approve|reject|request-revision` - RBAC usulan:review
- `/usulan/{id}/submit-revision` - RBAC usulan:buat

### Notifikasi API
- `/api/notifikasi/unread-count` - Badge count
- `/api/notifikasi/recent` - Recent notifications
- `/api/notifikasi/{id}/read` - Mark as read

### Static Assets
- `/asset/<filename>` - Served before routing with strict path validation
- Currently only: `logo-circle-unisbank.png`

---

## 📦 Services Inventory

| Service | Tanggung Jawab |
|---------|---------------|
| **AuthService** | Login, password, session/auth context |
| **UserService** | CRUD pengguna, profil, status, foto profil |
| **MasterDataService** | Jenis unit, unit kerja, kategori surat, instruksi disposisi |
| **SuratService** | Pendaftaran, pencarian, detail, arsip surat masuk |
| **DisposisiService** | Pembuatan disposisi, follow-up, tree, penerima valid, status |
| **TanggapanService** | Tanggapan penerima, instruksi lanjutan, review/complete |
| **UsulanService** | Usulan bottom-up dan riwayat review |
| **DelegasiService** | Aktivasi/nonaktif delegasi kewenangan |
| **NotifikasiService** | In-app/email/WhatsApp notifikasi |
| **ReportService** | Statistik dashboard, report, export |
| **FileStorageService** | Validasi upload, simpan file, metadata lampiran |
| **AuditLogService** | Pencatatan audit log |
| **EmailService** | Kirim email via SMTP |
| **WhatsAppService** | Kirim WhatsApp via external API |

---

## 🔒 Security

### Implemented
- ✅ CSRF protection pada login/logout
- ✅ Rate limiting IP-based (DB-backed + in-memory fallback)
- ✅ Session security (8-hour timeout, regenerate ID on login)
- ✅ Password policy (min 8 chars, complexity)
- ✅ Prepared statements untuk SQL
- ✅ HTML output escaping dengan `htmlspecialchars()`

### Known Issues / TODO
- ⚠️ Route dengan AuthMiddleware tetap perlu object-level authorization
- ⚠️ Akses file upload publik perlu authorization check per entity
- ⚠️ Notifikasi harus hanya bisa dibaca oleh penerima yang sah
- ⚠️ Beberapa UUID generator fallback pakai `mt_rand()` (should use `random_bytes()`)

---

## ⚙️ Configuration

### Environment Variables
```bash
DB_HOST=localhost
DB_PORT=5432
DB_DATABASE=disposisi_surat
DB_USERNAME=postgres
DB_PASSWORD=your_password
APP_BASE_URL=https://aidis.my.id
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USERNAME=
SMTP_PASSWORD=
SMTP_ENCRYPTION=tls
WA_DWIBUDI_BASE_URL=https://wa.dwibudi.my.id
WA_DWIBUDI_API_KEY=
WA_DWIBUDI_INSTANCE_ID=
```

### Upload Limits
- **Lampiran**: 10 MB (PDF, JPG, PNG, DOCX)
- **Foto Profil**: 2 MB (JPG, PNG)

### Key Settings
- **Pagination**: 20 per page
- **Session**: 8 hours
- **Lockout**: 5 failed attempts → 15 minutes
- **Overdue threshold**: 24 hours

---

## 🧪 Testing

### Run Tests
```bash
./vendor/bin/phpunit
```

### Test Structure
```
tests/
├── Controllers/
├── Services/
├── Middleware/
├── Helpers/
└── Integration/
```

### Notes
- Tests use SQLite in-memory for isolation
- SQLite tests won't catch PostgreSQL enum mismatch
- Always test negative paths for security fixes

---

## 📚 Documentation Files

| File | Isi |
|------|-----|
| `docs/README.md` | Wiki entry point |
| `docs/arsitektur.md` | Architecture patterns |
| `docs/alur-surat.md` | Mail flow & business logic |
| `docs/role-permission.md` | RBAC details |
| `docs/routes.md` | All routes |
| `docs/service-layer.md` | Service patterns |
| `docs/repository-layer.md` | Repository patterns |
| `docs/database.md` | Database schema |
| `docs/view-template.md` | Template rendering |
| `docs/conventions.md` | Coding standards |
| `docs/testing.md` | Testing guidelines |
| `docs/workflow.md` | Dev workflow |
| `docs/handoff.md` | Agent handoff template |
| `docs/TODO.md` | Known issues |
| `docs/dashboard-report.md` | Dashboard UI/UX |
| `docs/login-refactor.md` | Login refactor details |

---

## 🐛 Known Issues & History

### Recent Fixes
1. **Notifikasi URL Isolation** - Normalized internal notification links to root-relative paths (commit `f885ac2`)
2. **Login Security** - CSRF, rate limiting, session cookies, password complexity
3. **Login Refactor** - Logo centering, static asset serving

### Pending Issues (from TODO.md)
- Object-level authorization needs attention
- File upload access control
- Notification read authorization
- TanggapanService wiring in DisposisiController
- UUID generator should use `random_bytes()`

### Critical Production Issue (Resolved)
- **Enum mismatch**: Production DB `tipe_notifikasi` didn't have new enum values
- **Root cause**: Migration 002 not applied before code deployment
- **Fix**: Apply `ALTER TYPE tipe_notifikasi ADD VALUE IF NOT EXISTS '...'`
- **Lesson**: Always apply migrations before deploying code that uses new enum values

---

## 🚀 Deployment

### Installation
```bash
composer install
psql -U postgres -c "CREATE DATABASE disposisi_surat;"
psql -U postgres -d disposisi_surat -f database/full_dump.sql
```

### Dev Server
```bash
php -S localhost:8080 -t public public/router.php
```

### Default Login
| Email | Password | Role |
|-------|----------|------|
| admin@unisbank.ac.id | admin123 | Admin_Tata_Usaha |

---

## 🎯 Key Patterns & Decisions

1. **No Framework** - Pure PHP, no Laravel/Symfony
2. **3-Layer Architecture** - Controller → Service → Repository
3. **Nullable DI** - Controllers accept nullable dependencies for testability
4. **Standardized Response** - Services return `['success', 'message', 'data', 'errors']`
5. **Route + Object RBAC** - Never rely solely on route-level RBAC
6. **Flash Messages** - Use `App\Helpers\Response` for flash/errors/old input
7. **Template Rendering** - Variables set then include layout
8. **Static Assets** - Served before routing with strict validation
9. **Case-insensitive Lookup** - Unit kerja codes must handle legacy mixed-case data

---

## 📝 Agent Guidelines

### Before Working
1. Read `docs/README.md`
2. Read relevant wiki pages
3. Check `docs/TODO.md`
4. Review `docs/conventions.md`

### During Work
- Follow 3-layer architecture
- Use dependency injection
- Write tests
- Validate security (CSRF, SQL injection, authorization)
- Check RBAC for new routes

### After Work
1. Update wiki documentation
2. Run test suite
3. Fill handoff summary
4. Verify security checklist

---

*Knowledge compiled by Pi agent on 2026-06-25*
*Source: /var/www/aidis.my.id*
