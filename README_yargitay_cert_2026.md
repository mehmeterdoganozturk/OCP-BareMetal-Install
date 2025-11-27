
# README — Yargıtay Sertifika İşlemleri (CSR, PFX, CA Bundle, dönüşümler)

Bu doküman, `/home/erdogan/Downloads/yargitay_cert_2026/1864508042repl_2` dizininde çalışan sertifika işlemleri için adım adım talimatları ve açıklamaları içerir. İçerik açık, uygulanabilir komutlar ve nedenleriyle birlikte verilmiştir.

---

## İçindekiler
1. Amaç
2. Dosya listesi (mevcut)
3. CSR ve Private Key üretimi (örnek)
4. CA chain (bundle) oluşturma
5. PFX (PKCS#12) oluşturma (şifreli / şifresiz)
6. PFX doğrulama / içeriğini görüntüleme
7. Sertifika dönüştürmeleri (crt ↔ pem ↔ cer)
8. Private key izinleri (chmod) ve güvenlik
9. Örnek komutlar ve açıklamaları
10. Hata/uygulama notları
11. Sürüm bilgisi / Tarih

---

## 1) Amaç
Bu README, bir web sunucusu/kurum için oluşturulan CSR, private key, CA ara ve root sertifikalarının birleştirilmesi (bundle), PKCS#12 (.pfx) paketinin oluşturulması ve gerekli format dönüşümlerinin nasıl yapılacağını anlatır. Ayrıca güvenlik (dosya izinleri) ve sık karşılaşılan hatalar hakkında kısa notlar içerir.

---

## 2) Mevcut dosyalar (örnek)
Aşağıdaki dosyalar sizin dizininizde mevcut olarak listelenmiştir:

- `yargitay2026.key` — Private key (siz oluşturmuştunuz)
- `yargitay2026.crt` — Sunucu / alan adı sertifikası (CA tarafından imzalanmış)
- `USERTrustRSACertificationAuthority.crt` — Root CA
- `SectigoPublicServerAuthenticationRootR46_USERTrust.crt` — Intermediate CA
- `SectigoPublicServerAuthenticationCADVR36.crt` — Intermediate CA

---

## 3) CSR ve Private Key üretimi (örnek)
Aşağıdaki komut, RSA 2048-bit anahtar çifti ve CSR oluşturur. `-nodes` private key'i şifrelemez; eğer private key şifreli olsun isterseniz `-nodes` kaldırın.

```bash
openssl req -new -newkey rsa:2048 -nodes \
  -keyout yargitay2048.key \
  -out yargitay2048.csr \
  -addext "subjectAltName=DNS:.yargitay.gov.tr,DNS:.yargitaycb.gov.tr,DNS:www.gpo.gov.tr"
```

Komut açıklaması:
- `req -new` : yeni CSR isteği oluşturur.
- `-newkey rsa:2048` : 2048 bit RSA anahtar üretir.
- `-nodes` : private key'i şifrelemeden dosyaya yaz (no DES).
- `-keyout` : oluşturulan private key dosyası.
- `-out` : CSR dosyası.
- `-addext "subjectAltName=..."` : SAN (Subject Alternative Name) alanlarını CSR içine ekler (OpenSSL sürümünüz desteklemeli).

Not:
- Eğer OpenSSL sürümünüz `-addext`'i desteklemiyorsa, SAN'ları openssl.cnf üzerinden ya da bir config dosyası kullanarak eklemelisiniz. Ayrıca `Common Name` (CN) artık SAN ile desteklenir; modern uygulamalarda CN tek başına yeterli değildir.

---

## 4) CA chain (bundle) oluşturma
CA ara ve root sertifikalarını doğru sırada birleştirmek önemlidir. Sunucu sertifikasını doğrulamak için tarayıcı/istemci, önce sunucu sertifikasına, sonra ara CA'lara, en sonunda root CA'ya doğru bir zincir takip eder. Aşağıdaki örnek sıradır (ara -> root).

Dizin: `/home/erdogan/Downloads/yargitay_cert_2026/1864508042repl_2`

```bash
cd /home/erdogan/Downloads/yargitay_cert_2026/1864508042repl_2

cat SectigoPublicServerAuthenticationCADVR36.crt \
    SectigoPublicServerAuthenticationRootR46_USERTrust.crt \
    USERTrustRSACertificationAuthority.crt \
    > ca-bundle.crt
```

Açıklama:
- `cat` ile önce hizmeti sağlayan **Intermediate** CA'lar, en sonda Root CA yazılmalıdır. Bazı PKI sağlayıcıları farklı sıralama ister; web server (Apache, Nginx, HAProxy vb.) dökümantasyonuna göre ayarlayın.

---

## 5) PFX (PKCS#12) oluşturma
Private key, sunucu sertifikası ve CA chain'i tek bir `.pfx` dosyasında paketleyebilirsiniz.

### a) Şifresiz PFX (not: PKCS#12 formatı her zaman bir parola isteyebilir; `-nodes` gibi bir seçenek yoktur; `openssl pkcs12 -export` komutu şifre ister. Aşağıda örnek verilmektedir.)
```bash
openssl pkcs12 -export \
  -out yargitay2026.pfx \
  -inkey yargitay2026.key \
  -in yargitay2026.crt \
  -certfile ca-bundle.crt
```

Komuttaki anahtarlar:
- `-out` : üretilen pfx dosyası
- `-inkey` : private key
- `-in` : sunucu sertifikası
- `-certfile` : CA chain (bundle) dosyası (ara+root)

OpenSSL sizden bir export parolası girmenizi isteyebilir; bu parola PFX'i korur.

### b) Belirli bir isimle şifreli PFX oluşturma
Aynı komut kullanılır; çıktı dosya adını değiştirin:

```bash
openssl pkcs12 -export \
  -out yargitay2026sifreli.pfx \
  -inkey yargitay2026.key \
  -in yargitay2026.crt \
  -certfile ca-bundle.crt
```

Bu komut çalışırken `Enter Export Password:` ve onay ister. Girilen parola PFX dosyasını açmak için gereklidir.

---

## 6) PFX doğrulama / içeriğini görüntüleme
PFX içeriğini parola olmadan anahtarları göstermeden doğrulamak isterseniz:

```bash
openssl pkcs12 -info -in yargitay2026.pfx -nokeys
```

- `-nokeys` : private key içeriklerini göstermez (sadece sertifikaları ve CA zincirini listeler).
- Parola sorulacaktır; eğer PFX şifreliyse doğru parolayı girin.

Eğer PFX içerisinden private key almak isterseniz (parolayı bilmeniz gerekir):

```bash
openssl pkcs12 -in yargitay2026.pfx -nocerts -nodes -out extracted.key
```

- `-nocerts` : yalnızca private key çıkar
- `-nodes` : private key'i açık (şifresiz) olarak kaydeder; dikkatli olun.

---

## 7) Sertifika dönüşümleri (crt ↔ pem ↔ cer)
- `.crt`, `.cer`, `.pem` uzantıları genelde içerik tipine göre değişir; her zaman dosyanın içerik formatını kontrol edin (PEM = base64 ile `-----BEGIN CERTIFICATE-----` başlar; DER = binary).
- `.cer` uzantısı sıkça PEM veya DER içerebilir; uzantıyı değiştirmenin kendisi dosyayı bozmaz (yalnızca uygulamanın beklediği formatla uyumlu olmalısınız).

### CRT (PEM) → PEM (aynı içerik, sadece isim değişikliği)
```bash
cp yargitay2026.crt yargitay2026.pem
```

### CRT (PEM) → CRT (DER)
```bash
openssl x509 -in yargitay2026.crt -outform der -out yargitay2026.der
```

### DER → PEM
```bash
openssl x509 -inform der -in cert.der -out cert.pem
```

### CRT → CER (isim değiştirme / aynı format)
Eğer `yargitay2026.crt` PEM formatındaysa sadece isim değiştirerek `.cer` yapabilirsiniz:
```bash
cp yargitay2026.crt yargitay2026.cer
```
Ancak karşı tarafta `.cer` bekleyen uygulama DER formatı istiyorsa önce dönüştürün (bkz. DER dönüşümü).

---

## 8) Private key izinleri (güvenlik)
Private key dosyaları çok hassastır. Önerilen izinler:

```bash
# Sahip: root veya sertifika yöneticisi hesabı
sudo chown root:root yargitay2026.key

# OKUNABİLİR YALNIZCA SAHİP İÇİN
sudo chmod 600 yargitay2026.key
```

- `600` => sahibi okur/yazar, grup veya diğer kullanıcılar erişemez. Alternatif küçük ekiplerde `640` + özel grup kullanılabilir, ama en güvenli olan `600`'dür.
- Dosya izinleri ve sahipliği, web sunucusu (nginx, apache, haproxy vb.) private key'e erişecekse uygun şekilde ayarlanmalıdır (örn. `chown root:ssl-cert` ve `chmod 640` gibi).

---

## 9) Örnek komut seti — adım adım
Aşağıda tipik bir akış örneği verilmiştir:

```bash
# 1) Dizine geç
cd /home/erdogan/Downloads/yargitay_cert_2026/1864508042repl_2

# 2) CSR + Key (eğer henüz oluşturmadıysanız)
openssl req -new -newkey rsa:2048 -nodes \
  -keyout yargitay2048.key -out yargitay2048.csr \
  -addext "subjectAltName=DNS:.yargitay.gov.tr,DNS:.yargitaycb.gov.tr,DNS:www.gpo.gov.tr"

# 3) CA bundle oluştur
cat SectigoPublicServerAuthenticationCADVR36.crt \
    SectigoPublicServerAuthenticationRootR46_USERTrust.crt \
    USERTrustRSACertificationAuthority.crt > ca-bundle.crt

# 4) PFX oluştur (export parolası istenir)
openssl pkcs12 -export -out yargitay2026.pfx \
  -inkey yargitay2026.key -in yargitay2026.crt -certfile ca-bundle.crt

# 5) PFX doğrula (sertifika/zincir görüntüleme)
openssl pkcs12 -info -in yargitay2026.pfx -nokeys

# 6) crt -> pem
openssl x509 -in yargitay2026.crt -out yargitay2026.pem
```

---

## 10) Notlar & Sık karşılaşılan hatalar
- `-addext` hata veriyorsa: OpenSSL sürümünüz eski olabilir. Bu durumda SAN için bir openssl konfig dosyası kullanın. Örnek:
  ```ini
  [ req ]
  distinguished_name = req_distinguished_name
  req_extensions = v3_req
  prompt = no

  [ req_distinguished_name ]
  C = TR
  ST = ANKARA
  L = CANKAYA
  O = YARGITAY
  OU = SISTEM
  CN = *.yargitay.gov.tr
  emailAddress = sistem@yargitay.gov.tr

  [ v3_req ]
  subjectAltName = @alt_names

  [ alt_names ]
  DNS.1 = .yargitay.gov.tr
  DNS.2 = .yargitaycb.gov.tr
  DNS.3 = www.gpo.gov.tr
  ```

  Ve komut:
  ```bash
  openssl req -new -config req.cnf -key yargitay2048.key -out yargitay2048.csr
  ```

- `openssl pkcs12 -export` çalıştırırken "Enter Export Password" gelir; bu parola PFX dosyasını korur.
- `certificate chain` sırasına dikkat edin. Yanlış sıra tarayıcı hatalarına sebep olur.
- Eğer web sunucusu sertifikayı yüklemiyor ya da tarayıcı “certificate not trusted” diyorsa ara CA'ları eksik yüklemiş olabilirsiniz.

---

## 11) Sürüm bilgisi / Tarih
- Bu README: 27 November 2025
- Hazırlayan: Mehmet Erdoğan Öztürk tarafından sağlanan bilgiler temel alınarak otomatik üretilmiştir.

---

## Yardım / İleri adımlar
- Özel bir openssl konfig dosyasıyla CSR oluşturma örneği isterseniz, `req.cnf` örneğini sağlayıp tamamen dolduracağım.
- PFX içinden private key'i kaldırıp sadece sertifikayı tutmak isterseniz, ilave komutlar ekleyebilirim.
