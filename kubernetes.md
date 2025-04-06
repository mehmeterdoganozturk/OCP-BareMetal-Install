# Ubuntu 24.04 Üzerine HA Kubernetes Cluster Kurulumu (3 Master, 3 Worker + HAProxy)

Bu doküman, Ubuntu 24.04 LTS sunucular kullanarak 3 Master ve 3 Worker düğümden oluşan, önünde HAProxy ile yüksek erişilebilirlik (HA) sağlayan bir Kubernetes cluster'ının `kubeadm` ile nasıl kurulacağını adım adım açıklamaktadır.

## Genel Mimari

* **HAProxy (1 Sunucu/VM):** Kubernetes API sunucusuna (Kontrol Düzlemi - Port 6443) gelen istekleri 3 Master düğüme dağıtarak kontrol düzlemi için HA sağlar.
* **Master Düğümler (3 Sunucu/VM):** Kubernetes kontrol düzlemi bileşenlerini (API Server, etcd, Scheduler, Controller Manager) çalıştırır.
* **Worker Düğümler (3 Sunucu/VM):** Uygulama Pod'larını barındırır (Kubelet, Kube-proxy çalışır).

## Önemli Notlar

* Bu kılavuz `kubeadm` kullanır. Farklı kurulum yöntemleri mevcuttur.
* Komutlar `root` veya `sudo` yetkisi gerektirir.
* Tüm sunucuların belirtilen portlar üzerinden birbirleriyle ve HAProxy ile iletişim kurabildiğinden emin olun (Firewall ayarları).
* IP adreslerini ve hostname'leri kendi ortamınıza göre **mutlaka** değiştirin.
* Kuruluma başlamadan önce tüm sunucuların güncel olduğundan emin olun (`sudo apt update && sudo apt upgrade -y`).

## Adım 1: Ön Hazırlıklar (Tüm Sunucular: HAProxy, 3 Master, 3 Worker)

1.  **İşletim Sistemi:** Ubuntu 24.04 LTS Server.
2.  **Statik IP Adresleri (Örnek):**
    * HAProxy: `192.168.57.100` (**API Endpoint**)
    * Master 1: `192.168.57.101`
    * Master 2: `192.168.57.102`
    * Master 3: `192.168.57.103`
    * Worker 1: `192.168.57.111`
    * Worker 2: `192.168.57.112`
    * Worker 3: `192.168.57.113`


    ```
    alias update='sudo apt-get update && sudo apt-get upgrade -y && sudo apt-get dist-upgrade -y && sudo apt autoremove -y'
    ```

    ```
    sudo hostnamectl set-hostname haproxy.lab.local
    sudo hostnamectl set-hostname master1.lab.local
    sudo hostnamectl set-hostname master2.lab.local
    sudo hostnamectl set-hostname master3.lab.local
    sudo hostnamectl set-hostname worker1.lab.local
    sudo hostnamectl set-hostname worker2.lab.local
    sudo hostnamectl set-hostname worker3.lab.local
    ```
3.  **Hostname & /etc/hosts Ayarları:** Her sunucuya anlamlı bir hostname verin ve tüm sunuculardaki `/etc/hosts` dosyasını aşağıdaki gibi güncelleyin:
    ```
    192.168.57.100 haproxy haproxy.lab.local
    192.168.57.101 master1 master1.lab.local
    192.168.57.102 master2 master2.lab.local
    192.168.57.103 master3 master3.lab.local
    192.168.57.111 worker1 worker1.lab.local
    192.168.57.112 worker2 worker2.lab.local
    192.168.57.113 worker3 worker3.lab.local
    ```
4.  **Gerekli Portlar (Firewall İzinleri):**
    * **Master Düğümler:** TCP 6443, 2379-2380, 10250, 10257, 10259
    * **Worker Düğümler:** TCP 10250, 30000-32767 (NodePort Aralığı)
    * **HAProxy:** TCP 6443 (Gelen)
    * **Tüm Düğümler (CNI - Örnek Calico):** TCP 179, UDP 4789, IP Protocol 4

## Adım 2: HAProxy Kurulumu ve Yapılandırması (HAProxy Sunucusu Üzerinde)

**Not:** HAProxy sadece **Master** düğümlerin API sunucusuna (6443) erişimi dengelemek içindir. Worker düğümler buraya **eklenmez**.

1.  **Kurulum:**
    ```bash
    sudo apt update
    sudo apt install haproxy -y
    ```
2.  **Yapılandırma (`/etc/haproxy/haproxy.cfg`):**
    ```cfg
    global
        log /dev/log    local0
        log /dev/log    local1 notice
        chroot /var/lib/haproxy
        stats socket /run/haproxy/admin.sock mode 660 level admin expose-fd listeners
        stats timeout 30s
        user haproxy
        group haproxy
        daemon

    defaults
        log     global
        mode    tcp
        option  tcplog
        option  dontlognull
        timeout connect 5000
        timeout client  50000
        timeout server  50000

    frontend kubernetes-api
        bind *:6443
        mode tcp
        option tcplog
        default_backend kubernetes-master-nodes

    backend kubernetes-master-nodes
        mode tcp
        option tcplog
        option tcp-check
        balance roundrobin
        # Kendi Master IP adreslerinizi buraya girin:
        server master1 192.168.57.101:6443 check fall 3 rise 2
        server master2 192.168.57.102:6443 check fall 3 rise 2
        server master3 192.168.57.103:6443 check fall 3 rise 2
    ```
3.  **Servis Yönetimi:**
    ```bash
    sudo systemctl enable haproxy
    sudo systemctl restart haproxy
    sudo systemctl status haproxy
    ```

## Adım 3: Kubernetes Bileşenleri İçin Hazırlık (Tüm Master ve Worker Düğümler Üzerinde)

Aşağıdaki adımları **tüm 6 Kubernetes düğümünde** gerçekleştirin:

1.  **Swap'ı Devre Dışı Bırakma:**
    ```bash
    sudo swapoff -a
    sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
    ```
2.  **Kernel Modüllerini Yükleme:**
    ```bash
    sudo modprobe overlay
    sudo modprobe br_netfilter

    cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
    overlay
    br_netfilter
    EOF
    ```
3.  **Sysctl Ayarları:**
    ```bash
    cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
    net.bridge.bridge-nf-call-iptables  = 1
    net.bridge.bridge-nf-call-ip6tables = 1
    net.ipv4.ip_forward                 = 1
    EOF

    sudo sysctl --system
    ```
4.  **Container Runtime Kurulumu (containerd):**
    ```bash
    sudo apt update
    sudo apt install containerd -y

    sudo mkdir -p /etc/containerd
    sudo containerd config default | sudo tee /etc/containerd/config.toml
    sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/g' /etc/containerd/config.toml

    sudo systemctl restart containerd
    sudo systemctl enable containerd
    ```
5.  **Kubernetes Paketlerini Kurma (kubeadm, kubelet, kubectl):**
    ```bash
    sudo apt update
    sudo apt install -y apt-transport-https ca-certificates curl gpg

    # Google Cloud public signing key
    curl -fsSL [https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key](https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key) | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

    # Kubernetes apt repository (v1.29)
    echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] [https://pkgs.k8s.io/core:/stable:/v1.29/deb/](https://pkgs.k8s.io/core:/stable:/v1.29/deb/) /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

    sudo apt update
    sudo apt install -y kubelet kubeadm kubectl
    sudo apt-mark hold kubelet kubeadm kubectl # Versiyonları sabitle
    ```

## Adım 4: Kontrol Düzlemini Başlatma (Sadece İlk Master Düğüm - master1 Üzerinde)

1.  **kubeadm init:**
    ```bash
    # HAProxy IP'nizi ve seçtiğiniz Pod Network CIDR'ını kullanın (Calico için 10.244.0.0/16)
    sudo kubeadm init \
      --control-plane-endpoint "192.168.57.100:6443" \
      --upload-certs \
      --pod-network-cidr=10.244.0.0/16
    ```
2.  **ÖNEMLİ:** Komut çıktısındaki `kubeadm join` komutlarını (hem master hem worker için) ve `kubectl` yapılandırma adımlarını **kaydedin**.
3.  **kubectl'i Yapılandır:** (Çıktıdaki adımları uygulayın)
    ```bash
    mkdir -p $HOME/.kube
    sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
    sudo chown $(id -u):$(id -g) $HOME/.kube/config
    ```

## Adım 5: Diğer Master Düğümleri Cluster'a Ekleme (master2 ve master3 Üzerinde)

1.  `master1`'deki `kubeadm init` çıktısından aldığınız **Master join komutunu** `master2` ve `master3` üzerinde `sudo` ile çalıştırın. Komut şuna benzer olacaktır:
    ```bash
    # ÖRNEKTİR! Kendi çıktınızdaki komutu kullanın!
    sudo kubeadm join 192.168.57.100:6443 --token <token> \
        --discovery-token-ca-cert-hash sha256:<hash> \
        --control-plane --certificate-key <certificate_key>
    ```

## Adım 6: Pod Network (CNI) Kurulumu (Sadece Bir Master Düğüm Üzerinde)

1.  Bir CNI eklentisi (örneğin Calico) kurun. `kubectl` yetkisi olan bir master düğümde çalıştırın:
    ```bash
    # Calico manifest URL'si için resmi belgeleri kontrol edin
    kubectl apply -f [https://raw.githubusercontent.com/projectcalico/calico/v3.27/manifests/calico.yaml](https://raw.githubusercontent.com/projectcalico/calico/v3.27/manifests/calico.yaml)
    ```

## Adım 7: Worker Düğümleri Cluster'a Ekleme (worker1, worker2, worker3 Üzerinde)

1.  `master1`'deki `kubeadm init` çıktısından aldığınız **Worker join komutunu** `worker1`, `worker2` ve `worker3` üzerinde `sudo` ile çalıştırın. Komut şuna benzer olacaktır:
    ```bash
    # ÖRNEKTİR! Kendi çıktınızdaki komutu kullanın!
    sudo kubeadm join 192.168.57.100:6443 --token <token> \
        --discovery-token-ca-cert-hash sha256:<hash>
    ```

## Adım 8: Kurulumu Doğrulama (Bir Master Düğüm Üzerinde)

1.  **Düğüm Durumu:**
    ```bash
    kubectl get nodes -o wide
    ```
    Tüm düğümlerin `STATUS` sütununda `Ready` yazdığından emin olun.
2.  **Pod Durumu:**
    ```bash
    kubectl get pods -A
    ```
    Tüm pod'ların `STATUS` sütununda `Running` veya `Completed` yazdığından emin olun (Başlamaları biraz zaman alabilir).

## CPU, RAM ve SSD Disk Kullanımı Önerileri

* **CPU/RAM:**
    * **Master:** Min 2 CPU/4GB RAM, **Önerilen 4+ CPU/8+ GB RAM**.
    * **Worker:** Min 2 CPU/4GB RAM, **Önerilen:** Uygulama yüküne bağlıdır.
    * **HAProxy:** Genellikle 1-2 CPU/1-2 GB RAM yeterlidir.
* **SSD Disk Kullanımı:**
    * **Master:** `/var/lib/etcd` için **Kesinlikle önerilir!** Cluster performansı ve kararlılığı için kritiktir.
    * **Worker:** `/var/lib/kubelet`, `/var/lib/containerd` ve I/O yoğun uygulamalar için **Önerilir/Gereklidir**.
    * **HAProxy:** Genellikle zorunlu değildir.

## Sonraki Adımlar ve Önemli Konular

* **Ingress Controller:** Uygulamalara dışarıdan HTTP/S erişimi için (Nginx Ingress, Traefik vb.).
* **Depolama (Storage):** Kalıcı veriler için Persistent Volumes (PV), Storage Classes (SC) (NFS, Ceph vb.).
* **Monitoring ve Logging:** Prometheus/Grafana, EFK/PLG Stack.
* **Backup ve Restore:** `etcd` yedeği, Velero.
* **Güvenlik:** Network Policies, RBAC, Secrets yönetimi.

Bu adımları takip ederek HA Kubernetes cluster'ınızı başarıyla kurabilirsiniz.
