# OpenShift Container Platform Installing a user-provisioned cluster on bare metal

##  User Provisioned Infrastructure (UPI) Install Documentation.

![alt text](images/OCP_Cover.jpg)

**Architecture Diagram**

**Requirements to be used for openshift bare metal installation.** <br/>
Retrieved from https://console.redhat.com/openshift/install/metal/user-provisioned 
OpenShift installer, Pull secret key to be used in the installation yaml file, Command line interface CoreOS (RHCOS) ISO files are downloaded.

![alt text](./images/cluster01.png)
![alt text](./images/cluster02.png)
![alt text](./images/cluster03.png)

**Download Software**

oc get pods --all-namespaces <br/>
oc delete pods --field-selector=status.phase==Succeeded --all-namespaces
