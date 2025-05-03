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

﻿| Bootstrap 	RHCOS 	4 	16 GB 	100 GB 	300 |
| Control-plane 	RHCOS 	4 	16 GB 	100 GB 	300 |
| Compute |


| api.ocp.lab.local | 10.5.209.250 |
| api-int.ocp.lab.local | 10.5.209.250 |
| *.apps.ocp.lab.local  | 10.5.209.250 |
| bastion.ocp.lab.local | 10.5.209.250 - 00:50:56:86:bc:64 - 00505686bc64 |

| bootstrap.ocp.lab.local | 10.5.209.240  | 00:50:56:86:2a:09 | 005056862a09 |
| master01.ocp.lab.local  | 10.5.209.241  | 00:50:56:86:de:66 | 00505686de66 |
| master02.ocp.lab.local  | 10.5.209.242  | 00:50:56:86:b9:b8 | 00505686b9b8 |
| master03.ocp.lab.local  | 10.5.209.243  | 00:50:56:86:26:d4 | 0050568626d4 |

| worker01.ocp.lab.local | 10.5.209.251 | 00:50:56:86:f5:32 | 00505686f532 |
| worker02.ocp.lab.local | 10.5.209.252 | 00:50:56:86:39:25 | 005056863925 |

| quay.ocp.lab.local | 10.5.209.245 | 00:50:56:86:eb:52 | 00505686eb52 |
| gitlab.ocp.lab.local | 10.5.209.246 | 00:50:56:86:7b:ef | 005056867bef |
