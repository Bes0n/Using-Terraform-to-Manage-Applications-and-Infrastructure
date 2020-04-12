# Using Terraform to Manage Applications and Infrastructure
  

- [About Terraform](#about-terraform)
- [Setting Up Your Environment](#setting-up-your-environment)
    - [Using Cloud Playground](#using-cloud-playground)
    - [Setting up Docker Installing Terraform](#setting-up-docker-installing-terraform)
- [Terraform Basics](#terraform-basics)
    - [Terraform Commands](#terraform-commands)
    - [HashiCorp Configuration Language](#hashicorp-configuration-language)
    - [Terraform Console and Output](#terraform-console-and-output)
    - [Input Variables](#input-variables)
    - [Breaking Out Our Variables and Outputs](#breaking-out-our-variables-and-outputs)  
    - [Maps and Lookups](#maps-and-lookups)
    - [Terraform Workspaces](#terraform-workspaces)
- [Terraform Modules](#terraform-modules)
    - [Introduction to Modules](#introduction-to-modules)
    - [The Image Module](#the-image-module)
    - [The Container Module](#the-container-module)
    - [The Root Module](#the-root-module)
- [Terraform and Docker](#terraform-and-docker)  
    - [Managing Docker Networks](#managing-docker-networks)
    - [Managing Docker Volumes](#managing-docker-volumes)
    - [Creating Swarm Services](#creating-swarm-services)
    - [Using Secrets](#using-secrets)
- [Using Terraform in a CI/CD Environment](#using-terraform-in-a-cicd-environment)
    - [Building a Custom Jenkins Image](#building-a-custom-jenkins-image)
    - [Setting Up Jenkins](#setting-up-jenkins)
    - [Creating a Jenkins Job](#creating-a-jenkins-job)
    - [Building a Jenkins Pipeline Part 1](#building-a-jenkins-pipeline-part-1)
    - [Building a Jenkins Pipeline Part 2](#building-a-jenkins-pipeline-part-2)
    - [Building a Jenkins Pipeline Part 3](#building-a-jenkins-pipeline-part-3)
- [Terraform and AWS](#terraform-and-aws)
    - [Setting Up a Cloud Sandbox](#setting-up-a-cloud-sandbox)
    - [Our Architecture: What We're Going to Build](#our-architecture-what-were-going-to-build)
    - [Storage Part 1: The S3 Bucket and Random ID](#storage-part-1-the-s3-bucket-and-random-id)
    - [Storage Part 2: The Root Module](#storage-part-2-the-root-module)
    - [Networking Part 1: VPC, SG, Subnets](#networking-part-1-vpc-sg-subnets)
    - [Networking Part 2: The Root Module](#networking-part-2-the-root-module)
    - [Compute Part 1: AMI Data, Key Pair, and the File Function](#compute-part-1-ami-data-key-pair-and-the-file-function)
    - [Compute Part 2: The EC2 Instance](#compute-part-2-the-ec2-instance)
    - [Compute Part 3: The Root Module](#compute-part-3-the-root-module)
- [Troubleshooting](#troubleshooting)
    - [Troubleshooting Terraform Files](#troubleshooting-terraform-files)
- [Terraform State](#terraform-state)
    - [Terraform Formatting and Remote State](#terraform-formatting-and-remote-state)
    - [Using Remote State with Jenkins](#using-remote-state-with-jenkins)
- [Terraform and Kubernetes](#terraform-and-kubernetes)
    - [Setting up Kubernetes Installing Terraform](#setting-up-kubernetes-installing-terraform)
    - [Creating a Pod](#creating-a-pod)
    - [Creating a Pod and Service](#creating-a-pod-and-service)
    - [Creating a Deployment](#creating-a-deployment)
- [Terraform 0.12](#terraform-012)
    - [Setup and Disclaimer](#setup-and-disclaimer)
    - [Working with Resources](#working-with-resources)
    - [Input Variables](#input-variables)
    - [Output Variables](#output-variables)
    - [Dynamic Nested Blocks Part 1](#dynamic-nested-blocks-part-1)
    - [Dynamic Nested Blocks Part 2](#dynamic-nested-blocks-part-2)
    - [Expressions and Functions](#expressions-and-functions)

## About Terraform
- Terraform is a tool for building infrastructure
    - Allows simple version control
    - Available as open-source or enterprise software
        - Enterprise provides advanced collaboration and governance. 
    - Supports many popular service providers such as
        - AWS
        - OpenStack
        - Azure
        - GCP
        - Kubernetes
  
- Primary Terraform Features:
    - Infrastructure as Code (IaC)
        - Idempotent
        - High-level syntax
        - Easily reusable
    - Execution Plans
        - Show the intent of the deploy
        - Can help ensure everything in the development is intentional
    - Resource graph
        - Illustrates all changes and dependencies
  
- Some use cases for Terraform:
    - Hybrid clouds
        - Cloud-agnostic
        - Allows deployments to multiple providers simultaneously
    - Multi-tier architecture
        - Allows deployment of several layers of architecture
        - Is usually able to automatically deploy in the correct order
    - Software-defined networking
        - Able to deploy network architecture as well
  
- Terraform is a high-level infrastructure
    - Puppet, Chef, and other configuration management tools
        - Not intended for configuration management
        - Provides "provisioners" that can call these tools to perform the CM duties 
    - CloudFormation ant other IaC tools:
        - Many other tools are vendor-locked and only support one vendor. 
        - There are some tools that are similar to Terraform 
    - Boto and other lower-level tools
        - Terraform is a higher-level tool that tools such as Boto, which makes defining infrastructure easier. 

## Setting Up Your Environment
### Using Cloud Playground
In this video, we will create two Docker servers using Cloud Playground. These servers will be used to set up a Docker Swarm cluster.
  
We'll create two Cloud Servers from Cloud Playground, using the information below:
  
**Swarm Manager:**
Distribution: CentOS 7
Size: Medium
Tag: Docker Swarm Manager
  
**Swarm Worker:**
Distribution: CentOS 7
Size: Medium
Tag: Docker Swarm Worker

### Setting up Docker Installing Terraform
In this lesson, we will create a Docker Swarm cluster and install Terraform. We will start by installing Docker on both cloud servers, and then configure them to run in Swarm mode. After that we will install Terraform 0.11.13.
  
#### Installing Docker on the Swarm Manager and Worker
These actions will be executed on both the Swarm manager and worker nodes.

#### Update the operating system
```
sudo yum update -y
```

#### Prerequisites
Uninstall old versions:
```
sudo yum remove -y docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```

#### Install Docker CE
Install Utils:
```
sudo yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2
```
  
Add the Docker repository:
```
sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```
  
Install Docker CE:
```
sudo yum -y install docker-ce
```
  
Start Docker and enable it:
```
sudo systemctl start docker && sudo systemctl enable docker
```
  
Add `cloud_user` to the `docker` group:
```
sudo usermod -aG docker cloud_user
```
  
Test the Docker installation:
```
docker --version
```

#### Configuring Swarm Manager node
On the manager node, initialize the manager:
```
docker swarm init \
--advertise-addr [PRIVATE_IP]
```

#### Configure the Swarm Worker node
On the worker node, add the worker to the cluster:
```
docker swarm join --token [TOKEN] \
[PRIVATE_IP]:2377
```

#### Verify the Swarm cluster
List Swarm nodes:
```
docker node ls
```

#### Install Terraform
Install Terraform 0.11.13 on the Swarm manager:
```
sudo curl -O https://releases.hashicorp.com/terraform/0.11.13/terraform_0.11.13_linux_amd64.zip
sudo yum install -y unzip
sudo unzip terraform_0.11.13_linux_amd64.zip -d /usr/local/bin/
```
  
Test the Terraform installation:
```
terraform version
```

## Terraform Basics
### Terraform Commands
In this lesson, we begin working with Terraform commands. We will start by creating a very simple Terraform file that will pull down the an image from Docker Hub.
  
List the Terraform commands:
```
terraform
```
  
**Common commands:**
- `apply`: Builds or changes infrastructure
- `console`: Interactive console for Terraform interpolations
- `destroy`: Destroys Terraform-managed infrastructure
- `fmt`: Rewrites configuration files to canonical format
- `get`: Downloads and installs modules for the configuration
- `graph`: Creates a visual graph of Terraform resources
- `import`: Imports existing infrastructure into Terraform
- `init`: Initializes a new or existing Terraform configuration
- `output`: Reads an output from a state file
- `plan`: Generates and shows an execution plan
- `providers`: Prints a tree of the providers used in the configuration
- `push`: Uploads this Terraform module to Terraform Enterprise to run
- `refresh`: Updates local state file against real resources
- `show`: Inspects Terraform state or plan
- `taint`: Manually marks a resource for recreation
- `untaint`: Manually unmarks a resource as tainted
- `validate`: Validates the Terraform files
- `version`: Prints the Terraform version
- `workspace`: Workspace management
  
Set up the environment:
```
mkdir -p terraform/basics
cd terraform/basics
```
  
Create a Terraform script:
```
vi main.tf
```
  
`main.tf` contents:
```
# Download the latest Ghost image
resource "docker_image" "image_id" {
  name = "ghost:latest"
}
```
  
Initialize Terraform:
```
terraform init
```
  
Validate the Terraform file:
```
terraform validate
```
  
List providers in the folder:
```
ls .terraform/plugins/linux_amd64/
```

List providers used in the configuration:
```
terraform providers
```
  
Terraform Plan:
```
terraform plan
```
  
Useful flags for plan:
- `-out=path`: Writes a plan file to the given path. This can be used as input to the "apply" command.
- `-var 'foo=bar'`: Set a variable in the Terraform configuration. This flag can be set multiple times.
  
Terraform Apply:
```
terraform apply
```
  
Useful flags for `apply`:
- `-auto-approve`: This skips interactive approval of plan before applying.
- `-var 'foo=bar'`: This sets a variable in the Terraform configuration. It can be set multiple times.
  
Confirm your apply by typing **yes**. The apply will take a bit to complete.
  
List the Docker images:
```
docker image ls
```
  
Terraform Show:
```
terraform show
```
  
Terraform Destroy:
```
terraform destroy
```
  
Confirm your `destroy` by typing **yes**.
  
Useful flags for destroys:
- `-auto-approve`: Skip interactive approval of plan before applying.
  
Re-list the Docker images:
```
docker image ls
```
  
Using a plan:
```
terraform plan -out=tfplan
```
  
Applying a plan:
```
terraform apply tfplan
```
  
Show the Docker Image resource:
```
terraform show
```
  
Destroy the resource once again:
```
terraform destroy
```

### HashiCorp Configuration Language
In this lesson, we will cover the basics of the Terraform configuration language, as well as explore providers and resources. Continuing what we started in Terraform Commands, we will modify `main.tf` so we can deploy a Ghost container to Docker.
  
The syntax of Terraform configurations is called HashiCorp Configuration Language (HCL). It is meant to strike a balance between being human-readable and editable, and being machine-friendly. For machine-friendliness, Terraform can also read JSON configurations. For general Terraform configurations, however, we recommend using the HCL Terraform syntax.

#### Terraform code files
The Terraform language uses configuration files that are named with the `.tf` file extension. There is also a JSON-based variant of the language that is named with the `.tf.json` file extension.

#### Terraform Syntax
Here is an example of Terraform's HCL syntax:
```
resource "aws_instance" "example" {
  ami = "abc123"

  network_interface {
    # ...
  }
}
```

#### Syntax reference:
- Single line comments start with `#`.
- Multi-line comments are wrapped with `/*` and `*/`.
- Values are assigned with the syntax of `key = value`.
- Strings are in double-quotes.
- Strings can interpolate other values using syntax wrapped in `${}`, for example `${var.foo}`.
- Numbers are assumed to be base 10.
- Boolean values: true, false
- Lists of primitive types can be made with square brackets (`[]`), for example `["foo", "bar", "baz"]`.
- Maps can be made with braces (`{}`) and colons (`:`), for example `{ "foo": "bar", "bar": "baz" }`.

#### Style Conventions:
- Indent two spaces for each nesting level.
- With multiple arguments, align their equals signs.
  
Setup the environment:
```
cd terraform/basics
```

#### Deploying a container using Terraform
Redeploy the Ghost image: 
```
terraform apply
```
  
Confirm the apply by typing **yes**. The `apply` will take a bit to complete.
  
Open `main.tf`: 
```
vi main.tf
```
  
`main.tf` contents:
```
# Download the latest Ghost image
resource "docker_image" "image_id" {
  name = "ghost:latest"
}

# Start the Container
resource "docker_container" "container_id" {
  name  = "ghost_blog"
  image = "${docker_image.image_id.latest}"
  ports {
    internal = "2368"
    external = "80"
  }
}
```
  
Validate `main.tf`:
```
terraform validate
```
  
Terraform Plan: 
```
terraform plan
```
  
Apply the changes to `main.tf`: 
```
terraform apply
```
  
Confirm the `apply` by typing **yes**.
  
List the Docker containers: 
```
docker container ls
```
  
Access the Ghost blog by opening a browser and go to: 
```
http:://[SWARM_MANAGER_IP]
```
  
#### Cleaning up the environment
Reset the environment: 
```
terraform destroy
```
  
Confirm the `destroy` by typing **yes**.
  
### Tainting and Updating Resources
In this lesson, we are going to take a look at how to force a redeploy of resources using tainting. This is an extremely useful skill for when parts of a deployment need to be modified.

#### Tainting and Untainting Resources
Terraform commands:
  
- `taint`: Manually mark a resource for recreation 
- `untaint`: Manually unmark a resource as tainted
  
Tainting a resource: 
```
terraform taint [NAME]
```
  
Untainting a resource: 
```
terraform untaint [NAME]
```
  
Set up the environment: 
```
cd terraform/basics
```
  
Redeploy the Ghost image: 
```
terraform apply
```
  
Taint the Ghost blog resource: 
```
terraform taint docker_container.container_id
```
  
See what will be changed: 
```
terraform plan
```
  
Remove the taint on the Ghost blog resource: 
```
terraform untaint docker_container.container_id
```
  
Verity that the Ghost blog resource is untainted: 
```
terraform plan
```
  
#### Updating Resources
Let's edit `main.tf` and change the image to `ghost:alpine`.
  
Open `main.tf`: 
```
vi main.tf
```
  
`main.tf` contents:
```
# Download the latest Ghost image
resource "docker_image" "image_id" {
  name = "ghost:alpine"
}

# Start the Container
resource "docker_container" "container_id" {
  name  = "ghost_blog"
  image = "${docker_image.image_id.latest}"
  ports {
    internal = "2368"
    external = "80"
  }
}
```
  
Validate changes made to `main.tf`: 
```
terraform validate
```
  
See what changes will be applied: 
```
terraform plan
```
  
Apply image changes: 
```
terraform apply
```
  
List the Docker containers: 
```
docker container ls
```
  
See what image Ghost is using: 
```
docker image ls | grep [IMAGE]
```
  
Check again to see what changes will be applied: 
```
terraform plan
```
  
Apply container changes: 
```
terraform apply
```
  
See what image Ghost is now using: 
```
docker image ls | grep [IMAGE]
```

#### Cleaning up the environment
Reset the environment: 
```
terraform destroy
```
  
Confirm the `destroy` by typing **yes**.
  
List the Docker images: 
```
docker image ls
```
  
Remove the Ghost blog image: 
```
docker image rm ghost:latest
```
  
Reset `main.tf`: 
```
vi main.tf
```
  
`main.tf` contents:
```
# Download the latest Ghost image
resource "docker_image" "image_id" {
  name = "ghost:latest"
}

# Start the Container
resource "docker_container" "container_id" {
  name  = "ghost_blog"
  image = "${docker_image.image_id.latest}"
  ports {
    internal = "2368"
    external = "80"
  }
}
```

### Terraform Console and Output
In this lesson, we will use the Terraform Console to view various outputs that we can use for our scripts. The Terraform Console is extremely useful for troubleshooting and planning deployments.
  
Terraform commands:
- `terraform console`: Interactive console for Terraform interpolations
  
Set up the environment:
```
cd terraform/basics
```

#### Working with the Terraform console
Redeploy the Ghost image and container:
```
terraform apply
```
  
Show the Terraform resources:
```
terraform show
```
  
Start the Terraform console:
```
terraform console
```
  
Type the following in the console to get the container's name:
```
docker_container.container_id.name
```
  
Type the following in the console to get the container's IP:
```
docker_container.container_id.ip_address
```
  
Break out of the Terraform console by using **Ctrl+C**.
  
Destroy the environment:
```
terraform destroy
```

#### Output the name and IP of the Ghost blog container
Edit `main.tf`:
```
vi main.tf
```
  
`main.tf` contents:
```
# Download the latest Ghost Image
resource "docker_image" "image_id" {
  name = "ghost:latest"
}

# Start the Container
resource "docker_container" "container_id" {
  name  = "blog"
  image = "${docker_image.image_id.latest}"
  ports {
    internal = "2368"
    external = "80"
  }
}

#Output the IP Address of the Container
output "ip_address" {
  value       = "${docker_container.container_id.ip_address}"
  description = "The IP for the container."
}

#Output the Name of the Container
output "container_name" {
  value       = "${docker_container.container_id.name}"
  description = "The name of the container."
}
```
  
Validate changes made to `main.tf`: 
```
terraform validate
```
  
Apply changes to get output: 
```
terraform apply
```
  
#### Cleaning up the environment
Reset the environment:
```
terraform destroy
```
  
### Input Variables
Input variables serve as parameters for a Terraform file. A variable block configures a single input variable for a Terraform module. Each block declares a single variable.
  
Syntax:
```
variable [NAME] {
  [OPTION] = "[VALUE]"
}
```

#### Arguments
Within the block body (between `{ }`) is configuration for the variable, which accepts the following arguments:
- `type` (Optional): If set, this defines the type of the variable. Valid values are `string`, `list`, and `map`.
- `default` (Optional): This sets a default value for the variable. If no default is provided, Terraform will raise an error if a value is not provided by the caller.
- `description` (Optional): A human-friendly description for the variable.
  
Using variables during an apply:
```
terraform apply -var 'foo=bar'
```
  
Set up the environment:
```
cd terraform/basics
```
  
Edit main.tf:
```
vi main.tf
```
  
`main.tf` contents:
```
#Define variables
variable "image_name" {
  description = "Image for container."
  default     = "ghost:latest"
}

variable "container_name" {
  description = "Name of the container."
  default     = "blog"
}

variable "int_port" {
  description = "Internal port for container."
  default     = "2368"
}

variable "ext_port" {
  description = "External port for container."
  default     = "80"
}

# Download the latest Ghost Image
resource "docker_image" "image_id" {
  name = "${var.image_name}"
}

# Start the Container
resource "docker_container" "container_id" {
  name  = "${var.container_name}"
  image = "${docker_image.image_id.latest}"
  ports {
    internal = "${var.int_port}"
    external = "${var.ext_port}"
  }
}

#Output the IP Address of the Container
output "ip_address" {
  value       = "${docker_container.container_id.ip_address}"
  description = "The IP for the container."
}

output "container_name" {
  value       = "${docker_container.container_id.name}"
  description = "The name of the container."
}
```
  
Validate the changes:
```
terraform validate
```
  
Plan the changes:
```
terraform plan
```
  
Apply the changes using a variable:
```
terraform apply -var 'ext_port=8080'
```
  
Change the container name:
```
terraform apply -var 'container_name=ghost_blog' -var 'ext_port=8080'
```
  
Reset the environment:
```
terraform destroy -var 'ext_port=8080'
```

### Breaking Out Our Variables and Outputs
Setup your environment:
```
cd terraform/basics
```

Edit `variables.tf`:
```
vi variables.tf
```

`variables.tf` contents:
```
#Define variables
variable "container_name" {
  description = "Name of the container."
  default     = "blog"
}
variable "image_name" {
  description = "Image for container."
  default     = "ghost:latest"
}
variable "int_port" {
  description = "Internal port for container."
  default     = "2368"
}
variable "ext_port" {
  description = "External port for container."
  default     = "80"
}
```

Edit main.tf:
```
vi main.tf
```

main.tf contents:
```
# Download the latest Ghost Image
resource "docker_image" "image_id" {
  name = "${var.image_name}"
}

# Start the Container
resource "docker_container" "container_id" {
  name  = "${var.container_name}"
  image = "${docker_image.image_id.latest}"
  ports {
    internal = "${var.int_port}"
    external = "${var.ext_port}"
  }
}
```

Edit `outputs.tf`:
```
vi outputs.tf
```

`outputs.tf` contents:
```
#Output the IP Address of the Container
output "ip_address" {
  value       = "${docker_container.container_id.ip_address}"
  description = "The IP for the container."
}

output "container_name" {
  value       = "${docker_container.container_id.name}"
  description = "The name of the container."
}
```

Validate the changes:
```
terraform validate
```

Plan the changes:
```
terraform plan -out=tfplan -var container_name=ghost_blog
```

Apply the changes:
```
terraform apply tfplan
```

Destroy deployment:
```
terraform destroy -auto-approve -var container_name=ghost_blog
```

### Maps and Lookups
In this lesson, we will create a map to specify different environment variables based on conditions. This will allow us to dynamically deploy infrastructure configurations based on information we pass to the deployment.
  
Set up the environment:
```
cd terraform/basics
```

Edit `variables.tf`:
```
vi variables.tf
```

variables.tf contents:
```
#Define variables
variable "env" {
  description = "env: dev or prod"
}
variable "image_name" {
  type        = "map"
  description = "Image for container."
  default     = {
    dev  = "ghost:latest"
    prod = "ghost:alpine"
  }
}

variable "container_name" {
  type        = "map"
  description = "Name of the container."
  default     = {
    dev  = "blog_dev"
    prod = "blog_prod"
  }
}

variable "int_port" {
  description = "Internal port for container."
  default     = "2368"
}
variable "ext_port" {
  type        = "map"
  description = "External port for container."
  default     = {
    dev  = "8081"
    prod = "80"
  }
}
```

Validate the change:
```
terraform validate
```

Edit `main.tf`:
```
vi main.tf
```

`main.tf` contents:
```
# Download the latest Ghost Image
resource "docker_image" "image_id" {
  name = "${lookup(var.image_name, var.env)}"
}

# Start the Container
resource "docker_container" "container_id" {
  name  = "${lookup(var.container_name, var.env)}"
  image = "${docker_image.image_id.latest}"
  ports {
    internal = "${var.int_port}"
    external = "${lookup(var.ext_port, var.env)}"
  }
}
```

Plan the `dev` deploy:
```
terraform plan -out=tfdev_plan -var env=dev
```

Apply the `dev` plan:
```
terraform apply tfdev_plan
```

Plan the `prod` deploy:
```
terraform plan -out=tfprod_plan -var env=prod
```

Apply the `prod` plan:
```
terraform apply tfprod_plan
```

Destroy `prod` deployment:
```
terraform destroy -var env=prod -auto-approve
```

Use environment variables:
```
export TF_VAR_env=prod
```

Open the Terraform console:
```
terraform console
```

Execute a lookup:
```
lookup(var.ext_port, var.env)
```

Exit the console:
```
unset TF_VAR_env
```

### Terraform Workspaces
In this lesson, we will see how workspaces can help us deploy multiple environments. By using workspaces, we can deploy multiple environments simultaneously without the state files colliding.
  
#### Creating a workspace
Terraform commands:
- `workspace`: New, list, select and delete Terraform workspaces
  
Workspace subcommands:
- `delete`: Delete a workspace 
- `list`: List Workspaces 
- `new`: Create a new workspace 
- `select`: Select a workspace 
- `show`: Show the name of the current workspace
  
Setup the environment:
```
cd terraform/basics
```

Create a `dev` workspace:
```
terraform workspace new dev
```

Plan the `dev` deployment:
```
terraform plan -out=tfdev_plan -var env=dev
```

Apply the `dev` deployment:
```
terraform apply tfdev_plan
```

Change workspaces:
```
terraform workspace new prod
```

Plan the `prod` deployment:
```
terraform plan -out=tfprod_plan -var env=prod
```

Apply the `prod` deployment:
```
terraform apply tfprod_plan
```

Select the default workspace:
```
terraform workspace select default
```

Find what workspace we are using:
```
terraform workspace show
```

Select the `dev` workspace:
```
terraform workspace select dev
```

Destroy the `dev` deployment:
```
terraform destroy -var env=dev
```

Select the `prod` workspace:
```
terraform workspace select prod
```

Destroy the `prod` deployment:
```
terraform destroy -var env=prod
```

### Null Resources and Local-exec
In this lesson, we will utilize a Null Resource in order to perform local commands on our machine without having to deploy extra resources.
  
Setup the environment:
```
cd terraform/basics
```
  
`main.tf` contents:
```
# Download the latest Ghost Image
resource "docker_image" "image_id" {
  name = "${lookup(var.image_name, var.env)}"
}

# Start the Container
resource "docker_container" "container_id" {
  name  = "${lookup(var.container_name, var.env)}"
  image = "${docker_image.image_id.latest}"
  ports {
    internal = "${var.int_port}"
    external = "${lookup(var.ext_port, var.env)}"
  }
}

resource "null_resource" "null_id" {
  provisioner "local-exec" {
    command = "echo ${docker_container.container_id.name}:${docker_container.container_id.ip_address} >> container.txt"
  }
}
```
  
Reinitialize Terraform:
```
terraform init
```

Validate the changes:
```
terraform validate
```

Plan the changes:
```
terraform plan -out=tfplan -var env=dev
```

Apply the changes:
```
terraform apply tfplan
```

View the contents of `container.txt`:
```
cat container.txt
```

Destroy the deployment:
```
terraform destroy -auto-approve -var env=dev
```

## Terraform Modules
### Introduction to Modules
![img](https://github.com/Bes0n/Using-Terraform-to-Manage-Applications-and-Infrastructure/blob/master/images/img1.png)
  
Set up the environment:
```
mkdir -p modules/image
mkdir -p modules/container
```

Create files for the image:
```
cd ~/terraform/basics/modules/image
touch main.tf variables.tf outputs.tf
```

Create files for container:
```
cd ~/terraform/basics/modules/container
touch main.tf variables.tf outputs.tf
```

### The Image Module
Go to the image directory:
```
cd ~/terraform/basics/modules/image
```

Edit `main.tf`:
```
vi main.tf
```

`main.tf` contents:
```
# Download the Image
resource "docker_image" "image_id" {
  name = "${var.image_name}"
}
```

Edit `variables.tf`:
```
vi variables.tf
```

`variables.tf` contents:
```
variable "image_name" {
  description = "Name of the image"
}
```

Edit `outputs.tf`:
```
vi outputs.tf
```

`outputs.tf`: contents:
```
output "image_out" {
  value       = "${docker_image.image_id.latest}"
}
```

Initialize Terraform:
```
terraform init
```

Create the image plan:
```
terraform plan -out=tfplan -var 'image_name=ghost:alpine'
```

Deploy the image using the plan:
```
terraform apply -auto-approve tfplan
```

Destroy the image:
```
terraform destroy -auto-approve -var 'image_name=ghost:alpine'
```

### The Container Module
Go to the container directory:
```
cd ~/terraform/basics/modules/container
```

Edit `main.tf`:
```
vi main.tf
```

`main.tf` contents:
```
# Start the Container
resource "docker_container" "container_id" {
  name  = "${var.container_name}"
  image = "${var.image}"
  ports {
    internal = "${var.int_port}"
    external = "${var.ext_port}"
  }
}
```

Edit `variables.tf`:
```
vi variables.tf
```

`variables.tf` contents:
```
#Define variables
variable "container_name" {}
variable "image" {}
variable "int_port" {}
variable "ext_port" {}
```

Edit `outputs.tf`:
```
vi outputs.tf
```

`outputs.tf` contents:
```
#Output the IP Address of the Container
output "ip" {
  value = "${docker_container.container_id.ip_address}"
}

output "container_name" {
  value = "${docker_container.container_id.name}"
}
```

Initialize:
```
terraform init
```

Create the image plan:
```
terraform plan -out=tfplan -var 'container_name=blog' -var 'image=ghost:alpine' -var 'int_port=2368' -var 'ext_port=80'
```

Deploy container using the plan:
```
terraform apply tfplan
```

### The Root Module
In this lesson we will refactor the `root` module to use the image and container modules we created in the previous two lessons.

Go to the module directory:
```
cd ~/terraform/basics/modules/
```
  
```
touch {main.tf,variables.tf,outputs.tf}
```

Edit `main.tf`:
```
vi main.tf
```

`main.tf` contents:
```
# Download the image
module "image" {
  source = "./image"
  image_name  = "${var.image_name}"
}

# Start the container
module "container" {
  source             = "./container"
  image              = "${module.image.image_out}"
  container_name     = "${var.container_name}"
  int_port           = "${var.int_port}"
  ext_port           = "${var.ext_port}"
}
```

Edit `variables.tf`:
```
vi variables.tf
```

`variables.tf` contents:
```
#Define variables
variable "container_name" {
  description = "Name of the container."
  default     = "blog"
}
variable "image_name" {
  description = "Image for container."
  default     = "ghost:latest"
}
variable "int_port" {
  description = "Internal port for container."
  default     = "2368"
}
variable "ext_port" {
  description = "External port for container."
  default     = "80"
}
```

Edit `outputs.tf`:
```
vi outputs.tf
```

`outputs.tf` contents:
```
#Output the IP Address of the Container
output "ip" {
  value = "${module.container.ip}"
}

output "container_name" {
  value = "${module.container.container_name}"
}
```

Initialize Terraform:
```
terraform init
```

Create the image plan:
```
terraform plan -out=tfplan
```

Deploy the container using the plan:
```
terraform apply tfplan
```

Destroy the deployment:
```
terraform destroy -auto-approve
```
  

## Terraform and Docker
### Managing Docker Networks
In this lesson we will build on our knowledge of Terraform and Docker by learning about the `docker_network` resource.
  
Set up the environment:
```
mkdir -p ~/terraform/docker/networks
cd terraform/docker/networks
```

Create the files:
```
touch {variables.tf,image.tf,network.tf,main.tf}
```

Edit variables.tf:
```
vi variables.tf
```

`variables.tf` contents:
```
variable "mysql_root_password" {
  description = "The MySQL root password."
  default     = "P4sSw0rd0!"
}

variable "ghost_db_username" {
  description = "Ghost blog database username."
  default     = "root"
}

variable "ghost_db_name" {
  description = "Ghost blog database name."
  default     = "ghost"
}

variable "mysql_network_alias" {
  description = "The network alias for MySQL."
  default     = "db"
}

variable "ghost_network_alias" {
  description = "The network alias for Ghost"
  default     = "ghost"
}

variable "ext_port" {
  description = "Public port for Ghost"
  default     = "8080"
}
```

Edit `image.tf`:
```
vi image.tf
```

`image.tf` contents:
```
resource "docker_image" "ghost_image" {
  name = "ghost:alpine"
}

resource "docker_image" "mysql_image" {
  name = "mysql:5.7"
}
```

Edit `network.tf`:
```
vi network.tf
```

`network.tf` contents:
```
resource "docker_network" "public_bridge_network" {
  name   = "public_ghost_network"
  driver = "bridge"
}

resource "docker_network" "private_bridge_network" {
  name     = "ghost_mysql_internal"
  driver   = "bridge"
  internal = true
}
```

Edit `main.tf`:
```
vi main.tf
```

`main.tf` contents:
```
resource "docker_container" "blog_container" {
  name  = "ghost_blog"
  image = "${docker_image.ghost_image.name}"
  env   = [
    "database__client=mysql",
    "database__connection__host=${var.mysql_network_alias}",
    "database__connection__user=${var.ghost_db_username}",
    "database__connection__password=${var.mysql_root_password}",
    "database__connection__database=${var.ghost_db_name}"
  ]
  ports {
    internal = "2368"
    external = "${var.ext_port}"
  }
  networks_advanced {
    name    = "${docker_network.public_bridge_network.name}"
    aliases = ["${var.ghost_network_alias}"]
  }
  networks_advanced {
    name    = "${docker_network.private_bridge_network.name}"
    aliases = ["${var.ghost_network_alias}"]
  }
}

resource "docker_container" "mysql_container" {
  name  = "ghost_database"
  image = "${docker_image.mysql_image.name}"
  env   = [
    "MYSQL_ROOT_PASSWORD=${var.mysql_root_password}"
  ]
  networks_advanced {
    name    = "${docker_network.private_bridge_network.name}"
    aliases = ["${var.mysql_network_alias}"]
  }
}
```

Initialize Terraform:
```
terraform init
```

Validate the files:
```
terraform validate
```

Build a plan:
```
terraform plan -out=tfplan -var 'ext_port=8082'
```

Apply the plan:
```
terraform apply tfplan
```

Destroy the environment:
```
terraform destroy -auto-approve -var 'ext_port=8082'
```

#### Fixing `main.tf`
`main.tf` contents:
```
resource "docker_container" "mysql_container" {
  name  = "ghost_database"
  image = "${docker_image.mysql_image.name}"
  env   = [
    "MYSQL_ROOT_PASSWORD=${var.mysql_root_password}"
  ]
  networks_advanced {
    name    = "${docker_network.private_bridge_network.name}"
    aliases = ["${var.mysql_network_alias}"]
  }
}

resource "null_resource" "sleep" {
  depends_on = ["docker_container.mysql_container"]
  provisioner "local-exec" {
    command = "sleep 15s"
  }
}

resource "docker_container" "blog_container" {
  name  = "ghost_blog"
  image = "${docker_image.ghost_image.name}"
  depends_on = ["null_resource.sleep", "docker_container.mysql_container"]
  env   = [
    "database__client=mysql",
    "database__connection__host=${var.mysql_network_alias}",
    "database__connection__user=${var.ghost_db_username}",
    "database__connection__password=${var.mysql_root_password}",
    "database__connection__database=${var.ghost_db_name}"
  ]
  ports {
    internal = "2368"
    external = "${var.ext_port}"
  }
  networks_advanced {
    name    = "${docker_network.public_bridge_network.name}"
    aliases = ["${var.ghost_network_alias}"]
  }
  networks_advanced {
    name    = "${docker_network.private_bridge_network.name}"
    aliases = ["${var.ghost_network_alias}"]
  }
}
```

Build a plan:
```
terraform plan -out=tfplan -var 'ext_port=8082'
```

Apply the plan:
```
terraform apply tfplan
```

### Managing Docker Volumes
In this lesson, we will add a volume to our Ghost Blog/MySQL setup.
  
Destroy the existing environment:
```
terraform destroy -auto-approve -var 'ext_port=8082'
```

Setup an environment:
```
cp -r ~/terraform/docker/networks ~/terraform/docker/volumes
cd ../volumes/
```

Create `volumes.tf`:
```
vi volumes.tf
```

`volumes.tf` contents:
```
resource "docker_volume" "mysql_data_volume" {
  name = "mysql_data"
}
```

Edit `main.tf`:
```
vi main.tf
```

`main.tf` contents:
```
resource "docker_container" "mysql_container" {
  name  = "ghost_database"
  image = "${docker_image.mysql_image.name}"
  env   = [
    "MYSQL_ROOT_PASSWORD=${var.mysql_root_password}"
  ]
  volumes {
    volume_name    = "${docker_volume.mysql_data_volume.name}"
    container_path = "/var/lib/mysql"
  }
  networks_advanced {
    name    = "${docker_network.private_bridge_network.name}"
    aliases = ["${var.mysql_network_alias}"]
  }
}

resource "null_resource" "sleep" {
  depends_on = ["docker_container.mysql_container"]
  provisioner "local-exec" {
    command = "sleep 15s"
  }
}

resource "docker_container" "blog_container" {
  name  = "ghost_blog"
  image = "${docker_image.ghost_image.name}"
  depends_on = ["null_resource.sleep", "docker_container.mysql_container"]
  env   = [
    "database__client=mysql",
    "database__connection__host=${var.mysql_network_alias}",
    "database__connection__user=${var.ghost_db_username}",
    "database__connection__password=${var.mysql_root_password}",
    "database__connection__database=${var.ghost_db_name}"
  ]
  ports {
    internal = "2368"
    external = "${var.ext_port}"
  }
  networks_advanced {
    name    = "${docker_network.public_bridge_network.name}"
    aliases = ["${var.ghost_network_alias}"]
  }
  networks_advanced {
    name    = "${docker_network.private_bridge_network.name}"
    aliases = ["${var.ghost_network_alias}"]
  }
}
```

Initialize Terraform:
```
terraform init
```

Validate the files:
```
terraform validate
```

Build a plan:
```
terraform plan -out=tfplan -var 'ext_port=8082'
```

Apply the plan:
```
terraform apply tfplan
```

List Docker volumes:
```
docker volume inspect mysql_data
```

List the data in mysql_data:
```
sudo ls /var/lib/docker/volumes/mysql_data/_data
```

Destroy the environment:
```
terraform destroy -auto-approve -var 'ext_port=8082'
```

### Creating Swarm Services
In this lesson, we will convert our Ghost and MySQL containers over to using Swarm services. Swarm services are a more production-ready way of running containers.
  
Setup the environment:
```
cp -r volumes/ services
cd services
```

`variables.tf` contents:
```
variable "mysql_root_password" {
  description = "The MySQL root password."
  default     = "P4sSw0rd0!"
}

variable "ghost_db_username" {
  description = "Ghost blog database username."
  default     = "root"
}

variable "ghost_db_name" {
  description = "Ghost blog database name."
  default     = "ghost"
}

variable "mysql_network_alias" {
  description = "The network alias for MySQL."
  default     = "db"
}

variable "ghost_network_alias" {
  description = "The network alias for Ghost"
  default     = "ghost"
}

variable "ext_port" {
  description = "The public port for Ghost"
}
```

`images.tf` contents:
```
resource "docker_image" "ghost_image" {
  name = "ghost:alpine"
}

resource "docker_image" "mysql_image" {
  name = "mysql:5.7"
}
```

`network.tf` contents:
```
resource "docker_network" "public_bridge_network" {
  name   = "public_network"
  driver = "overlay"
}

resource "docker_network" "private_bridge_network" {
  name     = "mysql_internal"
  driver   = "overlay"
  internal = true
}
```

`volumes.tf` contents:
```
resource "docker_volume" "mysql_data_volume" {
  name = "mysql_data"
}
```

`main.tf` contents:
```
resource "docker_service" "ghost-service" {
  name = "ghost"

  task_spec {
    container_spec {
      image = "${docker_image.ghost_image.name}"

      env {
         database__client               = "mysql"
         database__connection__host     = "${var.mysql_network_alias}"
         database__connection__user     = "${var.ghost_db_username}"
         database__connection__password = "${var.mysql_root_password}"
         database__connection__database = "${var.ghost_db_name}"
      }
    }
    networks = [
      "${docker_network.public_bridge_network.name}",
      "${docker_network.private_bridge_network.name}"
    ]
  }

  endpoint_spec {
    ports {
      target_port    = "2368"
      published_port = "${var.ext_port}"
    }
  }
}

resource "docker_service" "mysql-service" {
  name = "${var.mysql_network_alias}"

  task_spec {
    container_spec {
      image = "${docker_image.mysql_image.name}"

      env {
        MYSQL_ROOT_PASSWORD = "${var.mysql_root_password}"
      }

      mounts = [
        {
          target = "/var/lib/mysql"
          source = "${docker_volume.mysql_data_volume.name}"
          type   = "volume"
        }
      ]
    }
    networks = ["${docker_network.private_bridge_network.name}"]
  }
}
```

Initialize Terraform:
```
terraform init
```

Validate the files:
```
terraform validate
```

Build a plan:
```
terraform plan -out=tfplan -var 'ext_port=8082'
```

Apply the plan:
```
terraform apply tfplan
```
```
docker service ls
```

```
docker container ls
```

Destroy the environment:
```
terraform destroy -auto-approve -var 'ext_port=8082'
```

### Using Secrets
In this lesson, we'll explore using Terraform to store sensitive data, by using Docker Secrets.
  
Setup the environment:
```
mkdir secrets
cd secrets
```

Encode the password with Base64:
```
echo 'p4sSWoRd0!' | base64
```

Create `variables.tf`:
```
vi variables.tf
```

`variables.tf` contents:
```
variable "mysql_root_password" {
  default     = "cDRzU1dvUmQwIQo="
}

variable "mysql_db_password" {
  default     = "cDRzU1dvUmQwIQo="
}
```

Create `image.tf`:
```
vi image.tf
```

`image.tf` contents:
```
resource "docker_image" "mysql_image" {
  name = "mysql:5.7"
}
```

Create `secrets.tf`:
```
vi secrets.tf
```

`secrets.tf` contents:
```
resource "docker_secret" "mysql_root_password" {
  name = "root_password"
  data = "${var.mysql_root_password}"
}

resource "docker_secret" "mysql_db_password" {
  name = "db_password"
  data = "${var.mysql_db_password}"
}
```

Create `networks.tf`:
```
vi networks.tf
```

`networks.tf` contents:
```
resource "docker_network" "private_overlay_network" {
  name     = "mysql_internal"
  driver   = "overlay"
  internal = true
}
```

Create `volumes.tf`:
```
vi volumes.tf
```

`volumes.tf` contents:
```
resource "docker_volume" "mysql_data_volume" {
  name = "mysql_data"
}
```

Create `main.tf`:
```
vi main.tf
```

`main.tf` contents:
```
resource "docker_service" "mysql-service" {
  name = "mysql_db"

  task_spec {
    container_spec {
      image = "${docker_image.mysql_image.name}"

      secrets = [
        {
          secret_id   = "${docker_secret.mysql_root_password.id}"
          secret_name = "${docker_secret.mysql_root_password.name}"
          file_name   = "/run/secrets/${docker_secret.mysql_root_password.name}"
        },
        {
          secret_id   = "${docker_secret.mysql_db_password.id}"
          secret_name = "${docker_secret.mysql_db_password.name}"
          file_name   = "/run/secrets/${docker_secret.mysql_db_password.name}"
        }
      ]

      env {
        MYSQL_ROOT_PASSWORD_FILE = "/run/secrets/${docker_secret.mysql_root_password.name}"
        MYSQL_DATABASE           = "mydb"
        MYSQL_PASSWORD_FILE      = "/run/secrets/${docker_secret.mysql_db_password.name}"
      }

      mounts = [
        {
          target = "/var/lib/mysql"
          source = "${docker_volume.mysql_data_volume.name}"
          type   = "volume"
        }
      ]
    }
    networks = [
      "${docker_network.private_overlay_network.name}"
    ]
  }
}
```

Initialize Terraform:
```
terraform init
```

Validate the files:
```
terraform validate
```

Build a plan:
```
terraform plan -out=tfplan
```

Apply the plan:
```
terraform apply tfplan
```

Find the MySQL container:
```
docker container ls
```

Use the exec command to log into the MySQL container:
```
docker container exec -it [CONTAINER_ID] /bin/bash
```

Access MySQL:
```
mysql -u root -p
```

Destroy the environment:
```
terraform destroy -auto-approve
```
  
## Using Terraform in a CI/CD Environment
### Building a Custom Jenkins Image
In this lesson, we will learn how to build a Jenkins Docker image that has Docker and Terraform baked in. We will be using this image throughout the remainder of this section.
  
Setup the environment:
```
mkdir -p jenkins
```

Create `Dockerfile`:
```
vi Dockerfile
```

`Dockerfile` contents:
```
FROM jenkins/jenkins:lts
USER root
RUN apt-get update -y && apt-get -y install apt-transport-https ca-certificates curl gnupg-agent software-properties-common
RUN curl -fsSL https://download.docker.com/linux/$(. /etc/os-release; echo "$ID")/gpg > /tmp/dkey; apt-key add /tmp/dkey
RUN add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/$(. /etc/os-release; echo "$ID") $(lsb_release -cs) stable"
RUN apt-get update -y
RUN apt-get install -y docker-ce docker-ce-cli containerd.io
RUN curl -O https://releases.hashicorp.com/terraform/0.11.13/terraform_0.11.13_linux_amd64.zip && unzip terraform_0.11.13_linux_amd64.zip -d /usr/local/bin/
USER ${user}
```

Build the Image:
```
docker build -t jenkins:terraform .
```

List the Docker images:
```
docker image ls
```

### Setting Up Jenkins
In this lesson, we will take the Jenkins image we built in the previous lesson, and deploy a Docker container using Terraform.
  
Edit `main.tf`:
```
vi main.tf
```

`main.tf` contents:
```
# Jenkins Volume
resource "docker_volume" "jenkins_volume" {
  name = "jenkins_data"
}

# Start the Jenkins Container
resource "docker_container" "jenkins_container" {
  name  = "jenkins"
  image = "jenkins:terraform"
  ports {
    internal = "8080"
    external = "8080"
  }

  volumes {
    volume_name    = "${docker_volume.jenkins_volume.name}"
    container_path = "/var/jenkins_home"
  }

  volumes {
    host_path      = "/var/run/docker.sock"
    container_path = "/var/run/docker.sock"
  }
}
```

Initialize Terraform:
```
terraform init
```

Plan the deployment:
```
terraform plan -out=tfplan
```

Deploy Jenkins:
```
terraform apply tfplan
```

Get the Admin password:
```
docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
```

### Creating a Jenkins Job
In this lesson, we will start working with Jenkins by creating a simple build job. This job will deploy a Docker container using Terraform, list the container, and then destroy it.
  
In the Jenkins dashboard, Click **New Item**.
  
Select **Freestyle Project**, and enter an item name of **DeployGhost**. Click **Ok**.
  
Under Source Code Management, select **Git**. Enter a Repository URL of https://github.com/linuxacademy/content-terraform-docker.git
  
In the Build section, click **Add build step** and select **Execute shell** from the dropdown.
  
Add the following in the Command area:
```
terraform init
terraform plan -out=tfplan
terraform apply tfplan
docker container ls
terraform destroy -auto-approve
```

Click **Save**.
  
Now, if we click Build Now in the left-hand menu, our project will start building. Clicking the little dropdown arrow next to #1 will give us a menu. Select **Console Output** to watch things build. Once we get a `Finished: SUCCESS` message, we're done.

### Building a Jenkins Pipeline Part 1
In this lesson, we will create the first Jenkins Pipeline that will deploy out a Ghost blog.
  
In the Jenkins dashboard, click **New Item** Enter an item name of **PipelinePart1**, and select Pipeline. Click **Ok**.
  
Check the box for *This project is parameterized*. Click **Add Parameter** and select *Choice Parameter*. Give it a Name of **action**. For Choices, enter **Deploy** and **Destroy**, and make sure they are on separate lines. Enter **The action that will be executed** as the *Description*.
  
Click **Add Parameter** and select **Choice Parameter** again. This time, name it **image_name**. Enter **ghost:latest** and **ghost:alpine** in the *Choices* box, making sure they are on separate lines. Enter **The image Ghost Blog will deploy** as a *Description*.
  
Click **Add Parameter** a third time, and select **String Parameter**. Give it a Name of **ext_port**. Set the *Default Value* to **80**. Enter **The Public Port** as the *Description*.
  
Down in the *Pipeline* section, give a *Definition* of **Pipeline script**, and add the following to the *Script*:
```
node {
  git 'https://github.com/linuxacademy/content-terraform-docker.git'
  if(action == 'Deploy') {
    stage('init') {
        sh """
            terraform init
        """
    }
    stage('plan') {
      sh label: 'terraform plan', script: "terraform plan -out=tfplan -input=false -var image_name=${image_name} -var ext_port=${ext_port}"
      script {
          timeout(time: 10, unit: 'MINUTES') {
              input(id: "Deploy Gate", message: "Deploy environment?", ok: 'Deploy')
          }
      }
    }
    stage('apply') {
        sh label: 'terraform apply', script: "terraform apply -lock=false -input=false tfplan"
    }
  }

  if(action == 'Destroy') {
    stage('plan_destroy') {
      sh label: 'terraform plan destroy', script: "terraform plan -destroy -out=tfdestroyplan -input=false -var image_name=${image_name} -var ext_port=${ext_port}"
    }
    stage('destroy') {
      script {
          timeout(time: 10, unit: 'MINUTES') {
              input(id: "Destroy Gate", message: "Destroy environment?", ok: 'Destroy')
          }
      }
      sh label: 'Destroy environment', script: "terraform apply -lock=false -input=false tfdestroyplan"
    }
  }
}
```

### Building a Jenkins Pipeline Part 2
In this lesson, we will create a Jenkins Pipeline to deploy out a Swarm service.
  
In the Jenkins dashboard, click **New Item** Enter an item name of **PipelinePart2**, and select Pipeline. Click **Ok**.
  
Check the box for *This project is parameterized*. Click **Add Parameter** and select *Choice Parameter*. Give it a Name of **action**. For Choices, enter **Deploy** and **Destroy**, and make sure they are on separate lines. Enter **The action that will be executed** as the *Description*.
  
Click **Add Parameter** and select **Choice Parameter** again. This time, name it **image_name**. Enter **ghost:latest** and **ghost:alpine** in the *Choices* box, making sure they are on separate lines. Enter **The image Ghost Blog will deploy** as a *Description*.
  
Click **Add Parameter** a third time, and select **String Parameter**. Give it a Name of **ext_port**. Set the *Default Value* to **80**. Enter **The Public Port** as the *Description*.
  
Down in the *Pipeline* section, give a *Definition* of **Pipeline script**, and add the following to the *Script*:
```
node {
  git 'https://github.com/linuxacademy/content-terraform-docker-service.git'
  if(action == 'Deploy') {
    stage('init') {
      sh label: 'terraform init', script: "terraform init"
    }
    stage('plan') {
      sh label: 'terraform plan', script: "terraform plan -out=tfplan -input=false -var image_name=${image_name} -var ghost_ext_port=${ghost_ext_port}"
      script {
          timeout(time: 10, unit: 'MINUTES') {
            input(id: "Deploy Gate", message: "Deploy environment?", ok: 'Deploy')
          }
      }
    }
    stage('apply') {
      sh label: 'terraform apply', script: "terraform apply -lock=false -input=false tfplan"
    }
  }

  if(action == 'Destroy') {
    stage('plan_destroy') {
      sh label: 'terraform plan', script: "terraform plan -destroy -out=tfdestroyplan -input=false -var image_name=${image_name} -var ghost_ext_port=${ghost_ext_port}"
    }
    stage('destroy') {
      script {
          timeout(time: 10, unit: 'MINUTES') {
              input(id: "Destroy Gate", message: "Destroy environment?", ok: 'Destroy')
          }
      }
      sh label: 'terraform apply', script: "terraform apply -lock=false -input=false tfdestroyplan"
    }
    stage('cleanup') {
      sh label: 'cleanup', script: "rm -rf terraform.tfstat"
    }
  }
}
```

### Building a Jenkins Pipeline Part 3
In this lesson, we will complete working with Jenkins by creating a pipeline that will create a MySQL Swarm service that uses Docker Secrets.
  
In the Jenkins dashboard, click **New Item** Enter an item name of **PipelinePart2**, and select Pipeline. Click **Ok**.
  
Check the box for *This project is parameterized*. Click **Add Parameter** and select *Choice Parameter*. Give it a Name of **action**. For Choices, enter **Deploy** and **Destroy**, and make sure they are on separate lines. Enter **The action that will be executed** as the *Description*.
  
Click **Add Parameter** and select **String Parameter**. For the name, enter **mysql_root_password**.. Enter **P4ssW0rd0!** in the *Default Value* box. Enter **MySQL root password**. as a *Description*.
  
For the next parameter, click **Add Parameter** once more and select **String Parameter**. For the name, enter **mysql_user_password**.. Enter **paSsw0rd0!** in the *Default Value* box. Enter **MySQL user password**. as a *Description*.

Down in the *Pipeline* section, give a *Definition* of **Pipeline script**, and add the following to the *Script*:
```
node {
  git 'https://github.com/linuxacademy/content-terraform-docker-secrets.git'
  if(action == 'Deploy') {
    stage('init') {
      sh label: 'terraform init', script: "terraform init"
    }
    stage('plan') {
      def ROOT_PASSWORD = sh (returnStdout: true, script: """echo ${mysql_root_password} | base64""").trim()
      def USER_PASSWORD = sh (returnStdout: true, script: """echo ${mysql_user_password} | base64""").trim()
      sh label: 'terraform plan', script: "terraform plan -out=tfplan -input=false -var mysql_root_password=${ROOT_PASSWORD} -var mysql_db_password=${USER_PASSWORD}"
      script {
          timeout(time: 10, unit: 'MINUTES') {
              input(id: "Deploy Gate", message: "Deploy ${params.project_name}?", ok: 'Deploy')
          }
      }
    }
    stage('apply') {
      sh label: 'terraform apply', script: "terraform apply -lock=false -input=false tfplan"
    }
  }

  if(action == 'Destroy') {
    stage('plan_destroy') {
      def ROOT_PASSWORD = sh (returnStdout: true, script: """echo ${mysql_root_password} | base64""").trim()
      def USER_PASSWORD = sh (returnStdout: true, script: """echo ${mysql_user_password} | base64""").trim()
      sh label: 'terraform plan', script: "terraform plan -destroy -out=tfdestroyplan -input=false -var mysql_root_password=${ROOT_PASSWORD} -var mysql_db_password=${USER_PASSWORD}"
    }
    stage('destroy') {
      script {
          timeout(time: 10, unit: 'MINUTES') {
              input(id: "Destroy Gate", message: "Destroy ${params.project_name}?", ok: 'Destroy')
          }
      }
      sh label: 'terraform apply', script: "terraform apply -lock=false -input=false tfdestroyplan"
    }
    stage('cleanup') {
      sh label: 'cleanup', script: "rm -rf terraform.tfstat"
    }
  }
}
```

## Terraform and AWS
### Setting Up a Cloud Sandbox
- We're going to create AWS sandbox from Linux academy and access it. 
- Be adviced that sandbox will be destroyed after some time 
- To allow accessing our AWS resources first of all we need to export our keys
  - export AWS_ACCESS_KEY_ID="AKIAW6UUHXHLLNJYDW5R"
  - export AWS_SECRET_ACCESS_KEY="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"


### Our Architecture: What We're Going to Build
We will have several modules for each component of our architecture
- Root:
  - Storage Module
  - Network Module
  - Compute Module

![img](https://github.com/Bes0n/Using-Terraform-to-Manage-Applications-and-Infrastructure/blob/master/images/img2.png)
  
High-level diagram will be following:
- Storage module is going to use S3 Bucket 
- Network module will deploy internet gateway with public and private route tables
- Two EC2 instances will be deployed and ssh keys generated for them. 
  
![img](https://github.com/Bes0n/Using-Terraform-to-Manage-Applications-and-Infrastructure/blob/master/images/img3.png)

### Storage Part 1: The S3 Bucket and Random ID
In this lesson, we will start working with AWS by creating a S3 Terraform module.
  
Environment setup:
```
mkdir -p ~/terraform/AWS/storage
cd ~/terraform/AWS/storage
```

Create main.tf:
```
vi main.tf
```

main.tf:
```
#---------storage/main.tf---------

# Create a random id
resource "random_id" "tf_bucket_id" {
  byte_length = 2
}

# Create the bucket
resource "aws_s3_bucket" "tf_code" {
    bucket        = "${var.project_name}-${random_id.tf_bucket_id.dec}"
    acl           = "private"

    force_destroy =  true

    tags {
      Name = "tf_bucket"
    }
}
```

Create variables.tf:
```
vi variables.tf
```

variables.tf:
```
#----storage/variables.tf----
variable "project_name" {}
```

outputs.tf:
```
vi outputs.tf
```

outputs.tf:
```
#----storage/outputs.tf----
output "bucketname" {
  value = "${aws_s3_bucket.tf_code.id}"
}
```

Initialize Terraform:
```
terraform init
```

Validate your files:
```
terraform validate
```

Plan the deployment:
```
export AWS_ACCESS_KEY_ID="[ACCESS_KEY]"
export AWS_SECRET_ACCESS_KEY="[SECRET_KEY]"
export AWS_DEFAULT_REGION="us-east-1"
terraform plan -out=tfplan -var project_name=la-terraform
```

Deploy the S3 bucket:
```
terraform apply tfplan
```

Destroy S3 bucket:
```
terraform destroy -auto-approve -var project_name=la-terraform
```

### Storage Part 2: The Root Module
In this lesson, we will start working on our root module. We'll start off by adding the storage module created in the previous lesson.
  
Environment setup:
```
cd ~/terraform/AWS
touch {main.tf,variables.tf,outputs.tf,terraform.tfvars}
```

Edit main.tf:
```
vi main.tf
```

main.tf:
```
#----root/main.tf-----
provider "aws" {
  region = "${var.aws_region}"
}

# Deploy Storage Resources
module "storage" {
  source       = "./storage"
  project_name = "${var.project_name}"
}
```

Edit variables.tf:
```
vi variables.tf
```

variables.tf:
```
#----root/variables.tf-----
variable "aws_region" {}

#------ storage variables
variable "project_name" {}
```

Edit terraform.tfvars:
```
vi terraform.tfvars
```

terraform.tfvars:
```
aws_region   = "us-east-1"
project_name = "la-terraform"
```

Edit outputs.tf:
```
vi outputs.tf
```

outputs.tf:
```
#----root/outputs.tf-----

#----storage outputs------
output "Bucket Name" {
  value = "${module.storage.bucketname}"
}
```

Initialize terraform:
```
export AWS_ACCESS_KEY_ID="[ACCESS_KEY]"
export AWS_SECRET_ACCESS_KEY="[SECRET_KEY]]"
terraform init
```

Validate code:
```
terraform validate
```

Deploy the S3 bucket:
```
terraform apply -auto-approve
```

Destroy S3 bucket:
```
terraform destroy -auto-approve
```

### Networking Part 1: VPC, SG, Subnets
In this lesson, we will start our networking resource deployment and we will deploy our Internet Gateway and route tables.
  
Environment setup:
```
mkdir -p  ~/terraform/AWS/networking
cd ~/terraform/AWS/networking
```

Touch the files:
```
touch {main.tf,variables.tf,outputs.tf,terraform.tfvars}
```

Edit main.tf:
```
vi main.tf
```

main.tf:
```
#----networking/main.tf----

data "aws_availability_zones" "available" {}

resource "aws_vpc" "tf_vpc" {
  cidr_block           = "${var.vpc_cidr}"
  enable_dns_hostnames = true
  enable_dns_support   = true

  tags {
    Name = "tf_vpc"
  }
}

resource "aws_internet_gateway" "tf_internet_gateway" {
  vpc_id = "${aws_vpc.tf_vpc.id}"

  tags {
    Name = "tf_igw"
  }
}

resource "aws_route_table" "tf_public_rt" {
  vpc_id = "${aws_vpc.tf_vpc.id}"

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = "${aws_internet_gateway.tf_internet_gateway.id}"
  }

  tags {
    Name = "tf_public"
  }
}

resource "aws_default_route_table" "tf_private_rt" {
  default_route_table_id  = "${aws_vpc.tf_vpc.default_route_table_id}"

  tags {
    Name = "tf_private"
  }
}

resource "aws_subnet" "tf_public_subnet" {
  count                   = 2
  vpc_id                  = "${aws_vpc.tf_vpc.id}"
  cidr_block              = "${var.public_cidrs[count.index]}"
  map_public_ip_on_launch = true
  availability_zone       = "${data.aws_availability_zones.available.names[count.index]}"

  tags {
    Name = "tf_public_${count.index + 1}"
  }
}

resource "aws_route_table_association" "tf_public_assoc" {
  count          = "${aws_subnet.tf_public_subnet.count}"
  subnet_id      = "${aws_subnet.tf_public_subnet.*.id[count.index]}"
  route_table_id = "${aws_route_table.tf_public_rt.id}"
}

resource "aws_security_group" "tf_public_sg" {
  name        = "tf_public_sg"
  description = "Used for access to the public instances"
  vpc_id      = "${aws_vpc.tf_vpc.id}"

  #SSH

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["${var.accessip}"]
  }

  #HTTP

  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["${var.accessip}"]
  }
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
}
```

Edit variables.tf:
```
vi variables.tf
```

variables.tf:
```
#----networking/variables.tf----
variable "vpc_cidr" {}

variable "public_cidrs" {
  type = "list"
}

variable "accessip" {}
```

Edit outputs.tf:
```
vi outputs.tf
```

outputs.tf:
```
#-----networking/outputs.tf----

output "public_subnets" {
  value = "${aws_subnet.tf_public_subnet.*.id}"
}

output "public_sg" {
  value = "${aws_security_group.tf_public_sg.id}"
}

output "subnet_ips" {
  value = "${aws_subnet.tf_public_subnet.*.cidr_block}"
}
```

terraform.tfvars:
```
vpc_cidr     = "10.123.0.0/16"
public_cidrs = [
  "10.123.1.0/24",
  "10.123.2.0/24"
]
accessip    = "0.0.0.0/0"
```

Initialize Terraform:
```
export AWS_ACCESS_KEY_ID="[ACCESS_KEY]"
export AWS_SECRET_ACCESS_KEY="[SECRET_KEY]]"
terraform init
```

Validate code:
```
terraform validate
```

Deploy Network:
```
terraform apply -auto-approve
```

Destroy Network:
```
terraform destroy -auto-approve
```

Delete terraform.tfvars:
```
rm terraform.tfvars
```

### Networking Part 2: The Root Module
In this lesson, we will add the networking module to the root module.
  
Environment setup:
```
cd ~/terraform/AWS
```

Edit main.tf:
```
vi main.tf
```

main.tf:
```
provider "aws" {
  region = "${var.aws_region}"
}

# Deploy Storage Resources
module "storage" {
  source       = "./storage"
  project_name = "${var.project_name}"
}

# Deploy Networking Resources
module "networking" {
  source       = "./networking"
  vpc_cidr     = "${var.vpc_cidr}"
  public_cidrs = "${var.public_cidrs}"
  accessip     = "${var.accessip}"
}
```

Edit variables.tf:
```
vi variables.tf
```

variables.tf:
```
#----root/variables.tf-----
variable "aws_region" {}

#------ storage variables
variable "project_name" {}

#-------networking variables
variable "vpc_cidr" {}
variable "public_cidrs" {
  type = "list"
}
variable "accessip" {}
```

Edit terraform.tfvars:
```
vi terraform.tfvars
```

terraform.tfvars:
```
aws_region   = "us-east-1"
project_name = "la-terraform"
vpc_cidr     = "10.123.0.0/16"
public_cidrs = [
  "10.123.1.0/24",
  "10.123.2.0/24"
]
accessip    = "0.0.0.0/0"
```

Reinitialize Terraform:
```
export AWS_ACCESS_KEY_ID="[ACCESS_KEY]"
export AWS_SECRET_ACCESS_KEY="[SECRET_KEY]]"
terraform init
```

Validate code:
```
terraform validate
```

Apply Changes:
```
terraform apply -auto-approve
```

Destroy environment:
```
terraform destroy -auto-approve
```

### Compute Part 1: AMI Data, Key Pair, and the File Function
In this lesson, we will start working on building out the resources for out AWS compute.
  
Environment setup:
```
mkdir -p  ~/terraform/AWS/compute
cd ~/terraform/AWS/compute
```

Touch the files:
```
touch {main.tf,variables.tf,outputs.tf}
```

Create a SSH key.
```
ssh-keygen
```

Edit main.tf:
```
vi main.tf
```

main.tf:
```
#----compute/main.tf#----
data "aws_ami" "server_ami" {
  most_recent = true

  owners = ["amazon"]

  filter {
    name   = "name"
    values = ["amzn-ami-hvm*-x86_64-gp2"]
  }
}

resource "aws_key_pair" "tf_auth" {
  key_name   = "${var.key_name}"
  public_key = "${file(var.public_key_path)}"
}
```

Edit variables.tf:
```
vi variables.tf
```

variables.tf:
```
#----compute/variables.tf----
variable "key_name" {}

variable "public_key_path" {}
```

Initialize Terraform:
```
export AWS_ACCESS_KEY_ID="[ACCESS_KEY]"
export AWS_SECRET_ACCESS_KEY="[SECRET_KEY]]"
terraform init
```

Validate changes:
```
terraform validate
```

Plan the changes:
```
terraform plan -out=tfplan -var 'key_name=tfkey' -var 'public_key_path=/home/cloud_user/.ssh/id_rsa.pub'
```

Apply the changes:
```
terraform apply -auto-approve
```
  
Provide the values for key_name and public_key_path: key_name: tfkey public_key_path: /home/cloud_user/.ssh/id_rsa.pub
  
Destroy environment:
```
terraform destroy -auto-approve
```
  
Provide the values for key_name and public_key_path: key_name: tfkey public_key_path: /home/cloud_user/.ssh/id_rsa.pub

### Compute Part 2: The EC2 Instance
In this lesson, we will finish off the Compute module by adding the aws_instance resource.
  
Edit main.tf:
```
vi main.tf
```

main.tf:
```
#-----compute/main.tf#-----

data "aws_ami" "server_ami" {
  most_recent = true

  owners = ["amazon"]

  filter {
    name   = "name"
    values = ["amzn-ami-hvm*-x86_64-gp2"]
  }
}

resource "aws_key_pair" "tf_auth" {
  key_name   = "${var.key_name}"
  public_key = "${file(var.public_key_path)}"
}

data "template_file" "user-init" {
  count    = 2
  template = "${file("${path.module}/userdata.tpl")}"

  vars {
    firewall_subnets = "${element(var.subnet_ips, count.index)}"
  }
}

resource "aws_instance" "tf_server" {
  count         = "${var.instance_count}"
  instance_type = "${var.instance_type}"
  ami           = "${data.aws_ami.server_ami.id}"

  tags {
    Name = "tf_server-${count.index +1}"
  }

  key_name               = "${aws_key_pair.tf_auth.id}"
  vpc_security_group_ids = ["${var.security_group}"]
  subnet_id              = "${element(var.subnets, count.index)}"
  user_data              = "${data.template_file.user-init.*.rendered[count.index]}"
}
```

Create userdata.tpl:
```
vi userdata.tpl
```

userdata.tpl:
```
#!/bin/bash
yum install httpd -y
echo "Subnet for Firewall: ${firewall_subnets}" >> /var/www/html/index.html
service httpd start
chkconfig httpd on
```

Edit variables.tf:
```
vi variables.tf
```

variables.tf:
```
#-----compute/variables.tf

variable "key_name" {}

variable "public_key_path" {}

variable "subnet_ips" {
  type = "list"
}

variable "instance_count" {}

variable "instance_type" {}

variable "security_group" {}

variable "subnets" {
  type = "list"
}
```

Edit outputs.tf:
```
vi outputs.tf
```

outputs.tf"
```
#-----compute/outputs.tf-----

output "server_id" {
  value = "${join(", ", aws_instance.tf_server.*.id)}"
}

output "server_ip" {
  value = "${join(", ", aws_instance.tf_server.*.public_ip)}"
}
```

### Compute Part 3: The Root Module
In this lesson, we will finish working with the EC2 resources by adding the compute module to the root module.
  
Edit main.tf:
```
vi main.tf
```

main.tf:
```
provider "aws" {
  region = "${var.aws_region}"
}

# Deploy Storage Resources
module "storage" {
  source       = "./storage"
  project_name = "${var.project_name}"
}

# Deploy Networking Resources
module "networking" {
  source       = "./networking"
  vpc_cidr     = "${var.vpc_cidr}"
  public_cidrs = "${var.public_cidrs}"
  accessip    = "${var.accessip}"
}

# Deploy Compute Resources
module "compute" {
  source          = "./compute"
  instance_count  = "${var.instance_count}"
  key_name        = "${var.key_name}"
  public_key_path = "${var.public_key_path}"
  instance_type   = "${var.server_instance_type}"
  subnets         = "${module.networking.public_subnets}"
  security_group  = "${module.networking.public_sg}"
  subnet_ips      = "${module.networking.subnet_ips}"
}
```

Edit variables.tf:
```
vi variables.tf
```

variables.tf:
```
#----root/variables.tf-----
variable "aws_region" {}

#------ storage variables
variable "project_name" {}

#-------networking variables
variable "vpc_cidr" {}
variable "public_cidrs" {
  type = "list"
}
variable "accessip" {}

#-------compute variables
variable "key_name" {}
variable "public_key_path" {}
variable "server_instance_type" {}
variable "instance_count" {
  default = 1
}
```

Edit outputs.tf:
```
vi outputs.tf
```

outputs.tf:
```
#----root/outputs.tf-----

#----storage outputs------

output "Bucket Name" {
  value = "${module.storage.bucketname}"
}

#---Networking Outputs -----

output "Public Subnets" {
  value = "${join(", ", module.networking.public_subnets)}"
}

output "Subnet IPs" {
  value = "${join(", ", module.networking.subnet_ips)}"
}

output "Public Security Group" {
  value = "${module.networking.public_sg}"
}

#---Compute Outputs ------

output "Public Instance IDs" {
  value = "${module.compute.server_id}"
}

output "Public Instance IPs" {
  value = "${module.compute.server_ip}"
}
```

terraform.tfvars:
```
aws_region   = "us-west-1"
project_name = "la-terraform"
vpc_cidr     = "10.123.0.0/16"
public_cidrs = [
  "10.123.1.0/24",
  "10.123.2.0/24"
]
accessip    = "0.0.0.0/0"
key_name = "tf_key"
public_key_path = "/home/cloud_user/.ssh/id_rsa.pub"
server_instance_type = "t2.micro"
instance_count = 2
```

Initialize Terraform:
```
export AWS_ACCESS_KEY_ID="[ACCESS_KEY]"
export AWS_SECRET_ACCESS_KEY="[SECRET_KEY]"
terraform init
```

Validate changes:
```
terraform validate
```

Plan the changes:
```
terraform plan
```

Apply the changes:
```
terraform apply
```

Destroy environment:
```
terraform destroy
```

## Troubleshooting
### Troubleshooting Terraform Files
In this lesson, we will talk about troubleshooting issues in the real world. Sometimes it's nothing more than "XYZ is broken. Go fix it!"
  
Test your troubleshooting abilities:
```
cd ~/terraform
git clone https://github.com/linuxacademy/content-terraform-labs.git troubleshooting
cd troubleshooting
git checkout troubleshooting-aws
terraform init
```

## Terraform State
### Terraform Formatting and Remote State
In this lesson, you will learn more about Terraform state and how to version it using a S3 bucket.
  
#### Create an S3 Bucket
- Search for S3 in **Find Services**.
- Click **Create Bucket**.
- Enter a **Bucket name**. The bucket name must be unique.
- Make sure the Region is **US East (N. Virginia)** Click **Next**.
- Click **Next** again on the Configure options page.
- Click **Next** again on the Set permissions page.
- Click **Create bucket** on the Review page.

#### Add the Terraform Folder to the Bucket
- Click on the bucket name.
- Click **Create folder**.
- Enter **terraform-aws** for the folder name.
- Click **Save**.

#### Add Backend to Your Scripts
From the Docker Swarm Manager navigate to the AWS directory:
  
- `cd ~/terraform/AWS`

#### Set the Environment Variables
```
export AWS_ACCESS_KEY_ID="[ACCESS_KEY]"
export AWS_SECRET_ACCESS_KEY="[SECRET_KEY]]"
export AWS_DEFAULT_REGION="us-east-1"
```
  
Create `terraform.tf`:
```
vi terraform.tf
```
  
`terraform.tf` contents:
```
terraform {
  backend "s3" {
    key    = "terraform-aws/terraform.tfstate"
  }
}
```
  
Initialize Terraform:
```
terraform init -backend-config "bucket=[BUCKET_NAME]"
```

Validate changes:
```
terraform validate
```

Plan the changes:
```
terraform plan
```

Apply the changes:
```
terraform apply -auto-approve
```

Destroy environment:
```
terraform destroy -auto-approve
```

### Using Remote State with Jenkins
In this lesson, we will and update our CI/CD process to use remote state with our Jenkins Pipelines. This will allow us to create two separate Jenkins Pipelines: one for deploying the infrastructure and one for destroying it.

#### Create an S3 Bucket
- Search for S3 in **Find Services**.
- Click **Create Bucket**.
- Enter a **Bucket name**. The bucket name must be unique.
- Make sure the Region is **US East (N. Virginia)** Click **Next**.
- Click **Next** again on the Configure options page.
- Click **Next** again on the Set permissions page.
- Click **Create bucket** on the Review page.

#### Add the Terraform Folder to the Bucket
- Click on the bucket name.
- Click **Create folder**.
- Enter **terraform-aws** for the folder name.
- Click **Save**.

#### Create the Jenkins Deploy Job
- Enter an item name and call it **DeployDockerService**. Select **Pipeline**. Click **Ok**.
- Click **Add Parameter** and select **String Parameter**. For the name enter **access_key_id**. Set the  Default Value to your Access Key Id.
- Click **Add Parameter** and select **String Parameter**. For the name enter **secret_access_key**. Set  the Default Value to your Secret Access Key.
- Click **Add Parameter** and select **String Parameter**. For the name enter **bucket_name**. Set the - Default Value to the name of your S3 Bucket.
- Click **Add Parameter** and select Choice Parameter. For the name enter **image_name**. For choices  enter ghost:latest and ghost:alpine. Make sure they are on separate lines.
- Click **Add Parameter** and select **String Parameter**. For the name enter **ghost_ext_port**. Set the Default Value to **80**.
  
In the Pipeline section add the following to Script:
```
env.AWS_ACCESS_KEY_ID = "${access_key_id}"
env.AWS_SECRET_ACCESS_KEY = "${secret_access_key}"
env.AWS_DEFAULT_REGION = 'us-east-1'

node {
  git (
    url: 'https://github.com/linuxacademy/content-terraform-docker-service.git',
    branch: 'remote-state'
  )
  stage('init') {
    sh label: 'terraform init', script: "terraform init -backend-config \"bucket=${bucket_name}\""
  }
  stage('plan') {
    sh label: 'terraform plan', script: "terraform plan -out=tfplan -input=false -var image_name=${image_name} -var ghost_ext_port=${ghost_ext_port}"
  }
  stage('apply') {
    sh label: 'terraform apply', script: "terraform apply -lock=false -input=false tfplan"
  }
}
```

#### Create the Jenkins Destroy Job
Enter an item name and call it **DestroyDockerService**.
In Copy from enter **DeployDockerService**. Click **Ok**.
  
Change Pipeline section to the following:
```
env.AWS_ACCESS_KEY_ID = "${access_key_id}"
env.AWS_SECRET_ACCESS_KEY = "${secret_access_key}"
env.AWS_DEFAULT_REGION = 'us-east-1'

node {
  git (
    url: 'https://github.com/linuxacademy/content-terraform-docker-service.git',
    branch: 'remote-state'
  )
  stage('init') {
    sh label: 'terraform init', script: "terraform init -backend-config \"bucket=${bucket_name}\""
  }
  stage('plan_destroy') {
    sh label: 'terraform plan', script: "terraform plan -destroy -out=tfdestroyplan -input=false -var image_name=${image_name} -var ghost_ext_port=${ghost_ext_port}"
  }
  stage('destroy') {
    sh label: 'terraform apply', script: "terraform apply -lock=false -input=false tfdestroyplan"
  }
}
```
  
Once Jenkins is running:
```
docker container ls
docker exec -it 73575a9ee4ac /bin/bash
```
  
Remove `.terraform` file from your project **DeployDockerService** located in `/var/jenkins_home/workspace/`
```
rm -r .terraform
```

## Terraform and Kubernetes
### Setting up Kubernetes Installing Terraform
In this lesson we will setup a Kuberentes master and install Terraform.
  
Add the following to `kube-config.yml`:
```
apiVersion: kubeadm.k8s.io/v1beta1
kind: ClusterConfiguration
networking:
  podSubnet: 10.244.0.0/16
apiServer:
  extraArgs:
    service-node-port-range: 8000-31274
```
  
Initialize Kubernetes:
```
sudo kubeadm init --config kube-config.yml
```
  
Copy admin.conf to your home directory:
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
  
Install Flannel:
```
sudo kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
```
  
Untaint the Kubernetes Master:
```
kubectl taint nodes --all node-role.kubernetes.io/master-
```

#### Install Terraform
Terraform will be installed on the Swarm manager.
  
Install Terraform 0.11.13:
```
sudo curl -O https://releases.hashicorp.com/terraform/0.11.13/terraform_0.11.13_linux_amd64.zip
sudo unzip terraform_0.11.13_linux_amd64.zip -d /usr/local/bin/
```
  
Test the Terraform installation:
```
terraform version
```

### Creating a Pod
In this video, we will start working with Kubernetes resources by creating a Pod.
  
Setup your environment:
```
mkdir -p ~/terraform/pod
cd ~/terraform/pod
```

vi `main.tf`:
```
resource "kubernetes_pod" "ghost_alpine" {
  metadata {
    name = "ghost-alpine"
  }

  spec {
    host_network = "true"
    container {
      image = "ghost:alpine"
      name  = "ghost-alpine"
    }
  }
}
```

Initialize Terraform:
```
terraform init
```

Validate main.tf:
```
terraform validate
```

Plan the deployment:
```
terraform plan
```

Deploy the pod:
```
terraform apply -auto-approve
```

List the Pods:
```
kubectl get pods
```

Reset the environment:
```
terraform destroy -auto-approve
```

### Creating a Pod and Service
In this lesson, we will create a pod and service using Terraform.
  
Setup your environment:
```
mkdir -p ~/terraform/service
cd ~/terraform/service
```

Create main.tf:
```
vi main.tf
```

main.tf contents:
```
resource "kubernetes_service" "ghost_service" {
  metadata {
    name = "ghost-service"
  }
  spec {
    selector {
      app = "${kubernetes_pod.ghost_alpine.metadata.0.labels.app}"
    }
    port {
      port = "2368"
      target_port = "2368"
      node_port = "8081"
    }
    type = "NodePort"
  }
}

resource "kubernetes_pod" "ghost_alpine" {
  metadata {
    name = "ghost-alpine"
    labels {
      app = "ghost-blog"
    }
  }

  spec {
    container {
      image = "ghost:alpine"
      name  = "ghost-alpine"
      port  {
        container_port = "2368"
      }
    }
  }
}
```

Initialize Terraform:
```
terraform init
```

Validate the files:
```
terraform validate
```

Plan the deployment:
```
terraform plan
```

Deploy the pod:
```
terraform apply -auto-approve
```

List the Pods:
```
kubectl get pods
```

List the Services:
```
kubectl get services
```

Reset the environment:
```
terraform destroy -auto-approve
```

### Creating a Deployment
In this lesson, we will use Terraform to create a Kubernetes deployment and service.
  
Setup your environment:
```
mkdir -p ~/terraform/deployment
cd ~/terraform/deployment
```

Create main.tf:
```
vi main.tf
```

main.tf contents:
```
resource "kubernetes_service" "ghost_service" {
  metadata {
    name = "ghost-service"
  }
  spec {
    selector {
      app = "${kubernetes_deployment.ghost_deployment.spec.0.template.0.metadata.0.labels.app}"
    }
    port {
      port        = "2368"
      target_port = "2368"
      node_port   = "8080"
    }

    type = "NodePort"
  }
}

resource "kubernetes_deployment" "ghost_deployment" {
  metadata {
    name = "ghost-blog"
  }

  spec {
    replicas = "1"

    selector {
      match_labels {
        app = "ghost-blog"
      }
    }

    template {
      metadata {
        labels {
          app = "ghost-blog"
        }
      }

      spec {
        container {
          name  = "ghost"
          image = "ghost:alpine"
          port {
            container_port = "2368"
          }
        }
      }
    }
  }
}
```

Initialize Terraform:
```
terraform init
```

Validate the files:
```
terraform validate
```

Plan the deployment:
```
terraform plan
```

Deploy the pod:
```
terraform apply -auto-approve
```

List the Deployments:
```
kubectl get deployments
kubectl get pods
kubectl delete pod [POD_ID]
```

Reset the environment:
```
terraform destroy -auto-approve
```

## Terraform 0.12
### Setup and Disclaimer
In this lesson, we will install Terraform 0.12.2 and talk about some of the pitfalls you may encounter working with such a new release.
  
Install Terraform 0.12:
```
cd /tmp
sudo curl -O https://releases.hashicorp.com/terraform/0.12.2/terraform_0.12.2_linux_amd64.zip
sudo unzip terraform_0.12.2_linux_amd64.zip
sudo cp terraform /usr/bin/terraform12
```

Test the Terraform installation:
```
terraform12 version
```

Setup a Terraform 0.12 directory:
```
mkdir /home/cloud_user/terraform/t12
cd /home/cloud_user/terraform/t12
cp -r /home/cloud_user/terraform/basics .
cd basics
rm -r .terraform
```

Test Docker by initializing Terraform:
```
terraform12 init
```

Copy AWS/storage to the Terraform 0.12 directory:
```
cd /home/cloud_user/terraform/t12
cp -r ../AWS/storage .
cd storage
```

Edit main.tf:
```
vi main.tf
```

main.tf contents:
```
#---------storage/main.tf---------

# Create a random id
resource "random_id" "tf_bucket_id" {
  byte_length = 2
}

# Create the bucket
resource "aws_s3_bucket" "tf_code" {
    bucket        = "${var.project_name}-${random_id.tf_bucket_id.dec}"
    acl           = "private"

    force_destroy =  true

    tags = {
      Name = "tf_bucket"
    }
}
```

Set up the AWS access key:
```
export AWS_ACCESS_KEY_ID="[ACCESS_KEY]"
export AWS_SECRET_ACCESS_KEY="[SECRET_KEY]]"
export AWS_DEFAULT_REGION="us-east-1"
```

Initialize Terraform:
```
terraform12 init
```

Deploy the S3 bucket:
```
terraform12 apply -var project_name=la-terraform -auto-approve
```

Destroy the S3 bucket:
```
terraform12 destroy -var project_name=la-terraform -auto-approve
```

Test with the older version of Terraform:
```
ls -la
rm -r .terraform terraform.tfstate*
terraform init
terraform apply -var project_name=la-terraform -auto-approve
terraform destroy -var project_name=la-terraform -auto-approve
```

### Working with Resources
In this lesson, we will start refactoring the storage module to use some of the new features of Terraform 0.12.x.
```
cd /home/cloud_user/terraform/t12/storage
rm -rf *
```

Create main.tf:
```
vi main.tf
```

main.tf contents:
```
# Create a random id
resource "random_id" "tf_bucket_id" {
  byte_length = 2
}

# Create the bucket
resource "aws_s3_bucket" "tf_code" {
    bucket        = format("la-terraform-%d", random_id.tf_bucket_id.dec)
    acl           = "private"

    force_destroy =  true

    tags = {
      Name = "tf_bucket"
    }
}
```

Set up the AWS access key:
```
export AWS_ACCESS_KEY_ID="[ACCESS_KEY]"
export AWS_SECRET_ACCESS_KEY="[SECRET_KEY]]"
export AWS_DEFAULT_REGION="us-east-1"
```

Initialize Terraform:
```
terraform12 init
```

Deploy the S3 bucket:
```
terraform12 apply -auto-approve
```

Destroy the S3 bucket:
```
terraform12 destroy -auto-approve
```

### Input Variables
In this lesson, we will continue refactoring the storage module by adding in variables.
  
Create variables.tf:
```
vi variables.tf
```

variables.tf contents:
```
variable "project_name" {
  type = string
}
```

Create main.tf:
```
vi main.tf
```

main.tf contents:
```
# Create a random id
resource "random_id" "tf_bucket_id" {
  byte_length = 2
}

# Create the bucket
resource "aws_s3_bucket" "tf_code" {
    bucket        = format("%s-%d", var.project_name, random_id.tf_bucket_id.dec)
    acl           = "private"
    force_destroy =  true
    tags          = {
      Name = "tf_bucket"
    }
}
```

Set up the AWS access key:
```
export AWS_ACCESS_KEY_ID="[ACCESS_KEY]"
export AWS_SECRET_ACCESS_KEY="[SECRET_KEY]]"
export AWS_DEFAULT_REGION="us-east-1"
```

Plan the deploy of the S3 bucket:
```
terraform12 plan -var project_name=la-terraform
```

Deploy the S3 bucket:
```
terraform12 apply -var project_name=la-terraform -auto-approve
```

Destroy the S3 bucket:
```
terraform12 destroy -var project_name=la-terraform -auto-approve
```

### Output Values
In this lesson, we will finish off refactoring the storage module by adding outputs of the S3 bucket name as well as the project_name variable.
  
Create outputs.tf:
```
vi outputs.tf
```

outputs.tf contents:
```
output "bucketname" {
  value = aws_s3_bucket.tf_code.id
}

output "project_name" {
  value = var.project_name
}
```

Initialize Terraform:
```
terraform12 init
```

Deploy the S3 bucket:
```
terraform12 apply -var project_name=la-terraform -auto-approve
```

Destroy the S3 bucket:
```
terraform destroy -var project_name=la-terraform -auto-approve
```

### Dynamic Nested Blocks Part 1
In this lesson we will begin working with dynamic nested blocks to dynamically construct nested blocks.
  
Set up the environment:
```
mkdir ~/terraform/t12/loops
cd ~/terraform/t12/loops
```

Create main.tf:
```
vi main.tf
```
  
Please note that `default = ["22", "22"]` has been updated to `default = ["22", "80"]`. The video has not been updated yet to reflect this change.
  
`main.tf` contents:
```
variable "vpc_cidr" {
  default = "10.123.0.0/16"
}

variable "accessip" {
  default = "0.0.0.0/0"
}

variable "service_ports" {
  default = ["22", "80"]
}

resource "aws_vpc" "tf_vpc" {
  cidr_block           = var.vpc_cidr
  enable_dns_hostnames = true
  enable_dns_support   = true

  tags = {
    Name = "tf_vpc"
  }
}

resource "aws_security_group" "tf_public_sg" {
  name        = "tf_public_sg"
  description = "Used for access to the public instances"
  vpc_id      = aws_vpc.tf_vpc.id

  dynamic "ingress" {
    for_each = var.service_ports
    content {
      from_port   = ingress.value
      to_port     = ingress.value
      protocol    = "tcp"
      cidr_blocks = [var.accessip]
    }
  }
}
```

Initialize Terraform:
```
export AWS_ACCESS_KEY_ID="[ACCESS_KEY]"
export AWS_SECRET_ACCESS_KEY="[SECRET_KEY]"
export AWS_DEFAULT_REGION="us-east-1"
terraform12 init
```

Plan the changes:
```
terraform12 plan
```


### Dynamic Nested Blocks Part 2
In this lesson, we will continue working with dynamic nested blocks. We will expand on it by using the for expression to loop through a list of maps.
  
Set up the environment:
```
mkdir ~/terraform/t12/dynamic
cd ~/terraform/t12/dynamic
```

Create main.tf:
```
vi main.tf
```

main.tf contents:
```
variable "vpc_cidr" {
  default = "10.123.0.0/16"
}

variable "accessip" {
  default = "0.0.0.0/0"
}

variable "service_ports" {
  default = [
    {
      from_port = "22",
      to_port   = "22"
    },
    {
      from_port = "80",
      to_port   = "80"
    }
  ]
}

resource "aws_vpc" "tf_vpc" {
  cidr_block           = var.vpc_cidr
  enable_dns_hostnames = true
  enable_dns_support   = true

  tags = {
    Name = "tf_vpc"
  }
}

resource "aws_security_group" "tf_public_sg" {
  name        = "tf_public_sg"
  description = "Used for access to the public instances"
  vpc_id      = aws_vpc.tf_vpc.id

  dynamic "ingress" {
    for_each = [ for s in var.service_ports: {
      from_port = s.from_port
      to_port = s.to_port
    }]

    content {
      from_port   = ingress.value.from_port
      to_port     = ingress.value.to_port
      protocol    = "tcp"
      cidr_blocks = [var.accessip]
    }
  }
}

output "ingress_port_mapping" {
  value = {
    for ingress in aws_security_group.tf_public_sg.ingress:
    format("From %d", ingress.from_port) => format("To %d", ingress.to_port)
  }
}
```

Initialize Terraform:
```
export AWS_ACCESS_KEY_ID="[ACCESS_KEY]"
export AWS_SECRET_ACCESS_KEY="[SECRET_KEY]"
export AWS_DEFAULT_REGION="us-east-1"
terraform12 init
```

Plan the changes:
```
terraform12 plan
```

Apply the changes:
```
terraform12 apply -auto-approve
```

Destroy the changes:
```
terraform12 destroy -auto-approve
```

### Expressions and Functions
In this lesson, we will look at the documentation for functions. Then, we will use the `cidrsubnet` function to calculate a subnet address within a given IP network address prefix.
  
Set up the environment:
```
mkdir ~/terraform/t12/functions
cd ~/terraform/t12/functions
```

Create main.tf:
```
vi main.tf
```

main.tf contents:
```
variable "vpc_cidr" {
  default = "10.123.0.0/16"
}

variable "accessip" {
  default = "0.0.0.0/0"
}

variable "subnet_numbers" {
  default = [1, 2, 3]
}

resource "aws_vpc" "tf_vpc" {
  cidr_block           = var.vpc_cidr
  enable_dns_hostnames = true
  enable_dns_support   = true

  tags = {
    Name = "tf_vpc"
  }
}

resource "aws_security_group" "tf_public_sg" {
  name        = "tf_public_sg"
  description = "Used for access to the public instances"
  vpc_id      = aws_vpc.tf_vpc.id

  ingress {
    from_port   = "22"
    to_port     = "22"
    protocol    = "tcp"
    cidr_blocks = [
      for num in var.subnet_numbers:
      cidrsubnet(aws_vpc.tf_vpc.cidr_block, 8, num)
    ]
  }
}
```

Initialize Terraform:
```
export AWS_ACCESS_KEY_ID="[ACCESS_KEY]"
export AWS_SECRET_ACCESS_KEY="[SECRET_KEY]"
export AWS_DEFAULT_REGION="us-east-1"
terraform12 init
```

Plan the changes:
```
terraform12 plan
```