# kubernetes_on_mac
Kubernetes on mac with minikube

## VT-x/AMD-v virtualization check 
Minikube requires that VT-x/AMD-v virtualization is enabled in BIOS. To check that this is enabled on OSX / macOS run:

pkshrestha@PKs-MacBook-Pro kubernetes_on_mac (develop) $ sysctl -a | grep machdep.cpu.features | grep VMX
machdep.cpu.features: FPU VME DE PSE TSC MSR PAE MCE CX8 APIC SEP MTRR PGE MCA CMOV PAT PSE36 CLFSH DS ACPI MMX FXSR SSE SSE2 SS HTT TM PBE SSE3 PCLMULQDQ DTES64 MON DSCPL VMX SMX EST TM2 SSSE3 FMA CX16 TPR PDCM SSE4.1 SSE4.2 x2APIC MOVBE POPCNT AES PCID XSAVE OSXSAVE SEGLIM64 TSCTMR AVX1.0 RDRAND F16C
pkshrestha@PKs-MacBook-Pro kubernetes_on_mac (develop) $ 
 #if there is output from the command we should be good 
## install Pre-requisites 
brew update && brew install kubectl && brew cask install docker minikube virtualbox
   # note: make sure virtual box 5.2 or above is there 


## start 
pkshrestha@PKs-MacBook-Pro kubernetes_on_mac (develop) $ minikube start
Starting local Kubernetes v1.7.5 cluster...
Starting VM...
Getting VM IP address...
Moving files into cluster...
Setting up certs...
Connecting to cluster...
Setting up kubeconfig...
Starting cluster components...
Kubectl is now configured to use the cluster.
pkshrestha@PKs-MacBook-Pro kubernetes_on_mac (develop) $ 

  
## Get kubernetes nodes 

pkshrestha@PKs-MacBook-Pro kubernetes_on_mac (develop) $ kubectl get nodes 
NAME       STATUS    ROLES     AGE       VERSION
minikube   Ready     <none>    16m       v1.7.5
pkshrestha@PKs-MacBook-Pro kubernetes_on_mac (develop) $ 


## Switch docker daemon to and from kubenetes 
### to kubernetes
eval $(minikube docker-env)
### revert to host docker daemon 
eval $(docker-machine env -u)

  ## define aliases for these in yoru ~/.bashrc or .zshrc 


## Build, deploy and run an image on your local k8s setup

### verify using kubernetes daemon you can pull images 
docker run -d -p 5000:5000 --restart=always --name registry registry:2


## Build, Deploy and Run in helloworld kubernetes 
 we will use the Dockerfile, index.html, myApp.yaml file present in this repository. 
### Build 
pkshrestha@PKs-MacBook-Pro kubernetes_on_mac (develop) $ docker build . --tag my-app
Sending build context to Docker daemon  58.88kB
Step 1 : FROM httpd:2.4-alpine
2.4-alpine: Pulling from library/httpd
Digest: sha256:e5ed73bf567206c6dae0c873f24fcd9038ee0f867c3852847c6a2128a4b58182
Status: Downloaded newer image for httpd:2.4-alpine
 ---> 0ce4c0d1e99a
Step 2 : COPY ./index.html /usr/local/apache2/htdocs/
 ---> e13b62ab5b1f
Removing intermediate container b7be4e23177b
Successfully built e13b62ab5b1f
pkshrestha@PKs-MacBook-Pro kubernetes_on_mac (develop) $ 
### Plublish to local docker registry 
pkshrestha@PKs-MacBook-Pro kubernetes_on_mac (develop) $ docker tag my-app localhost:5000/my-app:0.1.0
### verify 
docker images should show the images in the yaml file 
REPOSITORY                                             TAG                 IMAGE ID            CREATED             SIZE
my-app                                                 latest              cc949ad8c8d3        44 seconds ago      89.3MB
localhost:5000/my-app                                  0.1.0               cc949ad8c8d3        44 seconds ago      89.3MB
httpd                                                  2.4-alpine          fe26194c0b94        7 days ago          89.3MB
### Deploy and run 
pkshrestha@PKs-MacBook-Pro kubernetes_on_mac (develop) $ kubectl create -f myApp.yaml
deployment.extensions "my-app" created
service "my-app" created
pkshrestha@PKs-MacBook-Pro kubernetes_on_mac (develop) $ 

#### Verify pod and your service 
pkshrestha@PKs-MacBook-Pro kubernetes_on_mac (develop) $ kubectl get all
NAME                         READY     STATUS    RESTARTS   AGE
pod/my-app-904953151-gf6z9   1/1       Running   0          1m
NAME                           DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
deployment.extensions/my-app   1         1         1            1           1m
NAME                                     DESIRED   CURRENT   READY     AGE
replicaset.extensions/my-app-904953151   1         1         1         1m
NAME                     DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/my-app   1         1         1            1           1m
pkshrestha@PKs-MacBook-Pro kubernetes_on_mac (develop) $ 

#### test my-app deployment
The configuration exposes my-app outside of the cluster, you can get the address to access it by running:
pkshrestha@PKs-MacBook-Pro kubernetes_on_mac (develop) $ minikube service my-app --url
http://192.168.99.100:31083
pkshrestha@PKs-MacBook-Pro kubernetes_on_mac (develop) $ 




## Kubernetes GUI 
pkshrestha@PKs-MacBook-Pro kubernetes_on_mac (develop) $ minikube dashboard
Opening kubernetes dashboard in default browser...
pkshrestha@PKs-MacBook-Pro kubernetes_on_mac (develop) $ 

## delete deployment of my-app 
kubectl delete deploy my-app
deployment.extensions "my-app" deleted
pkshrestha@PKs-MacBook-Pro kubernetes_on_mac (develop) $ kubectl delete service my-app
service "my-app" deleted
pkshrestha@PKs-MacBook-Pro kubernetes_on_mac (develop) $ 



# Reset everything 
minikube stop;
minikube delete;
rm -rf ~/.minikube .kube;
brew uninstall kubectl;
brew cask uninstall docker virtualbox minikube;
