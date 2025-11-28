
# 🛡️ FreeIPA Client Kurulumu - Yargıtay İçin

Bu döküman, bir Ubuntu sunucusunun FreeIPA altyapısına dahil edilmesi için yapılan adımları içermektedir. Sistem yapılandırmaları, zaman senkronizasyonu ve istemci kurulumu detaylı şekilde aşağıda açıklanmıştır.

---

## 📦 1. Sistem Güncellemeleri ve Yeni Kullanıcı Oluşturulması

```bash
sudo apt-get update -y && sudo apt-get upgrade -y && sudo apt-get dist-upgrade -y && sudo apt autoremove -y
```

### 👤 Yeni kullanıcı oluşturma

```bash
sudo adduser yrg_sistem
sudo usermod -aG sudo yrg_sistem
```

---

## 🖥️ 2. Hostname ve Hosts Dosyası Ayarları

### 🏷️ Hostname Ayarı

```bash
sudo hostnamectl set-hostname postgrescluster-01.yargitay.gov.tr
```

### 🗂️ /etc/hosts Dosyasını Düzenle

```bash
sudo nano /etc/hosts
```

İçerik şu şekilde olmalıdır:

```
127.0.0.1       localhost
127.0.1.1       postgrescluster-01.yargitay.gov.tr
```

---

## 🕒 3. Zaman Dilimi ve NTP (Chrony) Ayarları

### 🕰️ Zaman Dilimi Ayarı

```bash
timedatectl
ls -l /etc/localtime
sudo timedatectl set-timezone Europe/Istanbul
```

### ⏲️ Chrony Kurulumu ve Yapılandırması

```bash
sudo apt-get install chrony -y
sudo nano /etc/chrony/chrony.conf
```

`/etc/chrony/chrony.conf` dosyasına şu satırları ekleyin veya aşağıdaki şekilde düzenleyin:

```
# Default Ubuntu havuzları devre dışı bırakılır
#pool ntp.ubuntu.com        iburst maxsources 4
#pool 0.ubuntu.pool.ntp.org iburst maxsources 1
#pool 1.ubuntu.pool.ntp.org iburst maxsources 1
#pool 2.ubuntu.pool.ntp.org iburst maxsources 2

# Kurum içi NTP sunucusu kullanılır
server ntp.yargitay.gov.tr
```

---

## 🔧 4. FreeIPA Client Kurulumu

### 📥 Paket Kurulumu

```bash
sudo apt-get install -y freeipa-client
```

### 🔐 Client Domain’e Katma

```bash
sudo ipa-client-install --mkhomedir \
  --server=ipa.yargitay.gov.tr \
  --domain=yargitay.gov.tr \
  --realm=YARGITAY.GOV.TR
```

Kurulum sırasında gelecek sorulara aşağıdaki gibi yanıt verin:

- **DNS discovery olmadan devam edilsin mi?** → `yes`  
- **Chrony yapılandırılsın mı?** → `no`  
- **Yapılandırma onaylansın mı?** → `yes`  
- **Kullanıcı adı (admin) ve şifre** girilir.

Başarılı çıktı örneği:

```
Enrolled in IPA realm YARGITAY.GOV.TR
Configured /etc/sssd/sssd.conf
Configured /etc/krb5.conf for IPA realm
Systemwide CA database updated
SSSD enabled
Client configuration complete.
The ipa-client-install command was successful
```

---

## 🔄 5. Sistem Yeniden Başlatma

```bash
sudo reboot
```

---

## 🧹 6. Geri Alma (Uninstall)

FreeIPA istemcisi sistemden tamamen kaldırmak için:

```bash
sudo ipa-client-install --uninstall -U
sudo apt remove freeipa-client
```

---

## 🔍 7. Test ve Durum Kontrolü

Kurulumdan sonra bağlantıyı test etmek için:

```bash
ipa service-find
```

---

> 📘 Bu döküman sadece FreeIPA istemci kurulumu ile ilgilidir. Sunucu kurulumu, kullanıcı ve grup yönetimi, sudo politikaları gibi diğer konular ayrı başlıklarda ele alınmalıdır.
