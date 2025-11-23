# OpenShift 4 UPI Bare Metal Kurulum Dokümanı (3 Master + 3 Worker + HAProxy)

Bu doküman, bare-metal üzerinde **User Provisioned Infrastructure (UPI)** yöntemi ile **3 Master + 3 Worker + HAProxy + Bastion** mimarisinde OpenShift 4 kurulumu için hazırlanmıştır.

## Topoloji Diyagramı

![OpenShift Topoloji Diyagramı](images/OCP.png)

## Gereksinimler Duruma Göre

| Adet (Şemadaki Adet) | Önerilen İşlemci (CPU)           | Önerilen Hafıza (RAM) | Önerilen Disk Alanı        | Önemli Notlar                                                                                                                        |
| -------------------- | -------------------------------- | --------------------- | -------------------------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| 3                    | 8 VCPUs (veya fiziksel çekirdek) | 32 GB RAM             | 250 GB SSD (NVMe önerilir) | Etcd için düşük gecikme süresi (low latency) hayati. Bu sunucular kararlılık ve yönetim yükü için güçlü olmalıdır.                   |
| 3                    | 8-16 VCPUs                       | 64 GB RAM             | 500 GB SSD                 | Uygulamalarınızın yoğunluğuna bağlı olarak RAM ve CPU en çok burada artırılmalıdır. Uygulama verisi için harici depolama kullanılır. |
| 1                    | 4 VCPUs                          | 16 GB RAM             | 120 GB SSD                 | Kurulum tamamlandıktan sonra kapatılacak geçici sunucudur.                                                                           |
| 1                    | 4 VCPUs                          | 8 GB RAM              | 100 GB                     | Tüm dış trafiği yönettiği için kararlı ve yeterli kaynaklara sahip olmalıdır.                                                        |

| IOPS Beklentisi | vCPU         | RAM    | Disk (OS & Ephemeral) | Kullanım Amacı ve Gerekçe                                                                                                                                              |
| --------------- | ------------ | ------ | --------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 5000+           | 16 vCPU      | 64 GB  | 250 GB (NVMe)         | Cluster büyüdükçe (Pod/Service sayısı arttıkça) API Server ve etcd bellek tüketimi artar. 64GB RAM, güncelleme (upgrade) süreçlerinin sorunsuz geçmesi için idealdir.  |
| 2000+           | 16 - 32 vCPU | 128 GB | 500 GB+ (SSD)         | Java/Spring Boot gibi bellek seven uygulamalar, veritabanları veya OpenShift Data Foundation (ODF) gibi depolama çözümleri için bu bellek miktarı "rahat" bir alandır. |
| Standart        | 8 vCPU       | 16 GB  | 200 GB                | Eğer SSL sonlandırma (Termination) Bastion üzerinde yapılacaksa CPU kritik hale gelir. Yüksek trafikli ingress akışını darboğaz olmadan işlemek için gereklidir.       |
| Standart        | 4 vCPU       | 16 GB  | 100 GB                | Kurulum süresini kısaltmak dışında bir etkisi yoktur. Kurulumdan sonra silineceği için bu değerler yeterlidir.                                                         |


| Gateway API                                                                                                                  | Geleneksel Ingress                                                                     |
| ---------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------- |
| Ağ yöneticisi, uygulama geliştiricisi ve küme operatörü rollerini ayrı nesnelerle (GatewayClass, Gateway, HTTPRoute) ayırır. | Tek bir Ingress nesnesi tüm sorumlulukları taşır.                                      |
| HTTP/S yanında TCP, UDP ve diğer protokoller için yerel destek sağlar.                                                       | Çoğunlukla sadece HTTP/S trafiği için kullanılır.                                      |
| Yönlendirme kurallarına özel eklentiler (Filters) eklemeye izin veren standart bir arayüz sunar.                             | Sağlayıcıya özel (vendor-specific) ek açıklamalar (annotations) kullanmayı gerektirir. |


## 1. DNS ve HAProxy Yapılandırması
_(Bu bölümde daha önce eklediğimiz DNS kayıtları ve HAProxy konfigürasyonu yer almaktadır.)_

## 2. OpenShift Kurulumu İçin 7 Adımlık Yapılandırma Blokları
1. **DNS Kayıtları**
2. **HAProxy Load Balancer Ayarları**
3. **İnstall-config.yaml Oluşturma**
4. **RHCOS Boot ve Ignition Dosyaları**
5. **Bootstrap ve Kontrol Düzlemi Kurulumu**
6. **Worker Node Kurulumu**
7. **Kurulum Sonrası Doğrulama**


## DNS Kayıtları
```
10.125.0.254    bastion.ocplab.yargitay.gov.tr
10.125.0.254    api.ocplab.yargitay.gov.tr
10.125.0.254    api-int.ocplab.yargitay.gov.tr
10.125.0.254    *.apps.ocplab.yargitay.gov.tr

10.125.0.200    bootstrap.ocplab.yargitay.gov.tr

10.125.0.201    master01.ocplab.yargitay.gov.tr
10.125.0.202    master02.ocplab.yargitay.gov.tr
10.125.0.203    master03.ocplab.yargitay.gov.tr

10.125.0.211    worker01.ocplab.yargitay.gov.tr
10.125.0.212    worker02.ocplab.yargitay.gov.tr
10.125.0.213    worker03.ocplab.yargitay.gov.tr
```

---

## 1. Gerekli Dosyaların İndirilmesi
Aşağıdaki bileşenler Red Hat konsolundan indirilmelidir:
- OpenShift Installer
- Pull Secret
- CLI (oc)
- RHCOS ISO

İndirme adresi:
```
https://console.redhat.com/openshift/install/metal/user-provisioned
```

---

## 2. Bastion Sunucusu Hazırlığı

### HTTPD kurulumu
```bash
dnf install httpd -y
sed -i 's/Listen 80/Listen 0.0.0.0:8080/' /etc/httpd/conf/httpd.conf
systemctl enable httpd
systemctl start httpd
curl localhost:8080
```

### HAProxy kurulumu
```bash
dnf install haproxy -y
systemctl enable haproxy
systemctl start haproxy
```
## HAProxy Yapılandırması (/etc/haproxy/haproxy.cfg)
```
#---------------------------------------------------------------------
# Global settings
#---------------------------------------------------------------------
global
    maxconn     20000
    log         /dev/log local0 info
    chroot      /var/lib/haproxy
    pidfile     /var/run/haproxy.pid
    user        haproxy
    group       haproxy
    daemon

    stats socket /var/lib/haproxy/stats

#---------------------------------------------------------------------
# Defaults
#---------------------------------------------------------------------
defaults
    log                     global
    mode                    tcp
    option                  dontlognull
    option                  tcp-check
    retries                 3
    maxconn                 20000
    timeout connect         60s
    timeout client          300s
    timeout server          300s
    timeout check           30s
    timeout queue           50s

#---------------------------------------------------------------------
# HAProxy Stats
#---------------------------------------------------------------------
listen stats
    bind *:9000
    mode http
    stats enable
    stats uri /stats
    stats refresh 10s

#---------------------------------------------------------------------
# Kubernetes API Server
#---------------------------------------------------------------------
frontend k8s_api_frontend
    bind :6443
    default_backend k8s_api_backend

backend k8s_api_backend
    balance source
    option tcp-check
    tcp-check connect port 6443
    default-server inter 5s rise 3 fall 3 check
    server bootstrap bootstrap.ocplab.yargitay.gov.tr:6443 check
    server master01 master01.ocplab.yargitay.gov.tr:6443 check
    server master02 master02.ocplab.yargitay.gov.tr:6443 check
    server master03 master03.ocplab.yargitay.gov.tr:6443 check

#---------------------------------------------------------------------
# OCP Machine Config Server
#---------------------------------------------------------------------
frontend ocp_machine_config_server_frontend
    bind :22623
    default_backend ocp_machine_config_server_backend

backend ocp_machine_config_server_backend
    balance source
    option tcp-check
    tcp-check connect port 22623
    default-server inter 5s rise 3 fall 3 check
    server bootstrap bootstrap.ocplab.yargitay.gov.tr:22623 check
    server master01 master01.ocplab.yargitay.gov.tr:22623 check
    server master02 master02.ocplab.yargitay.gov.tr:22623 check
    server master03 master03.ocplab.yargitay.gov.tr:22623 check

#---------------------------------------------------------------------
# OCP HTTP Ingress
#---------------------------------------------------------------------
frontend ocp_http_ingress_frontend
    bind :80
    default_backend ocp_http_ingress_backend

backend ocp_http_ingress_backend
    balance source
    option tcp-check
    tcp-check connect port 80
    default-server inter 5s rise 3 fall 3 check
    server worker01 worker01.ocplab.yargitay.gov.tr:80 check
    server worker02 worker02.ocplab.yargitay.gov.tr:80 check
    server worker03 worker03.ocplab.yargitay.gov.tr:80 check

#---------------------------------------------------------------------
# OCP HTTPS Ingress
#---------------------------------------------------------------------
frontend ocp_https_ingress_frontend
    bind :443
    default_backend ocp_https_ingress_backend

backend ocp_https_ingress_backend
    balance source
    option tcp-check
    tcp-check connect port 443
    default-server inter 5s rise 3 fall 3 check
    server worker01 worker01.ocplab.yargitay.gov.tr:443 check
    server worker02 worker02.ocplab.yargitay.gov.tr:443 check
    server worker03 worker03.ocplab.yargitay.gov.tr:443 check
```

---

### NFS Sunucusu Kurulumu
```bash
dnf install nfs-utils -y
mkdir -p /shares/registry
chown -R nobody:nobody /shares/registry
chmod -R 777 /shares/registry
echo "/shares/registry  10.0.0.0/8(rw,sync,root_squash,no_subtree_check,no_wdelay)" > /etc/exports
exportfs -rv
systemctl enable nfs-server rpcbind
systemctl start nfs-server rpcbind nfs-mountd
```

---

## 3. SSH Anahtar Oluşturma
```bash
ssh-keygen
```

---

## Extract Installer and Client
```bash
tar -xvf openshift-install-linux.tar.gz
openshift-client-linux.tar.gz
mv oc kubectl /usr/bin
```

## 4. Kurulum Dizini Oluşturma
```bash
mkdir ~/ocp-install
nano ~/ocp-install/install-config.yaml
```

### Örnek install-config.yaml
```yaml
apiVersion: v1
baseDomain: lab.local
compute:
- hyperthreading: Enabled
  name: worker
  replicas: 0
controlPlane:
  hyperthreading: Enabled
  name: master
  replicas: 3
metadata:
  name: ocp
networking:
  clusterNetwork:
  - cidr: 10.128.0.0/14
    hostPrefix: 23
  networkType: OVNKubernetes
  serviceNetwork:
  - 172.30.0.0/16
platform:
  none: {}
fips: false
pullSecret: '{"auths": ...}'
sshKey: 'ssh-ed25519 AAAA...'
```

> **Line 23**: pull-secret.txt içeriğini yapıştırın.  
> **Line 24**: `cat ~/.ssh/id_rsa.pub` çıktısını yapıştırın.

---

## 5. Manifest Oluşturma
```bash
~/openshift-install create manifests --dir ~/ocp-install
nano ~/ocp-install/manifests/cluster-scheduler-02-config.yml
```

`mastersSchedulable` değerini aşağıdaki gibi değiştirin:
```yaml
mastersSchedulable: false
```

---

## 6. Ignition Dosyalarının Oluşturulması
```bash
~/openshift-install create ignition-configs --dir ~/ocp-install/
```

### Dosyaları HTTP server’a kopyalayın
```bash
mkdir /var/www/html/ocp4
cp -R ~/ocp-install/* /var/www/html/ocp4
chown -R apache: /var/www/html/ocp4/
chmod 755 /var/www/html/ocp4/
curl localhost:8080/ocp4/
```

---

## 7. RHCOS Üzerinde İlk Ayarlar
```bash
sudo localectl set-keymap trq
sudo nano /etc/ssh/sshd_conf.d/40-rhcos-defaults.conf
```
Aşağıdaki satırı ekleyin:
```
PasswordAuthentication yes
```

Ardından:
```bash
sudo passwd core
sudo systemctl restart sshd --now
```

---
## CoreOS Installer Komutları (Bootstrap, Master, Worker)
```
sudo coreos-installer install /dev/sda \
  --ignition-url=http://10.125.0.254:8080/ocp4/bootstrap.ign \
  --insecure --insecure-ignition \
  --append-karg="ip=10.125.0.200::10.125.0.1:255.255.255.0:bootstrap.ocplab.yargitay.gov.tr:ens33:none nameserver=10.6.222.130" \
  --append-karg=rd.neednet=1

sudo coreos-installer install /dev/sda \
  --ignition-url=http://10.125.0.254:8080/ocp4/master.ign \
  --insecure --insecure-ignition \
  --append-karg="ip=10.125.0.201::10.125.0.1:255.255.255.0:master01.ocplab.yargitay.gov.tr:ens33:none nameserver=10.6.222.130" \
  --append-karg=rd.neednet=1

sudo coreos-installer install /dev/sda \
  --ignition-url=http://10.125.0.254:8080/ocp4/master.ign \
  --insecure --insecure-ignition \
  --append-karg="ip=10.125.0.202::10.125.0.1:255.255.255.0:master02.ocplab.yargitay.gov.tr:ens33:none nameserver=10.6.222.130" \
  --append-karg=rd.neednet=1

sudo coreos-installer install /dev/sda \
  --ignition-url=http://10.125.0.254:8080/ocp4/master.ign \
  --insecure --insecure-ignition \
  --append-karg="ip=10.125.0.203::10.125.0.1:255.255.255.0:master03.ocplab.yargitay.gov.tr:ens33:none nameserver=10.6.222.130" \
  --append-karg=rd.neednet=1

sudo coreos-installer install /dev/sda \
  --ignition-url=http://10.125.0.254:8080/ocp4/worker.ign \
  --insecure --insecure-ignition \
  --append-karg="ip=10.125.0.211::10.125.0.1:255.255.255.0:worker01.ocplab.yargitay.gov.tr:ens33:none nameserver=10.6.222.130" \
  --append-karg=rd.neednet=1

sudo coreos-installer install /dev/sda \
  --ignition-url=http://10.125.0.254:8080/ocp4/worker.ign \
  --insecure --insecure-ignition \
  --append-karg="ip=10.125.0.212::10.125.0.1:255.255.255.0:worker02.ocplab.yargitay.gov.tr:ens33:none nameserver=10.6.222.130" \
  --append-karg=rd.neednet=1

sudo coreos-installer install /dev/sda \
  --ignition-url=http://10.125.0.254:8080/ocp4/worker.ign \
  --insecure --insecure-ignition \
  --append-karg="ip=10.125.0.213::10.125.0.1:255.255.255.0:worker03.ocplab.yargitay.gov.tr:ens33:none nameserver=10.6.222.130" \
  --append-karg=rd.neednet=1
```

## 8. Bootstrap Süreci
Bootstrap node’u başlattıktan sonra bastion üzerinden şu komutu çalıştırın:
```bash
./openshift-install wait-for bootstrap-complete --dir ~/ocp-install
```

Bootstrap tamamlandıktan sonra:
- HAProxy üzerindeki **bootstrap backend satırlarını silin**.

Test:
```bash
watch -n 30 'curl -ks https://api.ocplab.yargitay.gov.tr:6443/version'
```

---

## 9. Cluster Kurulumunun Tamamlanması
```bash
./openshift-install wait-for install-complete --dir ~/ocp-install
```

Master node üzerinde:
```bash
sudo crictl ps | grep kube-apiserver
```

---

## 10. CLI ile Bağlantı
```bash
export KUBECONFIG=~/ocp-install/auth/kubeconfig
oc get nodes
```

---

## 11. Worker Node Join İşlemleri
Worker'lar eklendikten sonra CSRs approve edilir.

### CSR’leri görüntüleme
```bash
oc get csr
```

### Tüm pending CSRs onaylama
```bash
oc get csr -o go-template='{{range .items}}{{if not .status}}{{.metadata.name}}{{"\n"}}{{end}}{{end}}' | xargs oc adm certificate approve
oc get csr -o name | xargs oc adm certificate approve
```

### Node'ların Ready olmasını izleyin
```bash
watch -n5 oc get nodes
```

---

## 12. Kullanıcı Oluşturma
```bash
# htpasswd aracı gereklidir (genelde httpd-tools paketindedir)
# Komut yapısı: htpasswd -n -B -b <kullanici_adi> <sifre> | base64 -w0
htpasswd -n -B -b erdogan Erdogan.2024! | base64 -w0
nano oauth-htpasswd.yaml

apiVersion: v1
kind: Secret
metadata:
  name: htpasswd-secret
  namespace: openshift-config
type: Opaque
data:
  # Buradaki değer 'erdogan' kullanıcısı ve 'Erdogan.2024!' şifresinin base64 halidir.
  # Kendi ürettiğiniz farklı bir şifre varsa Adım 1'deki çıktı ile burayı değiştirin.
  htpasswd: ZXJkb2dhbjokMmkkMDUkLm5GSEI4a2pGZjlqL05xTzRCLk51Lm51c2p2ZzZ1LjhnL0VqLmI0OEEvQnB2R1U4Z1h5N3E=
---
apiVersion: config.openshift.io/v1
kind: OAuth
metadata:
  name: cluster
spec:
  identityProviders:
    # Giriş ekranında görünecek isim burada belirlenir.
    - name: Local_User
      mappingMethod: claim
      type: HTPasswd
      htpasswd:
        fileData:
          name: htpasswd-secret

oc apply -f oauth-htpasswd.yaml
oc adm policy add-cluster-role-to-user cluster-admin erdogan

htpasswd -n -B -b <username> <password>
oc apply -f ~/ocp4-metal-install/manifest/oauth-htpasswd.yaml

```

Admin yetkisi verme:
```bash
oc adm policy add-cluster-role-to-user cluster-admin admin
```

Cluster operatör durumu:
```bash
oc get co
```

---

## 13. /etc/hosts Güncelleme
```bash
sudo vi /etc/hosts
```
Aşağıdaki satırı ekleyin:
```
10.125.0.254 ocp-svc api.ocplab.yargitay.gov.tr console-openshift-console.apps.ocplab.yargitay.gov.tr oauth-openshift.apps.ocplab.yargitay.gov.tr downloads-openshift-console.apps.ocplab.yargitay.gov.tr alertmanager-main-openshift-monitoring.apps.ocplab.yargitay.gov.tr grafana-openshift-monitoring.apps.ocplab.yargitay.gov.tr prometheus-k8s-openshift-monitoring.apps.ocplab.yargitay.gov.tr thanos-querier-openshift-monitoring.apps.ocplab.yargitay.gov.tr
```
```
watch -n5 oc get nodes
```
---

## Kurulum Tamamlandı
Artık cluster tamamen çalışır durumdadır. Gerektiğinde log, CSR ve node durumlarını kontrol ederek devam edebilirsiniz.

---

---

## 4. Bootstrap Removal Sonrası Cluster Health
```bash
oc get csr
oc get nodes
oc get co
oc get pods -A
```
Bootstrap kaldırıldıktan sonra tüm control-plane bileşenlerinin `AVAILABLE=True` olması gerekir.

```
oc edit schedulers.config.openshift.io cluster
```
```
spec:
  mastersSchedulable: false
```
Bu, OpenShift’in control plane node’ları üzerinde workload (pod) schedule edilmesini engelleyen ana parametredir.

## Bu alan:
## false olduğunda:

1. **Master node’ları üzerinde hiçbir workload pod schedule edilmez.**
2. **Sadece OpenShift’in kendi control plane component’leri çalışır.**
3. **Normal operasyonda önerilen değerdir.**

## true olduğunda:

1. **Master node’ları aynı zamanda worker gibi davranabilir.**
2. **Pod’lar master’a schedule olabilir.**
3. **Küçük cluster (3 node) veya test ortamı için tercih edilir.**
4. **Üretimde tavsiye edilmez (yük artınca API ve etcd performansı düşer).**
