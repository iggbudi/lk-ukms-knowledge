# Arsitektur LK UKMs

Dokumen ini menjelaskan arsitektur sistem LK UKMs secara detail.

## 1. Ringkasan Eksekutif

| Aspek | Detail |
|-------|--------|
| **Nama Project** | LK UKMs - Sistem Proposal & LPJ |
| **Stack** | PHP 8.0+ Native, MySQL/MariaDB |
| **Frontend** | Tailwind CSS, Font Awesome, Chart.js |
| **Database** | MySQL/MariaDB (utf8mb4) |
| **Server** | Apache dengan mod_rewrite |
| **Status** | Production |

### Prinsip Arsitektural

```
1. Native PHP tanpa framework — kontrol penuh atas kode
2. MVC-like pattern — Pages (View), API (Controller), Database (Model)
3. Session-based auth — tidak依赖 JWT atau OAuth
4. Prepared statements — keamanan SQL injection
5. Role-based access — kontrol akses per role
```

## 2. Diagram Arsitektur Tingkat Tinggi

```mermaid
flowchart TB
    subgraph Client
        Browser[Browser]
        PWA[PWA Mobile]
    end
    
    subgraph Server
        Apache[Apache]
        PHP[PHP 8.0+]
        MySQL[(MySQL)]
    end
    
    subgraph External
        WhatsApp[WhatsApp API]
        Gmail[Gmail SMTP]
        CDN[Tailwind CDN]
    end
    
    Browser --> Apache
    PWA --> Apache
    Apache --> PHP
    PHP --> MySQL
    PHP --> WhatsApp
    PHP --> Gmail
    PHP --> CDN
```

## 3. Struktur Modul

```
lk.pjdigital.top/
├── index.php              # Entry point & router
├── api/                   # API endpoints (Controller)
├── pages/                 # View templates
├── templates/             # Shared layouts
├── includes/              # Core functions (Model/Helper)
├── config/                # Configuration
├── modals/                # Modal dialogs
├── public/                # Static assets
├── uploads/               # User files
└── scripts/               # Utility scripts
```

### Tanggung Jawab Modul

| Modul | Tanggung Jawab |
|-------|----------------|
| `index.php` | Routing, autentikasi dasar |
| `api/*.php` | Business logic, validasi, response JSON |
| `pages/*.php` | Rendering HTML, form handling |
| `templates/*.php` | Layout, navigasi, sidebar |
| `includes/*.php` | Helper functions, security, email |
| `config/*.php` | Database, konfigurasi |
| `public/*` | CSS, JS, icons |
| `uploads/*` | File user (proposal, surat, dll) |

## 4. Alur Request

### Alur GET Request

```mermaid
sequenceDiagram
    participant B as Browser
    participant I as index.php
    participant P as pages/*.php
    participant T as templates/*
    
    B->>I: GET /dashboard
    I->>I: Cek session auth
    I->>P: include pages/dashboard.php
    P->>T: include header.php
    P->>T: include sidebar.php
    P->>P: Render konten
    P->>T: include footer.php
    P-->>B: HTML response
```

### Alur POST Request (API)

```mermaid
sequenceDiagram
    participant B as Browser
    participant I as index.php
    participant A as api/*.php
    participant DB as MySQL
    
    B->>I: POST /api/proposal
    I->>I: Cek session auth
    I->>A: include api/proposal.php
    A->>A: Validasi CSRF token
    A->>A: requireRole()
    A->>DB: Prepared statement
    DB-->>A: Result
    A-->>B: JSON response
```

## 5. Sistem Role & Permission

```mermaid
flowchart LR
    subgraph Roles
        SA[Super Admin]
        Admin[Admin]
        User[User]
        Approver[Approver]
        Disp[Disposisi]
        RO[Read Only]
    end
    
    subgraph Access
        Full[Full Access]
        Manage[Manage Surat/Users]
        Submit[Submit Proposal/LPJ]
        Approve[Approve Proposal/LPJ]
        View[View Only]
    end
    
    SA --> Full
    Admin --> Manage
    User --> Submit
    Approver --> Approve
    Disp --> View
    RO --> View
```

### Matrix Permission

| Feature | Super Admin | Admin | User | Approver | Disposisi | Read Only |
|---------|:-----------:|:-----:|:----:|:--------:|:---------:|:---------:|
| Dashboard | ✅ | ✅ | ✅ | ✅ | ✅ | ✅ |
| Proposal | ✅ | ✅ | ✅ | ✅ | ❌ | ✅ |
| LPJ | ✅ | ✅ | ✅ | ✅ | ❌ | ✅ |
| Surat Masuk | ✅ | ✅ | ❌ | ❌ | ✅ | ✅ |
| Surat Keluar | ✅ | ✅ | ❌ | ❌ | ❌ | ❌ |
| Users | ✅ | ✅ | ❌ | ❌ | ❌ | ❌ |
| Pengumuman | ✅ | ✅ | ❌ | ❌ | ❌ | ❌ |
| Monitoring | ✅ | ✅ | ❌ | ❌ | ❌ | ❌ |
| System Settings | ✅ | ❌ | ❌ | ❌ | ❌ | ❌ |

## 6. Model Data

### Entity Relationship

```mermaid
erDiagram
    USERS {
        int id PK
        string username
        string password
        string nama
        string role
        string email
        string sebutan
        int force_change
    }
    
    PROPOSAL {
        int id PK
        string judul
        string jenis
        string pengaju
        string status
        string current_approver
        int parent_proposal_id FK
        text history
    }
    
    SURAT_MASUK {
        int id PK
        string nomor_surat
        string perihal
        string pengirim
        string disposisi
        string status
    }
    
    ALUR {
        int id PK
        string pengaju
        string acc1
        string acc2
        string acc3
        string acc4
        string acc5
        string acc6
    }
    
    PENGUMUMAN {
        int id PK
        string judul
        text isi
        string penulis
    }
    
    ACTIVITY_LOGS {
        int id PK
        string user
        string activity
        timestamp created_at
    }
    
    SYSTEM_CONFIG {
        string config_key PK
        text config_value
    }
    
    USERS ||--o{ PROPOSAL : "mengajukan"
    USERS ||--o{ ALUR : "memiliki alur"
    PROPOSAL ||--o| PROPOSAL : "parent (LPJ)"
    ALUR ||--o{ PROPOSAL : "approval chain"
```

### Tabel Utama

#### `users`
- Menyimpan data pengguna
- Password di-hash dengan bcrypt
- Field `force_change` untuk paksa ganti password

#### `proposal`
- Menyimpan proposal dan LPJ
- Dibedakan oleh field `jenis` (`Proposal` atau `LPJ`)
- `parent_proposal_id` menghubungkan LPJ ke proposal asal
- `history` menyimpan JSON riwayat approval

#### `alur`
- Menyimpan approval chain per pengaju
- acc1-acc6 adalah approver berurutan
- Approver bisa berupa username, role, atau nama

## 7. Approval Chain System

```mermaid
stateDiagram-v2
    [*] --> Draft
    Draft --> Menunggu: Submit
    Menunggu --> Diproses: Approver acc1 approve
    Diproses --> Disetujui: Semua approver setuju
    Diproses --> Revisi: Ada yang minta revisi
    Diproses --> Ditolak: Ada yang tolak
    Revisi --> Menunggu: User revisi & submit ulang
    Ditolak --> [*]
    Disetujui --> [*]
```

### Alur Approval

1. **User** submit proposal → status `Menunggu`
2. **Approver acc1** approve → status `Diproses`, current_approver → acc2
3. **Approver acc2** approve → current_approver → acc3
4. dst sampai acc6 (jika ada)
5. Semua approve → status `Disetujui`

### Normalisasi Approver

Gunakan `normalizeApprover()` untuk perbandingan:

```php
// Benar
if (normalizeApprover($currentApprover) === normalizeApprover($username)) {
    // Approver cocok
}

// Salah — tidak memperhatikan spasi/underscore
if (strtolower($currentApprover) === strtolower($username)) {
    // Bisa salah untuk "bem ftii" vs "bem_ftii"
}
```

## 8. Frontend Architecture

### Layout System

```mermaid
flowchart TB
    subgraph Page
        H[header.php]
        S[sidebar.php]
        C[Content]
        F[footer.php]
        MN[mobile_*_nav.php]
    end
    
    H --> S
    S --> C
    C --> F
    F --> MN
```

### Responsive Breakpoints

| Breakpoint | Layout | Navigasi |
|------------|--------|----------|
| >767px | Sidebar | Sidebar menu |
| ≤767px | Full-width | Bottom navigation |

### CSS Architecture

```
public/css/styles.css
├── Default styles (mobile-first)
├── @media (max-width: 767px) — Mobile styles
├── @media (pointer: coarse) — Touch device styles
└── @media (display-mode: standalone) — PWA styles
```

## 9. Security Model

### Authentication Flow

```mermaid
sequenceDiagram
    participant B as Browser
    participant I as index.php
    participant S as Session
    
    B->>I: POST /api/auth (login)
    I->>I: Validasi credentials
    I->>S: Set session data
    S-->>B: Session cookie
    
    B->>I: GET /dashboard
    I->>S: Cek session
    S-->>I: User data
    I->>I: Render page
```

### CSRF Protection

```javascript
// Frontend — ambil token dari header.php
const csrfToken = window.csrfToken;

// Kirim dengan setiap POST request
fetch('/api/proposal', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
        csrf_token: csrfToken,
        // ... data lainnya
    })
});
```

### Enforcing Roles

```php
// Di setiap API handler
requireRole(['Admin', 'Super Admin']); // Array = salah satu
requireRole('Super Admin');            // String = exact match
```

## 10. PWA Architecture

### Service Worker Strategy

```mermaid
flowchart TD
    Request[Request] --> IsNav{Navigate?}
    IsNav -->|Yes| NetworkFirst[Network First]
    IsNav -->|No| IsStatic{Static Asset?}
    IsStatic -->|Yes| CacheFirst[Cache First]
    IsStatic -->|No| Network[Network Only]
    
    NetworkFirst --> Cache[Cache]
    CacheFirst --> Cache
```

### Cache Versioning

- Format: `lkukms-pwa-v{number}`
- Update versi di `sw.js` untuk invalidate cache
- Cache lama dihapus otomatis pada activate event

## 11. Deployment

### Requirements

```
Server: Linux (Ubuntu/Debian recommended)
Web Server: Apache dengan mod_rewrite
PHP: 8.0+ dengan extensions: mysqli, mbstring, curl, gd
Database: MySQL 8.0+ atau MariaDB 10.4+
Composer: Untuk dependencies
```

### Build & Deploy

```bash
# Install dependencies
composer install

# Setup database
mysql -u root -p < config/schema.sql

# Konfigurasi database
nano config/database.php

# Set permissions
chmod -R 755 uploads/
chown -R www-data:www-data uploads/
```

## 12. Batasan & Future Work

| Saat Ini | Rencana |
|----------|---------|
| Single-server deployment | Docker containerization |
| Manual backup | Automated backup cron |
| Basic reporting | Advanced analytics |
| WhatsApp notifikasi | Multi-channel notifikasi |
| File-based uploads | Cloud storage integration |

## 13. Referensi File Kunci

| File | Topik |
|------|-------|
| `config/schema.sql` | Database schema |
| `includes/functions.php` | Core helper functions |
| `includes/security.php` | Auth & security helpers |
| `api/proposal.php` | Proposal & LPJ logic |
| `pages/proposal.php` | Proposal UI & actions |
| `public/css/styles.css` | Custom CSS & responsive |
| `sw.js` | Service worker & caching |

---

*Dokumen ini merefleksikan implementasi aktual per Juni 2026. Perbarui dokumen ini saat ada perubahan signifikan.*
