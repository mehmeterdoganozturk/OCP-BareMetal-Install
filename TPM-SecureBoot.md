# 🛡️ Lenovo P16s Fedora Güvenlik, TPM & NVIDIA Yapılandırma Rehberi (Eksiksiz)

Bu döküman; Lenovo P16s cihazında Secure Boot aktivasyonu, LUKS disk şifrelemesinin TPM 2.0 ile otomatize edilmesi ve NVIDIA sürücülerinin imzalanması süreçlerini içeren nihai rehberdir.

---

## 1. BIOS ve Donanım Yapılandırması
Sistemin Fedora imzalarını tanıması ve ileri düzey güvenlik protokollerini çalıştırması için BIOS'ta şu ayarlar yapılmıştır:
* **Secure Boot:** `Enabled`.
* **Third-Party Destek:** `Allow Microsoft Third-Party UEFI CA` seçeneği **ON** konumuna getirildi (Linux bootloader'ı için zorunludur).
* **Sertifika Yönetimi:** `Restore Factory Keys` yapılarak sistem "User Mode" seviyesine taşındı.
* **Rollback Protection:** `Secure Rollback Prevention` ayarı BIOS üzerinden `Enabled` yapıldı (Sistem yazılımının eski sürüme düşürülerek açık yaratılmasını engeller).

## 2. TPM 2.0 ile Otomatik Disk Kilidi Açma (LUKS)
Şifreli ana bölümün (`/dev/nvme0n1p3`), açılışta şifre sormadan TPM çipi üzerinden otomatik çözülmesi sağlanmıştır.

### Uygulanan Adımlar:
1.  **MOK Kaydı:** BIOS ve TPM güvenliği için Machine Owner Key (MOK) yapısı kuruldu.
2.  **TPM Kaydı (PCR 0+2+7):** ```bash
    sudo systemd-cryptenroll --tpm2-device=auto --tpm2-pcrs=0+2+7 /dev/nvme0n1p3
    ```
    *Bu komutla PCR 0 (BIOS), 2 (Donanım) ve 7 (Secure Boot) durumları mühürlenmiştir.*
3.  **Yapılandırma:** `/etc/crypttab` dosyasına `tpm2-device=auto` parametresi eklendi.
4.  **Initramfs Yenileme:** Değişikliklerin başlangıçta tanınması için imaj tazelendi:
    ```bash
    sudo dracut -f --regenerate-all
    ```

## 3. NVIDIA Sürücü İmzalama (Secure Boot Uyumu)
Secure Boot aktifken NVIDIA sürücülerinin yüklenmesi için izlenen imzalama süreci:

1.  **Anahtar Yönetimi:** `sudo kmodgenca -a` ile sistem içi imza anahtarları oluşturuldu.
2.  **MOK İçe Aktarma:** `sudo mokutil --import /etc/pki/akmods/certs/public_key.der` ile anahtar BIOS'a gönderildi.
3.  **Boot Onayı:** Yeniden başlatma sırasında mavi ekranda (MOK Management) **"Enroll MOK"** adımları tamamlandı.
4.  **Sürücü Derleme:** `sudo akmods --force` ile sürücüler yeni anahtarla imzalandı ve `nvidia-smi` çıktısı başarıyla alındı.

## 4. Güvenlik Seviyesi ve Analiz (HSI-2)
Sistem şu an **HSI-2!** güvenlik seviyesindedir.

* **✔ Başarılı Testler:** Secure Boot, TPM v2.0, IOMMU, Intel BootGuard, BIOS Rollback Protection, Kernel Lockdown.
* **⚠ Tainted Kernel:** NVIDIA kapalı kaynaklı olduğu için kernel "Tainted" (lekelenmiş) olarak görünür; bu teknik bir durumdur, güvenlik açığı değildir.
* **ℹ HSI-3/4 Notu:** `Encrypted RAM` donanımsal destek gerektirdiği, `Suspend-to-idle` ise stabiliteyi bozabileceği için bu seviyeler zorunlu tutulmamıştır.

---

## ⚠️ Kritik Hatırlatmalar
1.  **BIOS Yazma Hatası:** `failed to write bytes to 14` hatası, Secure Boot/Lockdown modunda Linux'un BIOS ayarlarına doğrudan müdahale etmesinin engellenmesidir. Bu tür ayarlar (Rollback vb.) manuel olarak BIOS ekranından yapılmalıdır.
2.  **Kernel Güncellemeleri:** `akmods` servisi sayesinde kernel güncellense bile sürücüler otomatik olarak imzalanacaktır.
3.  **Kurtarma:** TPM doğrulamasının başarısız olduğu durumlarda (donanım değişikliği vb.) sistem manuel LUKS şifrenizle açılabilir.

---
*Döküman Tarihi: 27 Şubat 2026*
