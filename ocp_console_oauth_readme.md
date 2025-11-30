# OpenShift Console & OAuth Route Custom Domain Configuration

Bu doküman OpenShift cluster üzerinde **Console** ve **OAuth** bileşenlerinin hostname adreslerini değiştirmek için izlenen adımların özet ve uygulanabilir halidir.

---

## 1. TLS Secret Oluşturma
Yeni domain için kullanılacak sertifika ve anahtar ile secret oluşturun.

```bash
och create secret tls console-tls-secret \
  --cert=yargitay2026.crt \
  --key=yargitay2026.key \
  -n openshift-config
```

---

## 2. OpenShift Console Hostname Güncelleme
Console routing bilgilerini düzenleyin.

### Edit (isteğe bağlı)
```bash
oc edit console.operator.openshift.io cluster
```
Gerekli alanlar:
```yaml
spec:
  route:
    hostname: console-ocp-ocplab.yargitay.gov.tr
    secret:
      name: console-tls-secret
```

### Patch (Önerilen Yöntem)
```bash
oc patch console.operator.openshift.io cluster --type=merge -p '{"spec": {"route": {"hostname": "console-ocp-ocplab.yargitay.gov.tr", "secret": {"name": "console-tls-secret"}}}}'
```

---

## 3. OAuth (Authentication Route) Güncelleme
Ingress üzerinde component route tanımlayın.

### Edit (isteğe bağlı)
```bash
oc edit ingress.config.openshift.io cluster
```
Eklenmesi gereken bölüm:
```yaml
spec:
  componentRoutes:
    - hostname: oauth-ocp-ocplab.yargitay.gov.tr
      name: oauth-openshift
      namespace: openshift-authentication
      servingCertKeyPairSecret:
        name: console-tls-secret
  domain: apps.ocplab.yargitay.gov.tr
```

### Patch (Önerilen Yöntem)
```bash
och patch ingress.config.openshift.io cluster --type=merge --patch '
  spec:
    componentRoutes:
    - hostname: oauth-ocp-ocplab.yargitay.gov.tr
      name: oauth-openshift
      namespace: openshift-authentication
      servingCertKeyPairSecret:
        name: console-tls-secret
'
```

---

## 4. Kontrol
```bash
och get routes -A | grep oauth
och get console.operator.openshift.io cluster -o yaml
```

Tüm route'lar için sertifika ve hostname doğrulanmalıdır.

---

## Sonuç
Bu işlemler tamamlandığında OpenShift Console artık şu URL üzerinden erişilebilir:

- Console: **https://console-ocp-ocplab.yargitay.gov.tr**
- OAuth Login: **https://oauth-ocp-ocplab.yargitay.gov.tr**

Cluster uygulama domaine ise şu şekilde kalır:

```
*.apps.ocplab.yargitay.gov.tr
```

