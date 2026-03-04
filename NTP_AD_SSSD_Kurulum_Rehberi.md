# NTP, Active Directory (AD) ve SSSD Yapılandırma Rehberi

**Oluşturulma Tarihi:** 04-03-2026 07:24

---

## 1️⃣ NTP (Chrony) Kurulumu ve Yapılandırması

### Paket Kurulumu

```bash
sudo apt update
sudo apt-get -y install chrony
```

### Konfigürasyon

```bash
sudo nano /etc/chrony/chrony.conf
```

Varsayılan Ubuntu havuzlarını kapatabilirsiniz:

```bash
#pool ntp.ubuntu.com iburst maxsources 4
#pool 0.ubuntu.pool.ntp.org iburst maxsources 1
#pool 1.ubuntu.pool.ntp.org iburst maxsources 1
#pool 2.ubuntu.pool.ntp.org iburst maxsources 2
```

#### Harici NTP Sunucusu

```bash
pool ntp.nict.jp iburst
```

#### Lokal NTP Sunucusu

```bash
server 10.6.212.50
```

#### Ağdan Erişime İzin Verme

```bash
allow 10.0.0.0/24
```

### Servis İşlemleri

```bash
sudo systemctl restart chrony
sudo systemctl start chronyd
sudo systemctl status chronyd
```

### Saat Dilimi Ayarı

```bash
sudo timedatectl list-timezones
sudo timedatectl set-timezone Europe/Istanbul
```

### Chrony Kontrol Komutları

```bash
chronyc activity
chronyc sourcestats -v
chronyc sources
chronyc sources -v
chronyc tracking
sudo chronyc ntpdata 91.189.94.4
sudo chronyc clients
```

### Trafik İzleme

```bash
sudo tcpdump port 123 -i wlp3s0
```

---

## 2️⃣ Active Directory Domain Join

### Gerekli Paketler

```bash
sudo apt update
sudo apt install realmd sssd sssd-tools libnss-sss libpam-sss adcli samba-common-bin packagekit policykit-1 acl -y
```

### Domain Join

```bash
sudo realm join -U yr503373 yargitay.gov.tr
sudo realm join -v -U yr503373 yargitay.gov.tr
```

---

## 3️⃣ SSSD Yapılandırması

```bash
sudo nano /etc/sssd/sssd.conf
```

> [!IMPORTANT]
> Dosya yetkisi **600** olmalıdır: `sudo chmod 600 /etc/sssd/sssd.conf`

#### Örnek 1: Genel Yapılandırma (erdogan.local)

```ini
[sssd]
domains = erdogan.local
config_file_version = 2
services = nss, pam

[domain/erdogan.local]
default_shell = /bin/bash
krb5_store_password_if_offline = True
cache_credentials = True
krb5_realm = ERDOGAN.LOCAL
realmd_tags = manages-system joined-with-adcli
id_provider = ad
ad_domain = erdogan.local
ldap_id_mapping = True
use_fully_qualified_names = False
fallback_homedir = /home/%u
access_provider = ad
```

#### Örnek 2: Grup Bazlı Erişim Kontrolü (yargitay.gov.tr)

```ini
[sssd]
domains = yargitay.gov.tr
config_file_version = 2
services = nss, pam

[domain/yargitay.gov.tr]
ad_domain = yargitay.gov.tr
krb5_realm = YARGITAY.GOV.TR
realmd_tags = manages-system joined-with-adcli
cache_credentials = True
id_provider = ad
krb5_store_password_if_offline = True
default_shell = /bin/bash
ldap_id_mapping = True
use_fully_qualified_names = False
fallback_homedir = /home/%u
access_provider = simple
simple_allow_groups = gg_web_server, sudo, gg_server
```

#### Servis Yeniden Başlatma

```bash
sudo systemctl restart sssd
```

---

## 4️⃣ Home Klasörü Otomatik Oluşturma

```bash
sudo pam-auth-update --enable mkhomedir
```

---

## 5️⃣ SSH Erişim Kontrolü

```bash
sudo realm deny --all
sudo realm permit -g gg_server
sudo realm permit -g gg_web_server
```

---

## 6️⃣ Sudo Yetkilendirme

```bash
sudo nano /etc/sudoers.d/domain_admins
```

Tam yetkili grup:

```bash
%sistem ALL=(ALL:ALL) ALL
```

Nginx servis yönetimi için sınırlı yetki:

```bash
%gg_web_server ALL=(ALL) /usr/bin/systemctl restart nginx, \
    /usr/bin/systemctl reload nginx, \
    /usr/bin/systemctl status nginx, \
    /usr/bin/systemctl start nginx, \
    /usr/bin/systemctl stop nginx, \
    /usr/bin/systemctl restart nginx.service, \
    /usr/bin/systemctl reload nginx.service, \
    /usr/bin/systemctl status nginx.service, \
    /usr/bin/systemctl start nginx.service, \
    /usr/bin/systemctl stop nginx.service
```

---

## 7️⃣ /var/www/html ACL & Yetkilendirme Yapılandırması

Bu yapılandırmada:

- **Grup:** `gg_server`
- **Grup:** `gg_web_server`
- **Kullanıcı:** `yrg_sistem`
- **Sahip:** `www-data`
- SGID aktif (yeni dosyalar otomatik doğru grupla gelir)
- Mevcut ve gelecekteki dosyalar için ACL tanımlıdır

### 7.1 Sahiplik ve Temel Linux İzinleri

```bash
sudo chown -R www-data:www-data /var/www/html
sudo chmod -R 775 /var/www/html
```

### 7.2 SGID Biti (Yeni Dosyalar İçin)

Yeni oluşturulan dosya ve klasörlerin grubu otomatik `www-data` olur:

```bash
sudo chmod g+s /var/www/html
```

Kontrol:

```bash
ls -ld /var/www/html
```

Çıktıda şu format görülmelidir:

```
drwxrwsr-x
```

### 7.3 ACL — Mevcut Dosyalar İçin Yetkiler

#### Gruplar

```bash
sudo setfacl -R -m g:gg_server:rwx /var/www/html
sudo setfacl -R -m g:gg_web_server:rwx /var/www/html
```

#### Kullanıcı

```bash
sudo setfacl -R -m u:yrg_sistem:rwx /var/www/html
```

### 7.4 Default ACL — Gelecekte Oluşacak Dosyalar İçin

#### Gruplar

```bash
sudo setfacl -R -d -m g:gg_server:rwx /var/www/html
sudo setfacl -R -d -m g:gg_web_server:rwx /var/www/html
```

#### Kullanıcı

```bash
sudo setfacl -R -d -m u:yrg_sistem:rwx /var/www/html
```

#### Sistem Stabilitesi İçin www-data Default ACL

```bash
sudo setfacl -R -d -m g:www-data:rwx /var/www/html
```

### 7.5 Yapılandırma Kontrolü

```bash
getfacl /var/www/html
```

Beklenen ACL girdileri:

```
group:gg_server:rwx
group:gg_web_server:rwx
user:yrg_sistem:rwx
default:group:gg_server:rwx
default:group:gg_web_server:rwx
default:user:yrg_sistem:rwx
```

### 7.6 Yetki Matrisi

| Yetkili       | Tür           | Yetki |
|---------------|---------------|-------|
| www-data      | Owner         | rwx   |
| gg_server     | Group (ACL)   | rwx   |
| gg_web_server | Group (ACL)   | rwx   |
| yrg_sistem    | User (ACL)    | rwx   |

### 7.7 AD Kullanıcı/Grup Kontrolü

```bash
getent group gg_web_server
getent group gg_server
id yrg_sistem
```

Görünmüyorsa:

```bash
sudo sss_cache -E
sudo systemctl restart sssd
```

---

## 8️⃣ Sorun Giderme

### SSSD Cache Temizleme

```bash
sudo sss_cache -E
sudo systemctl restart sssd
```

### Grup Kontrolü

```bash
getent group web
```

### Domain Kontrolleri

```bash
ping -c 3 yargitay.gov.tr
timedatectl
sudo realm list
id guler
getent passwd srv3373
su - srv3373
sudo -l -U srv3373
getfacl /var/www/html/intranet
sudo tail -f /var/log/auth.log
sudo systemctl status sssd
sudo journalctl -u sssd -f
```

---

## 9️⃣ Alternatif SSSD Konfigürasyonu (Simple Access Control)

```ini
[sssd]
domains = yargitay.gov.tr
config_file_version = 2
services = nss, pam

[domain/yargitay.gov.tr]
ad_domain = yargitay.gov.tr
krb5_realm = YARGITAY.GOV.TR
realmd_tags = manages-system joined-with-adcli
cache_credentials = True
id_provider = ad
krb5_store_password_if_offline = True
default_shell = /bin/bash
ldap_id_mapping = True
use_fully_qualified_names = False
fallback_homedir = /home/%u
access_provider = simple
simple_allow_groups = gg_web_server, sudo, gg_server
```

---

## 🔟 PAM Access Control

### /etc/pam.d/sshd İçine Ekle

```bash
account required pam_access.so
```

### /etc/security/access.conf

```conf
+ : (gg_web_server) : ALL
+ : (gg_server) : ALL
+ : (sudo) : ALL
+ : root : ALL
- : ALL : ALL
```

---

## ✅ Senaryo Testleri

| Senaryo         | Kullanıcı | Sunucu    | Komut                            | Beklenen Sonuç              |
|-----------------|-----------|-----------|----------------------------------|-----------------------------|
| SSH Erişimi     | Guler     | Server01  | `ssh guler@server01`             | ✅ Başarılı                 |
| SSH Erişimi     | Guler     | Server02  | `ssh guler@server02`             | ❌ Erişim Reddedildi        |
| Root Olma       | Guler     | Server01  | `sudo su`                        | ❌ Yetkisiz                 |
| Servis Yönetimi | Guler     | Server01  | `sudo systemctl restart nginx`   | ✅ Başarılı                 |
| Dosya Yazma     | Guler     | Server01  | `/var/www/html/intranet`         | ✅ Yazabilir (Grup: www-data) |
| Tam Yetki       | Erdogan   | Her İkisi | `sudo -i`                        | ✅ Root Olur                |

---

## 📌 Notlar

- SSSD konfigürasyon dosyası **600** yetkili olmalıdır.
- Zaman senkronizasyonu AD ortamında **kritik** öneme sahiptir.
- ACL ve SGID kombinasyonu grup bazlı yetkilendirmede en sağlıklı yöntemdir.
