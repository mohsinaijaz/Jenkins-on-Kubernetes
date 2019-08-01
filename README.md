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

$ kops create cluster --node-count=2 --node-size=t2.medium --master-size=t2.medium --zones=us-east-1a --name=${KOPS_CLUSTER_NAME}

$ brew install kubernetes-helm

$ kubectl create serviceaccount --namespace kube-system tiller

$ kubectl create clusterrolebinding tiller-cluster-rule --clusterrole=cluster-admin --serviceaccount=kube-system:tiller

$ helm init --service-account tiller







$ kops delete cluster --name ${KOPS_CLUSTER_NAME} --yes
