# ArkSigner Fedora Kurulum Notları
**Debian .deb paketinden Fedora’ya manuel kurulum**

## 1️⃣ Debian Paketini Aç

```
ar x arksigner-pub-2.3.12.deb
tar -xf data.tar.xz
```

## 2️⃣ Dosyaları Uygun Dizinlere Kopyala

```
sudo mkdir -p /opt/arksigner
sudo cp -r ~/Downloads/data/usr/bin/arksigner/* /opt/arksigner/
sudo cp -r ~/Downloads/data/etc/init.d/arksignerd /opt/arksigner/
```

## 3️⃣ PATH’e Sembolik Link Eklemek (Opsiyonel)

```
sudo ln -s /opt/arksigner/arksigner-universal /usr/local/bin/arksigner-universal
sudo ln -s /opt/arksigner/arksigner-service /usr/local/bin/arksigner-service
```

## 4️⃣ Systemd Servisi Oluştur

```
sudo nano /etc/systemd/system/arksigner.service
```

İçerik:

```
[Unit]
Description=ArkSigner Service
After=network.target pcscd.service
Wants=pcscd.service

[Service]
Type=simple
ExecStart=/opt/arksigner/arksigner-service
Restart=on-failure
User=root
Group=root

[Install]
WantedBy=multi-user.target
```

## 5️⃣ Servisi Başlat ve Etkinleştir

```
sudo systemctl daemon-reload
sudo systemctl enable --now arksigner.service
sudo systemctl status arksigner.service
```

## 6️⃣ Test Et

```
/opt/arksigner/arksigner-universal
```
