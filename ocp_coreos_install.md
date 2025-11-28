# OpenShift 4 Bare-Metal CoreOS Installer Komutları

Bu döküman, bootstrap, master ve worker nodelar için CoreOS kurulum komutlarını ve kurulum sonrası adımları içerir.

---

## 🔵 1. Bootstrap Node Kurulumu

```
sudo coreos-installer install /dev/sda   --ignition-url=http://10.5.209.250:8080/ocp4/bootstrap.ign   --insecure --insecure-ignition   --append-karg="ip=10.5.209.240::10.5.209.1:255.255.255.0:bootstrap.ocp.teoman.local:ens18:off nameserver=10.5.209.140"   --append-karg=rd.neednet=1
```

---

## 🔵 2. Master Node Kurulumları

### Master 01

```
sudo coreos-installer install /dev/sda   --ignition-url=http://10.5.209.250:8080/ocp4/master.ign   --insecure --insecure-ignition   --append-karg="ip=10.5.209.241::10.5.209.1:255.255.255.0:master01.ocp.teoman.local:ens18:off nameserver=10.5.209.140"   --append-karg=rd.neednet=1
```

### Master 02

```
sudo coreos-installer install /dev/sda   --ignition-url=http://10.5.209.250:8080/ocp4/master.ign   --insecure --insecure-ignition   --append-karg="ip=10.5.209.242::10.5.209.1:255.255.255.0:master02.ocp.teoman.local:ens18:off nameserver=10.5.209.140"   --append-karg=rd.neednet=1
```

### Master 03

```
sudo coreos-installer install /dev/sda   --ignition-url=http://10.5.209.250:8080/ocp4/master.ign   --insecure --insecure-ignition   --append-karg="ip=10.5.209.243::10.5.209.1:255.255.255.0:master03.ocp.teoman.local:ens18:off nameserver=10.5.209.140"   --append-karg=rd.neednet=1
```

---

## 🟢 3. Worker Node Kurulumları

### Worker 01
```
sudo coreos-installer install /dev/sda   --ignition-url=http://10.5.209.250:8080/ocp4/worker.ign   --insecure --insecure-ignition   --append-karg="ip=10.5.209.251::10.5.209.1:255.255.255.0:worker01.ocp.teoman.local:ens18:off nameserver=10.5.209.140"   --append-karg=rd.neednet=1
```

### Worker 02
```
sudo coreos-installer install /dev/sda   --ignition-url=http://10.5.209.250:8080/ocp4/worker.ign   --insecure --insecure-ignition   --append-karg="ip=10.5.209.252::10.5.209.1:255.255.255.0:worker02.ocp.teoman.local:ens18:off nameserver=10.5.209.140"   --append-karg=rd.neednet=1
```

### Worker 03
```
sudo coreos-installer install /dev/sda   --ignition-url=http://10.5.209.250:8080/ocp4/worker.ign   --insecure --insecure-ignition   --append-karg="ip=10.5.209.253::10.5.209.1:255.255.255.0:worker03.ocp.teoman.local:ens18:off nameserver=10.5.209.140"   --append-karg=rd.neednet=1
```

---

## 🧩 4. OpenShift Manifests ve Ignition Dosyalarını Oluşturma

```
~/openshift-install create manifests --dir ~/ocp-install
~/openshift-install create ignition-configs --dir ~/ocp-install/
```

---

## 🖥️ 5. Bastion Node Üzerinde Kurulum Takibi

### Bootstrap tamamlanmasını bekle:
```
./openshift-install wait-for bootstrap-complete --dir ~/ocp-install
```

### Kurulumun tamamen bitmesini bekle:
```
./openshift-install wait-for install-complete --dir ~/ocp-install
```

### API sürümünü izlemek için:
```
watch -n 30 'curl -ks https://api.ocp.teoman.local:6443/version'
```

---

## 🧪 6. Master Node Üzerinde kube-apiserver Kontrolü

```
sudo crictl ps | grep kube-apiserver
```

Bu çıktı master node’un kube-apiserver podunun çalıştığını doğrular.

---

