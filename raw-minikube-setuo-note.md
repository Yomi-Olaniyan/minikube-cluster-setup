#Kubect-Installations:

curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

sudo install -m 755 kubectl /usr/local/bin/kubectl

{rm kubectl}
{chmod +x kubectl}
{mv kubectl /usr/local/bin} NOTE: These 3 commands in curly brackets are not necessary!


#install minikube: curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 \&\& chmod +x minikube

Run: ls

Run: which minikube		   

Run: sudo mv minikube /usr/local/bin

Run: " which minikube " again     



NEXT IS SWITCH TO ROOT:sudo su

START MINIKUBE CLUSTER: Run:** minikube start --vm-driver=none

Run: ls .kube



OR: curl -LO https://github.com/kubernetes/minikube/releases/latest/download/minikube-linux-amd64

sudo install minikube-linux-amd64 /usr/local/bin/minikube \&\& rm minikube-linux-amd64



OR:
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64

#Install it

sudo install minikube-linux-amd64 /usr/local/bin/minikube

#Clean up

rm minikube-linux-amd64



#TO DELETE:    

minikube delete



**#Command-to-check-Linux-Version-being-Used**

cat /etc/os-release



**#To See the Minikube components inside my Docker-Container:**

docker exec -it minikube bash

docker ps



**#kubectl cluster-info**



**To See Minikube Components:**

minikube status



**For example:**

ubuntu@docker-server:~$ minikube status

minikube

type: Control Plane

host: Running

kubelet: Running

apiserver: Running

kubeconfig: Configured

2. minikube addons list**
3. minikube addons enable dashboard**

#TO GET OR SEE THE RUNNING K8S COMPONENTS: 
RUN: kubectl -n kube-system get pods

ubuntu@docker-server:~$ kubectl -n kube-system get pods

NAME                               READY   STATUS    RESTARTS      AGE

coredns-674b8bbfcf-bt9cm           1/1     Running   1 (14h ago)   16h

etcd-minikube                      1/1     Running   1 (14h ago)   16h

kube-apiserver-minikube            1/1     Running   1 (58m ago)   16h

kube-controller-manager-minikube   1/1     Running   1 (14h ago)   16h

kube-proxy-bcbtl                   1/1     Running   1 (14h ago)   14h

kube-scheduler-minikube            1/1     Running   1 (14h ago)   16h

storage-provisioner                1/1     Running   3 (57m ago)   16h

ubuntu@docker-server:~$



**4. minikube dashboard --url**

ubuntu@docker-server:~$ minikube dashboard --url
* Verifying dashboard health ...
* Launching proxy ...
* Verifying proxy health ...
http://127.0.0.1:41685/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/



**5. #To Check the kubectl proxy port if you used "minikube dashboard --url:**

a. netstat -tlnp | grep kubectl | tail -1



b. netstat -tlnp | grep kubectl > before.txt





**How do i create an ssh tunnel to have access to the dashboard at: http://127.0.0.1:33867/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/**

STARTING kubectl proxy server:
ubuntu@docker-server:~$ kubectl proxy
Starting to serve on 127.0.0.1:8001
#To test if the k8s default page is ok:
Run on the server: curl -v http://127.0.0.1:8001/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/

Run: kubectl proxy to start the proxy server. The proxy server is a mini-web server that connects you to API server of the cluster from your own Windows machine.

Creating The SSH Tunnel:
Run the command below on windows cmd prompt.
a. i. ssh -i "C:\\Users\\Master\\Downloads\\olaabKP.pem" -L 8081:127.0.0.1:8001 ubuntu@54.164.82.121
   ii. Open anther cmd page and run this: netstat -an | findstr 8081

b. ssh -i "C:\\Users\\Master\\Downloads\\olaabKP.pem" -L 8081:172.31.84.249:40479 ubuntu@54.164.82.121   (NOTE: Use this if created with "kubectl proxy --address='172.31.84.249' --port=38000 --disable-filter=true \&")**



**#WHEN DONE WITH THE ABOVE SETUPS, DO THIS:
PLEASE NOTE: I use 'yolaab' as my deployment name here -- it could be any name of your choice.

RUN: kubectl create deployment yolaab --image nginx

Run:  kubectl expose deployment yolaab --port 80     (NOTE: that port 80 is the typical nginx port where it listens)**
For example:
root@docker-server:~# kubectl expose deployment yolaab --port 80
service/yolaab exposed

TO REVEAL THE PORT NUMBER:
Run: kubectl get service yolaab
root@docker-server:~# kubectl get service yolaab
NAME        TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
yolaab   NodePort   10.101.3.198   <none>        80:31772/TCP   45m

NOTE THE PORT TYPE: If it's "ClusterIP", change it to NodePort so that you can browse from your laptop browser. Run: " kubectl edit service yolaab " to do this.

#BUILDING THE IMAGE \& Pushing it to Dockerhub Registry:
a. create a directory.
b. create a 'dockerfile' and put an nginx image inside.
c. create another file called "index.html" inside the directory.

1. Login to Docker hub on the CLI:** Run "  docker login -u olaab  "
2. Build the Image: Run "  *docker build -t 8th-august-image . "
3. Tag the image and put it in  container: " docker tag 8th-august-image olaab/minikube-orchestration:v1.2 "
4. Push the Image-container to registry: " docker push olaab/minikube-orchestration:v1.2 "

RUN: http://44.201.249.7:31772 in order to browse the NGINX IMAGE. If it doesn't browse, open the port number in the SGP.

To see your own Custom Image, Run (5) below:
5. Kubernetes Deployment is next: kubectl set image deployment/yolaab nginx=olaab/minikube-orchestration:v1.2
6. NEXT: browse " public-ip-of-ec2-instance:31772 ". NOTE: "31772" in this case is the port number mapped to port80 under "PORT(S)" above.
7. For example: " http://44.201.249.7:31772 ", where "44.201.249.7" is the pubic ip of the docker server"



#Run "  kubectl get nodes -o wide "

root@docker-server:~/kube-nginx# kubectl get nodes -o wide

NAME            STATUS   ROLES           AGE   VERSION   INTERNAL-IP     EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION    CONTAINER-RUNTIME

docker-server   Ready    control-plane   36h   v1.33.1   **172.31.84.249**   <none>        Ubuntu 24.04.2 LTS   6.14.0-1010-aws   docker://27.5.1



#NOTE:This internal IP " **172.31.84.249"** is the private IP of the ec2 instance, which in this context is the Docker-Server.


