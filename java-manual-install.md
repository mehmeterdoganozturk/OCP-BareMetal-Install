
# ☕ Java 8 Manual Installation (update-alternatives ile)

Oracle JDK 8'in manuel olarak kurulumunu ve sistem genelinde kullanılabilir hale getirilmesini sağlayan adımlar aşağıda detaylı şekilde verilmiştir.

---

## 📦 Arşivi Açın

```bash
tar -xvf jdk-8-linux-x64.tar.gz
```

> 📁 Arşivden çıkan klasör genellikle `jdk1.8.0_XXX` şeklinde adlandırılır. Oracle sürümüne göre bu isim değişebilir, kontrol edin.

---

## 🚚 Klasörü Taşıyın

```bash
sudo mv jdk1.8.0_361 /usr/local/jre1.8.0_361
```

---

## 🔗 update-alternatives ile Bağlantıları Oluşturun

```bash
sudo update-alternatives --install "/usr/bin/java" "java" "/usr/local/jre1.8.0_361/bin/java" 1
sudo update-alternatives --install "/usr/bin/javac" "javac" "/usr/local/jre1.8.0_361/bin/javac" 1
sudo update-alternatives --install "/usr/bin/javaws" "javaws" "/usr/local/jre1.8.0_361/bin/javaws" 1
```

---

## ✅ Java'yı Varsayılan Olarak Ayarlayın

```bash
sudo update-alternatives --set java /usr/local/jre1.8.0_361/bin/java
sudo update-alternatives --set javac /usr/local/jre1.8.0_361/bin/javac
sudo update-alternatives --set javaws /usr/local/jre1.8.0_361/bin/javaws
```

> 🔐 Alternatif olarak daha yüksek bir `priority` değeri verirseniz Oracle JDK sistemde öncelikli olur.

---

## 🔐 İzinleri Düzenleyin

```bash
sudo chmod a+x /usr/bin/java
sudo chmod a+x /usr/bin/javac
sudo chmod a+x /usr/bin/javaws
sudo chown -R root:root /usr/local/jre1.8.0_361
```

---

## ⚙️ Java Versiyonu Seçme (İsteğe Bağlı)

```bash
sudo update-alternatives --config java
```

```text
There are 3 choices for the alternative java (providing /usr/bin/java).

  Selection    Path                                            Priority   Status
------------------------------------------------------------
  0            /usr/lib/jvm/java-7-openjdk-amd64/jre/bin/java   1071      auto mode
  1            /usr/lib/jvm/java-7-openjdk-amd64/jre/bin/java   1071      manual mode
* 2            /usr/lib/jvm/jdk1.7.0/bin/java                   1         manual mode
  3            /usr/local/jre1.8.0_361/bin/java                 1         manual mode
```

> Kullanmak istediğiniz Java sürümünün numarasını girin (örnek: `3`).

---

## 🔁 Diğer Bileşenler için de Aynısını Yapın

```bash
sudo update-alternatives --config javac
sudo update-alternatives --config javaws
```

---

📝 **Not:** `java`, `javac`, `javaws` dışında da pek çok yürütülebilir dosya (`jconsole`, `jps`, `jvisualvm` vb.) aynı şekilde ayarlanabilir.
