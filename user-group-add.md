### **Kullanıcı ve Grup Yönetimi Komutları**

#### **1. Grup Oluşturma**
```bash
groupadd sysadm
```
- **sysadm** adında yeni bir grup oluşturulur.

#### **2. Kullanıcıları Gruba Ekleyerek Oluşturma**
```bash
useradd -G sysadm harry
useradd -G sysadm natasha
```
- **harry** ve **natasha** kullanıcıları oluşturulur ve **sysadm** grubuna eklenir.

#### **3. Giriş Yapamaz Kullanıcı Oluşturma**
```bash
useradd -s /sbin/nologin sarah
```
- **sarah** adında bir kullanıcı oluşturulur.
- `/sbin/nologin` ayarı sayesinde **sisteme giriş yapamaz**.

#### **4. Kullanıcılara Şifre Belirleme**
```bash
passwd sarah
passwd natasha
passwd sarah
```
- Kullanıcıların şifrelerini belirlemek için kullanılır.

#### **5. Komutların Konumlarını Bulma**
```bash
which useradd
which passwd
```
- `which useradd` çıktısı: `/usr/sbin/useradd`
- `which passwd` çıktısı: `/usr/bin/passwd`

---

### **Sudo Yetkilendirme İşlemleri (visudo)**
```bash
visudo
```
- **/etc/sudoers** dosyasını düzenlemek için kullanılır.

#### **sudoers İçerisindeki Yetkilendirmeler**
```bash
%sysadm      ALL=/usr/sbin/useradd
```
- **sysadm** grubundaki kullanıcılar, **sadece** `useradd` komutunu `sudo` ile çalıştırabilir.

```bash
harry        ALL=(ALL)        NOPASSWD: /usr/bin/passwd
```
- **harry**, `passwd` komutunu `sudo` ile **şifre girmeden** çalıştırabilir.

```bash
harry        ALL=ALL
```
- **harry**, `sudo` ile **tüm komutları çalıştırabilir**, ancak şifre girmesi gerekir.

```bash
harry ALL=(ALL)        NOPASSWD: ALL
```
- **harry**, `sudo` ile **şifre sormadan tüm komutları çalıştırabilir**.

---

### **Özet**
1. **sysadm grubu** oluşturuldu ve içine **harry ile natasha** eklendi.
2. **sarah** kullanıcısı oluşturuldu ve sisteme giriş yapamaz hale getirildi.
3. Kullanıcılara şifre atandı.
4. `visudo` ile sudo yetkilendirmeleri yapıldı:
   - **sysadm grubundaki kullanıcılar** yalnızca `useradd` komutunu sudo ile çalıştırabilir.
   - **harry**, `passwd` komutunu şifresiz çalıştırabilir.
   - **harry**'nin önce tüm sudo yetkileri verilmiş, sonra tüm komutları şifresiz çalıştırma yetkisi de eklenmiş.

Bu ayarlarla **harry** tam yetkili bir yönetici olmuş oluyor. Ancak **sysadm grubu** sadece yeni kullanıcılar ekleyebilir.

