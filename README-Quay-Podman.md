# Quay Registry (Self-Signed Certificate) -- Podman Kurulum ve Giriş Rehberi

Bu doküman, **self-signed sertifika kullanan kurumsal Quay Registry**
için Fedora/Linux üzerinde **Podman ile güvenli image pull ve login
işlemlerinin** nasıl yapılacağını adım adım açıklar.

------------------------------------------------------------------------

## ✅ Ortam Bilgileri

-   Registry: `quay.yargitay.gov.tr:8443`
-   Sertifika Türü: **Self-signed**
-   İşletim Sistemi: Fedora / RHEL Tabanlı
-   Container Runtime: **Podman**

------------------------------------------------------------------------

## 🔹 1. Sunucu Sertifikasını Alma

``` bash
openssl s_client -connect quay.yargitay.gov.tr:8443 -showcerts </dev/null
```

-----BEGIN CERTIFICATE-----\
...\
-----END CERTIFICATE-----

------------------------------------------------------------------------

## 🔹 2. Sertifikayı Sisteme Root CA Olarak Tanıtma

``` bash
sudo tee /etc/pki/ca-trust/source/anchors/yargitay-quay.crt
sudo update-ca-trust extract
```

------------------------------------------------------------------------

## 🔹 3. Podman İçin Özel Sertifika Dizini (ZORUNLU)

``` bash
sudo mkdir -p /etc/containers/certs.d/quay.yargitay.gov.tr:8443
sudo cp /etc/pki/ca-trust/source/anchors/yargitay-quay.crt         /etc/containers/certs.d/quay.yargitay.gov.tr:8443/ca.crt
sudo chmod 644 /etc/containers/certs.d/quay.yargitay.gov.tr:8443/ca.crt
```

------------------------------------------------------------------------

## 🔹 4. Podman Servisini Yeniden Başlatma

``` bash
systemctl --user restart podman
```

------------------------------------------------------------------------

## 🔹 5. Sertifika Doğrulama

``` bash
openssl s_client -connect quay.yargitay.gov.tr:8443 -servername quay.yargitay.gov.tr
```

------------------------------------------------------------------------

## 🔹 6. Temiz Sertifika Export

``` bash
openssl s_client -connect quay.yargitay.gov.tr:8443 -showcerts </dev/null 2>/dev/null | openssl x509 -outform PEM > quay.yargitay.gov.tr.crt
```

``` bash
openssl x509 -in quay.yargitay.gov.tr.crt -noout -subject -issuer
sudo cp quay.yargitay.gov.tr.crt /etc/pki/ca-trust/source/anchors/
sudo update-ca-trust extract
```

------------------------------------------------------------------------

## 🔹 7. Podman Login

``` bash
podman login quay.yargitay.gov.tr:8443
```

------------------------------------------------------------------------

## 🔹 8. Image Pull

``` bash
podman pull quay.yargitay.gov.tr:8443/yargitay-java-projects/otag-spring:latest
```

------------------------------------------------------------------------

## 🔹 9. Sertifika Dizini Kontrol

``` bash
ls -l /etc/containers/certs.d/quay.yargitay.gov.tr:8443/
```

------------------------------------------------------------------------

## 🔹 10. Podman Debug

``` bash
podman info --debug | grep -i cert -A5
```

------------------------------------------------------------------------

## ⚠️ Güvenlik Notu

``` bash
podman pull --tls-verify=false quay.yargitay.gov.tr:8443/image:tag
```

Bu yöntem **sadece geçici test içindir.**
