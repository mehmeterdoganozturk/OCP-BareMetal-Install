# GitLab Server Kurulum Rehberi (Ubuntu 24.04)

Bu doküman Ubuntu 24.04 Server üzerine GitLab EE (Enterprise Edition) kurulum süreçlerini açıklamaktadır. Aşağıdaki adımları takip ederek kendi GitLab sunucunuzu başarıyla devreye alabilirsiniz.

---

## 1. Sistem Güncelleme

İlk olarak mevcut paketleri güncelleyin.

```bash
sudo apt update && sudo apt upgrade -y
```

---

## 2. Gerekli Paketlerin Kurulumu

GitLab için ihtiyaç duyulan temel paketleri yükleyin.

```bash
sudo apt install -y curl openssh-server ca-certificates tzdata perl postfix
```

`postfix` kurulumu esnasında sunucu yapılandırması sorulabilir. Sunucunuz dış dünyaya mail göndermeyecekse "Local only" yapılandırması uygundur.

---

## 3. GitLab Repository Ekleme

GitLab EE deposunu sunucuya ekleyin.

```bash
curl -fsSL https://packages.gitlab.com/install/repositories/gitlab/gitlab-ee/script.deb.sh | sudo bash
```

---

## 4. GitLab Kurulumu

Kurulumdan önce `EXTERNAL_URL` parametresine erişim sağlayacağınız IP adresini veya domain adını ekleyin.

### Örnek IP ile Kurulum
```bash
sudo EXTERNAL_URL="http://192.168.1.100" apt-get install gitlab-ee
```

### Örnek Domain ile Kurulum
```bash
sudo EXTERNAL_URL="http://gitlab.sirketim.com" apt-get install gitlab-ee
```

Kurulum tamamlandığında otomatik olarak yeniden yapılandırma yapılacaktır.

---

## 5. İlk Root Şifresini Öğrenme

Kurulum sonrası GitLab ilk admin şifresini aşağıdaki dosyadan alabilirsiniz:

```bash
sudo cat /etc/gitlab/initial_root_password
```

Bu dosya yalnızca belirli bir süre saklanır. Görür görmez şifreyi not etmeyi unutmayın.

---

## 6. GitLab Web Arayüzüne Erişim

Tarayıcı üzerinden erişim:

```
http://sunucu_ip_adresiniz_veya_domain
```

Giriş bilgileri:

| Kullanıcı | Parola |
|----------|--------|
| `root`   | initial_root_password dosyasındaki şifre |

---

## 7. Ek Yapılandırmalar ve Öneriler

| Yapılandırma | Açıklama |
|-------------|----------|
| SSL/HTTPS kurulumu | Let’s Encrypt veya kendi sertifikalarınız ile aktif edilebilir. |
| Yedekleme | `/etc/gitlab/gitlab.rb` ve `/var/opt/gitlab/backups` dizinlerini düzenli yedekleyin. |
| GitLab Omnibus Config | Ana yapılandırma dosyası `/etc/gitlab/gitlab.rb` yolundadır. |

Yapılandırma değiştirdikten sonra yeniden yapılandırma komutu:

```bash
sudo gitlab-ctl reconfigure
```

---

## Kurulum Tamamlandı

Artık GitLab sunucunuz çalışmaktadır. Projelerinizi barındırabilir, CI/CD süreçleri oluşturabilir ve kullanıcı yönetimi yapabilirsiniz.

---

Hazırlayan: Mehmet Erdoğan Öztürk
