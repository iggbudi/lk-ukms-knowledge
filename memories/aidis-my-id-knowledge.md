# Knowledge Export: aidis.my.id (Sistem Disposisi Surat UNISBANK)

> Exported: 2026-06-25  
> Source: /var/www/aidis.my.id  
> Server: aidis.my.id

---

## 🧠 Key Learnings

### 1. Project Architecture
- **Pattern**: 3-layer (Controller → Service → Repository)
- **Backend**: PHP 8.2+ native (NO frameworks)
- **Database**: PostgreSQL 15+
- **Frontend**: HTML/CSS/JS vanilla (NO frameworks)
- **Testing**: PHPUnit 10 with SQLite in-memory

### 2. Critical Constraints
- ❌ No Laravel, Symfony, or any PHP framework
- ❌ No React, Vue, Angular, or any frontend framework
- ❌ No external dependencies without approval (except PHPMailer)
- ✅ Must use prepared statements
- ✅ Must escape HTML with htmlspecialchars()
- ✅ Must validate CSRF tokens
- ✅ Must check object-level authorization (not just route-level)

### 3. RBAC System
- 11 roles: Rektor, Sekretaris_Rektor, Wakil_Rektor, Admin_Wakil_Rektor, Admin_Fakultas, Direktur, Dekan, Kepala_Divisi, Kaprodi, Staf, Admin_Tata_Usaha
- Route-level RBAC via middleware
- Object-level authorization in services (CRITICAL)

### 4. Database Knowledge
- PostgreSQL enums for type safety
- Migration 001: Initial schema
- Migration 002: Unit flow additions (new roles, notification types)
- IMPORTANT: Must apply migrations before deploying code that uses new enum values

### 5. Service Response Format
```php
[
  'success' => bool,
  'message' => string,
  'data'    => array,    // optional
  'errors'  => array,    // optional
]
```

### 6. Known Issues Resolved
- **Notifikasi URL**: Normalized internal links to root-relative paths
- **Login Security**: CSRF, rate limiting, session cookies implemented
- **Enum Mismatch**: Production DB missing migration 002 enum values (FIXED)

### 7. Pending Issues
- Object-level authorization needs attention
- File upload access control
- Notification read authorization
- TanggapanService wiring in DisposisiController
- UUID generator should use random_bytes()

---

## 📊 Project Stats

| Metric | Value |
|--------|-------|
| PHP files (src) | 50 |
| Template files | 32 |
| Test files | 22 |
| Controllers | 8 |
| Services | 14 |
| Repositories | 17 |
| Helpers | 6 |
| Middleware | 3 |
| Roles | 11 |
| Database tables | 19+ |

---

## 🔧 Development Workflow

### Before Working
1. Read `docs/README.md`
2. Read relevant wiki pages
3. Check `docs/TODO.md`
4. Review `docs/conventions.md`

### During Work
- Follow 3-layer architecture
- Use dependency injection
- Write tests
- Validate security
- Check RBAC

### After Work
1. Update wiki documentation
2. Run test suite: `./vendor/bin/phpunit`
3. Fill handoff summary
4. Verify security checklist

---

## 🚀 Quick Reference

### Key Commands
```bash
# Install dependencies
composer install

# Run tests
./vendor/bin/phpunit

# Start dev server
php -S localhost:8080 -t public public/router.php

# Database setup
psql -U postgres -d disposisi_surat -f database/full_dump.sql
```

### Default Login
| Email | Password | Role |
|-------|----------|------|
| admin@unisbank.ac.id | admin123 | Admin_Tata_Usaha |

### Important Files
- Front Controller: `public/index.php`
- Config: `config/app.php`, `config/database.php`
- Wiki Entry: `docs/README.md`
- Known Issues: `docs/TODO.md`

---

## 🔗 Related Documentation

- Full wiki: `wiki/aidis-my-id-disposisi-surat.md`
- Architecture: `docs/arsitektur.md`
- Routes: `docs/routes.md`
- RBAC: `docs/role-permission.md`
- Database: `docs/database.md`
- Services: `docs/service-layer.md`
- Conventions: `docs/conventions.md`
- Testing: `docs/testing.md`

---

*Knowledge compiled by Pi agent from project at /var/www/aidis.my.id*
