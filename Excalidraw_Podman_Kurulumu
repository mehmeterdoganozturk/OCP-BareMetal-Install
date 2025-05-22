# 📦 Excalidraw Podman Kurulumu (Fedora 42)

Bu döküman, Fedora 42 üzerinde Excalidraw'ı Podman kullanarak nasıl kuracağınızı, konteyneri sistem açılışında otomatik başlatmayı ve masaüstü için `.desktop` kısayolu oluşturmayı adım adım açıklar.

---

## 📁 1. Excalidraw’ı Klonla

```bash
cd ~/Downloads
git clone https://github.com/excalidraw/excalidraw.git
cd excalidraw
```

---

## ⚙️ 2. Web Uygulamasını Derle

```bash
cd excalidraw-app
npm install
npm run build
```

> Derleme tamamlandığında `build/` klasörü oluşur.

---

## 📦 3. Podman için Dockerfile ve Klasör Hazırla

```bash
cd ..
mkdir excalidraw-container
cp -r excalidraw-app/build excalidraw-container/
cd excalidraw-container
```

### `Dockerfile` oluştur:

```Dockerfile
# excalidraw-container/Dockerfile
FROM nginx:alpine
COPY build/ /usr/share/nginx/html
EXPOSE 80
```

---

## 🔧 4. Podman İmajını Oluştur ve Konteyneri Başlat

```bash
podman build -t excalidraw:latest .
podman run -d --name excalidraw-app -p 8080:80 localhost/excalidraw:latest
```

---

## 🔁 5. Konteyneri Sistem Başlangıcında Otomatik Başlat

### A. systemd servis dosyası oluştur:

```bash
podman generate systemd --name excalidraw-app --files --new
```

### B. systemd kullanıcı dizinine kopyala:

```bash
mkdir -p ~/.config/systemd/user
cp container-excalidraw-app.service ~/.config/systemd/user/
```

### C. Servisi etkinleştir:

```bash
systemctl --user daemon-reexec
systemctl --user daemon-reload
systemctl --user enable --now container-excalidraw-app.service
```

### D. Linger özelliğini aktif et (gerekirse):

```bash
loginctl enable-linger $USER
```

---

## 🖥️ 6. Masaüstü Uygulama Kısayolu Oluştur

```bash
nano ~/.local/share/applications/excalidraw.desktop
```

### İçerik:

```ini
[Desktop Entry]
Name=Excalidraw (Podman)
Exec=xdg-open http://localhost:8080
Icon=utilities-terminal
Type=Application
Categories=Utility;
StartupNotify=false
```

> Dilersen `Icon=` satırına bir PNG dosya yolu ekleyebilirsin: `Icon=/home/kullaniciadi/Downloads/excalidraw-icon.png`

### Uygulama olarak tanıt:

```bash
chmod +x ~/.local/share/applications/excalidraw.desktop
```

---

## ✅ Test Et

- Bilgisayarı yeniden başlat.
- Aşağıdaki komutla konteynerin çalıştığını doğrula:

```bash
podman ps
```

- Uygulama menüsünden **Excalidraw (Podman)** girişini kullanarak uygulamayı aç.

---

## 🎉 Hazırsın!

Artık Excalidraw uygulaman Podman ile çalışıyor, sistem açıldığında otomatik başlıyor ve masaüstü/uygulama menüsünden erişilebiliyor.
