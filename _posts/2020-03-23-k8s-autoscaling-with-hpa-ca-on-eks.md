---
title: Very Simple Kubernetes Autoscaling(HPA, CA) on EKS with Helm
updated: 2020-03-23 17:18
---

HPA(Horizontal Pod Autoscaler)로 k8s의 pod이 scaling 되도록 설정하고, scale-out 중 cluster의 resource가 부족하면 CA(Cluster Autoscaler)설정를 통해 클러스터(여기서는 EC2 instance)를 scaling하는 것을 해봅니다.

##### ap-northeast-1 지역에서 진행한다고 가정하고 작성했습니다. (Amazon Linux 2 AMI t2.micro 인스턴스 내에서 실습 진행)


### AWS 계정 설정

AdminAccess 권한이 있는 IAM user로 configure 후 AdminAccess 있는지 확인.

```sh
aws configure

aws opsworks describe-my-user-profile

aws iam list-attached-user-policies --user-name {YOUR_USER_PROFILE_NAME}
```

![aws_configure]( {{ site.baseurl }}/assets/img/2020-03-23-k8s/aws_configure.png)

<div class="divider"></div>

### eksctl, kubectl 설치

eksctl is a simple CLI tool for creating clusters on EKS - 
Amazon’s new managed Kubernetes service for EC2 [(eksctl.io)](https://eksctl.io){:target="_blank"}


```sh
sudo curl --silent --location -o /usr/local/bin/kubectl https://amazon-eks.s3-us-west-2.amazonaws.com/1.14.6/2019-08-22/bin/linux/amd64/kubectl

sudo chmod +x /usr/local/bin/kubectl

curl --silent --location "https://github.com/weaveworks/eksctl/releases/download/latest_release/eksctl_$(uname -s)_amd64.tar.gz" | tar xz -C /tmp

sudo mv -v /tmp/eksctl /usr/local/bin

eksctl version
```

![eksctl_kubectl]( {{ site.baseurl }}/assets/img/2020-03-23-k8s/eksctl_kubectl.png)

<div class="divider"></div>


### eksctl 명령어 이용해서 클러스터 생성하기

--managed: EKS-managed nodegroup 으로 구성 (c.f. fargate)

--asg-access: node들에게 autoscaling 하기 위한 권한 부여

```sh
# create cluster with new VPC
eksctl create cluster --name=hello-world --nodes=3 --managed --alb-ingress-access \
  --asg-access --region=ap-northeast-1 --node-type t3.small


# with existing VPC (private VPC와 public VPC를 각각 지정. 아래 subnet은 예시이고 자신의 subnet으로 넣어서 진행해야 함)
eksctl create cluster --name=hello-world --nodes=3 --managed --alb-ingress-access \
  --asg-access --region=ap-northeast-1 --node-type t3.small \
  --vpc-private-subnets=subnet-0aff7039162e39a77,subnet-0184c82a14838a3d4 \
  --vpc-public-subnets=subnet-0c5c8682d11203a5f,subnet-07a9nf0f1e9c7e576
```

![eksctl_create]( {{ site.baseurl }}/assets/img/2020-03-23-k8s/eksctl_create.png)

<div class="divider"></div>


### Helm 설치

Helm 설치 후 stable repository 추가

```sh
curl -sSL https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash

helm version --short  # check "v3.1.2+gd878d4d" alike

helm repo add stable https://kubernetes-charts.storage.googleapis.com/
```

<div class="divider"></div>

### Install metrics-server with Helm

Metrics server is a cluster-wide aggregator of resource usage data. These metrics will drive the scaling behavior of the deployments. 
라고 하는데 deployment들의 scaling을 가능하게 해주는 것. Helm 을 이용해 설정.

```sh
helm install horizontal-pod-autoscaler stable/metrics-server

kubectl get apiservice v1beta1.metrics.k8s.io -o yaml
```

![hpa]( {{ site.baseurl }}/assets/img/2020-03-23-k8s/hpa.png)
<div class="divider"></div>

![hpa_check]( {{ site.baseurl }}/assets/img/2020-03-23-k8s/hpa_check.png)
<div class="divider"></div>



### HPA를 이용할 리소스 생성

hpa-example 이미지를 사용하여 autoscale 진행할 deployment 구성(cpu=1000m: 하나의 core의 100%를 pod에 할당한다는 의미(c.f. 500m=한 코어의 50% 할당))

```sh
kubectl run php-apache --image=k8s.gcr.io/hpa-example --requests=cpu=1000m --expose --port=80

# create an autoscaler for replication set php-apache, with target CPU utilization set to 30% 
kubectl autoscale deployment php-apache --cpu-percent=30 --min=1 --max=10
```

![php-apache]( {{ site.baseurl }}/assets/img/2020-03-23-k8s/php-apache.png)

<div class="divider"></div>

### CA config on console

[여기](https://console.aws.amazon.com/ec2/autoscaling/home#AutoScalingGroups:){:target="_blank"}를 통해 EC2 Auto Scaling Groups 접근

스크린샷에 나온 것처럼 edit 후 Max 8 정도로 설정

![console_1]( {{ site.baseurl }}/assets/img/2020-03-23-k8s/console_1.png)

<div class="divider"></div>
![console_2]( {{ site.baseurl }}/assets/img/2020-03-23-k8s/console_2.png)

<div class="divider"></div>


### Complete CA config with Helm

helm의 cluster-autoscaler chart를 이용해 CA 설정.(`{YOUR_EKS_CLUSTER_NAME}`에 자신의 클러스터 이름을 넣고 진행)
[(github)](https://github.com/helm/charts/tree/master/stable/cluster-autoscaler){:target="_blank"}

```sh
helm install cluster-autoscaler stable/cluster-autoscaler --set \
  autoDiscovery.clusterName={YOUR_EKS_CLUSTER_NAME},awsRegion=ap-northeast-1,rbac.create=true
```

![ca_helm]( {{ site.baseurl }}/assets/img/2020-03-23-k8s/ca_helm.png)

<div class="divider"></div>

### Generate Load to Trigger Scaling

설정은 마쳤고 autoscaling이 잘 되는지 보기 위해 적절한 load를 줘봅니다. 

```sh
kubectl run -i --tty load-generator --image=busybox /bin/sh

# 'hey'라는 web application을 통해 request 생성 [github.com/rakyll/hey](https://github.com/rakyll/hey){:target="_blank"}
wget https://storage.googleapis.com/hey-release/hey_linux_amd64

chmod 700 hey_linux_amd64

# send one request per second(-q 1) for 3 minutes(-z 3m)
./hey_linux_amd64 -q 1 -z 3m http://php-apache

# 각각의 명령어에 대해 변하는 것 확인
watch kubectl get hpa
watch kubectl get pods
watch kubectl get nodes
```

scale-out 초기
![phase_1]( {{ site.baseurl }}/assets/img/2020-03-23-k8s/phase_1.png)
<div class="divider"></div>

console EC2 인스턴스 생성(node scale-out)
![ec2_console]( {{ site.baseurl }}/assets/img/2020-03-23-k8s/ec2_console.png)
<div class="divider"></div>

pod, node들 늘어난 것
![phase_2]( {{ site.baseurl }}/assets/img/2020-03-23-k8s/phase_2.png)
<div class="divider"></div>

3분 후 scale-down
![phase_3]( {{ site.baseurl }}/assets/img/2020-03-23-k8s/phase_3.png)
<div class="divider"></div>

각 그림에 따라 pod이 늘어나고 node가 늘어났다가 3분 후 request가 끝나고 다시 scale-down 하는 것을 볼 수 있습니다.

scale down은 k8s의 policy에 따라 천천히 이루어지는데 EKS에서는 설정하는 것이 아직 지원되지 않는 것으로 보입니다.[관련 링크](https://github.com/aws/containers-roadmap/issues/159){:target="_blank"}

<div class="divider"></div>

### Cleanup

```sh
kubectl delete deploy load-generator php-apache

kubectl delete service php-apache

kubectl delete hpa php-apache

helm uninstall horizontal-pod-autoscaler cluster-autoscaler
```

<div class="divider"></div>

fargate로 노드 구성해보는 것도 궁금하고 해보고싶다.
흰색 터미널로 스크린샷 찍으니까 구분이 잘 안되는 것 같다.

