# 🛡️ Lenovo P16s Fedora Güvenlik ve TPM Yapılandırma Rehberi

Bu döküman, Lenovo P16s cihazında Secure Boot aktivasyonu ve LUKS disk şifrelemesinin TPM 2.0 ile otomatize edilmesi sürecini özetler.

---

## 1. Donanım ve BIOS Yapılandırması
Sistemin Fedora imzalarını tanıması ve güvenli önyükleme yapabilmesi için BIOS üzerinde şu adımlar uygulanmıştır:

* **Secure Boot:** `Enabled` konuma getirildi.
* **Third-Party Destek:** `Allow Microsoft Third-Party UEFI CA` seçeneği **ON** yapıldı.
* **Sertifika Yönetimi:** `Restore Factory Keys` yapılarak sistem "User Mode" seviyesine getirildi.

## 2. Yazılımsal Kontrol ve Doğrulama
İşletim sistemi seviyesinde güvenliğin sağlandığı terminal üzerinden teyit edilmiştir:

* **Durum Sorgulama:** `mokutil --sb-state` komutu ile Secure Boot'un işletim sistemi tarafından tanındığı doğrulandı.
* **Görsel Teyit:** GNOME Ayarları > Gizlilik ve Güvenlik ekranında "Secure Boot is Active" ve "Linux Kernel Verification" ibarelerinin **Yeşil** olduğu görüldü.

## 3. TPM 2.0 ile Otomatik Disk Kilidi Açma
`lsblk` çıktısına göre `/dev/nvme0n1p3` üzerinde bulunan şifreli bölümün, her açılışta şifre sormadan TPM çipi üzerinden açılması sağlanmıştır.

### Uygulanan Komutlar:
1.  **TPM Kaydı (PCR 0+2+7):** ```bash
    sudo systemd-cryptenroll --tpm2-device=auto --tpm2-pcrs=0+2+7 /dev/nvme0n1p3
    ```
    *Bu komutla "New TPM2 token enrolled as key slot 2" onayı alınmıştır.*

2.  **Yapılandırma Dosyası:** `/etc/crypttab` dosyasına `tpm2-device=auto` parametresi eklenerek sistemin açılışta TPM'i sorgulaması talimatı verilmiştir.

3.  **Başlangıç İmajı Yenileme:** Değişikliklerin kernel seviyesinde tanınması için dracut çalıştırılmıştır:
    ```bash
    sudo dracut -f --regenerate-all
    ```

---

## ⚠️ Önemli Notlar ve Bakım

* **Eski Kayıtlar:** Güvenlik ekranında alt kısımda görünen kırmızı çarpı işaretleri (2025 tarihli), sistemin geçmişindeki başarısız denemelerin log kaydıdır. Güncel durum (2026) yeşil kalkanla korunmaktadır.
* **Kurtarma:** Eğer BIOS güncellemesi sonrası sistem şifre isterse, manuel şifreni girip sistem açıldıktan sonra `dracut` komutunu tekrar çalıştırman yeterlidir.
* **Bluetooth Uyarısı:** `dracut` sırasında görülen Bluetooth mesajı bir hata değildir, dizüstü bilgisayarlar için standart bir bilgilendirmedir.

---
*Son Güncelleme: 27 Şubat 2026*
