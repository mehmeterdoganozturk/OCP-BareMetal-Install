# 🛡️ Lenovo P16s Fedora Güvenlik, TPM & NVIDIA Yapılandırma Rehberi (Final)

Bu döküman; Secure Boot aktivasyonu, LUKS disk şifrelemesinin TPM 2.0 ile otomatize edilmesi ve NVIDIA sürücülerinin imzalanması süreçlerini kapsar.

---

## 1. Donanım ve BIOS Yapılandırması
Sistemin Fedora imzalarını tanıması için BIOS üzerinde şu adımlar uygulanmıştır:
* **Secure Boot:** `Enabled` konuma getirildi.
* **Third-Party Destek:** `Allow Microsoft Third-Party UEFI CA` seçeneği **ON** yapıldı.
* **Sertifika Yönetimi:** `Restore Factory Keys` yapılarak sistem "User Mode" seviyesine getirildi.
* **Rollback Protection:** `Secure Rollback Prevention` ayarı BIOS üzerinden `Enabled` yapılarak eski sürüm açıklarına karşı koruma sağlandı.

## 2. TPM 2.0 ile Otomatik Disk Kilidi Açma (LUKS)
Şifreli ana bölümün (`/dev/nvme0n1p3`), açılışta şifre sormadan TPM çipi üzerinden otomatik çözülmesi sağlanmıştır.

### Uygulanan İşlemler:
1.  **MOK Kaydı:** BIOS/TPM güvenliği için MOK anahtarı oluşturuldu.
2.  **TPM Kaydı (PCR 0+2+7):** ```bash
    sudo systemd-cryptenroll --tpm2-device=auto --tpm2-pcrs=0+2+7 /dev/nvme0n1p3
    ```
3.  **Yapılandırma:** `/etc/crypttab` dosyasına `tpm2-device=auto` parametresi eklendi.
4.  **Initramfs Yenileme:** `sudo dracut -f --regenerate-all` ile değişiklikler kernel seviyesine işlendi.

## 3. NVIDIA Sürücü İmzalama (Secure Boot Uyumu)
Secure Boot aktifken NVIDIA sürücülerinin çalışması için "Machine Owner Key" (MOK) süreci tamamlanmıştır:

1.  **Anahtar Oluşturma:** `sudo kmodgenca -a` ile imza anahtarları hazırlandı.
2.  **MOK İçe Aktarma:** `sudo mokutil --import /etc/pki/akmods/certs/public_key.der` komutu ile anahtar BIOS'a gönderildi.
3.  **Mavi Ekran (MOK Management):** Reboot sonrası "Enroll MOK" adımları tamamlanarak anahtar doğrulandı.
4.  **Sürücü Derleme:** `sudo akmods --force` ile sürücüler yeni anahtarla imzalandı.

## 4. Mevcut Güvenlik Durumu Doğrulaması
* **`mokutil --sb-state`:** "SecureBoot enabled" olarak teyit edildi.
* **`nvidia-smi`:** Ekran kartının aktif olduğu ve çekirdek ile iletişim kurduğu doğrulandı.
* **`fwupdmgr security`:** HSI-3 seviyesine kadar tüm kritik güvenlik kontrolleri (TPM, IOMMU, BootGuard vb.) başarıyla geçildi.

---

## ⚠️ Bakım ve Hatırlatmalar
* **Kernel Güncellemeleri:** `akmods` sayesinde yeni kernel güncellemelerinde sürücüler otomatik olarak imzalanmaya devam edecektir.
* **Tainted Kernel:** Nvidia kapalı kaynaklı olduğu için sistemde "Tainted" uyarısı görülmesi normaldir, bir hata değildir.
* **Kurtarma:** Sistem bir sebeple TPM üzerinden açılmazsa, kurulumda belirlediğiniz manuel LUKS şifrenizle her zaman giriş yapabilirsiniz.

---
*Son Güncelleme: 27 Şubat 2026*
