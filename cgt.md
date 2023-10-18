# Setting up an Amazon EKS Cluster with Terraform

In this article, I'll guide you through setting up an Amazon Elastic Kubernetes Service (EKS) cluster using Terraform. EKS is a managed Kubernetes service provided by Amazon Web Services (AWS) that allows for easy deployment and management of Kubernetes clusters.

## Why Use Amazon EKS?

Creating a Kubernetes cluster locally using tools like kops or minikube can be complex and require ongoing management. Amazon EKS simplifies this process by handling the cluster creation and management for you. You can easily scale your cluster based on your requirements.

## Getting Started

To begin, navigate to the AWS Management Console and search for "EKS". Amazon EKS is essentially another flavor of Kubernetes, so if you're familiar with Kubernetes, you'll find EKS easy to use.

## Terraform for Quick EKS Setup

While EKS is convenient, it can be expensive. In this guide, we'll quickly set up an EKS cluster using Terraform and delete it soon after to minimize costs.

## Understanding EKS Cluster and Worker Nodes

- When setting up an EKS cluster, we provision the master node, which is the control plane of the Kubernetes cluster.
- We also create worker nodes, which are standard nodes in the Kubernetes cluster where your applications will run.
- It's important to note that the pricing models for worker nodes and the EKS cluster itself are different, and you need to consider both.

## Accessing Your Cluster and Deploying Applications

Once the EKS cluster is created, you can access and manage it using the Kubeconfig file. The Kubeconfig file contains the necessary configuration details to connect to your cluster.
You can then deploy your containerized applications and manage them within the Kubernetes environment.

## Utilizing Terraform for EKS Cluster and VPC

To create a Kubernetes cluster with Terraform, we leverage its capabilities to also set up the associated Amazon Virtual Private Cloud (VPC). Terraform's modular approach is a significant advantage, as it allows you to reuse existing modules, saving you from starting from scratch.

## Leveraging Terraform Modules

Terraform's strength lies in its extensive module ecosystem. You don't need to write all the code yourself. You can take advantage of various Terraform modules available for free. You can easily find these modules by searching for "Terraform modules" on your preferred search engine.

Explore the Terraform module registry: [Terraform Module Registry](https://registry.terraform.io/browse/modules?product_intent=terraform)


## Understanding Terraform Modules
Modules are containers for multiple resources that are used together. A module consists of a collection of .tf and/or .tf.json files kept together in a directory.

Modules are the main way to package and reuse resource configurations with Terraform.

Terraform modules are a powerful feature. They encapsulate reusable configurations, making it easier to create and manage resources. AWS provides Terraform modules that simplify the creation and management of resources like VPCs.

### VPC Module

The VPC module allows us to create a Virtual Private Cloud (VPC) for our EKS cluster. It provides flexibility through various customizable variables.

### EKS Module

The EKS module helps us create the EKS cluster and its managed node groups. It handles the provisioning and scaling of the nodes in our cluster.

## Creating a VPC and EKS Cluster

Our first step is to set up a Virtual Private Cloud (VPC) where we'll create the Amazon Elastic Kubernetes Service (EKS) cluster along with its nodes.

Let's take a closer look at the VPC module:

### Module VPC
Usage

#### Variables
- Name
- Sidr
- Azs
- Private Subnets
- Public Subnets
- Enable Nat Gateways

```sh
module "vpc" {
  source = "terraform-aws-modules/vpc/aws"
  version = "3.14.2"

  name = "terra-eks-vpc"

  cidr = "172.20.0.0/16"
  azs = slice(data.aws_availability_zones.available.names, 0, 3)

  private_subnets = ["172.20.1.0/24", "172.20.2.0/24", "172.20.3.0/24"]
  public_subnets = ["172.20.4.0/24", "172.20.5.0/24", "172.20.6.0/24"]

  enable_nat_gateway = true
  single_nat_gateway = true
  enable_dns_hostnames = true

  public_subnet_tags = {
    "kubernetes.io/cluster/${local.cluster_name}" = "shared"
    "kubernetes.io/role/elb" = 1
  }

  private_subnet_tags = {
    "kubernetes.io/cluster/${local.cluster_name}" = "shared"
    "kubernetes.io/role/internal-elb" = 1
  }
}
```
#### Module Declaration:
```sh
module "vpc" {
```
This declares a Terraform module named "vpc".

#### Module Source and Version:
```sh
source = "terraform-aws-modules/vpc/aws"
version = "3.14.2"
```
Specifies the source and version of the module to use. In this case, it's using the `terraform-aws-modules` organization's VPC module version 3.14.2.

#### VPC Name and CIDR Block:
```sh
name = "terra-eks-vpc"
cidr = "172.20.0.0/16"
```
Sets the name and CIDR block for the VPC.

#### Availability Zones (AZs):
```sh
azs = slice(data.aws_availability_zones.available.names, 0, 3)
```
Determines the availability zones to use for the VPC. It's using the first 3 AZs available in the AWS region.

#### Subnet Configurations:
```sh
private_subnets = ["172.20.1.0/24", "172.20.2.0/24", "172.20.3.0/24"]
public_subnets = ["172.20.4.0/24", "172.20.5.0/24", "172.20.6.0/24"]
```
Defines the CIDR blocks for private and public subnets within the VPC.

#### NAT Gateway and DNS Settings:
```sh
enable_nat_gateway = true
single_nat_gateway = true
enable_dns_hostnames = true
```
Enables NAT gateways, a single NAT gateway, and DNS hostnames within the VPC.

#### Tags for Subnets:
```sh
public_subnet_tags = {
  "kubernetes.io/cluster/${local.cluster_name}" = "shared"
  "kubernetes.io/role/elb" = 1
}

private_subnet_tags = {
  "kubernetes.io/cluster/${local.cluster_name}" = "shared"
  "kubernetes.io/role/internal-elb" = 1
}
```
Sets tags for public and private subnets, which can be used for Kubernetes networking purposes.

### Module EKS
Usage

```sh
module "eks" {
  source = "terraform-aws-modules/eks/aws"
  version = "19.0.4"

  cluster_name = local.cluster_name
  cluster_version = "1.27"

  vpc_id = module.vpc.vpc_id
  subnet_ids = module.vpc.private_subnets
  cluster_endpoint_public_access = true

  eks_managed_node_group_defaults = {
    ami_type = "AL2_x86_64"

  }

  eks_managed_node_groups = {
    one = {
      name = "node-group-1"

      instance_types = ["t3.small"]

      min_size = 1
      max_size = 3
      desired_size = 2
    }

    two = {
      name = "node-group-2"

      instance_types = ["t3.small"]

      min_size = 1
      max_size = 2
      desired_size = 1
    }
  }
}
```

#### Module Declaration:
```sh
module "eks" {
```
This declares a Terraform module named "eks" for managing an EKS cluster.

#### Module Source and Version:
```sh
source = "terraform-aws-modules/eks/aws"
version = "19.0.4"
```
Specifies the source and version of the module to use. In this case, it's using the `terraform-aws-modules` organization's EKS module version 19.0.4.

#### Cluster Name and Version:
```sh
cluster_name = local.cluster_name
cluster_version = "1.27"
```
Sets the name and version of the EKS cluster.

#### VPC and Subnet IDs:
```sh
vpc_id = module.vpc.vpc_id
subnet_ids = module.vpc.private_subnets
```
Specifies the VPC ID and the IDs of the private subnets where the EKS nodes will be placed.

#### Cluster Endpoint Public Access:
```sh
cluster_endpoint_public_access = true
```
Enables public access to the EKS cluster endpoint.

#### EKS Managed Node Group Defaults:
```sh
eks_managed_node_group_defaults = {
  ami_type = "AL2_x86_64"
}
```
Sets defaults for the managed node groups, specifying the Amazon Machine Image (AMI) type.

#### EKS Managed Node Groups:
```sh
eks_managed_node_groups = {
  one = {
    name = "node-group-1"
    instance_types = ["t3.small"]
    min_size = 1
    max_size = 3
    desired_size = 2
  },
  two = {
    name = "node-group-2"
    instance_types = ["t3.small"]
    min_size = 1
    max_size = 2
    desired_size = 1
  }
}
```
Defines two managed node groups named "node-group-1" and "node-group-2" with specified configurations for instance types, minimum size, maximum size, and desired size.

So essentially, our Terraform code will consist of these two modules, along with additional details such as provider, backend, endpoints, and related configurations.





## Using Terraform for EKS Setup

To set up EKS with Terraform, follow these steps:

1. Install Terraform using your platform's package manager.

2. Create an IAM user in AWS and download the access key.

3. Configure Terraform using `terraform init`.

4. Customize variables in the `variables.tf` file, like the region and cluster name.

5. Run `terraform plan` to see what resources will be created.

6. Run `terraform apply` to create the resources.

7. Validate the setup by generating the kubeconfig file and using `kubectl` to interact with the cluster.

8. Once validated, run `terraform destroy` to clean up the resources and minimize costs.

## Conclusion

Setting up an Amazon EKS cluster using Terraform provides an efficient and cost-effective way to manage your Kubernetes workloads. Make sure to carefully manage your AWS resources to control costs effectively.

You can find the source code and additional explanations in the [GitHub repository](https://github.com/coder/repo-profile/project/tree/TerraForm-x).

Step 2: Setting Up the AWS Configuration
To configure AWS credentials for Terraform, create an IAM user with the necessary permissions and generate an access key. Configure the AWS CLI using the access key and region using the aws configure command.

```sh
aws configure
```
Step 3: Modifying Variables
In the variables.tf file, modify the region variable to match the AWS region where you want to create the EKS cluster. Also, adjust the cluster_name variable to your desired cluster name.

```sh
variable "region" {
  default = "us-east-1" # Change this to your desired AWS region
}

variable "cluster_name" {
  default = "my-eks-cluster" # Change this to your desired cluster name
}
```

Step 4: Terraform Plan
Run the following command to create an execution plan:

```sh
terraform plan

```

This command shows you what actions Terraform will take before actually doing it. Verify the plan and ensure it aligns with your expectations.

Step 5: Applying the Terraform Configuration
Apply the Terraform configuration to create the EKS cluster and associated resources:

```sh
terraform apply
```

You will be prompted to confirm the action. Enter yes to proceed.

Step 6: Configuring kubectl
After the EKS cluster is successfully created, configure kubectl to connect to your cluster. Run the following command, replacing my-eks-cluster with your cluster name:

```sh
aws eks --region $(terraform output region) update-kubeconfig --name my-eks-cluster

```

This command updates the kubeconfig file with the necessary cluster details.

Step 7: Verifying the Cluster
Verify that the cluster is running by checking the nodes:

```sh
kubectl get nodes
```

You should see the nodes in your EKS cluster.

Step 8: Cleaning Up
Once you're done testing, it's crucial to clean up and destroy the resources to avoid unnecessary charges. Run the following command to destroy the resources:

```sh
terraform destroy
```

Enter yes to confirm the destruction.

Conclusion
Setting up an Amazon EKS cluster using Terraform provides an efficient and cost-effective way to manage your Kubernetes workloads. Make sure to carefully manage your AWS resources to control costs effectively.

You can find the source code and additional explanations in the GitHub repository.

This article was written by [Your Name] on [Date]. If you have any questions or feedback, feel free to reach out at [your@email.com].

Keep learning and happy coding!

Disclaimer: The steps and information in this article are subject to change as AWS services and Terraform evolve. Always refer to the official AWS and Terraform documentation for the most up-to-date and accurate information.
