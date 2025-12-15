
# ⏱️ NTP Server: Chrony ile NTP Sunucu Yapılandırması

Ubuntu sistemlerde Chrony kullanarak NTP sunucusu kurulumu ve istemci yapılandırma adımları aşağıda detaylı şekilde açıklanmıştır.

---

## 📦 Kurulum

```bash
sudo apt update
sudo apt-get -y install chrony
```

---

## ⚙️ Yapılandırma

### `/etc/chrony/chrony.conf` Dosyasını Düzenleyin

```bash
sudo nano /etc/chrony/chrony.conf
```

#### ✏️ Varsayılan `pool` satırlarını yorum satırı yapın ve kendi zaman sunucunuzu tanımlayın:

```conf
#pool ntp.ubuntu.com iburst maxsources 4
#pool 0.ubuntu.pool.ntp.org iburst maxsources 1
#pool 1.ubuntu.pool.ntp.org iburst maxsources 1
#pool 2.ubuntu.pool.ntp.org iburst maxsources 2

# Örneğin Japonya NICT sunucusu
pool ntp.nict.jp iburst

# Ya da kendi lokal NTP sunucunuz
server 10.6.212.50

# Ağa zaman senkronizasyonu izni vermek için:
allow 10.0.0.0/24
```

---

## 🔁 Servisi Başlat / Yeniden Başlat

```bash
sudo systemctl restart chrony
sudo systemctl start chronyd
sudo systemctl status chronyd
```

---

## 🌐 Zaman Dilimi Ayarı

```bash
sudo timedatectl list-timezones
sudo timedatectl set-timezone Europe/Istanbul
```

---

## 🧪 Doğrulama Komutları

```bash
chronyc activity
chronyc sourcestats -v
chronyc sources
chronyc sources -v
chronyc tracking
sudo chronyc ntpdata 91.189.94.4
```

---

## 🧩 İstemci Bilgilerini Görüntüleme

```bash
sudo chronyc clients
```

---

## 📡 Ağ Üzerinden NTP Trafiğini İzleme

```bash
sudo tcpdump port 123 -i wlp3s0
```

---

> 📝 Bu döküman bir NTP sunucusu (Chrony) kurulumunu içermektedir. Hem sunucu hem istemci tarafı yapılandırmalarını kapsar.

---------------------------
sudo nano /etc/netplan/50-cloud-init.yaml

network:
  version: 2
  ethernets:
    ens160:
      dhcp4: no
      addresses: [192.168.122.88/24]
      routes:
        - to: default
          via: 192.168.122.1
      nameservers:
        addresses: [192.168.122.250] # Active Directory IP
        search: [erdogan.local]

sudo netplan apply

# Paketler
sudo apt update
sudo apt install realmd sssd sssd-tools libnss-sss libpam-sss adcli samba-common-bin packagekit policykit-1 acl -y

# Domain'e Join
sudo realm join -U Administrator erdogan.local
sudo realm join -v -U Administrator erdogan.local


SSSD Yapılandırması (Kullanıcı Formatı)
Kullanıcıların user@domain yerine sadece user olarak girmesi için.
sudo nano /etc/sssd/sssd.conf(Yetkisi 600 olmalı):

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

sudo systemctl restart sssd

Home Klasörü Oluşturma

sudo pam-auth-update --enable mkhomedir

Hangi AD gruplarının SSH yapabileceğini belirler.

sudo realm deny --all
sudo realm permit -g sistem
sudo realm permit -g web

Sudo (Yönetici) Yetkileri
sudo nano /etc/sudoers.d/domain_admins dosyası:

# Sistem grubu şifresiz/şifreli tam yetkili
%sistem ALL=(ALL:ALL) ALL

# Web grubu sadece Nginx servislerini yönetebilir (systemctl)
%web ALL=(ALL) /usr/bin/systemctl restart nginx, /usr/bin/systemctl reload nginx, /usr/bin/systemctl status nginx, /usr/bin/systemctl start nginx, /usr/bin/systemctl stop nginx, /usr/bin/systemctl restart nginx.service, /usr/bin/systemctl reload nginx.service, /usr/bin/systemctl status nginx.service, /usr/bin/systemctl start nginx.service, /usr/bin/systemctl stop nginx.service

Dosya ve Klasör İzinleri (ACL & SGID)
Web grubunun /var/www/html/intranet üzerinde çalışabilmesi ve dosya sahipliklerinin düzgün kalması için.

# 1. Klasör oluşturulur ve sahibi www-data yapılır
sudo mkdir -p /var/www/html/intranet
sudo chown -R www-data:www-data /var/www/html/intranet
sudo chmod -R 775 /var/www/html/intranet

# 2. SGID Biti: Yeni dosyaların GRUBU otomatik www-data olur
sudo chmod g+s /var/www/html/intranet

# 3. ACL: Web grubuna okuma/yazma izni verilir (Mevcut + Gelecek dosyalar)
sudo setfacl -R -m g:web:rwx /var/www/html/intranet
sudo setfacl -R -d -m g:web:rwx /var/www/html/intranet
sudo setfacl -R -d -m g:www-data:rwx /var/www/html/intranet

-------
# 1. Ana klasörün sahibini ve grubunu www-data yap (Garanti olsun)
sudo chown -R www-data:www-data /var/www/html

# 2. Ana klasöre yazma izni ver
sudo chmod -R 775 /var/www/html

# 3. SGID Bitini aç (Çok Önemli: Yeni açılan klasörlerin grubu otomatik www-data olur)
sudo chmod g+s /var/www/html

# 4. ACL: Web grubuna mevcut dosyalar için tam yetki ver
sudo setfacl -R -m g:web:rwx /var/www/html

# 5. ACL (Default): Gelecekte oluşturulacak dosya/klasörler için kuralı miras bırak
sudo setfacl -R -d -m g:web:rwx /var/www/html

# 6. ACL (Default): www-data yetkisini de miras bırak (Sistem bozulmasın diye)
sudo setfacl -R -d -m g:www-data:rwx /var/www/html
----------------
# 1. Mevcut dosyalar için 'ubuntu' kullanıcısına (u:ubuntu) tam yetki ver
sudo setfacl -R -m u:yrg_sistem:rwx /var/www/html

# 2. Gelecek dosyalar için de yetkiyi otomatikleştir (Default ACL)
sudo setfacl -R -d -m u:yrg_sistem:rwx /var/www/html
--------------------
olmaz ise
# 1. Önbelleği tamamen temizle
sudo sss_cache -E

# 2. Servisi yeniden başlat
sudo systemctl restart sssd

# 3. Grubun gelip gelmediğini tekrar kontrol et
getent group web
-----------------

# Opsiyoneldir, yukarıdaki ACL yöntemi varken şart değildir
sudo usermod -aG www-data guler
sudo usermod -aG www-data emre
sudo usermod -aG www-data melih

-----------

Server02 Yapılandırması (Sadece Sistem)
A. Giriş İzinleri
Web grubunun erişimi tamamen engellenir.


sudo realm deny --all
sudo realm permit -g sistem

Sudo Yetkileri
sudo nano /etc/sudoers.d/domain_admins dosyası:

%sistem ALL=(ALL:ALL) ALL

Senaryo,			Kullanıcı,	Sunucu,		Komut,							Beklenen Sonuç
SSH Erişimi,		Guler,		Server01,	ssh guler@server01,				✅ Başarılı
SSH Erişimi,		Guler,		Server02,	ssh guler@server02,				❌ Erişim Reddedildi
Root Olma,			Guler,		Server01,	sudo su,						❌ Yasaklandı
Servis Yönetimi,	Guler,		Server01,	sudo systemctl restart nginx,	✅ Başarılı
Dosya Yazma,		Guler,		Server01,	/var/www/html/intranet,			✅ Yazar (Grup: www-data olur)
Tam Yetki,			Erdogan,	Her İkisi,	sudo -i,						✅ Root olur

---------------------
--Trobleshooting--

ping -c 3 erdogan.local

timedatectl

Trust Relationship
sudo realm list

Kullanıcı Bilgisi Çek.

id guler

veritabanında sorgula
getent passwd erdogan

su - guler

# Admin kullanıcısı ile şu komutu çalıştır:
sudo -l -U guler

ACL Listesini Görüntüle:

getfacl /var/www/html/intranet

loglar
sudo tail -f /var/log/auth.log

SSSD Servis Hataları:

sudo systemctl status sssd
# veya detaylı log için:
sudo journalctl -u sssd -f

Değişikliklerin anında yansıması için önbelleği temizle:

# SSSD önbelleğini tamamen siler
sudo sss_cache -E

# Servisi yeniden başlatır
sudo systemctl restart sssd
