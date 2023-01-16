# terraform-aws-eks-cluster-autoscaler

## If you had used "EKS module" to provision your EKS cluster, you could refer to the below configuration.

```hcl

variable "aws_region" {
  description = "Please enter the region used to deploy this infrastructure"
  type        = string
  default = "us-west-2"  
}

data "terraform_remote_state" "eks" {
  backend = "s3"
  config = {
    bucket = "backend-terraform-250887682577"
    key    = "infra-structure/aws-3/eks/terraform.tfstate"
    region = var.aws_region
  }
}

module "eks-cluster-autoscaler" {
  source  = "mrnim94/eks-cluster-autoscaler/aws"
  version = "0.0.4"

  aws_region = var.aws_region
  environment = "dev"
  business_divsion = "nimtechnology"

  eks_cluster_certificate_authority_data = data.terraform_remote_state.eks.outputs.cluster_certificate_authority_data
  eks_cluster_endpoint = data.terraform_remote_state.eks.outputs.cluster_endpoint
  eks_cluster_id = data.terraform_remote_state.eks.outputs.cluster_id
  aws_iam_openid_connect_provider_arn = data.terraform_remote_state.eks.outputs.oidc_provider_arn
}

```

## In other case. YOu can use this configuration

```hcl

variable "aws_region" {
  description = "Please enter the region used to deploy this infrastructure"
  type        = string
  default = "us-west-2"  
}

variable "cluster_id" {
  description = "Enter full name of EKS Cluster"
  type        = string
  default = "devops-nimtechnology" 
}


data "aws_eks_cluster" "eks_k8s" {
  name = var.cluster_id
}

module "eks-cluster-autoscaler" {
  source  = "mrnim94/eks-cluster-autoscaler/aws"
  version = "0.0.5"

  aws_region = var.aws_region
  environment = "dev"
  business_divsion = "nimtechnology"

  eks_cluster_certificate_authority_data = data.aws_eks_cluster.eks_k8s.certificate_authority[0].data
  eks_cluster_endpoint = data.aws_eks_cluster.eks_k8s.endpoint
  eks_cluster_id = var.cluster_id
  aws_iam_openid_connect_provider_arn = "arn:aws:iam::${element(split(":", "${data.aws_eks_cluster.eks_k8s.arn}"), 4)}:oidc-provider/${element(split("//", "${data.aws_eks_cluster.eks_k8s.identity[0].oidc[0].issuer}"), 1)}"
}
```

## Recheck

```
root@LP11-D7891:~# kubectl get all -n kube-system
NAME                                                               READY   STATUS    RESTARTS   AGE
pod/aws-node-8jd96                                                 1/1     Running   0          40m
pod/coredns-57ff979f67-d2lf6                                       1/1     Running   0          45m
pod/coredns-57ff979f67-pgl2k                                       1/1     Running   0          45m
pod/kube-proxy-5jzhg                                               1/1     Running   0          40m
pod/nimtechnology-dev-ca-aws-cluster-autoscaler-5d45898bc5-qz8rt   1/1     Running   0          4s

NAME                                                  TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)         AGE
service/kube-dns                                      ClusterIP   172.20.0.10      <none>        53/UDP,53/TCP   45m
service/nimtechnology-dev-ca-aws-cluster-autoscaler   ClusterIP   172.20.120.186   <none>        8085/TCP        20m

NAME                        DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
daemonset.apps/aws-node     1         1         1       1            1           <none>          45m
daemonset.apps/kube-proxy   1         1         1       1            1           <none>          45m

NAME                                                          READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/coredns                                       2/2     2            2           45m
deployment.apps/nimtechnology-dev-ca-aws-cluster-autoscaler   1/1     1            1           20m

NAME                                                                     DESIRED   CURRENT   READY   AGE
replicaset.apps/coredns-57ff979f67                                       2         2         2       45m
replicaset.apps/nimtechnology-dev-ca-aws-cluster-autoscaler-5d45898bc5   1         1         1       21m
```

You also refer to My Post regarding Cluster AutoScaler on EKS.

[![Image](https://nimtechnology.com/wp-content/uploads/2022/09/image-345-1536x816.png "[AWS] Discovering how to design Cluster Autoscaler on EKS. ")](https://nimtechnology.com/2022/09/29/aws-discovering-how-to-design-cluster-autoscaler-on-eks/)