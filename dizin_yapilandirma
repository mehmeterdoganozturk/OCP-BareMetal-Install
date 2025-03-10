# Paylaşılan Dizin Yapılandırma Kılavuzu

Bu doküman, `/shared/sysadm/` dizininin nasıl oluşturulacağı, izinlerinin nasıl ayarlanacağı ve nasıl kullanılacağına dair detaylı adımları içermektedir.

## 1. Dizin Oluşturma
Aşağıdaki komut ile `/shared/sysadm/` dizini oluşturulur:

```bash
mkdir -p /shared/sysadm
```
- `-p` seçeneği, üst dizin (`/shared`) yoksa onu da oluşturur.

## 2. Grup Sahipliğini Değiştirme
`sysadm` grubunun bu dizine sahip olması için aşağıdaki komut kullanılır:

```bash
chgrp sysadm /shared/sysadm/
```

## 3. İzinleri Ayarlama
Aşağıdaki komut ile dizinin izinleri ayarlanır:

```bash
chmod 2770 /shared/sysadm/
```
Bu komut:
- **`2` (SGID - Set Group ID)**: Bu dizine eklenen dosyaların grup sahipliği otomatik olarak `sysadm` olur.
- **`770` (rwxrwx---)**:
  - **Sahibi (owner)**: `rwx` (okuma, yazma, çalıştırma)
  - **Grubu (group)**: `rwx` (okuma, yazma, çalıştırma)
  - **Diğerleri (others)**: `---` (erişim yok)

## 4. İzinleri Kontrol Etme
Aşağıdaki komut ile dizinin izinleri görüntülenebilir:

```bash
ls -ld /shared/sysadm/
```
Örnek çıktı:

```bash
drwxrws--- 2 root sysadm 4096 Mar 10 12:34 /shared/sysadm/
```
- **`d`** → Dizin olduğunu gösterir.
- **`rwxrws---`** →
  - `rwx` (sahibi = root)
  - `rws` (grup = sysadm, SGID aktif)
  - `---` (diğerleri erişemez)
- **`root sysadm`** → Sahip `root`, grup `sysadm`.

## 5. Kullanıcıları Gruba Ekleme
`sysadm` grubundaki kullanıcılar bu dizini tam yetkiyle kullanabilir. Kullanıcı eklemek için:

```bash
usermod -aG sysadm kullanici_adi
```

Bu yapılandırma sayesinde `/shared/sysadm/` dizini güvenli bir şekilde grup tarafından paylaşılabilir. Başka konular hakkında da bilgi almak isterseniz bana ulaşabilirsiniz! 🚀
