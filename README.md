# aws-k8s-stash

Kubernetes setup
```
curl -LO https://github.com/kubernetes/kops/releases/download/1.8.1/kops-linux-amd64 
sudo mv kops-linux-amd64 /usr/local/bin/kops && sudo chmod a+x /usr/local/bin/kops

curl -LO https://storage.googleapis.com/kubernetes-release/release/v1.7.16/bin/linux/amd64/kubectl
sudo mv kubectl /usr/local/bin/kubectl && sudo chmod a+x /usr/local/bin/kubectl
```
Kops and awscli (debian flavoured)
```
apt-get install awscli
```
configure AWS:
```
aws configure
```
AWS bucket Kops stores the configuration of the deployment in this bucket
```
aws s3api create-bucket --bucket kops-cassandra-blog --region eu-west-1 --create-bucket-configuration LocationConstraint=us-west-1
```
Now generate a public/private key-pair:
```
ssh-keygen -f cassandra-test
```
This key-pair is used to access the EC2 machines. Create cluster:
```
kops create cluster \
--cloud=aws \
--name=kops-cassandra-blog.k8s.local \
--zones=eu-west-1a,eu-west-1b,eu-west-1c \
--master-size="t2.small" \
--master-zones=eu-west-1a,eu-west-1b,eu-west-1c \
--node-size="t2.small" \
--ssh-public-key="cassandra-test.pub" \
--state=s3://cassandra-test \
--node-count=6
```
Now apply the cluster definition
```
kops update cluster --name=kops-cassandra-blog.k8s.local --state=s3://cassandra-test --yes
```
```
#NAME               STATUS  AGE  VERSION  ZONE
 ip-172-20-112-210  Ready   1m   v1.8.7   eu-west-1c
 ip-172-20-58-140   Ready   1m   v1.8.7   eu-west-1a
 ip-172-20-85-234   Ready   1m   v1.8.7   eu-west-1b
```
```
kubectl get no -L failure-domain.beta.kubernetes.io/zone -l kubernetes.io/role=node 
```
As can be seen in the output, each availability zone has two Kubernetes nodes:
```
#NAME               STATUS    AGE  VERSION  ZONE
 ip-172-20-114-66   Ready     1m   v1.8.7   eu-west-1c
 ip-172-20-116-132  Ready     1m   v1.8.7   eu-west-1c
 ip-172-20-35-200   Ready     1m   v1.8.7   eu-west-1a
 ip-172-20-42-220   Ready     1m   v1.8.7   eu-west-1a
 ip-172-20-94-29    Ready     1m   v1.8.7   eu-west-1b
 ip-172-20-94-34    Ready     1m   v1.8.7   eu-west-1b
```
Follow with cassandra/mongo.services configuration.

kops delete cluster --name=kops-cassandra-blog.k8s.local --state=s3://cassandra-test --yes
