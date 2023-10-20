# Setting up an Amazon EKS Cluster with Terraform

In this article, I'll guide you through setting up an Amazon Elastic Kubernetes Service (EKS) cluster using Terraform. EKS is a managed Kubernetes service provided by Amazon Web Services (AWS) that allows for easy deployment and management of Kubernetes clusters.

## Prerequisites

### Setting Up Terraform and IAM User
To get started with Terraform, you'll need to install it and set up an IAM user with appropriate access. Here's a step-by-step guide:

#### Installing Terraform
1. **Windows (using Chocolatey):**
To install Terraform on Windows, you can use the Chocolatey package manager. If you haven't installed Chocolatey already, search google on how to install then install it.

Open your terminal (e.g., Git Bash) and run the following command:
    ```sh
    choco install terraform
    ```
2. **MacOS (using Homebrew):**
For MacOS users, Terraform can be installed using Homebrew. If you don't have Homebrew installed, you can set it up by visiting the Homebrew website.

Open your terminal and run this command:
    ```sh
    brew install terraform
    ```
This will install Terraform on your system.

#### Creating an IAM User
1. **Create an IAM User:**
If you don't already have an IAM user, you'll need to create one in your AWS account. You can choose any name you prefer.

2. **Assign Administrator Access:**
While creating the IAM user, make sure to grant it the "AdministratorAccess" policy to provide full access to AWS resources. You can manage these settings in the AWS Management Console.

3. **Access Key:**
After creating the IAM user, go to its "Security Credentials" section. Here, you'll find an option to create an access key. Click "Create Access Key."

4. **Download Access Key:**
Download the access key ID and the corresponding secret access key. Keep these credentials safe and never share them in public places like code repositories.

#### Configuring AWS Credentials
1. **Open Your Terminal:**
You can use a terminal like Git Bash on Windows or the built-in terminal on MacOS.

2. **Run aws configure:**
Use the following command to configure aws credential using your IAM user credentials, specifying the AWS region and setting the output format to JSON:
    ```sh
    aws configure
    ```
You will be prompted to enter the access key ID, secret access key, region (e.g., "us-east-1" for the US East region), and output format (choose "json").

With these steps, you'll have Terraform installed and properly configured with your IAM user's access credentials, allowing you to interact with AWS resources as needed in your infrastructure code.

### **Install Kubectl on your system**

[Kubectl](https://kubernetes.io/docs/reference/kubectl/kubectl/) is a command-line tool for interacting with Kubernetes clusters. Here are the steps to install `kubectl` on both Mac and Windows:

#### Installing kubectl on Mac

1. **Homebrew Installation**:
   - If you don't have Homebrew installed, open your terminal and run this command to install it:
     ```sh
     /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
     ```

2. **Install kubectl**:
   - Run the following command to install `kubectl` using Homebrew:
     ```sh
     brew install kubectl
     ```

3. **Verify Installation**:
   - To ensure `kubectl` was installed successfully, run the following command:
     ```sh
     kubectl version --client
     ```

#### Installing kubectl on Windows

1. **Chocolatey Installation**:
   - If you don't have Chocolatey installed, open Command Prompt as an administrator and run this command to install Chocolatey:
     ```powershell
     Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))
     ```

2. **Install kubectl**:
   - Run the following command in Command Prompt to install `kubectl` using Chocolatey:
     ```powershell
     choco install kubernetes-cli
     ```

3. **Verify Installation**:
   - To verify that `kubectl` was installed successfully, open Command Prompt and run the following command:
     ```sh
     kubectl version --client
     ```

That's it! You've now successfully installed `kubectl`. It will be used it to interact with our Kubernetes clusters.



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


vpc.tf
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
Determines the availability zones to use for the VPC. It's using the first 3 AZs available in the AWS region. Returned as a list, stored in the variable `azs` Or you can just create a list here and mention it manually.


#### Subnet Configurations:
    ```sh
    private_subnets = ["172.20.1.0/24", "172.20.2.0/24", "172.20.3.0/24"]
    public_subnets = ["172.20.4.0/24", "172.20.5.0/24", "172.20.6.0/24"]
    ```
Defines the CIDR blocks for private and public subnets within the VPC. If making your subnets two remember to change the azs from 0 to 3 to 0 to 2.


#### NAT Gateway and DNS Settings:
    ```sh
    enable_nat_gateway = true
    single_nat_gateway = true
    enable_dns_hostnames = true
    ```
Enables NAT gateways, a single NAT gateway, and DNS hostnames within the VPC. So it will create a nat gateway and you will have three nat gateways if you have three private subnets.

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
Sets tags for public and private subnets, which can be used for Kubernetes networking purposes. From the documention you have to specify tags. The given variable name local.cluster_name will be generated after the cluster is created

### Module EKS
Usage
eks-cluster.tf
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
The name of the node group instance type is "T3 small." When you use this specific instance type, you'll be charged for the usage of T3 small instances and the cluster itself.

Here's a cost-saving tip: If you use this setup for less than an hour, your bill will be quite low. It's advisable not to keep it running for an extended period to minimize your expenses.

This setup also takes care of autoscaling. Think of it as a way to automatically adjust the number of instances in your group based on the current workload. We've set the rules for this autoscaling group as follows:

At least one instance (minimum size), but no more than three instances (maximum size), with an initial count of two instances (desired size).
This means that initially, you'll have two instances running. If the demand increases, it can automatically scale up to a maximum of three instances, thanks to the autoscaling group. So you won't have to worry about manually adjusting the number of instances.

You can also choose larger instance sizes if needed. This setup is divided into two node groups. In the first group, we've set the desired size to one, so it will have a total of three instances: two smaller ones and a control plane, although the control plane instance won't be visible to you. We refer to the control plane as the heart of the cluster.

Additionally, there will be an endpoint and a public endpoint, which we've set to "true." This provides you with an accessible endpoint to interact with your setup. (So we get the endpoint to access it. No more than that.)


So essentially, our Terraform code will consist of these two modules, along with additional details such as provider, backend, endpoints, and related configurations.


### Setting Up Dependencies for the Project

Let's outline the dependencies required for the project, including providers and backend configuration. Make sure to customize this setup to your specific needs.

terraform.tf
    ```sh
    terraform {
    required_providers {
        aws = {
        source = "hashicorp/aws"
        version = "~> 4.46.0"
        }

        random = {
        source = "hashicorp/random"
        version = "~> 3.4.3"
        }

        tls = {
        source = "hashicorp/tls"
        version = "~> 4.0.4"
        }

        cloudinit = {
        source = "hashicorp/cloudinit"
        version = "~> 2.2.0"
        }

        kubernetes = {
        source = "hashicorp/kubernetes"
        version = "~> 2.16.1"
        }
    }

    backend "s3" {
        bucket         	   = "terra-eks12"
        key              	   = "state/terraform.tfstate"
        region         	   = "us-east-1"
    }

    required_version = "~> 1.3"
    }
    ```
This Terraform configuration file (`terraform.tf`) sets up the necessary providers, defines the backend for storing Terraform state, and specifies the required Terraform version.

Breakdown of the terraform Block
- **required_providers:** In this block, multiple providers are defined, including AWS(VPC), Random(For encrypting the key certificates), TLS, CloudInit, and Kubernetes. Each provider is assigned a specific source and version to ensure compatibility with our infrastructure.

#### Required Providers
For this project, we will be using AWS modules, specifically the VPC module.

#### AWS VPC

- **Provider**: AWS
- **Module**: VPC

#### Dependencies

To run our infrastructure code, we have a few additional dependencies:

- **Random**: We use the random provider for generating random values, which can be helpful for generating encryption keys and certificates.
- **TLS**: TLS is essential for secure communication, and we use it in our project for encryption.
- **Cloud Init**: Cloud Init is a tool for customizing cloud instances during the launch process. It's essential for configuring our instances.
- **Kubernetes**: Kubernetes for EKS

#### Backend Configuration

To maintain the state of our Terraform resources, we need to set up a backend. In this example, we're using an S3 bucket as our backend. You can choose a different backend if needed, but ensure it's well-suited to your project.

Here are the steps to configure the backend:

1. Create an S3 Bucket: You can either use an existing S3 bucket or create a new one. To create a new bucket, follow these steps:
   
   - Log in to your AWS console.
   - Create a new S3 bucket with a unique name. In this example, we've created a bucket named "terra-eks12." You should choose a different name.
   - Specify the region for the bucket. In our case, it's "us-east-1." Ensure that the bucket and the cluster are in the same region. While not mandatory, it's a good practice.

2. Mention the Bucket Name: Make sure to specify the S3 bucket name you created or selected for the backend.

- **backend "s3"**
This block configures the backend where Terraform state files are stored. In this case, an S3 bucket named "terra-eks12" is specified for state file storage. The key parameter identifies the location and name of the state file within the bucket. Additionally, the region is set to `"us-east-1"` to ensure the region consistency between the backend and your infrastructure.

- **required_version:** Lastly, the required Terraform version is set to `"~> 1.3,"` ensuring that Terraform will be used within the specified version range.


### Setting up the Variables file
We will have two simple variables defined. The region and the Cluster name.

variables.tf
    ```sh
    variable "region" {
    description = "AWS region"
    type = string
    default = "us-east-1"
    }

    variable "clusterName" {
    description = "Name of the EKS cluster"
    type = string
    default = "terra-eks"
    }
    ```
The Breakdown

- **variable "region" {**
This line starts the definition of a new variable named `"region."` Variables in Terraform allow you to parameterize your configuration, making it easier to reuse and customize your infrastructure code.

- **description = "AWS region"**
This line sets a description for the `"region" variable.` The description is a human-readable explanation of what the variable is used for. In this case, it describes that the variable is meant to specify the AWS region.

- **type = string**
Here, the line specifies the data type of the variable. The `"region"` variable is of type `"string,"` which means it can hold text values, such as `"us-east-1."`

- **default = "us-east-1"**
This line sets a default value for the `"region"` variable. The default value is `"us-east-1,"` which means that if no value is explicitly provided when using this variable, Terraform will use `"us-east-1"` as the default value.

The next set of lines defines another variable named "clusterName." The structure and purpose of this variable are very similar to the "region" variable, but it serves a different purpose:

- **variable "clusterName" {**
Starts the definition of a new variable named `"clusterName."`

- **description = "Name of the EKS cluster"**
Sets a description for the `"clusterName"` variable, indicating that it is used to specify the name of an Amazon Elastic Kubernetes Service (EKS) cluster.

- **type = string**
Specifies that the `"clusterName"` variable is of type `"string,"` which means it can hold text values.

- **default = "terra-eks"**
Sets a default value for the `"clusterName"` variable. The default value is `"terra-eks."`


### Set up maintf file
main.tf
    ```sh
    provider "kubernetes" {
    host = module.eks.cluster_endpoint
    cluster_ca_certificate = base64decode(module.eks.cluster_certificate_authority_data)
    }

    provider "aws" {
    region = var.region
    }

    data "aws_availability_zones" "available" {}

    locals {
    cluster_name = var.clusterName
    }
    ```
The Breakdown

Here, we're setting up the configuration for two key components: the Kubernetes provider and the AWS provider.

#### Kubernetes Provider Configuration:
We're configuring the Kubernetes provider to interact with a Kubernetes cluster. This provider needs two important values:

- **Host:** This value represents the address of the Kubernetes cluster. But at this point, we don't have this address. We'll get it from another part of our code: module.eks.cluster_endpoint. Think of it as finding the URL for the control plane of the cluster, similar to what you'd have in a kubeconfig file. The module.eks is responsible for creating the cluster, and it will provide this address.

- **Cluster Certificate:** This is used for secure communication with the Kubernetes cluster. We're decoding this certificate from module.eks.cluster_certificate_authority_data using base64 decoding. The module.eks provides this data, and we're storing it in the `cluster_ca_certificate` variable. These values are essential for setting up our Kubernetes provider to interact with the cluster.

#### AWS Provider Configuration:
We're also configuring the AWS provider, which is used to manage resources in AWS. This provider requires specifying the AWS region. In our case, the region is specified as `var.region.` The `var.region` variable is defined in our variables file, and it is set to "US East 1." This means that when we work with AWS resources in this configuration, it will be in the "US East 1" region.

- **provider "kubernetes" {**
This line starts the definition of a Kubernetes provider configuration block. It specifies that Terraform will interact with a Kubernetes cluster. The block contains the following settings:

- **host = module.eks.cluster_endpoint:** This setting defines the host address of the Kubernetes cluster. It uses the `module.eks.cluster_endpoint` variable to obtain the endpoint of the Amazon Elastic Kubernetes Service (EKS) cluster created with Terraform.
And in this we need to pass the value for host. (So we don't have the host yet 
So once this is created, it is going to return the endpoint of the cluster) 

- **cluster_ca_certificate = base64decode(module.eks.cluster_certificate_authority_data):** This setting specifies the cluster's CA certificate data. It uses the `module.eks.cluster_certificate_authority_data` variable, which likely contains the certificate authority data for the EKS cluster. The `base64decode` function decodes the `base64-encoded` data. (This is required to use the kubernetes module)

- **provider "aws" {**
This line starts the definition of an AWS provider configuration block. It specifies that Terraform will interact with AWS resources. The block contains the following setting:

- **region = var.region:** This setting defines the AWS region to use for AWS resources. It uses the `var.region` variable, which allows you to specify the region for AWS operations. This region should match the one specified in the `var.region` variable defined in our variables file.

- **data "aws_availability_zones" "available" {}**
(So we will be creating VPC. And we don't know the the names of the availability zones in this region us-east-1 this is going to fetch that `data availability_zones available`, and this is going to get the list of all the zones in that region. And that is mentioned in the VPC module. )
This line specifies a data block to retrieve information about AWS availability zones. This information can be used in our Terraform configuration for making decisions related to resource placement and redundancy. The block has the following attributes:

- **aws_availability_zones:** This is the data source for availability zones.

- **"available":** This is the name given to the data block, and you can refer to it later in your configuration using this name.

- **locals {**
This line starts the definition of a local block. Locals are used to define variables that are specific to your Terraform configuration and are not meant for direct interaction with resources or providers. In this block, a local variable is defined:

- **cluster_name = var.clusterName:** This local variable named cluster_name is assigned the value of the `var.clusterName` variable, which allows you to specify the name of the EKS cluster.

`var.clusterName` is defined in our variable file, and this local variable makes it more convenient to reference within our configuration.

### Run Terraform Commands
## Terraform `init` Command
1. **Run terraform init**
    ```sh
    terraform init
    ```

`terraform init` is a fundamental command in Terraform, a widely used infrastructure as code (IaC) tool. This command is essential for initializing a new or existing Terraform configuration within a working directory. Here's a breakdown of what `terraform init` accomplishes:

- **Plugin Installation:** The primary purpose of `terraform init` is to download and install the necessary provider plugins. Terraform relies on these plugins to interact with a variety of cloud and infrastructure platforms, including AWS, Azure, Google Cloud, and more. These plugins facilitate the creation and management of resources on the designated platform.

- **Initializing Backend Configuration:** If your Terraform configuration specifies a backend for remote state storage (e.g., using an S3 bucket in AWS or a remote system), `terraform init` also handles the initialization of the backend configuration. This includes configuring the required connection details and storage settings to manage your Terraform state remotely.

- **Downloading Modules:** In cases where your configuration references external modules (reusable blocks of Terraform code), `terraform init` takes care of downloading these modules from the specified sources, which can be Git repositories. This ensures that your configuration can utilize these modules as foundational building blocks for your infrastructure.

- **Validation:** As part of the initialization process, Terraform conducts a basic validation of your configuration files. This step ensures that your files are correctly formatted and free of syntax errors. If any issues are detected, Terraform will report them during the initialization.

- **Lock File Creation:** `terraform init` generates a lock file that records the precise versions of the provider plugins and modules that are being utilized. This lock file is instrumental in maintaining configuration consistency and preventing inadvertent changes due to updates in provider plugins or modules.

- **Initialization Summary:** Upon completing these tasks, `terraform init` typically provides a summary of the initialized environment. This summary includes details about the configured plugins, modules, and backend settings.

In summary, `terraform init` sets the stage for your Terraform environment, ensuring that all the required components are in place and ready for actions like creating, updating, or destroying infrastructure resources, as defined in your Terraform configuration. This command is a critical first step before executing any other Terraform commands, such as `terraform plan` or `terraform apply`.

2. **Run terraform plan**
Run the following command to create an execution plan:

    ```sh
    terraform plan
    ```
This command shows you what actions Terraform will take before actually doing it. Verify the plan and ensure it aligns with your expectations.

On this project, the plan is to add 53 resources.

3. **Applying the Terraform Configuration**
Apply the Terraform configuration to create the EKS cluster and associated resources:

    ```sh
    terraform apply
    ```
You will be prompted to confirm the action. Enter yes to proceed. This might take some time.

After some waiting, my cluster has been successfully created. You can now access it through the cluster's endpoint. To view the cluster from the AWS Management Console, switch to the North Virginia region.

In the console, you'll find your Kubernetes cluster, and its endpoint will be displayed there. Please be cautious and do not attempt to delete the cluster directly from the console; instead, manage it exclusively through Terraform.

Now, let's take a closer look at the node groups. We have two node groups: one with two nodes and another with just one node.

Regarding networking, you should find the Virtual Private Cloud (VPC) there, and within the VPC, there are a total of three subnets—three public and three private subnets. Now, let's explore the VPC itself.

Here's a view of our VPC, and within the subnets, you should see a total of six subnets, comprising three public and three private subnets.

4. **Configuring kubectl**
After the EKS cluster is successfully created, configure kubectl to connect to your cluster. Run the following command, replacing my-eks-cluster with your cluster name:

    ```sh
    aws eks update-kubeconfig --region us-east-1 --name terra-eks
    ```
The terminal will output the location of your kubeconfig file after that command

Run this to see the kube config file
    ```sh
    cat ~/.kube/config
    ```
Now run
    ```sh
    kubectl get nodes
    ```
Running the kubectl get nodes command will provide you with information about the nodes that are part of your EKS cluster. The output of this command displays details about the nodes in your cluster, including their current status and available resources. Here's what the output typically shows:

- **NAME**: This column lists the names of the individual nodes within your EKS cluster. Each node represents an Amazon Elastic Compute Cloud (EC2) instance that is part of your Kubernetes cluster.

- **STATUS**: The "STATUS" column indicates the current status of each node. Nodes can be in various states, such as "Ready," "NotReady," "SchedulingDisabled," etc. "Ready" nodes are typically healthy and available for scheduling workloads.

- **ROLES**: If your EKS cluster has node groups with specific roles, this column will specify the role assigned to each node. For example, a node may be designated as a "worker" node.

- **AGE**: The "AGE" column displays how long each node has been part of the cluster, in terms of age.

- **VERSION**: This column indicates the Kubernetes version running on each node.

By running kubectl get nodes, you can monitor the health and status of the nodes in your EKS cluster. If you encounter issues or want to check node availability, this command provides essential insights into the cluster's infrastructure.

5. **Delete Cluster**
    ```sh
    terraform destroy
    ```
Enter `yes` when it prompts

That will delete everything

## Conclusion

Setting up an Amazon EKS cluster using Terraform provides an efficient and cost-effective way to manage your Kubernetes workloads. Make sure to carefully manage your AWS resources to control costs effectively.

You can find the source code and additional explanations in the [GitHub repository](https://github.com/Pearlicia/setup-eks-cluster-using-terraform).


This article was written by [Felicia Ebikon]. If you have any questions or feedback, feel free to reach out at [feliciaebikon@email.com].

Keep learning and happy coding!

Disclaimer: The steps and information in this article are subject to change as AWS services and Terraform evolve. Always refer to the official AWS and Terraform documentation for the most up-to-date and accurate information.
