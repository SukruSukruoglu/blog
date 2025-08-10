# 🚀 Adakale Project - PHP/MySQL Migration Rehberi

Bu rehber, React uygulamanızdaki tüm verileri PHP/MySQL sistemine taşımanız için adım adım talimatlar içerir.

## 📋 Ön Gereksinimler

- ✅ PHP 7.4+ (PDO ve GD extension)
- ✅ MySQL 5.7+ veya MariaDB 10.3+
- ✅ Web sunucu (Apache/Nginx)
- ✅ Composer (isteğe bağlı)

## 🏗️ 1. Database Kurulumu

### 1.1 MySQL Database Oluşturma
```sql
CREATE DATABASE adakale_project CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
USE adakale_project;
```

### 1.2 Tabloları Oluşturma
```bash
mysql -u root -p adakale_project < backend/sql/create_tables.sql
```

### 1.3 Database Bağlantı Ayarları
`backend/config/database.php` dosyasını düzenleyin:
```php
private $host = 'localhost';           // Veritabanı sunucusu
private $db_name = 'adakale_project';  // Veritabanı adı
private $username = 'root';            // Kullanıcı adı
private $password = '';                // Şifre
```

## 📦 2. Veri Export İşlemi

### 2.1 React Uygulamasından Export
1. Admin panele giriş yapın
2. **Export** sekmesine gidin
3. **"JSON Export İndir"** butonuna tıklayın
4. `adakale_export_YYYY-MM-DD.json` dosyası indirilecek

### 2.2 Görsel URL Listesi İndirme
1. Export sekmesindeki **"URL Listesini İndir"** butonuna tıklayın
2. `image_urls.txt` dosyası indirilecek

## 🔄 3. Veri Import İşlemi

### 3.1 JSON Verilerini Import Etme
```bash
cd backend/import/
php import_json.php ../../adakale_export_2024-01-01.json
```

Bu script şunları import eder:
- ✅ Proje görselleri ve içerikleri
- ✅ Blog yazıları ve kategorileri  
- ✅ Fotoğraf galerisi
- ✅ Kitap listesi ve yorumlar
- ✅ Site ayarları

### 3.2 Import Sonrası Kontrol
```sql
-- Tabloları kontrol edin
SELECT COUNT(*) FROM projects;
SELECT COUNT(*) FROM blog_posts;
SELECT COUNT(*) FROM photos;
SELECT COUNT(*) FROM books;
SELECT COUNT(*) FROM site_settings;
```

## 🖼️ 4. Görsel Migration

### 4.1 Görselleri İndirme
```bash
cd backend/tools/
php download_images.php ../../image_urls.txt
```

### 4.2 Upload Klasörü İzinleri
```bash
chmod 755 backend/uploads/
chown www-data:www-data backend/uploads/
```

### 4.3 Görsel Yollarını Güncelleme
İndirilen görseller için database'deki URL'leri güncelleyin:
```bash
cd backend/migration/
php migrate_images.php
```

## 🌐 5. Web Sunucu Ayarları

### 5.1 Apache (.htaccess)
```apache
RewriteEngine On
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule ^api/(.*)$ /backend/api/$1 [L]

# CORS Headers
Header add Access-Control-Allow-Origin "*"
Header add Access-Control-Allow-Headers "Content-Type"
Header add Access-Control-Allow-Methods "GET, POST, PUT, DELETE"
```

### 5.2 Nginx
```nginx
location /api/ {
    try_files $uri $uri/ /backend/api/index.php?$query_string;
}

location /backend/uploads/ {
    expires 1y;
    add_header Cache-Control "public, immutable";
}
```

## 🔧 6. API Endpoint'lerini Test Etme

### 6.1 Proje Listesi
```bash
curl http://localhost/api/projects.php
```

### 6.2 Blog Yazıları
```bash
curl http://localhost/api/blog.php
```

### 6.3 Fotoğraflar
```bash
curl http://localhost/api/photos.php
```

## 📁 7. Dizin Yapısı

Migration sonrası dizin yapınız şöyle olmalı:
```
adakale-project/
├── backend/
│   ├── api/              # API endpoint'leri
│   ├── config/           # Database konfigürasyonu
│   ├── uploads/          # Yüklenen görseller
│   ├── migration/        # Migration scriptleri
│   └── sql/             # Database şemaları
├── src/                 # React uygulaması (varsa)
└── MIGRATION_GUIDE.md   # Bu rehber
```

## 🔐 8. Güvenlik Ayarları

### 8.1 Admin Şifresi Değiştirme
```sql
-- Güçlü bir şifre oluşturun
UPDATE admin_users 
SET password_hash = '$2y$10$YOUR_HASHED_PASSWORD' 
WHERE username = 'admin1981';
```

### 8.2 Upload Güvenliği
```php
// backend/api/upload.php içinde
$allowedTypes = ['image/jpeg', 'image/png', 'image/gif', 'image/webp'];
$maxFileSize = 5 * 1024 * 1024; // 5MB
```

## 🚨 9. Sorun Giderme

### 9.1 Database Bağlantı Sorunları
```bash
# MySQL servisi kontrol
systemctl status mysql

# Bağlantı testi
mysql -u root -p -e "SHOW DATABASES;"
```

### 9.2 PHP Hataları
```bash
# PHP hata loglarını kontrol
tail -f /var/log/apache2/error.log
tail -f /var/log/nginx/error.log
```

### 9.3 İzin Sorunları
```bash
# Web sunucu kullanıcısına izin ver
chown -R www-data:www-data backend/uploads/
chmod -R 755 backend/uploads/
```

## ✅ 10. Migration Checklist

- [ ] Database oluşturuldu ve tablolar kuruldu
- [ ] JSON export alındı ve import edildi
- [ ] Görsel URL listesi indirildi
- [ ] Görseller sunucuya yüklendi
- [ ] Database'deki URL'ler güncellendi
- [ ] API endpoint'leri test edildi
- [ ] Web sunucu ayarları yapıldı
- [ ] Admin şifresi güncellendi
- [ ] Upload izinleri ayarlandı
- [ ] Backup alındı

## 🆘 Destek

Migration sırasında sorun yaşarsanız:
1. Hata mesajlarını kaydedin
2. Database ve PHP log'larını kontrol edin
3. Test verilerini kullanarak adım adım ilerleyin
4. Backup'larınızı koruyun

---

**🎉 Migration tamamlandığında Adakale Project'iniz tamamen PHP/MySQL sistemi üzerinde çalışacak!**
