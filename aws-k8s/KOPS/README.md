Install/configure awscli

```
apt install -y python3-pip
pip3 install awscli
aws configure (or export AWS_ACCESS_KEY_ID= ; export AWS_SECRET_ACCESS_KEY=; export AWS_DEFAULT_REGION=us-east-1)
```

Created a new IAM user called k8s-saas with a new group (k8s-saas) that has access to AmazonEC2FullAccess, IAMFullAccess, AmazonS3FullAccess, and AmazonVPCFullAccess.

```
AmazonEC2FullAccess
AmazonRoute53FullAccess
AmazonS3FullAccess
IAMFullAccess
AmazonVPCFullAccess
```

Create kops user using aws cli:
```

aws iam create-group --group-name k8s-saas
aws iam attach-group-policy --policy-arn arn:aws:iam::aws:policy/AmazonEC2FullAccess --group-name k8s-saas
aws iam attach-group-policy --policy-arn arn:aws:iam::aws:policy/AmazonRoute53FullAccess --group-name k8s-saas
aws iam attach-group-policy --policy-arn arn:aws:iam::aws:policy/AmazonS3FullAccess --group-name k8s-saas
aws iam attach-group-policy --policy-arn arn:aws:iam::aws:policy/IAMFullAccess --group-name k8s-saas
aws iam attach-group-policy --policy-arn arn:aws:iam::aws:policy/AmazonVPCFullAccess --group-name k8s-saas
aws iam create-user --user-name k8s-saas
aws iam add-user-to-group --user-name k8s-saas --group-name k8s-saas
aws iam create-access-key --user-name k8s-saas

#Record the SecretAccessKey and AccessKeyID in the returned JSON output.
```

Grab access and secrets keys (from previous JSON output for example) and configure your k8s-saas profile with: 
```
aws configure (or edit ~/.aws/credentials & ~/.aws/config  files)
```
You can check those creds at:
```
$ cat ~/.aws/credentials 
[default]
aws_access_key_id = 
aws_secret_access_key = 
[k8s-saas]
aws_access_key_id = 
aws_secret_access_key = 

$ cat ~/.aws/config 
cat: cat: No such file or directory
[default]
region = us-east-2
[k8s-saas]
region = us-east-1
```
Next, let's create the s3 bucket with versioning that we need for our kube state (us-east-1). I will call it `k8s-saas-kops-state-dev`. Make sure you turn on versioning.

Create S3 bucket using aws cli:

```
aws s3api create-bucket --bucket k8s-saas-kops-state-dev --region us-east-1 
aws s3api put-bucket-versioning --bucket k8s-saas-kops-state-dev --versioning-configuration Status=Enabled

```
Now let's work on the Kops and Kubernetes segment.

to install kops on linux:
```
curl -LO https://github.com/kubernetes/kops/releases/download/$(curl -s https://api.github.com/repos/kubernetes/kops/releases/latest | grep tag_name | cut -d '"' -f 4)/kops-linux-amd64
chmod +x kops-linux-amd64
sudo mv kops-linux-amd64 /usr/local/bin/kops
```
For other operating systems see their documentation.

Note: What is KOPS? We like to think of it as kubectl for clusters. kops helps you create, destroy, upgrade and maintain production-grade, highly available, Kubernetes clusters from the command line. AWS (Amazon Web Services) is currently officially supported, with GCE and OpenStack in beta support, and VMware vSphere in alpha, and other platforms planned.

KOPS Features

    Automates the provisioning of Kubernetes clusters in AWS and GCE
    Deploys Highly Available (HA) Kubernetes Masters
    Built on a state-sync model for dry-runs and automatic idempotency
    Ability to generate Terraform
    Supports managed kubernetes add-ons
    Command line autocompletion
    YAML Manifest Based API Configuration
    Templating and dry-run modes for creating Manifests
    Choose from eight different CNI Networking providers out-of-the-box
    Supports upgrading from kube-up
    Capability to add containers, as hooks, and files to nodes via a cluster manifest



then install kubectl:
```
curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl
```
For more information or installation methods, see the docs.

Configuring and Provisioning our Cluster

Now the we have installed the requirements, let's export our variables and provision our cluster.

We need our AWS profile, keys, and S3 bucket name that we created. Export the following:
```
export AWS_PROFILE="k8s-saas"
export AWS_ACCESS_KEY_ID=$(aws configure get k8s-saas.aws_access_key_id)
export AWS_SECRET_ACCESS_KEY=$(aws configure get k8s-saas.aws_secret_access_key)
export KOPS_STATE_STORE=s3://k8s-saas-kops-state-dev
export KUBECONFIG=~/.kube/k8s-saas-AWS-KOPS
```
Once we have done that, let's run kops in the command line to create a master and 3 nodes.(I named mine saas.k8s.local):
note: Create a new ssh key called k8s-saas with `ssh-keygen`
```


$ ls  ~/.ssh/k8s-saas*
-rw------- 1 davar davar 1679 Jan 16 09:53 /home/davar/.ssh/k8s-saas
-rw-r--r-- 1 davar davar  394 Jan 16 09:53 /home/davar/.ssh/k8s-saas.pub
```
Create k8s cluster with kops:

```
kops create cluster \
             --name="saas.k8s.local" \
             --zones="us-east-1a" \
             --master-size="t2.micro" \
             --node-size="t2.micro" \
             --node-count="3" \
             --ssh-public-key="~/.ssh/k8s-saas.pub"
```

Warning: Yes this is free tier, but make sure you setup billing alerts and remember to tear down your cluster when you are not practicing. This will absolutely burn through the free tier compute limit after a day or two of running

Example Outout:

```
...
Must specify --yes to apply changes

Cluster configuration has been created.

Suggestions:
 * list clusters with: kops get cluster
 * edit this cluster with: kops edit cluster saas.k8s.local
 * edit your node instance group: kops edit ig --name=saas.k8s.local nodes
 * edit your master instance group: kops edit ig --name=saas.k8s.local master-us-east-1a

Finally configure your cluster with: kops update cluster --name saas.k8s.local --yes

```
check:

```
$ kops get cluster
NAME		CLOUD	ZONES
saas.k8s.local	aws	us-east-1a
```

All configuration and your cluster state is stored in the S3 bucket that you created:
```
$ aws s3 ls s3://k8s-saas-kops-state-dev/saas.k8s.local/
                           PRE instancegroup/
                           PRE pki/
2021-01-16 10:03:53       5191 cluster.spec
2021-01-16 10:03:52       1101 config
```
Create k8s cluster with kops (final step: --yes):

```
kops update cluster --name saas.k8s.local --yes
```

If you get any errors, try running:
```
kops update cluster --name saas.k8s.local
```
and to validate:
```
kops validate cluster --name saas.k8s.local
```
Once this cluster is complete, you should be able to see it in your EC2 Dashboard. It will also save your configuration to .kube/ (~/.kube/k8s-saas-AWS-KOPS) in you home directory.

try:
```
kubectl get nodes
```
You should be able to see the nodes you just created.

You can now use this cluster to try things out, but again, i'd recommend k3s/minikube for testing. This is a good piece of knowledge to have in your professional tool belt.


When you are finished, go ahead and bring it down to save your free tier compute hours:
```
kops delete cluster saas.k8s.local --yes
```
and verify that the cluster has been terminated in your EC2. And remember, your cluster state is stored in the S3 bucket that you created! (so delete S3 bucket too)
```
$ aws s3 rm s3://k8s-saas-kops-state-dev/ --recursive
$ aws s3api delete-bucket --bucket k8s-saas-kops-state-dev --region us-east-1
```
