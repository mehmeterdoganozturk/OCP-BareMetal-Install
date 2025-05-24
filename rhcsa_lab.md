# **RHCSA Sınavı İçin Sanal Lab Ortamı Yapılandırması**

Bu belge, Red Hat Certified System Administrator (RHCSA) EX200 sınavına hazırlık amacıyla kurulan sanal lab ortamının ve özellikle Bastion Host'un temel yapılandırmasını içermektedir.

## **Ortam Topolojisi**

Sanal lab ortamınız aşağıdaki bileşenlerden oluşmaktadır:

* **Bastion Host (Red Hat Enterprise Linux 9\)**:  
  * İki ağ arayüzü:  
    * enp1s0: NAT bağlantısı üzerinden internete çıkış. DHCP ile otomatik IP alır (örn: 192.168.122.222).  
    * enp2s0: İç Ağ (Lab) bağlantısı. Statik IP: 192.168.100.1/24. İç ağdaki sunucular için varsayılan ağ geçidi (Gateway) görevi görür.  
  * Kullanıcılar: erdogan (sudo yetkili), root.  
* **İç Ağ Sunucuları (servera, serverb, workstation)**:  
  * Her bir sunucuda tek ağ arayüzü (enp1s0) bulunur.  
  * servera: 192.168.100.10/24  
  * serverb: 192.168.100.11/24  
  * workstation: 192.168.100.12/24  
  * **Gateway (Tüm İç Ağ Sunucuları İçin)**: 192.168.100.1 (Bastion Host'un iç IP'si).  
  * **DNS (Tüm İç Ağ Sunucuları İçin)**: 8.8.8.8 (veya tercih edilen başka bir genel DNS sunucusu).  
  * Tüm iç ağ sunucularından Bastion Host'a SSH erişimi mümkündür.

## **Bastion Host (RHEL 9\) Detaylı Yapılandırma Adımları**

Bu adımlar, Bastion Host üzerinde sudo yetkisine sahip erdogan kullanıcısı ile veya doğrudan root kullanıcısı ile yapılmalıdır.

### **1\. Ağ Arayüzlerini Doğrulama ve Kalıcı Yapılandırma**

Arayüzlerin doğru IP adreslerine sahip olduğundan ve sistem yeniden başlatıldığında da kalıcı olduğundan emin olun.  
\# Geçerli ağ bağlantılarını kontrol edin  
nmcli connection show

\# enp2s0 için statik IP yapılandırması (eğer yapılmadıysa veya kalıcı değilse)  
\# Not: Eğer 'enp2s0' için daha önce bir bağlantı oluşturduysanız, önce onu silmeniz gerekebilir:  
\# sudo nmcli connection del \<mevcut\_enp2s0\_baglanti\_adi\>

sudo nmcli connection add type ethernet con-name enp2s0-internal ifname enp2s0 ip4 192.168.100.1/24  
sudo nmcli connection modify enp2s0-internal connection.autoconnect yes \# Sistem açılışında otomatik bağlan  
sudo nmcli connection up enp2s0-internal \# Bağlantıyı aktive et

\# enp1s0 (NAT/Internet) için DHCP bağlantısını kontrol edin  
\# Bu arayüz genellikle varsayılan olarak DHCP ile otomatik IP alır ve kalıcıdır.  
\# Sadece doğrulamak için:  
\# nmcli device status  
\# nmcli connection show enp1s0 (veya ilgili bağlantı adı)

\# Kontrol için IP adreslerini listeleyin  
ip a

### **2\. IP Yönlendirmeyi (IP Forwarding) Etkinleştirme**

İç ağdaki sunucuların internete çıkabilmesi için Bastion Host'un IP paketlerini yönlendirmesi gerekir.  
\# Geçici olarak IP yönlendirmeyi etkinleştir  
sudo sysctl \-w net.ipv4.ip\_forward=1

\# Kalıcı hale getir (sistem yeniden başlatıldığında da aktif kalır)  
echo "net.ipv4.ip\_forward \= 1" | sudo tee /etc/sysctl.d/99-ip-forward.conf \> /dev/null  
sudo sysctl \-p /etc/sysctl.d/99-ip-forward.conf

\# Kontrol  
sysctl net.ipv4.ip\_forward \# Çıktı "net.ipv4.ip\_forward \= 1" olmalı

### **3\. Güvenlik Duvarı (Firewalld) Yapılandırması ve NAT (Masquerading)**

Firewalld, RHEL'de varsayılan güvenlik duvarıdır. İç ağdan internete çıkış için NAT (Network Address Translation) sağlamak ve Bastion Host'u korumak kritik öneme sahiptir.  
\# Firewalld servisinin çalıştığından emin olun  
sudo systemctl status firewalld  
sudo systemctl enable \--now firewalld

\# 1\. Ağ Arayüzlerini Doğru Zone'lara Ata:  
\# Dış arayüzü (enp1s0) 'external' zone'a ekle  
sudo firewall-cmd \--zone=external \--add-interface=enp1s0 \--permanent

\# İç arayüzü (enp2s0) 'internal' zone'a ekle  
sudo firewall-cmd \--zone=internal \--add-interface=enp2s0 \--permanent

\# 2\. 'external' zone'da Masquerading'i (NAT) etkinleştir:  
\# Bu, iç ağdan gelen trafiğin Bastion'un dış IP'si ile internete çıkmasını sağlar.  
sudo firewall-cmd \--zone=external \--add-masquerade \--permanent

\# 3\. 'internal' zone'un varsayılan hedefinin trafiği kabul etmesini sağla:  
\# Bu, iç ağdan gelen paketlerin Firewalld tarafından yönlendirilmesine izin verir.  
\# Bu adım, "Packet filtered" hatasının çözümüdür.  
sudo firewall-cmd \--zone=internal \--set-target=ACCEPT \--permanent

\# 4\. Gerekirse SSH servisine izin ver (Bastion Host'a erişim için):  
\# Genellikle 'internal' zone'da SSH varsayılan olarak açıktır.  
sudo firewall-cmd \--zone=internal \--add-service=ssh \--permanent

\# 5\. Tüm değişiklikleri uygulamak için Firewalld'ı yeniden yükle:  
sudo firewall-cmd \--reload

\# 6\. Kontrol (Çıktılarda 'forward: yes', 'masquerade: yes' ve 'target: ACCEPT' olduğundan emin olun)  
sudo firewall-cmd \--list-all \--zone=internal  
sudo firewall-cmd \--list-all \--zone=external

### **4\. Kullanıcı ve İzin Yönetimi (Bastion Host Üzerinde)**

erdogan kullanıcınızın sudo yetkisine sahip olduğundan emin olun.  
\# erdogan kullanıcısının gruplarını kontrol edin (çıktıda 'wheel' grubu olmalı)  
id erdogan

\# Eğer 'wheel' grubunda değilse, ekleyin:  
\# sudo usermod \-aG wheel erdogan

\# /etc/sudoers dosyasında 'wheel' grubunun sudo yetkisi olduğundan emin olun:  
\# sudo visudo \# veya sudo cat /etc/sudoers  
\# Aşağıdaki satırın uncomment edilmiş (başında \# olmayan) olduğundan emin olun:  
\# %wheel  ALL=(ALL)       ALL

### **5\. SELinux Konfigürasyonu**

SELinux, RHEL'de varsayılan olarak enforcing modda çalışır ve sistem güvenliğini artırır. RHCSA sınavı için SELinux'u kapatmayın, aksine nasıl yönetileceğini öğrenin.  
\# SELinux durumunu kontrol edin  
getenforce \# Çıktı "Enforcing" olmalı

\# Eğer "Permissive" veya "Disabled" ise, "Enforcing" moda geçirmek için (sistem yeniden başlatma gerektirebilir):  
\# sudo sed \-i 's/SELINUX=permissive/SELINUX=enforcing/' /etc/selinux/config  
\# VEYA  
\# sudo sed \-i 's/SELINUX=disabled/SELINUX=enforcing/' /etc/selinux/config  
\# sudo reboot

## **İç Ağ Sunucuları (servera, serverb, workstation) Yapılandırması**

Her bir iç ağ sunucusu üzerinde aşağıdaki ağ ayarlarını yapın. Bu sunucuların Bastion Host üzerinden internete çıkabilmesi için Gateway'leri Bastion'un iç IP'si olmalıdır.  
\# Örnek olarak 'servera' için (IP adresi 192.168.100.10 olarak varsayılmıştır)  
\# Diğer sunucular için IP adreslerini (192.168.100.11, 192.168.100.12) buna göre değiştirin.

\# Ağ bağlantısını oluşturun ve IP, Gateway, DNS bilgilerini tanımlayın  
\# Not: Eğer 'enp1s0' için daha önce bir bağlantı oluşturduysanız, önce onu silmeniz gerekebilir.  
\# sudo nmcli connection del \<mevcut\_enp1s0\_baglanti\_adi\>

sudo nmcli connection add type ethernet con-name enp1s0-internal ifname enp1s0 ip4 192.168.100.10/24 gw4 192.168.100.1  
sudo nmcli connection modify enp1s0-internal ipv4.dns "8.8.8.8" \# Genel DNS  
sudo nmcli connection modify enp1s0-internal connection.autoconnect yes \# Sistem açılışında otomatik bağlan

\# Bağlantıyı aktive edin  
sudo nmcli connection up enp1s0-internal

\# Kontrol  
ip a \# Doğru IP adresini kontrol edin  
ip r \# Varsayılan Gateway'in 192.168.100.1 olduğunu kontrol edin  
cat /etc/resolv.conf \# nameserver 8.8.8.8 olduğunu kontrol edin

## **Ortam Testleri**

Tüm yapılandırmaları tamamladıktan sonra, ağ bağlantısını test edin:

* **Bastion Host'tan:**  
  * İnternet erişimi: ping google.com  
  * İç ağdaki sunuculara erişim: ping 192.168.100.10 (servera)  
  * İç ağdaki sunuculara SSH: ssh servera (veya IP adresiyle)  
* **İç Ağ Sunucularından (servera, serverb, workstation):**  
  * İnternet erişimi: ping 8.8.8.8  
  * İnternet erişimi (isim çözümleme ile): ping google.com  
  * Bastion Host'a SSH: ssh 192.168.100.1 (Bastion Host)  
  * Diğer iç ağ sunucularına SSH: ssh serverb

Bu yapılandırma, RHCSA sınavı için sağlam bir temel sunar ve ağ konularında pratik yapmanızı sağlar. Bol pratikle başarılar dilerim\!
