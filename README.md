# Using Terraform to Manage Applications and Infrastructure
  

- [About Terraform](#about-terraform)
- [Setting Up Your Environment](#setting-up-your-environment)
    - [Using Cloud Playground](#using-cloud-playground)
    - [Setting up Docker Installing Terraform](#setting-up-docker-installing-terraform)
- [Terraform Basics](#terraform-basics)
    - [Terraform Commands](#terraform-commands)
    - [HashiCorp Configuration Language](#hashicorp-configuration-language)

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
`sudo yum update -y`

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
`docker --version`

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
`docker node ls`

#### Install Terraform
Install Terraform 0.11.13 on the Swarm manager:
```
sudo curl -O https://releases.hashicorp.com/terraform/0.11.13/terraform_0.11.13_linux_amd64.zip
sudo yum install -y unzip
sudo unzip terraform_0.11.13_linux_amd64.zip -d /usr/local/bin/
```
  
Test the Terraform installation:
`terraform version`

## Terraform Basics
### Terraform Commands
In this lesson, we begin working with Terraform commands. We will start by creating a very simple Terraform file that will pull down the an image from Docker Hub.
  
List the Terraform commands:
`terraform`
  
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
`terraform init`
  
Validate the Terraform file:
`terraform validate`
  
List providers in the folder:
`ls .terraform/plugins/linux_amd64/`

List providers used in the configuration:
`terraform providers`
  
Terraform Plan:
`terraform plan`
  
Useful flags for plan:
- `-out=path`: Writes a plan file to the given path. This can be used as input to the "apply" command.
- `-var 'foo=bar'`: Set a variable in the Terraform configuration. This flag can be set multiple times.
  
Terraform Apply:
`terraform apply`
  
Useful flags for `apply`:
- `-auto-approve`: This skips interactive approval of plan before applying.
- `-var 'foo=bar'`: This sets a variable in the Terraform configuration. It can be set multiple times.
  
Confirm your apply by typing **yes**. The apply will take a bit to complete.
  
List the Docker images:
`docker image ls`
  
Terraform Show:
`terraform show`
  
Terraform Destroy:
`terraform destroy`
  
Confirm your `destroy` by typing **yes**.
  
Useful flags for destroys:
- `-auto-approve`: Skip interactive approval of plan before applying.
  
Re-list the Docker images:
`docker image ls`
  
Using a plan:
`terraform plan -out=tfplan`
  
Applying a plan:
`terraform apply tfplan`
  
Show the Docker Image resource:
`terraform show`
  
Destroy the resource once again:
`terraform destroy`

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
`cd terraform/basics`

#### Deploying a container using Terraform
Redeploy the Ghost image: `terraform apply`
  
Confirm the apply by typing **yes**. The `apply` will take a bit to complete.
  
Open `main.tf`: `vi main.tf`
  
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
  
Validate `main.tf`: `terraform validate`
  
Terraform Plan: `terraform plan`
  
Apply the changes to `main.tf`: `terraform apply`
  
Confirm the `apply` by typing **yes**.
  
List the Docker containers: `docker container ls`
  
Access the Ghost blog by opening a browser and go to: `http:://[SWARM_MANAGER_IP]`
  
#### Cleaning up the environment
Reset the environment: `terraform destroy`
  
Confirm the `destroy` by typing **yes**.
  
### Tainting and Updating Resources
In this lesson, we are going to take a look at how to force a redeploy of resources using tainting. This is an extremely useful skill for when parts of a deployment need to be modified.

#### Tainting and Untainting Resources
Terraform commands:
  
- `taint`: Manually mark a resource for recreation 
- `untaint`: Manually unmark a resource as tainted
  
Tainting a resource: `terraform taint [NAME]`
  
Untainting a resource: `terraform untaint [NAME]`
  
Set up the environment: `cd terraform/basics`
  
Redeploy the Ghost image: `terraform apply`
  
Taint the Ghost blog resource: `terraform taint docker_container.container_id`
  
See what will be changed: `terraform plan`
  
Remove the taint on the Ghost blog resource: `terraform untaint docker_container.container_id`
  
Verity that the Ghost blog resource is untainted: `terraform plan`
  
#### Updating Resources
Let's edit `main.tf` and change the image to `ghost:alpine`.
  
Open `main.tf`: `vi main.tf`
  
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
  
Validate changes made to `main.tf`: `terraform validate`
  
See what changes will be applied: `terraform plan`
  
Apply image changes: `terraform apply`
  
List the Docker containers: `docker container ls`
  
See what image Ghost is using: `docker image ls | grep [IMAGE]`
  
Check again to see what changes will be applied: `terraform plan`
  
Apply container changes: `terraform apply`
  
See what image Ghost is now using: `docker image ls | grep [IMAGE]`

#### Cleaning up the environment
Reset the environment: `terraform destroy`
  
Confirm the `destroy` by typing **yes**.
  
List the Docker images: `docker image ls`
  
Remove the Ghost blog image: `docker image rm ghost:latest`
  
Reset `main.tf`: `vi main.tf`
  
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