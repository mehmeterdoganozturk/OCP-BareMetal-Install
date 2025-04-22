---
title: Java Manual install
updated: 2024-06-23 14:28:09Z
created: 2024-06-23 14:28:08Z
latitude: 39.93336350
longitude: 32.85974190
altitude: 0.0000
---

   Java Manual install
   
   tar -xvf jdk-8-linux-x64.tar.gz (64-bit)

    The JDK 8 package is extracted into ./jdk1.8.0 directory. N.B.: Check carefully this folder name since Oracle seem to change this occasionally with each update.

    Now move the JDK 8 directory to /usr/local/jre1.8.0_361

    Now run

sudo update-alternatives --install "/usr/bin/java" "java" "/usr/local/jre1.8.0_361/bin/java" 1
sudo update-alternatives --install "/usr/bin/javac" "javac" "/usr/local/jre1.8.0_361/bin/javac" 1
sudo update-alternatives --install "/usr/bin/javaws" "javaws" "/usr/local/jre1.8.0_361/bin/javaws" 1

sudo update-alternatives --set java /usr/local/jre1.8.0_361/bin/java
sudo update-alternatives --set javaws /usr/local/jre1.8.0_361/bin/javaws
sudo update-alternatives --set javac /usr/local/jre1.8.0_361/bin/javac

    This will assign Oracle JDK a priority of 1, which means that installing other JDKs will replace it as the default. Be sure to use a higher priority if you want Oracle JDK to remain the default.

    Correct the file ownership and the permissions of the executables:

sudo chmod a+x /usr/bin/java
sudo chmod a+x /usr/bin/javac
sudo chmod a+x /usr/bin/javaws
sudo chown -R root:root /usr/lib/jvm/jre1.8.0_361

    N.B.: Remember - Java JDK has many more executables that you can similarly install as above. java, javac, javaws are probably the most frequently required. This answer lists the other executables available.

    Run

    sudo update-alternatives --config java

    You will see output similar to the one below - choose the number of jdk1.8.0 - for example 3 in this list (unless you have have never installed Java installed in your computer in which case a sentence saying "There is nothing to configure" will appear):

    $ sudo update-alternatives --config java
    There are 3 choices for the alternative java (providing /usr/bin/java).

      Selection    Path                                            Priority   Status
    ------------------------------------------------------------
      0            /usr/lib/jvm/java-7-openjdk-amd64/jre/bin/java   1071      auto mode
      1            /usr/lib/jvm/java-7-openjdk-amd64/jre/bin/java   1071      manual mode
    * 2            /usr/lib/jvm/jdk1.7.0/bin/java                   1         manual mode
      3            /usr/lib/jvm/jdk1.8.0/bin/java                   1         manual mode

    Press enter to keep the current choice[*], or type selection number: 3
    update-alternatives: using /usr/lib/jvm/jdk1.8.0/bin/java to provide /usr/bin/java (java) in manual mode

    Repeat the above for:

    sudo update-alternatives --config javac
    sudo update-alternatives --config javaws