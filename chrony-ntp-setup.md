
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
