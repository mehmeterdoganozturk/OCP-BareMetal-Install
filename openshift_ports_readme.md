# OpenShift 4 -- Kritik Portlar (Açıklamalı)

## 1. API ve Yönetim Portları

  Port             Protokol   Açıklama
  ---------------- ---------- ----------------------------------------------
  **6443**         TCP        Kubernetes API Server -- tüm yönetim trafiği
  **22623**        TCP        Machine Config Server -- Ignition dosyaları
  **2379--2380**   TCP        etcd cluster iletişimi

## 2. Node -- Control Plane Trafiği

  Port        Protokol   Açıklama
  ----------- ---------- -------------------------
  **10250**   TCP        Kubelet API
  **10257**   TCP        kube-controller-manager
  **10259**   TCP        kube-scheduler

## 3. Worker Node Uygulama Trafiği

  Port       Protokol   Açıklama
  ---------- ---------- ----------------------
  **80**     TCP        Router HTTP giriş
  **443**    TCP        Router HTTPS giriş
  **1936**   TCP        HAProxy router stats

## 4. Cluster Internal Servisleri

  Port       Protokol   Açıklama
  ---------- ---------- -----------------------
  **9000**   TCP        Machine-config-daemon
  **953**    TCP        DNS
  **5353**   UDP        mDNS discovery

## 5. Bootstrap Node Gereksinimleri

  Port        Protokol   Açıklama
  ----------- ---------- ------------------
  **22623**   TCP        master.ign çekme
  **6443**    TCP        Bootstrap → API
  **22**      TCP        SSH (opsiyonel)
  **67/68**   UDP        DHCP

## 6. Registry & Internal Services

  Port       Protokol   Açıklama
  ---------- ---------- -------------------
  **5000**   TCP        Internal registry
  **3128**   TCP        Proxy
  **53**     TCP/UDP    DNS

## 7. Load Balancer Portları

### API VIP

-   **6443**\
-   **22623**

### Apps VIP

-   **80**\
-   **443**
