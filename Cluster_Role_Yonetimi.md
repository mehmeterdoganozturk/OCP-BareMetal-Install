# OpenShift Cluster Role Yönetimi

Bu doküman, OpenShift üzerinde `oc adm policy add-cluster-role-to-user` komutunun kullanımını ve yaygın olarak kullanılan rollerin açıklamalarını içermektedir.

## 🔹 Yaygın Kullanılan Cluster Roller ve İşlevleri

| Cluster Role | Açıklama |
|-------------|---------|
| **cluster-admin** | En yüksek yetkiye sahiptir. Kullanıcıya **OpenShift'in tüm kaynaklarına tam erişim** verir. **Admin (yönetici) yetkisi** sağlar. **Sistem yöneticileri için uygundur.** |
| **admin** | Bir namespace içinde **tüm kaynakları yönetebilir**, ancak cluster seviyesinde yetkisi yoktur. Yalnızca kendi projesini yönetebilir. |
| **edit** | Kullanıcıya bir namespace içinde **kaynakları değiştirme** yetkisi verir ama **rol atayamaz**. **Geliştiriciler için uygundur.** |
| **view** | Kullanıcıya sadece **okuma (read-only) yetkisi** verir. Projeyi görebilir ama değiştiremez. |
| **system:openshift:scc:anyuid** | Pod’ları **root olmayan UID ile çalıştırma kısıtlamasını kaldırır.** Root yetkisi gerektiren container'lar için kullanılır. |
| **system:openshift:scc:privileged** | Kullanıcıya **güvenlik sınırlamaları kaldırılmış (privileged) pod çalıştırma yetkisi** verir. |
| **system:image-puller** | Kullanıcıya veya Service Account’a **projedeki container image’larını çekme yetkisi** verir. |
| **system:image-builder** | Kullanıcıya OpenShift'te **image build etme yetkisi** verir. |
| **system:image-pusher** | Kullanıcıya OpenShift image registry'ye **image push etme yetkisi** verir. |
| **basic-user** | Kullanıcıya temel OpenShift API erişimi sağlar ama **kaynakları düzenleyemez.** |
| **self-provisioner** | Kullanıcının kendi başına yeni projeler (namespaces) oluşturmasını sağlar. |

## 🔹 Kullanım Örnekleri

### 📌 1️⃣ Kullanıcıya Cluster Admin Yetkisi Verme (Tüm Yetkiler)
```sh
oc adm policy add-cluster-role-to-user cluster-admin user1
```
> ✅ `user1` artık **tüm cluster üzerinde tam yetkiye sahiptir**.
> ⚠️ **Dikkat:** Bu yetkiyi sadece güvenilir yöneticilere vermelisin!  

---

### 📌 2️⃣ Kullanıcıya Bir Namespace İçinde Admin Yetkisi Verme
```sh
oc adm policy add-role-to-user admin user1 -n my-project
```
> ✅ `user1`, **my-project içinde tüm kaynakları yönetebilir**, ama cluster genelinde yönetici değildir.

---

### 📌 3️⃣ Kullanıcıya Yalnızca Okuma Yetkisi (view) Verme
```sh
oc adm policy add-cluster-role-to-user view user1
```
> ✅ `user1`, **kaynakları görüntüleyebilir ama değiştiremez.**

---

### 📌 4️⃣ Service Account’a Özel Yetkiler Verme
#### 📌 Örnek: Bir SA’ya Admin Yetkisi Verme
```sh
oc adm policy add-role-to-user admin -z web-sa -n my-project
```
> ✅ `web-sa` adlı Service Account, **my-project namespace içinde yönetici yetkisine sahip olur.**

---

### 📌 5️⃣ Privileged Pod Çalıştırmak İçin Yetki Verme
```sh
oc adm policy add-scc-to-user privileged -z web-sa -n my-project
```
> ✅ `web-sa`, **privileged modda pod çalıştırabilir.**

---

## 📌 Özet:
- **`add-cluster-role-to-user`**, bir kullanıcıya **cluster genelinde yetki verir**.
- **`add-role-to-user`**, yalnızca belirli bir namespace içindeki yetkileri ayarlar.
- **Güvenlik açısından dikkatli ol!** `cluster-admin` gibi güçlü rolleri **sadece gerekli kişilere** ver.
- **Pod’un root çalıştırması gerekiyorsa**, `privileged` veya `anyuid` SCC yetkilerini eklemelisin.

📌 **İhtiyacına uygun rolü belirleyip kullanabilirsin!** 🚀

# OpenShift'te Pod'a Root Yetkisi Verme ve Yetkilendirme

OpenShift, güvenlik politikaları gereği Pod'ların root olarak çalışmasını varsayılan olarak engeller. Eğer bir Pod'un root olarak çalışması gerekiyorsa, **SCC (Security Context Constraints)** kullanarak gerekli yetkileri vermen gerekir.

## SCC (Security Context Constraints) Nedir?

SCC, OpenShift'te **Pod'ların güvenliğini** sağlamak için kullanılan bir güvenlik mekanizmasıdır. SCC, **Pod'ların hangi kullanıcılarla çalışabileceğini, hangi yetkilere sahip olabileceğini ve hangi kaynaklara erişebileceğini belirler**.

Bazı uygulamalar **root olarak çalışmaya ihtiyaç duyar**. Örneğin:
- Özel bir sistem servisi çalıştıran bir konteyner
- Kernel modülleri yükleyen bir Pod
- Düşük seviyeli ağ konfigürasyonu değiştiren bir uygulama

Bunu sağlamak için Pod’un **privileged** veya **anyuid** SCC yetkilerine ihtiyacı vardır.

---

## SCC Türleri ve Root Yetkisi Verme

### 1. Privileged SCC

Pod’a **tüm sistem kaynaklarına erişim sağlayarak** çalışmasına izin verir.

#### Nasıl Kullanılır?
```sh
oc adm policy add-scc-to-user privileged -z my-service-account -n my-namespace
```

#### Ne Yapar?
- Pod **her türlü işlem yapabilir**
- Root yetkisiyle çalışmasına izin verilir
- Donanım ve çekirdek modüllerine erişebilir

---

### 2. AnyUID SCC

Pod’un **istediği kullanıcı kimliğiyle (UID) çalışmasına izin verir**. Varsayılan olarak OpenShift, Pod’un rastgele bir UID ile çalışmasını zorunlu kılar. Eğer uygulama root veya belirli bir UID gerektiriyorsa, `anyuid` SCC eklenmelidir.

#### Nasıl Kullanılır?
```sh
oc adm policy add-scc-to-user anyuid -z my-service-account -n my-namespace
```

#### Ne Yapar?
- Pod **root veya belirli bir UID ile çalışabilir**
- OpenShift’in zorunlu UID atamasını devre dışı bırakır
- Konteyner içinde kullanıcı değiştirmeye izin verir

---

## Service Account ile Root Yetkisi Verme

Bir Pod’a root yetkisi vermek için **Service Account’a özel SCC atayabiliriz**.

### Örnek: Root Yetkili Bir Pod Çalıştırma
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: my-sa
  namespace: my-namespace
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: root-pod
  namespace: my-namespace
spec:
  replicas: 1
  selector:
    matchLabels:
      app: root-app
  template:
    metadata:
      labels:
        app: root-app
    spec:
      serviceAccountName: my-sa  # Service Account ataması yapıldı
      containers:
      - name: root-container
        image: alpine
        command: ["/bin/sh", "-c", "sleep 3600"]
        securityContext:
          privileged: true  # Pod’a tam root yetkisi verildi
```

Bu Pod root olarak çalışacaktır.

#### Eğer Pod Çalışmazsa SCC Yetkisi Ver
```sh
oc adm policy add-scc-to-user privileged -z my-sa -n my-namespace
```

---

## Özet: Hangi SCC’yi Ne Zaman Kullanmalıyım?

| SCC Türü | Ne İşe Yarar? | Komut |
|----------|--------------|-------|
| **privileged** | Pod’a **tam root yetkisi** verir, kernel seviyesinde işlem yapabilir | `oc adm policy add-scc-to-user privileged -z my-sa -n my-namespace` |
| **anyuid** | Pod’un herhangi bir UID ile çalışmasını sağlar, root kullanıcı olabilir | `oc adm policy add-scc-to-user anyuid -z my-sa -n my-namespace` |

Bu SCC’leri **Service Account’a** ekleyerek **Pod’un root olarak çalışmasını sağlayabilirsin**. Ama **güvenlik açısından dikkatli olunmalıdır**.

Eğer SCC kullanmadan sadece root yetkili bir Pod istiyorsan:
```yaml
securityContext:
  runAsUser: 0
```
ekleyerek UID’yi 0 (root) yapabilirsin, ama bu **varsayılan SCC politikalarına takılabilir**.

---
