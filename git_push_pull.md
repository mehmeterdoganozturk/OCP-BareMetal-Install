# Git Repository Oluşturma ve GitLab'a Gönderme Rehberi

Bu doküman, **yerel bir Spring Boot (veya genel bir yazılım) projesinin Git ile versiyonlanıp GitLab'a gönderilmesi** sürecinde kullanılan temel komutları **adım adım ve açıklamalı** olarak açıklar. Aşağıdaki açıklamalar, projeyi ilk kez Git'e alırken izlenen tipik ve doğru akışı temel alır.

---

## 🎯 Amaç

- Yerel bir projeyi Git repository haline getirmek
- İlk commit'i oluşturmak
- GitLab üzerindeki uzak repository ile bağlantı kurmak
- Projeyi `main` branch'i üzerinden GitLab'a göndermek

---

## 1️⃣ `git init`

```bash
git init
```

**Açıklama:**  
Bulunduğunuz dizini bir **Git repository** haline getirir.

**Teknik detay:**
- `.git/` adlı gizli dizini oluşturur
- Commit geçmişi, branch bilgileri ve konfigürasyonlar burada tutulur

> Bu komut proje başına **yalnızca bir kez** çalıştırılır.

---

## 2️⃣ `git add .`

```bash
git add .
```

**Açıklama:**  
Mevcut dizindeki tüm dosyaları **staging area**'ya ekler.

**Önemli noktalar:**
- `.gitignore` dosyasındaki kurallara uyar
- `target/`, `node_modules/` gibi dizinler ignore ediliyorsa eklenmez

Kontrol için:
```bash
git status
```

---

## 3️⃣ `git commit -m "deneme"`

```bash
git commit -m "deneme"
```

**Açıklama:**  
Staging area'daki dosyaları kalıcı bir **commit** haline getirir.

**Notlar:**
- Commit, projenin o anki durumunun bir anlık görüntüsüdür
- Gerçek projelerde commit mesajı daha açıklayıcı olmalıdır

Örnek:
```bash
git commit -m "Initial project commit"
```

---

## 4️⃣ `git branch`

```bash
git branch
```

**Açıklama:**  
Mevcut branch'leri listeler.

**Beklenen durum:**
- İlk commit sonrası genellikle `master` branch'i görünür

---

## 5️⃣ `git remote add origin <REPO_URL>`

```bash
git remote add origin http://gitlab.ocplab.yargitay.gov.tr/web/otag-spring.git
```

**Açıklama:**  
Yerel repository ile GitLab üzerindeki uzak repository arasında bağlantı kurar.

**Detay:**
- `origin` uzak repository için kullanılan varsayılan takma addır
- Bu komut veri göndermez, sadece bağlantıyı tanımlar

---

## 6️⃣ `git branch -M main`

```bash
git branch -M main
```

**Açıklama:**  
Mevcut branch'in adını zorlayarak (`-M`) `main` olarak değiştirir.

**Neden gerekli?**
- GitLab varsayılan branch olarak `main` kullanır
- `master` / `main` uyumsuzluğunu önler

---

## 7️⃣ `git push -uf origin main`

```bash
git push -uf origin main
```

**Açıklama:**  
Yerel `main` branch'indeki commit'leri GitLab'a gönderir.

### Parametreler:
- `-u` : Yerel branch ile uzak branch arasında **upstream** ilişkisi kurar
- `-f` : **Force push** yapar, uzak branch'i zorla ezer

⚠ **Uyarı:**  
`-f` parametresi, uzak repodaki commit geçmişini silebilir.  
Sadece **ilk push** veya **boş repo** durumlarında kullanılmalıdır.

**Güvenli alternatif:**
```bash
git push -u origin main
```

---

## 8️⃣ `git remote -v`

```bash
git remote -v
```

**Açıklama:**  
Tanımlı uzak repository'leri ve URL'lerini listeler.

**Örnek çıktı:**
```text
origin  http://gitlab.ocplab.yargitay.gov.tr/web/otag-spring.git (fetch)
origin  http://gitlab.ocplab.yargitay.gov.tr/web/otag-spring.git (push)
```

---

## ✅ Önerilen Doğru Akış (Özet)

```bash
git init
git add .
git commit -m "Initial commit"
git branch -M main
git remote add origin <REPO_URL>
git push -u origin main
```

---

## 📌 Notlar

- `.gitignore` dosyası ilk `git add` öncesinde oluşturulmalıdır
- `force push (-f)` ekip çalışmalarında **kullanılmamalıdır**
- İlk push sonrası günlük kullanım için sadece `git add`, `git commit`, `git push` yeterlidir

---

## 👤 Hazırlayan

Mehmet Erdoğan Öztürk

