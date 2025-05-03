# OpenShift Container Platform Installing a user-provisioned cluster on bare metal

##  User Provisioned Infrastructure (UPI) Install Documentation.

![alt text](images/OCP_Cover.jpg)

**Architecture Diagram**

**Requirements to be used for openshift bare metal installation.** <br/>
Retrieved from https://console.redhat.com/openshift/install/metal/user-provisioned 
OpenShift installer, Pull secret key to be used in the installation yaml file, Command line interface CoreOS (RHCOS) ISO files are downloaded.

![alt text](images/04.png)
![alt text](1.png)
![alt text](cluster2.png)

**Download Software**

oc get pods --all-namespaces <br/>
oc delete pods --field-selector=status.phase==Succeeded --all-namespaces

***********************************************************************************************

# OpenShift Container Platform (OCP) Kurulum Dokümanı

## Makine Özellikleri ve DNS Kayıtları

### Node Tipleri ve Kaynakları
| Tip          | OS     | CPU | RAM  | Boot Disk | Data Disk |
|--------------|--------|-----|------|-----------|-----------|
| Bootstrap    | RHCOS  | 4   | 16GB | 100GB     | 300GB     |
| Control-plane| RHCOS  | 4   | 16GB | 100GB     | 300GB     |
| Compute      | -      | -   | -    | -         | -         |

### DNS ve IP Yapılandırması
#### API ve Uygulama Giriş Noktaları
- `api.ocp.lab.local` - `10.5.209.250`
- `api-int.ocp.lab.local` - `10.5.209.250`
- `*.apps.ocp.lab.local` - `10.5.209.250`
- `bastion.ocp.lab.local` - `10.5.209.250` (MAC: `00:50:56:86:bc:64`)

#### Node'lar
- **Bootstrap**:  
  `bootstrap.ocp.lab.local` - `10.5.209.240` (MAC: `00:50:56:86:2a:09`)
- **Master Node'ları**:  
  `master01.ocp.lab.local` - `10.5.209.241`  
  `master02.ocp.lab.local` - `10.5.209.242`  
  `master03.ocp.lab.local` - `10.5.209.243`
- **Worker Node'ları**:  
  `worker01.ocp.lab.local` - `10.5.209.251`  
  `worker02.ocp.lab.local` - `10.5.209.252`
- **Servisler**:  
  `quay.ocp.lab.local` - `10.5.209.245`  
  `gitlab.ocp.lab.local` - `10.5.209.246`

#### Altyapı Sunucuları
- `DC-01` - `10.5.209.31`
- `DC-02` - `10.5.209.32`
- `DHCP-01` - `10.5.209.51`

---

## HAProxy Konfigürasyonu
```haproxy
global
  log         127.0.0.1 local2
  pidfile     /var/run/haproxy.pid
  maxconn     4000
  daemon

defaults
  mode                    http
  log                     global
  option                  dontlognull
  option http-server-close
  option                  redispatch
  retries                 3
  timeout http-request    10s
  timeout queue           1m
  timeout connect         10s
  timeout client          1m
  timeout server          1m
  timeout http-keep-alive 10s
  timeout check           10s
  maxconn                 3000

# API Server (6443)
listen api-server-6443
  bind *:6443
  mode tcp
  balance roundrobin
  server bootstrap bootstrap.ocp4.example.com:6443 check backup
  server master0 master0.ocp4.example.com:6443 check
  server master1 master1.ocp4.example.com:6443 check
  server master2 master2.ocp4.example.com:6443 check

# Machine Config Server (22623)
listen machine-config-server-22623
  bind *:22623
  mode tcp
  server bootstrap bootstrap.ocp4.example.com:22623 check backup
  server master0 master0.ocp4.example.com:22623 check
  server master1 master1.ocp4.example.com:22623 check
  server master2 master2.ocp4.example.com:22623 check

# Ingress Router (80/443)
listen ingress-router-80
  bind *:80
  mode tcp
  balance source
  server compute0 compute0.ocp4.example.com:80 check
  server compute1 compute1.ocp4.example.com:80 check
