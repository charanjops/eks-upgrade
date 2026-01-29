# eks-upgrade
1. Pre-upgrade checks

Set the current context
aws eks update-kubeconfig --region us-east-1 --name demo-eks

Check the context and aws user configuration
kubectl config current-context
aws sts get-caller-identity

Check current versions
kubectl version --client=false
aws eks describe-cluster --name demo-eks --query cluster.version 

Backup cluster state

Backup manifests (GitOps repo ideally)

Snapshot critical data:

EBS volumes

RDS / external DBs

Export:

kubectl get all -A -o yaml > cluster-backup.yaml

2. Upgrade EKS Control Plane

aws eks update-cluster-version --name <cluster-name> --kubernetes-version <target-version>

Monitor:

aws eks describe-cluster --name <cluster-name> --query cluster.status

3. Upgrade EKS Add-ons
3.1 VPC CNI
aws eks update-addon \
  --cluster-name <cluster-name> \
  --addon-name vpc-cni

3.2 CoreDNS
aws eks update-addon \
  --cluster-name <cluster-name> \
  --addon-name coredns

3.3 kube-proxy
aws eks update-addon \
  --cluster-name <cluster-name> \
  --addon-name kube-proxy

3.4 CSI Drivers (if used)
aws eks update-addon \
  --cluster-name <cluster-name> \
  --addon-name aws-ebs-csi-driver


Verify:

kubectl get pods -n kube-system

4. Upgrade Worker Nodes
Option A: Managed Node Groups (recommended)
4.1 Create new node group (blue/green)

New AMI

Same labels, taints, instance types

Same scaling config

aws eks create-nodegroup ...

4.2 Cordon + drain old nodes
kubectl cordon <node>
kubectl drain <node> \
  --ignore-daemonsets \
  --delete-emptydir-data

Option B: Rolling update existing node group
aws eks update-nodegroup-version \
  --cluster-name <cluster-name> \
  --nodegroup-name <nodegroup-name>

5. Validate workloads
5.1 Cluster health
kubectl get nodes
kubectl get pods -A