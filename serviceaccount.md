## 🎯 **Hedefler**

1.  `yetkiliuser` adında bir **Service Account** tanımlayacağız.
2.  `linux` adlı OpenShift projesinde (namespace) çalışacak.
3.  `ubuntu.yaml` adlı **Deployment** dosyası oluşturulacak ve pod **root yetkili** çalışacak.
4.  Pod içinde **NTP servisi** kurulacak.
5.  Pod içindeki **kullanıcı sadece NTP servisini restart edebilecek**, diğer servisleri değiştiremeyecek.

* * *

## 📌 **Adım 1: Service Account Tanımlama**

Bu Service Account, pod'un root çalışmasını ve belirli yetkilere sahip olmasını sağlayacak.

&nbsp;

`apiVersion: v1kind: ServiceAccountmetadata: name: yetkiliuser namespace: linux`

Bu service account'ı oluşturmak için şu komutu kullanabilirsin:

&nbsp;

`oc apply -f serviceaccount.yaml`

* * *

## 📌 **Adım 2: Deployment (ubuntu.yaml) - Root Yetkili Pod**

Bu deployment:

- **Root olarak çalıştırılır** (`runAsUser: 0`).
- **Privileged modda açılır**, böylece sistem seviyesinde değişiklik yapabilir.
- İçinde **NTP servisi kurulur** (`ntp` paketi ile).
- Bir döngü içinde çalışarak, NTP servisini restart etme yetkisine sahip olur.

&nbsp;

`apiVersion: apps/v1kind: Deploymentmetadata: name: ubuntu-ntp namespace: linuxspec: replicas: 1 selector: matchLabels: app: ubuntu-ntp template: metadata: labels: app: ubuntu-ntp spec: serviceAccountName: yetkiliuser # Kullanılacak Service Account securityContext: runAsUser: 0 # Root yetkisi ile çalıştır containers: - name: ubuntu-container image: ubuntu:latest command: ["/bin/bash", "-c", "apt update && apt install -y ntp && while true; do sleep 3600; done"] securityContext: privileged: true # Full root yetkisi allowPrivilegeEscalation: true capabilities: add: ["NET_ADMIN", "SYS_TIME"] # NTP için gerekli yetkiler`

Bunu oluşturmak için:

`oc apply -f ubuntu.yaml`

* * *

## 📌 **Adım 3: Role ve RoleBinding ile Sadece NTP Servisini Restart Etme Yetkisi Vermek**

Aşağıdaki **Role**, pod içindeki `yetkiliuser` kullanıcısına **sadece NTP servisini restart etme yetkisi** verir.

&nbsp;

`apiVersion: rbac.authorization.k8s.io/v1kind: Rolemetadata: namespace: linux name: ntp-managerrules:- apiGroups: [""] resources: ["pods/exec"] verbs: ["create"]`

Bu rolü bağlamak için **RoleBinding** kullanacağız:

&nbsp;

`apiVersion: rbac.authorization.k8s.io/v1kind: RoleBindingmetadata: name: ntp-manager-binding namespace: linuxsubjects:- kind: ServiceAccount name: yetkiliuser namespace: linuxroleRef: kind: Role name: ntp-manager apiGroup: rbac.authorization.k8s.io`

Bu dosyaları OpenShift'e uygulamak için:

&nbsp;

`oc apply -f role.yamloc apply -f rolebinding.yaml`

* * *

## 🎯 **Sonuç**

Bu yapılandırma ile:  
✅ Pod **root yetkisiyle açılır** ve **ntp servisini çalıştırır**.  
✅ Kullanıcı sadece **NTP servisini restart edebilir**, başka servislere müdahale edemez.  
✅ OpenShift **RBAC** ile yetkilendirme sağlanmıştır.

💡 **NTP servisini restart etmek için pod içinde şu komut çalıştırılabilir:**

`oc exec -it <pod-name> -- systemctl restart ntp`
