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