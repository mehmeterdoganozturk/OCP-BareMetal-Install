
# HAProxy + Kubernetes Cluster Yönetimi

Bu döküman, HAProxy'nin 3 master ve 3 worker node'lu bir Kubernetes cluster ile yönetimi ve erişimi için yapılan temel işlemleri özetler.

---

## 🛠️ Ortam Genel Bilgileri

- **Cluster:** 3 Master, 3 Worker node
- **HAProxy Sunucusu:** API isteklerini ve ingress trafiğini yönlendirir
- **OS:** Ubuntu tabanlı sunucular
- **HAProxy:** TCP modunda çalışıyor, backendlerde worker node’lar var

---

## 🔑 HAProxy TCP Konfigürasyonu (Örnek)

`/etc/haproxy/haproxy.cfg` içerisinde:

```haproxy
frontend nginx-frontend
    bind *:30080
    mode tcp
    option tcplog
    default_backend nginx-backend

backend nginx-backend
    mode tcp
    balance roundrobin
    option tcp-check
    default-server inter 3s fall 3 rise 2
    server worker01 10.5.209.251:30080 check
    server worker02 10.5.209.252:30080 check
    server worker03 10.5.209.253:30080 check
```

Bu yapılandırma, HAProxy'nin worker node’lara istekleri round-robin olarak yönlendirmesini sağlar.

---

## 📡 HAProxy Admin Socket Kullanımı

Admin socket ile durum görüntülemek için önce `socat` yüklenir:

```bash
sudo apt update && sudo apt install socat -y
```

Socket üzerinden komut çalıştırma:

```bash
echo "show info" | sudo socat unix-connect:/run/haproxy-master.sock stdio
```

> Eğer `Unknown command` hatası alırsanız `help` komutu ile kullanılabilir komutları görebilirsiniz.

---

## 🧩 Kubectl Kullanımı (HAProxy Üzerinden)

`.kube/config` dosyası HAProxy sunucusuna kopyalandıktan sonra `kubectl` kurulumu:

```bash
curl -LO "https://dl.k8s.io/release/$(curl -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
```

`kubectl` sürüm kontrolü:

```bash
kubectl version --client
```

`kubectl` için kubeconfig ayarı:

```bash
export KUBECONFIG=/home/ubuntu/.kube/config
echo 'export KUBECONFIG=/home/ubuntu/.kube/config' >> ~/.bashrc
source ~/.bashrc
```

Node'ları listeleme:

```bash
kubectl get nodes
```

> Eğer `connection refused` hatası alırsanız kubeconfig yolunu kontrol edin.

---

## 🔍 Kontrol Noktaları

- `kubectl get nodes` çıktısı tüm master ve worker node’ları listeliyor mu?
- HAProxy loglarında backend health check durumları doğru görünüyor mu?
- Admin socket üzerinden `show info`, `show stat` komutları düzgün çalışıyor mu?

---

Bu rehber ile HAProxy üzerinden Kubernetes cluster yönetimi yapılabilir hale gelir.
