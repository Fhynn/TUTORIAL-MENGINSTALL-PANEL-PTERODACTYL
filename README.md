# TUTORIAL-MENGINSTALL-PANEL-PTERODACTYL
Tutorial ini sangat mudah dipahami



# 🦅 Panduan Instalasi Pterodactyl Panel
### Panduan Lengkap Setup Ubuntu 24.04 LTS

> **Tutorial lengkap setup Pterodactyl Panel + Wings Daemon di VPS Ubuntu 24.04 untuk menjalankan aplikasi Node.js (contoh: Bot WhatsApp)**
> 
> **Dijelaskan step by step agar mudah dipahami & langsung jalan** ✅

---

**📝 Created by Fhinz**  
*Tutorial ini dibuat dengan penuh cinta untuk memudahkan setup Pterodactyl Panel*

---

## 📋 Spesifikasi yang Direkomendasikan

### Spesifikasi Minimum VPS
- **OS**: Ubuntu 24.04 LTS (x64)
- **CPU**: 2 core minimum
- **RAM**: 4 GB minimum (8 GB direkomendasikan)
- **Storage**: 50 GB+ SSD
- **Network**: Koneksi internet stabil

### Kebutuhan Domain
Siapkan **2 subdomain** yang mengarah ke IP VPS:
- `panel.domain.com` → **Interface Panel**
- `node.domain.com` → **Wings Daemon**

### Port yang Diperlukan
- **80** - HTTP (akan redirect ke HTTPS)
- **443** - HTTPS akses Panel
- **8080** - Komunikasi Wings daemon
- **2022** - Akses file SFTP
- **25565-25570** - Alokasi server game (contoh range)

---

## 🚀 Langkah 1: Persiapan Server Awal

### Update System & Install Tools Dasar
```bash
# Update repository paket
apt update && apt -y upgrade

# Install tools penting
apt -y install software-properties-common curl wget git unzip tar htop ufw

# Set timezone (sesuaikan kebutuhan)
timedatectl set-timezone Asia/Jakarta

# Verifikasi timezone
timedatectl
```

### Konfigurasi Firewall (Opsional tapi Direkomendasikan)
```bash
# Aktifkan UFW
ufw enable

# Izinkan port penting
ufw allow 22    # SSH
ufw allow 80    # HTTP
ufw allow 443   # HTTPS
ufw allow 8080  # Wings
ufw allow 2022  # SFTP

# Cek status
ufw status verbose
```

---

## 🛢️ Langkah 2: Instalasi Database & Web Server

### Install Layanan Inti
```bash
# Install Nginx, MariaDB, dan Redis
apt -y install nginx mariadb-server redis-server

# Start dan enable services
systemctl enable --now nginx mariadb redis-server

# Cek status layanan
systemctl status nginx mariadb redis-server
```

### Amankan Instalasi MariaDB
```bash
mysql_secure_installation
```

**Jawaban Setup Keamanan MariaDB:**
- Enter current password: `[ENTER]` (kosong untuk instalasi baru)
- Set root password: `Y` → Masukkan password kuat
- Remove anonymous users: `Y`
- Disallow root login remotely: `Y`
- Remove test database: `Y`
- Reload privilege tables: `Y`

---

## 🐘 Langkah 3: Instalasi & Konfigurasi PHP 8.3

### Install PHP dan Extension yang Diperlukan
```bash
# Install PHP 8.3 dan extension wajib
apt -y install php8.3 php8.3-fpm php8.3-cli php8.3-gd php8.3-mysql \
               php8.3-mbstring php8.3-bcmath php8.3-xml php8.3-curl \
               php8.3-zip php8.3-intl php8.3-sqlite3

# Verifikasi instalasi PHP
php -v
```

### Install Composer (PHP Package Manager)
```bash
# Download dan install Composer
curl -sS https://getcomposer.org/installer | php
mv composer.phar /usr/local/bin/composer
chmod +x /usr/local/bin/composer

# Verifikasi instalasi Composer
composer --version
```

---

## 🗄️ Langkah 4: Setup Database untuk Panel

### Buat Database dan User
```bash
# Akses MariaDB
mysql -u root -p
```

**Jalankan perintah SQL berikut:**
```sql
-- Buat database untuk Pterodactyl
CREATE DATABASE panel;

-- Buat user khusus dengan password kuat
CREATE USER 'pterodactyl'@'127.0.0.1' IDENTIFIED BY 'passwordku123!';

-- Berikan semua hak akses ke database panel
GRANT ALL PRIVILEGES ON panel.* TO 'pterodactyl'@'127.0.0.1';

-- Refresh privileges
FLUSH PRIVILEGES;

-- Keluar dari MariaDB
EXIT;
```

**⚠️ PENTING**: Ganti `passwordku123!` dengan password yang lebih kuat!

---

## 🦅 Langkah 5: Download & Install Pterodactyl Panel

### Download Panel
```bash
# Buat direktori dan masuk ke dalamnya
mkdir -p /var/www/pterodactyl
cd /var/www/pterodactyl

# Download versi terbaru
curl -Lo panel.tar.gz https://github.com/pterodactyl/panel/releases/latest/download/panel.tar.gz

# Extract dan hapus file zip
tar -xzvf panel.tar.gz && rm panel.tar.gz

# Install dependencies PHP
composer install --no-dev --optimize-autoloader
```

### Set Permission yang Tepat
```bash
# Set ownership ke www-data
chown -R www-data:www-data /var/www/pterodactyl

# Set permission direktori
chmod -R 755 /var/www/pterodactyl

# Set permission khusus untuk storage dan cache
chmod -R 775 /var/www/pterodactyl/storage /var/www/pterodactyl/bootstrap/cache
```

---

## ⚙️ Langkah 6: Konfigurasi Environment (.env)

### Buat File Konfigurasi
```bash
# Copy template .env
cp .env.example .env

# Edit file konfigurasi
nano .env
```

### Contoh Konfigurasi .env Lengkap
```env
# === APLIKASI ===
APP_ENV=production
APP_DEBUG=false
APP_KEY=
APP_TIMEZONE=Asia/Jakarta
APP_URL=https://panel.domain.com

# === DATABASE ===
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=panel
DB_USERNAME=pterodactyl
DB_PASSWORD=passwordku123!

# === CACHE & QUEUE ===
CACHE_DRIVER=file
QUEUE_CONNECTION=redis
SESSION_DRIVER=file

# === SECURITY ===
HASHIDS_SALT=RahasiaBanget123!
HASHIDS_LENGTH=8

# === EMAIL (SMTP) ===
MAIL_MAILER=smtp
MAIL_HOST=smtp.gmail.com
MAIL_PORT=587
MAIL_USERNAME=your-email@gmail.com
MAIL_PASSWORD=your-app-password
MAIL_ENCRYPTION=tls
MAIL_FROM_ADDRESS=no-reply@panel.domain.com
MAIL_FROM_NAME="Pterodactyl Panel"

# === REDIS ===
REDIS_HOST=127.0.0.1
REDIS_PASSWORD=
REDIS_PORT=6379
```

### Generate Key & Migrasi Database
```bash
# Generate application key
php artisan key:generate --force

# Jalankan migrasi database dan seed data
php artisan migrate --seed --force

# Buat user admin (ikuti prompt)
php artisan p:user:make
```

**Info User Admin:**
- Email: masukkan email aktif
- Username: admin (atau sesuai keinginan)
- First Name & Last Name: isi sesuai keinginan
- Password: buat password kuat
- Admin: Y (untuk akses penuh)

---

## 🌐 Langkah 7: Konfigurasi Nginx & SSL

### Buat Konfigurasi Nginx untuk Panel
```bash
# Buat file konfigurasi
nano /etc/nginx/sites-available/pterodactyl.conf
```

**Isi file konfigurasi:**
```nginx
server {
    listen 80;
    server_name panel.domain.com;

    root /var/www/pterodactyl/public;
    index index.php index.html;

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        include snippets/fastcgi-php.conf;
        fastcgi_pass unix:/run/php/php8.3-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include fastcgi_params;
    }

    location ~ /\.ht {
        deny all;
    }

    # Security headers
    add_header X-Content-Type-Options nosniff;
    add_header X-Frame-Options DENY;
    add_header X-XSS-Protection "1; mode=block";
}
```

### Aktifkan Konfigurasi Nginx
```bash
# Link ke sites-enabled
ln -s /etc/nginx/sites-available/pterodactyl.conf /etc/nginx/sites-enabled/

# Hapus default config
rm /etc/nginx/sites-enabled/default

# Test konfigurasi
nginx -t

# Reload Nginx
systemctl reload nginx
```

### Install SSL Certificate
```bash
# Install Certbot
apt -y install certbot python3-certbot-nginx

# Generate SSL certificate
certbot --nginx -d panel.domain.com

# Verifikasi SSL auto-renewal
certbot renew --dry-run
```

---

## 🛠️ Langkah 8: Setup Queue Worker & Cron Jobs

### Install dan Konfigurasi Supervisor
```bash
# Install Supervisor
apt -y install supervisor

# Buat konfigurasi worker
nano /etc/supervisor/conf.d/pteroq.conf
```

**Isi file pteroq.conf:**
```ini
[program:pteroq]
command=php /var/www/pterodactyl/artisan queue:work --queue=high,standard,low --sleep=3 --tries=3 --timeout=120
directory=/var/www/pterodactyl
user=www-data
autostart=true
autorestart=true
startsecs=1
startretries=3
stdout_logfile=/var/log/supervisor/pteroq.log
stderr_logfile=/var/log/supervisor/pteroq.log
```

### Aktifkan Worker
```bash
# Reload konfigurasi Supervisor
supervisorctl reread
supervisorctl update

# Start worker
supervisorctl start pteroq

# Cek status worker
supervisorctl status
```

### Setup Cron Job untuk Scheduled Tasks
```bash
# Tambah cron job untuk user root
(crontab -l 2>/dev/null; echo "* * * * * php /var/www/pterodactyl/artisan schedule:run >> /dev/null 2>&1") | crontab -

# Verifikasi cron job
crontab -l
```

---

## 👤 Langkah 9: Akses Panel & Verifikasi

### Test Akses Panel
1. Buka browser dan akses: `https://panel.domain.com`
2. Login dengan akun admin yang dibuat tadi
3. Pastikan dashboard muncul tanpa error

### Troubleshooting Panel
```bash
# Jika ada error permission
chown -R www-data:www-data /var/www/pterodactyl
chmod -R 755 /var/www/pterodactyl
chmod -R 775 /var/www/pterodactyl/storage /var/www/pterodactyl/bootstrap/cache

# Jika queue tidak jalan
supervisorctl restart pteroq

# Cek log error
tail -f /var/log/nginx/error.log
tail -f /var/www/pterodactyl/storage/logs/laravel.log
```

---

## 🐳 Langkah 10: Instalasi Docker & Wings

### Install Docker
```bash
# Download dan install Docker
curl -fsSL https://get.docker.com | sh

# Enable dan start Docker
systemctl enable --now docker

# Verifikasi instalasi Docker
docker --version
docker run hello-world
```

### Download Wings Daemon
```bash
# Buat direktori konfigurasi
mkdir -p /etc/pterodactyl /var/lib/pterodactyl

# Download Wings binary
cd /usr/local/bin
curl -L -o wings https://github.com/pterodactyl/wings/releases/latest/download/wings_linux_amd64

# Buat executable
chmod +x /usr/local/bin/wings

# Verifikasi Wings
wings --version
```

---

## 🗂️ Langkah 11: Setup Node di Panel Admin

### 1. Buat Location (Lokasi)
- Login ke Panel Admin → **Locations**
- Klik **Create New**
- **Short Code**: `id-jkt-1`
- **Description**: `Jakarta - Indonesia`

### 2. Buat Node Baru
- Admin → **Nodes** → **Create New**
- **Name**: `Node Jakarta`
- **Location**: Pilih yang tadi dibuat
- **FQDN**: `node.domain.com`
- **Communicate Over SSL**: ✅ **Enabled**
- **Behind Proxy**: ❌ **Disabled**
- **Daemon Server File Directory**: `/var/lib/pterodactyl/volumes`
- **Memory**: Total RAM VPS (contoh: 3584 MB untuk VPS 4GB)
- **Disk**: Total disk tersedia (contoh: 40960 MB untuk 50GB)
- **Daemon Port**: `8080`
- **Daemon SFTP Port**: `2022`

### 3. Download Konfigurasi Wings
Setelah node dibuat:
1. Klik nama node yang baru dibuat
2. Tab **Configuration**
3. Copy isi **Configuration File**
4. Simpan ke VPS:

```bash
nano /etc/pterodactyl/config.yml
```

Paste konfigurasi yang dicopy dari panel.

### 4. SSL Certificate untuk Node
```bash
# Stop Nginx sementara
systemctl stop nginx

# Generate SSL untuk node domain
certbot certonly --standalone -d node.domain.com

# Start Nginx kembali
systemctl start nginx
```

---

## 🕹️ Langkah 12: Setup Wings sebagai Service

### Buat Wings Service File
```bash
nano /etc/systemd/system/wings.service
```

**Isi file wings.service:**
```ini
[Unit]
Description=Pterodactyl Wings Daemon
After=docker.service network-online.target
Wants=network-online.target
Requires=docker.service
PartOf=docker.service

[Service]
User=root
WorkingDirectory=/etc/pterodactyl
LimitNOFILE=4096
PIDFile=/var/run/wings/daemon.pid
ExecStart=/usr/local/bin/wings
Restart=on-failure
StartLimitInterval=180
StartLimitBurst=30
RestartSec=5s

[Install]
WantedBy=multi-user.target
```

### Jalankan Wings Service
```bash
# Reload systemd daemon
systemctl daemon-reload

# Enable dan start Wings
systemctl enable --now wings

# Cek status Wings
systemctl status wings

# Cek log Wings (jika ada masalah)
journalctl -u wings -f
```

---

## 🎮 Langkah 13: Konfigurasi Allocation Ports

### Tambah IP dan Port Range
1. Panel Admin → **Nodes** → Pilih node
2. Tab **Allocation**
3. **Assign New Allocations**
4. **IP Address**: `IP VPS kamu` (contoh: 128.199.101.245)
5. **Ports**: `25565-25570` (atau range yang diinginkan)
6. Klik **Submit**

### Verifikasi Allocation
- Pastikan status allocation muncul sebagai **unassigned**
- Ports harus bisa diakses dari luar (cek firewall)

---

## 📦 Langkah 14: Import Egg Node.js

### Download Egg Node.js
```bash
# Download egg Node.js Generic
cd /tmp
wget https://raw.githubusercontent.com/parkervcp/eggs/master/software/nodejs/egg-node-js-generic.json
```

### Import ke Panel
1. Panel Admin → **Nests** → **Import Egg**
2. Upload file `egg-node-js-generic.json`
3. **Associated Nest**: Pilih **Generic**
4. Klik **Import**

### Verifikasi Import
- Admin → **Nests** → **Generic**
- Pastikan **Node.js Generic** muncul di daftar eggs

---

## 🤖 Langkah 15: Deploy Bot WhatsApp (Contoh)

### 1. Buat Server Baru
Panel Admin → **Servers** → **Create New**

**Server Details:**
- **Server Name**: `Bot WhatsApp`
- **Server Description**: `Bot WhatsApp otomatis`
- **Server Owner**: Pilih user
- **Node**: Pilih node yang dibuat
- **Default Allocation**: Pilih dari list

**Resource Limits:**
- **Memory**: `1024 MB`
- **Swap**: `0 MB`
- **Disk**: `2048 MB`
- **Block I/O**: `500`
- **CPU Limit**: `100%`

**Nest Configuration:**
- **Nest**: Generic
- **Egg**: Node.js Generic

**Docker Image**: `ghcr.io/parkervcp/nodejs:18`

### 2. Upload File Bot
**Cara 1 - Via Panel Files:**
1. Masuk ke server → Tab **Files**
2. Upload semua file bot (package.json, index.js, dll)

**Cara 2 - Via SFTP:**
- **Host**: `node.domain.com`
- **Port**: `2022`
- **Username**: Sesuai di panel
- **Password**: Sesuai di panel

### 3. Install Dependencies & Jalankan Bot
Masuk ke **Console** server:

```bash
# Install dependencies
npm install

# Jalankan bot (test)
node index.js
```

---

## 🔄 Langkah 16: Auto-Restart Bot (Agar Tidak Mati)

### Metode 1: Menggunakan PM2
```bash
# Install PM2 global
npm install -g pm2

# Jalankan bot dengan PM2
pm2-runtime start index.js --name bot-wa
```

### Metode 2: Schedule Restart Otomatis
1. Server → **Schedules** → **Create Schedule**
2. **Name**: `Auto Restart Bot`
3. **Cron Expression**: `*/5 * * * *` (setiap 5 menit)
4. **Tasks** → **Add Task**
5. **Action**: **Start the server**

### Metode 3: Startup Script
Edit **Startup Command** di server:
```bash
if [[ -d node_modules ]]; then npm start; else npm install && npm start; fi
```

---

## ✅ Verifikasi Setup Lengkap

### Cek Semua Service Berjalan
```bash
# Cek status semua service penting
systemctl status nginx mariadb redis-server wings

# Cek Wings connection ke Panel
journalctl -u wings --since "5 minutes ago"

# Cek resource usage
htop
```

### Test Koneksi Panel ↔ Wings
1. Panel Admin → **Nodes** → Pilih node
2. **Configuration** tab → Status harus **Online** 🟢
3. Jika **Offline** 🔴, cek konfigurasi Wings

---

## 🎯 URL Akses Akhir

| Service | URL | Keterangan |
|---------|-----|------------|
| **Panel Admin** | `https://panel.domain.com` | Dashboard admin |
| **Panel User** | `https://panel.domain.com` | Dashboard user |
| **Wings Daemon** | `https://node.domain.com:8080` | API Wings |
| **SFTP Access** | `node.domain.com:2022` | Upload file |

---

## 🛠️ Troubleshooting Umum

### ❌ Panel Error 500
```bash
# Cek log error
tail -f /var/www/pterodactyl/storage/logs/laravel.log

# Fix permission
chown -R www-data:www-data /var/www/pterodactyl
chmod -R 775 /var/www/pterodactyl/storage

# Clear cache
php artisan config:clear
php artisan cache:clear
```

### ❌ Wings Tidak Connect
```bash
# Cek status Wings
systemctl status wings

# Restart Wings
systemctl restart wings

# Cek log Wings
journalctl -u wings -f

# Cek konfigurasi
cat /etc/pterodactyl/config.yml
```

### ❌ SSL Certificate Error
```bash
# Renew certificate manual
certbot renew

# Check certificate expiry
certbot certificates
```

### ❌ Bot Crash "MODULE_NOT_FOUND"
```bash
# Masuk ke console server
npm install

# Jika masih error, hapus node_modules
rm -rf node_modules package-lock.json
npm install
```

### ❌ Memory/Disk Full Error
- **Panel**: Tingkatkan limit di **Server → Settings → Resource Management**
- **VPS**: Upgrade specs atau cleanup disk

### ❌ Database Connection Error
```bash
# Test koneksi database
mysql -u pterodactyl -p -h 127.0.0.1 panel

# Restart MariaDB jika perlu
systemctl restart mariadb
```

---

## 🔧 Maintenance & Update

### Update Panel
```bash
cd /var/www/pterodactyl

# Backup database
mysqldump -u pterodactyl -p panel > panel_backup_$(date +%Y%m%d).sql

# Download update
curl -L https://github.com/pterodactyl/panel/releases/latest/download/panel.tar.gz | tar -xzv

# Update dependencies
composer install --no-dev --optimize-autoloader

# Run migrations
php artisan migrate --force

# Clear cache
php artisan view:clear
php artisan config:clear
```

### Update Wings
```bash
# Stop Wings
systemctl stop wings

# Download versi terbaru
curl -L -o /usr/local/bin/wings https://github.com/pterodactyl/wings/releases/latest/download/wings_linux_amd64
chmod +x /usr/local/bin/wings

# Start Wings
systemctl start wings
```

### Backup Rutin
```bash
# Script backup harian
nano /root/backup_pterodactyl.sh
```

**Isi script backup:**
```bash
#!/bin/bash
DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_DIR="/root/backups"

mkdir -p $BACKUP_DIR

# Backup database
mysqldump -u pterodactyl -p'passwordku123!' panel > $BACKUP_DIR/panel_$DATE.sql

# Backup panel files
tar -czf $BACKUP_DIR/panel_files_$DATE.tar.gz /var/www/pterodactyl

# Backup Wings config
cp /etc/pterodactyl/config.yml $BACKUP_DIR/wings_config_$DATE.yml

# Hapus backup lama (>7 hari)
find $BACKUP_DIR -type f -mtime +7 -delete

echo "Backup completed: $DATE"
```

```bash
# Buat executable dan test
chmod +x /root/backup_pterodactyl.sh
/root/backup_pterodactyl.sh

# Tambah ke cron (backup harian jam 2 pagi)
(crontab -l 2>/dev/null; echo "0 2 * * * /root/backup_pterodactyl.sh") | crontab -
```

---

## 🏆 Tips Optimasi Performance

### 1. Optimasi PHP-FPM
```bash
nano /etc/php/8.3/fpm/pool.d/www.conf
```

Cari dan ubah:
```ini
pm = dynamic
pm.max_children = 20
pm.start_servers = 4
pm.min_spare_servers = 2
pm.max_spare_servers = 6
```

### 2. Optimasi Nginx
```bash
nano /etc/nginx/nginx.conf
```

Tambahkan di dalam `http` block:
```nginx
# Gzip compression
gzip on;
gzip_vary on;
gzip_min_length 1024;
gzip_types text/plain text/css text/xml text/javascript application/javascript application/xml+rss application/json;

# File caching
location ~* \.(jpg|jpeg|png|gif|ico|css|js)$ {
    expires 1y;
    add_header Cache-Control "public, immutable";
}
```

### 3. Optimasi MariaDB
```bash
nano /etc/mysql/mariadb.conf.d/50-server.cnf
```

Tambahkan di `[mysqld]`:
```ini
innodb_buffer_pool_size = 1G
innodb_log_file_size = 256M
query_cache_type = 1
query_cache_size = 64M
```

```bash
# Restart services
systemctl restart php8.3-fpm nginx mariadb
```

---

## 🎉 Selesai! Setup Berhasil

### ✅ Yang Sudah Dikerjakan:
- ✅ Panel Pterodactyl terpasang dan berjalan
- ✅ Wings Daemon terkoneksi dengan Panel
- ✅ SSL Certificate aktif untuk kedua domain
- ✅ Database dan Redis berjalan optimal
- ✅ Queue worker dan cron jobs aktif
- ✅ Node.js environment siap untuk bot
- ✅ Auto-restart dan monitoring berjalan

### 🎯 Langkah Selanjutnya:
1. **Buat User Regular**: Panel Admin → Users → Create New
2. **Upload Bot Code**: Via Files atau SFTP
3. **Install Dependencies**: `npm install` di console
4. **Jalankan Bot**: `node index.js` atau dengan PM2
5. **Monitor Performance**: Cek resource usage secara berkala

### 📞 Support & Resources:
- **Official Docs**: https://pterodactyl.io/panel/1.0/
- **Community Discord**: https://discord.gg/pterodactyl
- **GitHub Issues**: https://github.com/pterodactyl/panel/issues

---

## 🎊 Bonus: Template Bot WhatsApp Sederhana

Buat file `package.json`:
```json
{
  "name": "whatsapp-bot",
  "version": "1.0.0",
  "description": "Simple WhatsApp Bot",
  "main": "index.js",
  "scripts": {
    "start": "node index.js",
    "dev": "nodemon index.js"
  },
  "dependencies": {
    "whatsapp-web.js": "^1.23.0",
    "qrcode-terminal": "^0.12.0"
  }
}
```

Buat file `index.js`:
```javascript
const { Client, LocalAuth } = require('whatsapp-web.js');
const qrcode = require('qrcode-terminal');

const client = new Client({
    authStrategy: new LocalAuth()
});

client.on('qr', (qr) => {
    console.log('Scan QR Code di bawah dengan WhatsApp:');
    qrcode.generate(qr, {small: true});
});

client.on('ready', () => {
    console.log('✅ Bot WhatsApp siap digunakan!');
});

client.on('message', msg => {
    if (msg.body === '!ping') {
        msg.reply('🏓 Pong! Bot aktif dan berjalan di Pterodactyl!');
    }
});

client.initialize();

// Handle process termination
process.on('SIGINT', () => {
    console.log('🛑 Bot dihentikan...');
    client.destroy();
    process.exit(0);
});
```

**Upload kedua file ini ke server, lalu:**
```bash
npm install
npm start
```

---

*🎉 **Selamat!** Pterodactyl Panel dengan Wings Daemon sudah siap digunakan untuk hosting aplikasi Node.js seperti Bot WhatsApp. Panel ini bisa menjalankan multiple bot sekaligus dengan resource management yang baik.*

**💡 Pro Tip**: Selalu monitor resource usage dan lakukan backup rutin untuk menjaga stabilitas sistem!

---

## 👨‍💻 Credits

**Tutorial by: Fhinz**  
*Dibuat dengan dedikasi tinggi untuk community developer Indonesia* 🇮🇩

**💬 Need Help?**  
Jika ada pertanyaan atau butuh bantuan setup, jangan sungkan untuk bertanya!

**🌟 Enjoyed this tutorial?**  
Star repository ini dan share ke teman-teman developer lainnya!
