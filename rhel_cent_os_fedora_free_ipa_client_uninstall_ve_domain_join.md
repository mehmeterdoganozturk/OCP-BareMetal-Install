# RHEL / CentOS / Fedora
## FreeIPA Client Uninstall ve Domain Join Dökümanı

Bu doküman **Red Hat Enterprise Linux, CentOS Stream ve Fedora** sistemlerinde **FreeIPA Client** kaldırma (uninstall) ve yeniden **domain join** işlemleri için hazırlanmıştır. Ubuntu/Debian adımları özellikle dışarıda bırakılmıştır.

---

## 1. Mevcut IPA Client ve Domain Temizliği (Uninstall)

> Amaç: Sistemi tamamen domain'den ayırmak ve temiz bir yeniden katılım (join) için hazırlamak.

```bash
# IPA client uninstall
sudo ipa-client-install --uninstall
```

```bash
# Realm'den çıkış
sudo realm leave yargitay.gov.tr
```

```bash
# Paketlerin kaldırılması
sudo dnf remove -y ipa-client realmd sssd sssd-tools adcli oddjob oddjob-mkhomedir samba-common-tools
```

```bash
# Artık dosyaların temizlenmesi
sudo rm -f /etc/sssd/sssd.conf
sudo rm -f /etc/krb5.keytab
sudo rm -rf /var/lib/sss/db/*
```

```bash
# Servisleri durdur
sudo systemctl stop sssd
```

---

## 2. Network ve DNS Kontrolleri

FreeIPA ve Kerberos için **DNS ve zaman senkronizasyonu kritiktir**.

### DNS Kontrol
```bash
resolvectl status
```

> IPA/AD DNS sunucuları görünmelidir.

### Zaman Kontrolü
```bash
timedatectl
```

> `System clock synchronized: yes` olmalıdır.

Gerekirse:
```bash
sudo dnf install -y chrony
sudo systemctl enable --now chronyd
```

---

## 3. Gerekli Paketlerin Kurulumu

```bash
sudo dnf install -y \
realmd \
sssd \
sssd-tools \
adcli \
oddjob \
oddjob-mkhomedir \
samba-common-tools \
krb5-workstation \
policycoreutils-python-utils \
acl
```

---

## 4. Domain Join (IPA Client Kurulumu)

### Standart IPA Client Join

```bash
sudo ipa-client-install \
  --domain=yargitay.gov.tr \
  --realm=YARGITAY.GOV.TR \
  --server=ipa.yargitay.gov.tr \
  --mkhomedir \
  --principal=yr503373
```

> Kurulum sırasında parola istenir.

### Alternatif: realmd ile Join

```bash
sudo realm join -v yargitay.gov.tr -U yr503373
```

---

## 5. SSSD Yapılandırması (Kısa Kullanıcı Adı)

Kullanıcıların `user@domain` yerine **sadece `user`** ile giriş yapabilmesi için:

```bash
sudo nano /etc/sssd/sssd.conf
```

```ini
[sssd]
domains = yargitay.gov.tr
config_file_version = 2
services = nss, pam

[domain/yargitay.gov.tr]
id_provider = ipa
ipa_domain = yargitay.gov.tr
ipa_server = ipa.yargitay.gov.tr
ipa_hostname = $(hostname -f)

use_fully_qualified_names = False
fallback_homedir = /home/%u
default_shell = /bin/bash
cache_credentials = True
```

```bash
sudo chmod 600 /etc/sssd/sssd.conf
sudo systemctl restart sssd
```

---

## 6. NSS Ayarları Kontrolü

```bash
sudo nano /etc/nsswitch.conf
```

Aşağıdaki satırlar **sss** içermelidir:

```
passwd:     files sss
group:      files sss
shadow:     files sss
services:   files sss
netgroup:   files sss
```

---

## 7. Home Dizini Otomatik Oluşturma

```bash
sudo systemctl enable --now oddjobd
sudo authselect enable-feature with-mkhomedir
sudo authselect apply-changes
```

---

## 8. Domain Erişim Kontrolleri (SSH Yetkileri)

### Tüm Domain Kullanıcılarını Engelle
```bash
sudo realm deny --all
```

### Sadece Belirli Gruplara İzin Ver
```bash
sudo realm permit -g gg_server
sudo realm permit -g gg_web_server
sudo realm permit -g ycb_sistem
```

---

## 9. Sudo Yetkileri (Domain Grupları)

```bash
sudo nano /etc/sudoers.d/domain_admins
```

```sudoers
%gg_server ALL=(ALL:ALL) ALL
%gg_web_server ALL=(ALL:ALL) ALL
```

> Dosya izni:
```bash
sudo chmod 440 /etc/sudoers.d/domain_admins
```

---

## 10. Kontroller ve Testler

### Domain Bilgisi
```bash
realm list
```

### Kullanıcı Bilgisi
```bash
id srv3894
getent passwd srv3894
```

### Kullanıcıya Geçiş
```bash
su - srv3894
```

### Sudo Yetkisi
```bash
sudo -l -U srv3894
```

---

## 11. Cache ve Servis Yenileme

```bash
sudo sss_cache -E
sudo systemctl restart sssd
```

---

## 12. Log ve Troubleshooting

```bash
ping -c 5 yargitay.gov.tr
```

```bash
journalctl -u sssd -f
```

```bash
tail -f /var/log/secure
```

---

**Bu doküman yalnızca RHEL / CentOS / Fedora sistemleri için hazırlanmıştır.**

