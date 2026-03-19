# 🛡️ Lenovo P16s Fedora Güvenlik ve Donanım Yapılandırma Rehberi

Bu rehber, Lenovo P16s üzerinde Secure Boot aktivasyonu, TPM 2.0 ile otomatik disk açma ve NVIDIA sürücü imzalama süreçlerini özetler.

---

## 1. BIOS Yapılandırması

Sistemin Fedora imzalarını tanıması için BIOS'ta şu adımlar uygulanmıştır:

- **Secure Boot:** Enabled  
- **Allow Microsoft Third-Party UEFI CA:** Enabled (Linux modülleri için şarttır)  
- **Secure Rollback Prevention:** Enabled (Manuel olarak BIOS'tan aktif edildi)  

---

## 2. TPM 2.0 ve LUKS Otomasyonu

Disk şifresinin (`/dev/nvme0n1p3`) açılışta otomatik çözülmesi için uygulanan komutlar:

```bash
# TPM Kaydı
sudo systemd-cryptenroll --tpm2-device=auto --tpm2-pcrs=0+2+7 /dev/nvme0n1p3

# /etc/crypttab yapılandırması
# Dosyaya şu parametre eklendi:
tpm2-device=auto

# Initramfs güncelleme
sudo dracut -f --regenerate-all
```

---

## 3. NVIDIA Sürücü İmzalama

Secure Boot açıkken ekran kartının çalışması için Machine Owner Key (MOK) süreci:

```bash
# Anahtar oluşturma
sudo kmodgenca -a

# MOK kaydı
sudo mokutil --import /etc/pki/akmods/certs/public_key.der

# Reboot sonrası:
# MOK Management ekranında "Enroll MOK" seçilir

# Sürücü derleme
sudo akmods --force
```

---

## 4. Güvenlik Raporu Analizi (fwupdmgr)

- **HSI-2! Seviyesi:** Sistem temel ve ileri güvenlik seviyelerini başarıyla geçmiştir.  
- **Linux Kernel Lockdown:** Aktif. Secure Boot çekirdeği koruyor.  
- **Tainted Kernel:** NVIDIA sürücüsü kapalı kaynak olduğu için bu uyarı normaldir.  
- **Encrypted RAM:** Donanım desteklemediği için pasif.  

---

## 🔄 BIOS Güncellemesi Sonrası "Anahtar Tazeleme"

BIOS (Firmware) güncellendiğinde PCR 0 değeri değişir ve TPM otomatik açılışı durdurur.

Tekrar aktif etmek için:

```bash
# Eski kaydı sil
sudo systemd-cryptenroll --wipe-slot=tpm2 /dev/nvme0n1p3

# Yeni durumu TPM'e kaydet
sudo systemd-cryptenroll --tpm2-device=auto --tpm2-pcrs=0+2+7 /dev/nvme0n1p3

# Initramfs güncelle
sudo dracut -f --regenerate-all
```

---

## ⚠️ Kritik Notlar

- **BIOS Yazma Hatası:**  
  `failed to write bytes to 14` hatası normaldir. Secure Boot/Lockdown modunda BIOS'a yazılım üzerinden müdahale kısıtlıdır.

- **Kernel Güncellemeleri:**  
  `akmods` sayesinde kernel güncellense bile sürücüler otomatik olarak imzalanır.
