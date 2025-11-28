# CRC (CodeReady Containers) Kurulum ve Yeniden Boyutlandırma Notları

## 1️⃣ CRC Kurulum Ayarları

Bellek ve CPU yapılandırmasını ayarla:

```
crc config set memory 32768
crc config set cpus 10
```

CRC'yi başlat:

```
crc start --pull-secret-file /home/erdogan/Downloads/crc/pull-secret.txt
```

---

## 2️⃣ Disk Alanını Büyütme (crc.qcow2)

CRC'yi durdur:

```
crc stop
```

qcow2 diskini büyüt:

```
qemu-img resize ~/.crc/machines/crc/crc.qcow2 80G
```

CRC'yi yeniden başlat:

```
crc start --pull-secret-file /home/erdogan/Downloads/crc/pull-secret.txt
```

---

## 3️⃣ Tam Temizlik (Full Reset)

```
crc stop
crc delete -f
crc cleanup
rm -Rf ~/.crc
rm -f ~/.kube/config
```

Bu adımlar CRC’yi tamamen temizleyip sıfırdan kurulum için hazır hale getirir.
