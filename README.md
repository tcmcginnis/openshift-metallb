This is a cookbook style set of yaml to utilize MetalLB out of the box.

To prep:
1) install metallb operator
2) customize the 02-ip-addresspool.yml yaml to reflect your internal IP network.

To Install:
   Run "oc create -f" on each numbered yaml config file

To test:
   oc get svc -n metallb-demo-app 
   NAME               TYPE           CLUSTER-IP       EXTERNAL-IP     PORT(S)          AGE
   demo-app-service   LoadBalancer   172.30.206.100   10.100.59.201   8080:32623/TCP   22s

   curl http://10.100.59.201:8080
   MetalLB Demo application right here

