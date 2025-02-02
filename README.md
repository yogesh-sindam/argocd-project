# argocd-project
this repository testing for deploy application on multiple cluster using GitOps tool argoCD 
##
GitOps tools like argocd, flux 
how argocd is used at production levle 



##########################
deploying a application on multiple cluster

pre-requisites:
1. 3 EKS cluster (kube1 for Hub, 2 for spoke)
  Hub-Spoke deployment:

steps:
1. create a 3 AWS EKS cluster with name
2. Install argocd on Hub cluster
3. UI login to Argocd
4. add Spoke cluster to hub cluster
5. deploy application on both spoke cluster 

##########################
**step1 start:**
create a cluster with the followinfig g command:
eksctl create cluster --name hub-cluster --region  
eksctl create cluster --name spoke-cluster-1 --region ap-southeast-2
eksctl create cluster --name spoke-cluster-2 --region ap-southeast-2

check how many cluster with the namespce ap-southeast-2
kubectl get cluster 
$Kubectl config get-contexts | grep ap-southeast-2

kubectl config use-contexts | namespace of cluster
kubectl config current-context

all the action is preformed on hub cluster

**step2 start:**
install argocd with this command 
kubectl create namespace argocd

kubect get pods -n argocd 
for this project we are creating a  HTTP model 
run server without TLS
$ kubctl get cm -n argocd
$kubctl edit configmap arfocd-cmd-params-cm -n argocd 
data:  
  # Run server without TLS
  server.insecure: "false"   
  
to secure your application you should never use this 
for creating a secure HTTPS (TLS with secure)
you need to sign in to ca-certificates autontication.
$kubectl get svc -n argocd 
$kubectl edit svc argocd --server -n argocd
ClusterIp--> NodePort
$kubectl edit svc argocd --server -n argocd
edit inbound rule for hub server
edit port, protocal( all traffic for testing)

**step3 start**:
username is admin
kubectl get secrets -n argocd 
kubectl edit secrets argocd initial pass -n argocd
copy the password(ghj34fhfhtu4)
this is in encrypted format, need to decode 
echo ghj34fhfhtu4 | base64 --decode
abcdhdfk(this is actial password)
log-in the **argocd page:**


**step4 start:**

need to add 2 spoke cluster to hub cluster
you loggedin to argo cd through UI
**NOTE-** you can add the cluster directly using argocd api
to add the cluster you need to log-in to argocd CLI api also 

$argocd login argocdip
username: admin
pass: abcdhdfk

$argocd cluster add context_of_cluster --server ip_address_of_argocd

**NOTE: ** Do the same for spoke 2 cluster also

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
$kubectl get all

### applicaton is deployed on multiple cluster ###

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







