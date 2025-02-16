# argocd-project
this repository testing for deploy application on multiple cluster using GitOps tool argoCD 
##
GitOps tools like argocd, flux 
how argocd is used at production levle 
##########################

1. create a 3 AWS EKS cluster with name
2. Install argocd on Hub cluster
3. UI login to Argocd
4. add Spoke cluster to hub cluster
5. deploy application on both spoke cluster 

##########################
**step1 start:**
create a cluster with the followinfig g command:
```
$eksctl create cluster --name hub-cluster --region ap-southeast-2
$eksctl create cluster --name spoke-cluster-1 --region ap-southeast-2
$eksctl create cluster --name spoke-cluster-2 --region ap-southeast-2
```
check how many cluster with the namespce ap-southeast-2
```
$kubectl get cluster 
$Kubectl config get-contexts | grep ap-southeast-2
CURRENT   NAME                                                        CLUSTER                                    AUTHINFO                                                    NAMESPACE
          iam-root-account@hub-cluster.ap-southeast-2.eksctl.io       hub-cluster.ap-southeast-2.eksctl.io       iam-root-account@hub-cluster.ap-southeast-2.eksctl.io
*         iam-root-account@spoke-cluster-1.ap-southeast-2.eksctl.io   spoke-cluster-1.ap-southeast-2.eksctl.io   iam-root-account@spoke-cluster-1.ap-southeast-2.eksctl.io
```
```
$kubectl config use-contexts namespace_of_cluster
$kubectl config use-contexts iam-root-account@hub-cluster.ap-southeast-2.eksctl.io
$kubectl config current-context
iam-root-account@hub-cluster.ap-southeast-2.eksctl.io
```
all the action is preformed on hub cluster
##########################

**step2 start:**
install argocd with this command 
```
$kubectl create namespace argocd
$kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
$kubect get pods -n argocd 

NAME                                                READY   STATUS    RESTARTS   AGE
argocd-application-controller-0                     1/1     Running   0          2m49s
argocd-applicationset-controller-684cd5f5cc-547nz   1/1     Running   0          2m50s
argocd-dex-server-77c55fb54f-xbfx8                  1/1     Running   0          2m50s
argocd-notifications-controller-69cd888b56-66fv6    1/1     Running   0          2m49s
argocd-redis-55c76cb574-bz8xf                       1/1     Running   0          2m49s
argocd-repo-server-584d45d88f-h9qfr                 1/1     Running   0          2m49s
argocd-server-8667f8577-xh8q5                       1/1     Running   0          2m49s
```
for this project we are creating a  HTTP model 
run server without TLS
```
$kubectl get configmap -n argocd
          OR
$kubectl get cm -n argocd
NAME                        DATA   AGE
argocd-cm                   0      4m1s
argocd-cmd-params-cm        0      4m1s
argocd-gpg-keys-cm          0      4m1s
argocd-notifications-cm     0      4m1s
argocd-rbac-cm              0      4m1s
argocd-ssh-known-hosts-cm   1      4m1s
argocd-tls-certs-cm         0      4m1s
kube-root-ca.crt            1      4m55s
```
$kubctl edit configmap argocd-cmd-params-cm -n argocd 
data:  
  # Run server without TLS
  server.insecure: "false"
  -->
  server.insecure: "true" 
  -->
to secure your application you should never use HTTP for organization 
create a secure HTTPS (TLS with secure)
you need to sign in to ca-certificates autontication.

$kubectl get svc -n argocd 
$kubectl edit svc argocd --server -n argocd
type: ClusterIp
--> 
type:NodePort
-->

check the service is changed form ClusterIp to NodePort
$kubectl edit svc argocd --server -n argocd
argocd-server                             ClusterIP    10.100.161.58    <none>        80:32326/TCP,443:32079/TCP   12m
-->
argocd-server                             NodePort    10.100.161.58    <none>        80:32326/TCP,443:32079/TCP   12m
-->

edit inbound rule for hub server
edit port, protocal( all traffic for testing)

hit url: pulic ip of hub ec2-instance (103.105.0.0:32326)
########################## 

**step3 start**:
username is admin
$kubectl get secrets -n argocd

argocd-initial-admin-secret   Opaque   1      17m
argocd-notifications-secret   Opaque   0      17m
argocd-redis                  Opaque   1      17m
argocd-secret                 Opaque   5      17m

$kubectl edit secrets argocd-initial-admin-secre -n argocd
b0Q3cXByR2hVbHEydzVYVQ==
copy the password(b0Q3cXByR2hVbHEydzVYVQ==)
this is in encrypted format, need to decode 
$echo b0Q3cXByR2hVbHEydzVYVQ== | base64 --decode
abcdhdfk(this is actial password without "%")
log-in the **argocd page:**
##########################

**step4 start:**

need to add 2 spoke cluster to hub cluster
you log-in to argo cd through UI
**NOTE-** you can't add the cluster directly using argocd User interface
to add the cluster you need to log-in to argocd CLI api also 

$argocd login argocdip
$argocd login 103.105.0.0:32326 
username: admin
pass: abcdhdfk

$argocd cluster add context_of_cluster --server ip_address_withport_argocd_page
for example:
  $argocd cluster add iam-root-account@spoke-cluster-1.ap-southeast-2.eksctl.io --server 103.105.0.0:32326
**NOTE: ** Do the same for remaining spoke cluster as wll.
##########################
 
**step5 start:**
Go to argocd Home page:
application-> create new application 
  Application: guestbook1(randomname)
  project: default:
  sync policy: automatic
  repo url: give github repo url http(repo should be public in this project )
  path: manifests/guest-book
  Destination:
  clusterURL: (spoke1)select 1 cluster which is need to be added 
  namespace: default
  create:
do this to add another cluster also 

check your application deployed or not 
$kubectl config use-context spoke1
 f.g: $kubectl config use-contex iam-root-account@spoke-cluster-1.ap-southeast-2.eksctl.io
$kubectl get all
$kubectl edit configmap
you can see this 
data:
  ui_properties_file_name: "yogesh-interface.properties"


### applicaton is deployed on multiple cluster ###
##########################

lets make a changes in configmap.yml manifest 
will our argo cd will identify the change and it should be deployed on to the cluster
data:
  ui_properties_file_name: "yogesh-interface.properties"
-->
data:
  ui_properties_file_name: "abhishek-interface.properties"


Go to ArgoCD admin page 
argocd will auto-sync for 3min timeframe
you can manually sync click on **sync app** --> select 2 cluster --> sync 







