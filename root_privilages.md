# OpenShift'te Pod Root Yetkilendirmesi ve Güvenli Kullanımı

## 📌 İki Deployment YAML Dosyasının Karşılaştırılması

İki YAML dosyası da bir **Alpine** container'ı çalıştırıyor ve bir **Service Account (web-sa)** kullanıyor. Ancak root yetkilendirme konusunda farklı yaklaşımlar içeriyor.

---

## **1. Deployment - Privileged Mode ile Root Yetkili**

Bu YAML dosyası **tam yetkili (privileged) bir pod** oluşturur:

- **runAsUser: 0** → Pod doğrudan root kullanıcısı olarak çalışır.
- **privileged: true** → Pod, host üzerinde tam root yetkisine sahip olur.
- **allowPrivilegeEscalation: true** → Container içinde root yetkisi yükseltilebilir.
- **Güvenlik riski taşır**, çünkü pod sistem kaynaklarına tam erişim sağlar.

---

## **2. Deployment - Daha Kısıtlı Root Yetkisi ile Güvenli Kullanım**

Bu YAML dosyası **daha güvenli root çalıştırma yöntemini** içerir:

- **seccompProfile: RuntimeDefault** → OpenShift’in varsayılan güvenlik profili kullanılır.
- **runAsNonRoot: false** → Root olarak çalışmasına izin verilir ancak **gerekmedikçe kullanılmamalıdır**.
- **allowPrivilegeEscalation: false** → Pod içindeki süreçlerin root yetkisi yükseltmesine izin verilmez.
- **capabilities: drop: ["ALL"]** → Ekstra yetkiler kısıtlanır.
- **Daha güvenli bir seçenektir** ve OpenShift politikalarına uyumludur.

---

## ⚖️ **Hangi Root Yetkilendirme Daha Güvenli?**

| Özellik | 1. YAML (Privileged Mode) | 2. YAML (Daha Güvenli) |
|---------|-------------------------|----------------------|
| **Root Kullanımı** | ✅ (**runAsUser: 0**) | ✅ (**runAsNonRoot: false**) |
| **Privileged Mode** | ✅ (**privileged: true**) | ❌ (**privileged yok**) |
| **Yetki Artırma** | ✅ (**allowPrivilegeEscalation: true**) | ❌ (**allowPrivilegeEscalation: false**) |
| **Seccomp Profili** | ❌ Yok | ✅ (**RuntimeDefault**) |
| **Ekstra Yetkiler** | ❌ Hepsi açık | ✅ (**capabilities: drop: ["ALL"]**) |
| **Güvenlik Riski** | 🚨 Yüksek | 🟢 Daha güvenli |
| **Önerilen Kullanım** | Güvenlik gerektirmeyen test ortamları | **OpenShift uyumlu güvenli pod çalıştırma** |

**🔹 Eğer OpenShift güvenlik politikalarına uymak istiyorsan, 2. YAML dosyası en uygun olanıdır.**

---

## 📌 OpenShift’te Root Yetkisi Verme

Eğer pod’un **root yetkisiyle çalışması gerekiyorsa**, aşağıdaki adımları takip edebilirsin:

### **1. "anyuid" SCC Yetkisi Ver**

Bu, pod’un **root olarak çalışmasına izin verir**:
```sh
oc adm policy add-scc-to-user anyuid -z web-sa -n <namespace>
```

### **2. Privileged SCC Yetkisi Gerekiyor mu Kontrol Et**

Bazı özel durumlarda **privileged SCC** kullanman gerekebilir. Bunu eklemek için:
```sh
oc adm policy add-scc-to-user privileged -z web-sa -n <namespace>
```
Ancak **güvenlik riski nedeniyle mümkünse kaçınılmalıdır**.

### **3. Uygulamayı Root Olmadan Çalıştıracak Şekilde Düzenle**

En iyi yöntem, uygulamanın **root yetkisi gerektirmeyecek şekilde çalışmasını sağlamaktır**. Mümkünse **runAsNonRoot: true** kullanarak root dışı bir kullanıcıyla çalıştır.

---

## **Özet**

- **Mutlaka root çalıştırman gerekiyorsa**, **2. YAML dosyası** daha güvenlidir.
- **Privileged Mode (1. YAML)** çok güçlü yetkiler içerdiği için **sadece zorunlu olduğunda** kullanılmalıdır.
- **OpenShift güvenlik politikalarına uymak için** "anyuid" SCC kullan ve **privileged SCC’den mümkünse kaçın**.
- **Mümkünse root yetkisi gerektirmeyen bir kullanıcı ile çalıştır**.

