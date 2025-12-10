# ---

**OpenShift MetalLB Cookbook**

This repository provides a cookbook-style set of YAML manifests to deploy and configure [MetalLB](https://metallb.universe.tf/) on Red Hat OpenShift "out of the box." It includes configuration for the MetalLB instance itself as well as a demo application to verify the load balancing functionality.

## **Table of Contents**

* [Prerequisites]
* [Repository Structure](https://www.google.com/search?q=%23repository-structure)  
* [Installation](https://www.google.com/search?q=%23installation)  
  * [1\. Install MetalLB Operator](https://www.google.com/search?q=%231-install-metallb-operator)  
  * [2\. Configure MetalLB](https://www.google.com/search?q=%232-configure-metallb)  
  * [3\. Deploy Demo Application](https://www.google.com/search?q=%233-deploy-demo-application)  
* [Verification](https://www.google.com/search?q=%23verification)  
* [Cleanup](https://www.google.com/search?q=%23cleanup)

## **Prerequisites**

Before utilizing these manifests, ensure you have:

* Access to an **OpenShift Container Platform** cluster.  
* The **oc CLI** tool installed and authenticated to your cluster.  
* Cluster administrator privileges (cluster-admin).

## **Repository Structure**

The configuration files are numbered to indicate the order of application:

| File | Description |
| :---- | :---- |
| 01-metallb.yml | Installs the MetalLB custom resource instance. |
| 02-ip-addresspool.yml | Defines the IP address pool for Layer 2 mode. **(Requires Customization)** |
| 02-ip-addresspool-bgp.yml | Alternate configuration for BGP mode. |
| 03-l2-advertisements.yml | Configures Layer 2 advertisements. |
| 04-demoapp-namespace.yml | Creates the namespace metallb-demo-app. |
| 05-demoapp-pod.yml | Deploys a simple demo pod. |
| 06-demoapp-LB-service.yml | Exposes the demo pod via a LoadBalancer service. |

## **Installation**

### **1\. Install MetalLB Operator**

Ensure the MetalLB Operator is installed in your OpenShift cluster via the OperatorHub. This set of manifests assumes the operator is already running and waiting for a MetalLB instance to be defined.

### **2\. Configure MetalLB**

1. Customize the IP Address Pool:  
   Open 02-ip-addresspool.yml and edit the addresses list to reflect an available range within your internal network.  
   YAML  
   spec:  
     addresses:  
     \- 192.168.1.240-192.168.1.250 \# \<--- Update this range

2. Apply the MetalLB Configuration:  
   Apply the instance configuration, your customized address pool, and the L2 advertisement.  
   Bash  
   oc create \-f 01-metallb.yml  
   oc create \-f 02-ip-addresspool.yml  
   oc create \-f 03-l2-advertisements.yml  
   **Note:** If you are using BGP instead of Layer 2, use 02-ip-addresspool-bgp.yml and skip the L2 advertisements file.

### **3\. Deploy Demo Application**

Deploy the namespace, pod, and service to test the load balancer.

Bash

oc create \-f 04-demoapp-namespace.yml  
oc create \-f 05-demoapp-pod.yml  
oc create \-f 06-demoapp-LB-service.yml

## **Verification**

### **Check Service Status**

Query the service to confirm MetalLB has provisioned an EXTERNAL-IP from your configured pool.

Bash

oc get svc \-n metallb-demo-app

**Example Output:**

Plaintext

NAME               TYPE           CLUSTER-IP       EXTERNAL-IP     PORT(S)          AGE  
demo-app-service   LoadBalancer   172.30.206.100   10.100.59.201   8080:32623/TCP   22s

### **Connectivity Test**

Verify connectivity by curling the assigned External IP on the exposed port (8080).

Bash

curl http://\<EXTERNAL-IP\>:8080

**Example:**

Bash

curl http://10.100.59.201:8080

**Expected Output:**

Plaintext

MetalLB Demo application right here

## **Cleanup**

To remove the demo application and MetalLB configuration:

Bash

\# Delete demo app resources  
oc delete \-f 06-demoapp-LB-service.yml  
oc delete \-f 05-demoapp-pod.yml  
oc delete \-f 04-demoapp-namespace.yml

\# Delete MetalLB configuration  
oc delete \-f 03-l2-advertisements.yml  
oc delete \-f 02-ip-addresspool.yml  
oc delete \-f 01-metallb.yml  
