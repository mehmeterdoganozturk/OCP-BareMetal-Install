Sunucu Yönetimi, Domain Join ve Yetkilendirme Rehberi

Bu döküman; sistem zaman senkronizasyonu, Active Directory (AD) entegrasyonu, gelişmiş yetkilendirme (Sudoers) ve dosya izinleri (ACL) yapılandırmalarını kapsamaktadır.
1. Zaman Senkronizasyonu (Chrony)

Sistem saatinin güncel olması, özellikle Domain yapılarında Kerberos biletlerinin geçerliliği için kritiktir.
Kurulum ve Yapılandırma
Bash

# Sistem güncelleme ve kurulum
sudo apt update
sudo apt-get -y install chrony

# Yapılandırma dosyasını düzenle
sudo nano /etc/chrony/chrony.conf

chrony.conf içerisine eklenecek örnekler:
Code snippet

# Varsayılan Ubuntu sunucularını kapatıp özel sunucu ekleme
# pool ntp.ubuntu.com iburst maxsources 4

# Özel NTP Sunucuları
pool ntp.nict.jp iburst
server 10.6.212.50 iburst

# Lokal ağdaki istemcilere zaman servisi verme izni
allow 10.0.0.0/24

Servis Yönetimi ve Kontrol
Bash

sudo systemctl restart chrony
sudo systemctl start chronyd
sudo systemctl status chronyd

# Saat dilimi ayarı
sudo timedatectl list-timezones
sudo timedatectl set-timezone Europe/Istanbul

# Durum izleme komutları
chronyc activity        # Aktif kaynak sayısı
chronyc sources -v      # Bağlı sunucuların detaylı listesi
chronyc tracking        # Senkronizasyon hassasiyeti
sudo chronyc ntpdata 91.189.94.4

2. Active Directory (AD) Entegrasyonu

Ubuntu sistemini Windows Domain yapısına dahil etmek ve kullanıcıları yönetmek için kullanılır.
Gerekli Paketlerin Kurulumu
Bash

sudo apt update
sudo apt install realmd sssd sssd-tools libnss-sss libpam-sss adcli samba-common-bin packagekit policykit-1 acl -y

Domain'e Dahil Olma (Join)
Bash

# Domain'i keşfetme ve katılma
sudo realm join -v -U Administrator erdogan.local

SSSD Yapılandırması

Kullanıcıların user@domain yerine sadece user olarak giriş yapabilmesi için /etc/sssd/sssd.conf dosyasını düzenleyin.
Önemli: Dosya yetkisi 600 olmalıdır.
Bash

sudo nano /etc/sssd/sssd.conf

Örnek Yapılandırma:
Ini, TOML

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

Ev Dizini (Home) Otomatik Oluşturma

Kullanıcı ilk kez giriş yaptığında klasörünün oluşması için:
Bash

sudo pam-auth-update --enable mkhomedir

3. Erişim ve Sudo Yetkileri (RBAC)
Giriş İzinlerini Kısıtlama
Bash

# Tüm domain kullanıcılarını engelle, sadece belirli gruplara izin ver
sudo realm deny --all
sudo realm permit -g sistem
sudo realm permit -g web

Gelişmiş Sudoer Tanımları

/etc/sudoers.d/domain_admins dosyasını oluşturarak gruplara özel yetki verin:
Bash

# 1. Sistem grubu: Her komutu çalıştırabilir
%sistem ALL=(ALL:ALL) ALL

# 2. Web grubu: Sadece Nginx servislerini yönetebilir
%web ALL=(ALL) /usr/bin/systemctl restart nginx, /usr/bin/systemctl reload nginx, /usr/bin/systemctl status nginx, /usr/bin/systemctl start nginx, /usr/bin/systemctl stop nginx, /usr/bin/systemctl restart nginx.service, /usr/bin/systemctl reload nginx.service, /usr/bin/systemctl status nginx.service, /usr/bin/systemctl start nginx.service, /usr/bin/systemctl stop nginx.service

4. Gelişmiş Dosya İzinleri (ACL & SGID)

/var/www/html dizininde farklı grupların (örneğin web grubu) dosya oluşturabilmesi ve sahipliğin bozulmaması için:
Bash

# Ana klasör sahipliği
sudo chown -R www-data:www-data /var/www/html
sudo chmod -R 775 /var/www/html

# SGID Bitini Aç (Yeni dosyalar otomatik www-data grubuna ait olur)
sudo chmod g+s /var/www/html

# ACL: Mevcut dosyalar için web grubuna tam yetki
sudo setfacl -R -m g:web:rwx /var/www/html

# ACL: Gelecekte oluşturulacak (Default) dosyalara yetkiyi miras bırak
sudo setfacl -R -d -m g:web:rwx /var/www/html
sudo setfacl -R -d -m g:www-data:rwx /var/www/html

# Belirli bir kullanıcıya (ör: yrg_sistem) özel tam yetki
sudo setfacl -R -m u:yrg_sistem:rwx /var/www/html
sudo setfacl -R -d -m u:yrg_sistem:rwx /var/www/html

5. Güvenlik ve PAM Kısıtlamaları

Daha katı bir erişim kontrolü için /etc/security/access.conf kullanılabilir.

    PAM modülünü aktifleştir: /etc/pam.d/sshd içine ekle:
    account required pam_access.so

    Kısıtlamaları tanımla: /etc/security/access.conf

Code snippet

# Sadece AD grubuna ve yerel adminlere izin ver
+ : (gg_web_server) : ALL
+ : (gg_server) : ALL
+ : (sudo) : ALL
+ : root : ALL

# Geri kalan herkesi engelle
- : ALL : ALL

6. Test ve Doğrulama Senaryoları
Senaryo	Kullanıcı	Sunucu	Beklenen Sonuç
SSH Erişimi	Guler	Server01	✅ Başarılı
SSH Erişimi	Guler	Server02	❌ Erişim Reddedildi
Root Olma	Guler	Server01	❌ Yasaklandı
Nginx Yönetimi	Guler	Server01	✅ Başarılı (Sudo ile)
Dosya Yazma	Guler	Server01	✅ Başarılı (Grup: www-data)
Tam Yetki	Erdogan	Her İkisi	✅ Başarılı (Root olur)
7. Sorun Giderme (Troubleshooting)
Servis ve Bağlantı Kontrolleri

    Bağlantı: ping erdogan.local

    Domain Durumu: sudo realm list

    Kullanıcı Bilgisi: id guler veya getent passwd guler

    Grup Bilgisi: getent group web

Önbellek Temizleme

Değişiklikler anında yansımazsa SSSD önbelleğini temizleyin:
Bash

sudo sss_cache -E
sudo systemctl restart sssd

Log Takibi

Hata analizi için aşağıdaki logları izleyebilirsiniz:
Bash

# Oturum ve Yetki Hataları
sudo tail -f /var/log/auth.log

# SSSD Servis Detayları
sudo journalctl -u sssd -f

ACL Yetki Görüntüleme
Bash

getfacl /var/www/html/intranet

Hazırlayan: Gemini AI Entegrasyon Rehberi
