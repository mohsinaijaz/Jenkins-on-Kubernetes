# Jenkins-on-Kubernetes

**Below are the steps to install KOPS and setup Jenkins, this setup provides Jenkins master running in K8's cluster on top of AWS. The benefit of this is will be that Jenkins master will create new pods in K8's when new builds are trigged, once build is complete the pod will be terminated.**  

Here are the steps to install Kubernetes cluster using Kops in AWS.

link to kops https://github.com/kubernetes/kops

User who will create the cluster must have the following IAM permissions:
1. AmazonEC2FullAccess
2. AmazonRoute53FullAccess
3. AmazonS3FullAccess
4. IAMFullAccess
5. AmazonVPCFullAccess

Kops will create the following in your AWS account, make sure to delete the cluster to avoid charges.

#

Setup aws cli to interact with aws account

$ aws configure

Install kops and kubectl interact with the cluster

$ brew update && brew install kops kubectl

$ aws s3api create-bucket --bucket jenkins-k8-state-store --region us-east-1

$ aws s3api put-bucket-versioning --bucket jenkins-k8-state-store  --versioning-configuration Status=Enabled

$ export KOPS_CLUSTER_NAME=jenkins.k8s.local

$ export KOPS_STATE_STORE=s3://jenkins-k8-state-store

**Options for the Kops create command**

--master-size=

--master-count=

--node-size=

--node-count=

--cloud=

--zones=

--cloud-labels=

--bastion=

--topology=

--networking=

--state=

--dns-zone=

--dns=

**We will not customize out cluster and keep it pretty basic.**

$ kops create cluster --node-count=2 --node-size=t2.medium --master-size=t2.medium --zones=us-east-1a --name=${KOPS_CLUSTER_NAME}

**Once the cluster is up, this will take about 15 min. Validate the cluster before setting up anything.**

**Install helm on your MAC**

$ brew install kubernetes-helm

**Create service account for tiller on the cluster**

$ kubectl create serviceaccount --namespace kube-system tiller

**Create a cluster role binding for tiller service account and grant cluster admin access**

$ kubectl create clusterrolebinding tiller-cluster-rule --clusterrole=cluster-admin --serviceaccount=kube-system:tiller

**Initialize helm, this will create a tiller pod running in kube-system**

$ helm init --service-account tiller

**Now we will install Jenkins using helm chart, also I modified the values file with AdminUser and AdminPassword. Using the below command you will download the chart and can modify**

$ helm inspect stable/jenkins > /tmp/jenkins.values

**You can also modify anything else, now will we will install using our custom values file**

$ helm install stable/jenkins --values /tmp/jenkins.values --name test-jenkins

**Once Jenkins pod and services are ready you will see load balancer in AWS would have been created, browse to the LB and specify port 8080. Login using the credentials you added in the values file.**

**Update Jenkins and lets configure Jenkins to create slave pods when builds are trigged**

**Credentials --> system --> Global credentials --> Add credentials --> Kind Kubernetes**

**Now we will grant access to Jenkins to create salve pods on our behalf**

$ kubectl create clusterrolebinding permissive-binding --clusterrole=cluster-admin --user=admin --user=kubelet --group=system:serviceaccounts

**Now login to jenkins, Manage Jenkins --> configure system --> Kubernetes section Add the secret file and TEST Connection**

**Create a build and watch the cluster create a pod for that build, once build is complete the pod will be terminated.**

$ kops delete cluster --name ${KOPS_CLUSTER_NAME} --yes
