# Kubernetes The Hard Way (On Azure)

This project has been my first experience with kubernetes. I did encounter some issues that I had to overcome, namely with certificate generation and running certain commands in PowerShell vs Bash. Overall, I would say I learned a great deal about not only kubernetes, but as well azure resource provisioning, configuration, linux binary installations, linux daemons / services, and more.

My goal is to continue to learn about kubernetes and achieve the CKA (Certified Kubernetes Administrator). From there I will determine the path forward, looking at CKAD and more.

> Thank you for taking the time to check this out. I hope that the images I provided can help show my progress through deploying this cluster. I also describe some of the hiccups I encountered and how I solved them.

## Progress Pics

### SSH into a node
<img src="https://i.imgur.com/n7oWyTu.png">

* Here you can see that I provisioned the worker and controller nodes, and was able to successfully ssh into one. Also you can see that the controller has some of the certs and kubeconfigs already inside.

---
### Health check
<img src="https://i.imgur.com/AUYDSi7.png">

* You can see here I am accessing health from within a controller node. This is the first time I was able to successfully get etcd up and running. I ran into multiple issues with this and had to restart the project a few times to completely solve it.

1. The scripts provided in the original github had to be modified to be ran in PowerShell.
2. Because I was messing with the scripts, any error wouldn't present itself until I went to utilize the certificates I generated or use the config files.
3. Led to a lot of regression, confusion, and learning - for instance, I realized that public-ip resource won't get the IP until it's used with a NIC and VM. Held myself up trying to figure out why creating fresh public-ips had ipAddress=null.
4. Etcd was a nightmare to get running on the first controller, and I found a comment on the issues tab of the kthw-azure github that mentioned "just configure all 3 controllers etcd and the first one will work". No real answer there, but I was head-scratching quite a bit on controller-0 where I would see etcd fail to load and auto-restart infinitely until I configured the other two controller nodes etcd.

The below image is an example of a curl command failing in PowerShell but working in bash. The certificate was correct, all signs point to yes, but it does not work.

<img src="https://i.imgur.com/gkkifwE.png">

---
### Worker nodes up
<img src="https://i.imgur.com/eZmOjU4.png">

* Finally I was able to have controllers talk to the api and find the available nodes.

---
### Health Check Now Remote
<img src="https://i.imgur.com/EbRCwxy.png">
* Similar health check as above, but connecting remotely.

---
### Route Tables
<img src="https://i.imgur.com/9WxAeXj.png">
* Successfully configured the route table resource in Azure to enable networking between the nodes.

---
### Pod nslookup
<img src="https://i.imgur.com/enl69cv.png">
* Successfully created a busybox deployment. In the pod, you can see remote exec of nslookup working.

---
### Secrets confirmed
<img src="https://i.imgur.com/EMlNX4P.png">
* This image serves to verify that the kubernetes secrets were working correctly. You can see on the right side of the hexdump that aescbc encrypted the data at rest, and the secret was stored in etcd.

---
### Deployments confirmed
<img src="https://i.imgur.com/jDpim6R.png">
* Here you can see a simple nginx deployment created and spun up.

---
### Port-Forwarding
<img src="https://i.imgur.com/pdzANQ3.png">
* Verified that the port-forwarding was configured correctly by listening over port 80 and using a separate terminal to curl and get response.

---
### Logs
<img src="https://i.imgur.com/XWJX3iN.png">
* Verified that the logs are able to be retrieved from a pod remotely.

---
### Remote exec
<img src="https://i.imgur.com/qlNeM5e.png">
* Running a version check on the nginx pod and getting that response remotely.

---
### Expose services
<img src="https://i.imgur.com/brHjs9R.png">
* Verified that the worker nodes can have the app exposed, meaning the firewall rule was configured correctly and the NodePort service was exposed.

---
### Dashboard live and misc
<img src="https://i.imgur.com/s8z9K0c.png">
<img src="https://i.imgur.com/hlWkJxX.png">
<img src="https://i.imgur.com/RVRE3IV.png">
<img src="https://i.imgur.com/xgIB62o.png">
<img src="https://i.imgur.com/nKBsiWX.png">

## Final Thoughts

Again, I appreciate anyone who may have looked through this document. I really enjoyed tinkering with this small entry to kubernetes and am eager to learn more. I added some images of my Azure Portal and resources to show how that all looked before I took it down. Cost management, am I right?

I am happy to answer any and all questions about this project!