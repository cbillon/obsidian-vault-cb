Wondering, what is Terraform? Let’s find out about it.

Infrastructure as Code (IaC) is a widespread terminology among DevOps professionals. It is the process of managing and provisioning the complete IT infrastructure (comprises both physical and virtual machines) using machine-readable definition files. It is a software engineering approach toward operations. It helps in automating the complete data center by using programming scripts.

With all the features that Infrastructure as Code provides, it has multiple challenges:

- Need to learn to code
- Don’t know the change impact.
- Need to revert the change
- Can’t track changes
- Can’t automate a resource
- Multiple environments for infrastructure

[Terraform](https://geekflare.com/cloud/terraform-interview-questions-and-answers/) has been created to solve these challenges.

**What is Terraform?**

[Terraform](https://www.terraform.io/) is an open-source infrastructure as Code tool developed by HashiCorp. It is used to define and provision the complete infrastructure using an easy-to-learn declarative language.

It is an infrastructure provisioning tool where you can store your cloud infrastructure setup as codes. It’s very similar to tools such as [CloudFormation](https://aws.amazon.com/cloudformation/), which you would use to automate your AWS infrastructure, but you can only use that on AWS. With Terraform, you can use it on other [cloud hosting platforms](https://geekflare.com/hosting/best-cloud-hosting-platforms/) as well.

![[maxresdefault.jpg|YouTube video]]

Below are some of the benefits of using Terraform.

- Does orchestration, not just configuration management
- Supports multiple providers such as AWS, Azure, GCP, DigitalOcean and many more
- Provide immutable infrastructure where configuration changes smoothly
- Uses easy to understand language, HCL (HashiCorp configuration language)
- Easily portable to any other provider
- Supports Client only architecture, so no need for additional configuration management on a server

## Terraform Core concepts

Below are the core concepts/terminologies used in Terraform:

- **Variables**: Also used as input-variables, it is key-value pair used by Terraform modules to allow customization.
- **Provider**: It is a plugin to interact with APIs of service and access its related resources.
- **Module**: It is a folder with Terraform templates where all the configurations are defined
- **State**: It consists of cached information about the infrastructure managed by Terraform and the related configurations.
- **Resources**: It refers to a block of one or more infrastructure objects (compute instances, virtual networks, etc.), which are used in configuring and managing the infrastructure.
- **Data Source**: It is implemented by providers to return information on external objects to terraform.
- **Output Values**: These are return values of a terraform module that can be used by other configurations.
- **Plan**: It is one of the stages where it determines what needs to be created, updated, or destroyed to move from real/current state of the infrastructure to the desired state.
- **Apply**: It is one of the stages where it applies the changes real/current state of the infrastructure in order to move to the desired state.

## Terraform Lifecycle

Terraform lifecycle consists of – **init**, **plan**, **apply**, and **destroy**.

![[terraform-lifecycle.png|terraform lifecycle - geekflare]]

- Terraform init initializes the working directory which consists of all the configuration files
- Terraform plan is used to create an execution plan to reach a desired state of the infrastructure. Changes in the configuration files are done in order to achieve the desired state.
- Terraform apply then makes the changes in the infrastructure as defined in the plan, and the infrastructure comes to the desired state.
- Terraform destroy is used to delete all the old infrastructure resources, which are marked tainted after the apply phase.

## How Terraform Works?

Terraform has two main components that make up its architecture:

- Terraform Core
- Providers

![[terraform-architecture.png|terraform architecture - geekflare]]

### Terraform Core

Terraform core uses two input sources to do its job.

The **first** input source is a Terraform configuration that you, as a user, configure. Here, you define what needs to be created or provisioned. And the **second** input source is a state where terraform keeps the up-to-date state of how the current set up of the infrastructure looks like.

So, what terraform core does is it takes the input, and it figures out the plan of what needs to be done. It compares the state, what is the current state, and what is the configuration that you desire in the end result. It figures out what needs to be done to get to that desired state in the configuration file. It figures what needs to be created, what needs to be updated, what needs to be deleted to create and provision the infrastructure.

### Providers

The second component of the architecture are providers for specific technologies. This could be cloud providers like AWS, Azure, GCP, or other infrastructure as a service platform. It is also a provider for more high-level components like Kubernetes or other platform-as-a-service tools, even some software as a self-service tool.

It gives you the possibility to create infrastructure on different levels.

For example – create an AWS infrastructure, then deploy Kubernetes on top of it and then create services/components inside that Kubernetes cluster.

Terraform has over a hundred providers for different technologies, and each provider then gives terraform user access to its resources. So through AWS provider, for example, you have access to hundreds of AWS resources like EC2 instances, the AWS users, etc. With Kubernetes provider, you access to commodities, resources like services and deployments and namespaces, etc.

So, this is how Terraform works, and this way, it tries to help you provision and cover the complete application setup from infrastructure all the way to the application.

Let’s do some practical stuff. 👨‍💻

We will install Terraform on Ubuntu and provision a very basic infrastructure.

## Install Terraform

Download the latest terraform package.

Refer to the [official download page](https://www.terraform.io/downloads.html) to get the latest version for the respective OS.

```
geekflare@geekflare:~$ wget https://releases.hashicorp.com/terraform/0.13.0/terraform_0.13.0_linux_amd64.zip
--2020-08-14 16:55:38--
https://releases.hashicorp.com/terraform/0.13.0/terraform_0.13.0_linux_amd64.zip
Resolving releases.hashicorp.com (releases.hashicorp.com)... 151.101.153.183, 2a04:4e42:24::439
Connecting to releases.hashicorp.com (releases.hashicorp.com)|151.101.153.183|:443... connected.
HTTP request sent, awaiting response... 200 OK
Length: 34851622 (33M) [application/zip]
Saving to: ‘terraform_0.13.0_linux_amd64.zip’

terraform_0.13.0_linux_amd64.zip
100%[=================================================================>] 33.24M
90.3KB/s in 5m 28s

2020-08-14 17:01:06 (104 KB/s) - ‘terraform_0.13.0_linux_amd64.zip’ saved [34851622/34851622]
```

Extract the downloaded package.

```
geekflare@geekflare:~$ unzip terraform_0.13.0_linux_amd64.zip
Archive:
terraform_0.13.0_linux_amd64.zip
inflating: terraform
```

Move the terraform executable file to the path shown below. Check the terraform version.

```
geekflare@geekflare:~$ sudo mv terraform /usr/local/bin/
[sudo] password for geekflare:
geekflare@geekflare:~$ terraform -v
Terraform v0.13.0
```

You can see these are the available commands in terraform for execution.

```
geekflare@geekflare:~$ terraform
Usage: terraform [-version] [-help] <command> [args]

The available commands for execution are listed below.
The most common, useful commands are shown first, followed by
less common or more advanced commands. If you're just getting
started with Terraform, stick with the common commands. For the
other commands, please read the help and docs before usage.

Common commands:
apply Builds or changes infrastructure
console Interactive console for Terraform interpolations
destroy Destroy Terraform-managed infrastructure
env Workspace management
fmt Rewrites config files to canonical format
get Download and install modules for the configuration
graph Create a visual graph of Terraform resources
import Import existing infrastructure into Terraform
init Initialize a Terraform working directory
login Obtain and save credentials for a remote host
logout Remove locally-stored credentials for a remote host
output Read an output from a state file
plan Generate and show an execution plan
providers Prints a tree of the providers used in the configuration
refresh Update local state file against real resources
show Inspect Terraform state or plan
taint Manually mark a resource for recreation
untaint Manually unmark a resource as tainted
validate Validates the Terraform files
version Prints the Terraform version
workspace Workspace management

All other commands:
0.12upgrade Rewrites pre-0.12 module source code for v0.12
0.13upgrade Rewrites pre-0.13 module source code for v0.13
debug Debug output management (experimental)
force-unlock Manually unlock the terraform state
push Obsolete command for Terraform Enterprise legacy (v1)
state Advanced state management
```

## Provision AWS EC2 Instance Using Terraform

In this demo, I am going to launch a new AWS EC2 instance using Terraform.

Create a working directory for this Terraform demo.

```
geekflare@geekflare:~$ mkdir terraform_demo
```

Go to the directory and create a terraform configuration file where you define the provider and resources to launch an AWS EC2 instance.

```
geekflare@geekflare:~$ cd terraform_demo/
geekflare@geekflare:~/terraform_demo$ gedit awsec2.tf

provider "aws" {
access_key = "B5KG6Fe5GUKIATUF5UD"
secret_key = "R4gb65y56GBF6765ejYSJA4YtaZ+T6GY7H"
region = "us-west-2"
}

resource "aws_instance" "terraform_demo" {
ami = "ami-0a634ae95e11c6f91"
instance_type = "t2.micro"
}
```

_Note: I have changed the access and secret keys 😛, you need to use your own._

From the configuration mentioned above, you can see I am mentioning the provider like AWS. Inside the provider, I am giving AWS user credentials and regions where the instance must be launched.

In resources, I am giving AMI details of Ubuntu (ami-0a634ae95e11c6f91) and mentioning the instance type should be t2.micro

You can see how easy and readable the configuration file is, even if you are not a die-hard coder.

### terraform init

Now, the first step is to initialize terraform.

```
geekflare@geekflare:~/terraform_demo$ terraform init

Initializing the backend...

Initializing provider plugins...
- Using previously-installed hashicorp/aws v3.2.0

The following providers do not have any version constraints in configuration,
so the latest version was installed.

To prevent automatic upgrades to new major versions that may contain breaking
changes, we recommend adding version constraints in a required_providers block
in your configuration, with the constraint strings suggested below.

* hashicorp/aws: version = "~> 3.2.0"

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
```

### terraform plan

Next is the plan stage; it will create the execution graph for creating and provisioning the infrastructure.

```
geekflare@geekflare:~/terraform_demo$ terraform plan
Refreshing Terraform state in-memory prior to plan...
The refreshed state will be used to calculate this plan, but will not be
persisted to local or remote state storage.

------------------------------------------------------------------------

An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
+ create

Terraform will perform the following actions:

# aws_instance.terraform_demo will be created
+ resource "aws_instance" "terraform_demo" {
+ ami = "ami-0a634ae95e11c6f91"
+ arn = (known after apply)
+ associate_public_ip_address = (known after apply)
+ availability_zone = (known after apply)
+ cpu_core_count = (known after apply)
+ cpu_threads_per_core = (known after apply)
+ get_password_data = false
+ host_id = (known after apply)
+ id = (known after apply)
+ instance_state = (known after apply)
+ instance_type = "t2.micro"
+ ipv6_address_count = (known after apply)
+ ipv6_addresses = (known after apply)
+ key_name = (known after apply)
+ outpost_arn = (known after apply)
+ password_data = (known after apply)
+ placement_group = (known after apply)
+ primary_network_interface_id = (known after apply)
+ private_dns = (known after apply)
+ private_ip = (known after apply)
+ public_dns = (known after apply)
+ public_ip = (known after apply)
+ secondary_private_ips = (known after apply)
+ security_groups = (known after apply)
+ source_dest_check = true
+ subnet_id = (known after apply)
+ tenancy = (known after apply)
+ volume_tags = (known after apply)
+ vpc_security_group_ids = (known after apply)

+ ebs_block_device {
+ delete_on_termination = (known after apply)
+ device_name = (known after apply)
+ encrypted = (known after apply)
+ iops = (known after apply)
+ kms_key_id = (known after apply)
+ snapshot_id = (known after apply)
+ volume_id = (known after apply)
+ volume_size = (known after apply)
+ volume_type = (known after apply)
}

+ ephemeral_block_device {
+ device_name = (known after apply)
+ no_device = (known after apply)
+ virtual_name = (known after apply)
}

+ metadata_options {
+ http_endpoint = (known after apply)
+ http_put_response_hop_limit = (known after apply)
+ http_tokens = (known after apply)
}

+ network_interface {
+ delete_on_termination = (known after apply)
+ device_index = (known after apply)
+ network_interface_id = (known after apply)
}

+ root_block_device {
+ delete_on_termination = (known after apply)
+ device_name = (known after apply)
+ encrypted = (known after apply)
+ iops = (known after apply)
+ kms_key_id = (known after apply)
+ volume_id = (known after apply)
+ volume_size = (known after apply)
+ volume_type = (known after apply)
}
}

Plan: 1 to add, 0 to change, 0 to destroy.

------------------------------------------------------------------------

Note: You didn't specify an "-out" parameter to save this plan, so Terraform
can't guarantee that exactly these actions will be performed if
"terraform apply" is subsequently run.
```

### terraform apply

The apply stage will execute the configuration file and launch an AWS EC2 instance. When you run apply command, it will ask you, “Do you want to perform these actions?”, you need to type yes and hit enter.

```
geekflare@geekflare:~/terraform_demo$ terraform apply

An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
+ create

Terraform will perform the following actions:

# aws_instance.terraform_demo will be created
+ resource "aws_instance" "terraform_demo" {
+ ami = "ami-0a634ae95e11c6f91"
+ arn = (known after apply)
+ associate_public_ip_address = (known after apply)
+ availability_zone = (known after apply)
+ cpu_core_count = (known after apply)
+ cpu_threads_per_core = (known after apply)
+ get_password_data = false
+ host_id = (known after apply)
+ id = (known after apply)
+ instance_state = (known after apply)
+ instance_type = "t2.micro"
+ ipv6_address_count = (known after apply)
+ ipv6_addresses = (known after apply)
+ key_name = (known after apply)
+ outpost_arn = (known after apply)
+ password_data = (known after apply)
+ placement_group = (known after apply)
+ primary_network_interface_id = (known after apply)
+ private_dns = (known after apply)
+ private_ip = (known after apply)
+ public_dns = (known after apply)
+ public_ip = (known after apply)
+ secondary_private_ips = (known after apply)
+ security_groups = (known after apply)
+ source_dest_check = true
+ subnet_id = (known after apply)
+ tenancy = (known after apply)
+ volume_tags = (known after apply)
+ vpc_security_group_ids = (known after apply)

+ ebs_block_device {
+ delete_on_termination = (known after apply)
+ device_name = (known after apply)
+ encrypted = (known after apply)
+ iops = (known after apply)
+ kms_key_id = (known after apply)
+ snapshot_id = (known after apply)
+ volume_id = (known after apply)
+ volume_size = (known after apply)
+ volume_type = (known after apply)
}

+ ephemeral_block_device {
+ device_name = (known after apply)
+ no_device = (known after apply)
+ virtual_name = (known after apply)
}

+ metadata_options {
+ http_endpoint = (known after apply)
+ http_put_response_hop_limit = (known after apply)
+ http_tokens = (known after apply)
}

+ network_interface {
+ delete_on_termination = (known after apply)
+ device_index = (known after apply)
+ network_interface_id = (known after apply)
}

+ root_block_device {
+ delete_on_termination = (known after apply)
+ device_name = (known after apply)
+ encrypted = (known after apply)
+ iops = (known after apply)
+ kms_key_id = (known after apply)
+ volume_id = (known after apply)
+ volume_size = (known after apply)
+ volume_type = (known after apply)
}
}

Plan: 1 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
Terraform will perform the actions described above.
Only 'yes' will be accepted to approve.

Enter a value: yes

aws_instance.terraform_demo: Creating...
aws_instance.terraform_demo: Still creating... [10s elapsed]
aws_instance.terraform_demo: Still creating... [20s elapsed]
aws_instance.terraform_demo: Still creating... [30s elapsed]
aws_instance.terraform_demo: Still creating... [40s elapsed]
aws_instance.terraform_demo: Creation complete after 44s [id=i-0eec33286ea4b0740]

Apply complete! Resources: 1 added, 0 changed, 0 destroyed.
```

Go to your AWS EC2 dashboard, and you will see a new instance with the instance id mentioned at the end of apply command has been created.

![[terraform-aws-ec2.png|terraform aws ec2 - geekflare]]

You have successfully launched an AWS EC2 instance using Terraform.

### terraform destroy

Finally, if you want to delete the infrastructure, you need to run the destroy command.

```
geekflare@geekflare:~/terraform_demo$ terraform destroy
aws_instance.terraform_demo: Refreshing state... [id=i-0eec33286ea4b0740]

An execution plan has been generated and is shown below.
Resource actions are indicated with the following symbols:
- destroy

Terraform will perform the following actions:

# aws_instance.terraform_demo will be destroyed
- resource "aws_instance" "terraform_demo" {
- ami = "ami-0a634ae95e11c6f91" -> null
- arn = "arn:aws:ec2:us-west-2:259212389929:instance/i-0eec33286ea4b0740" -> null
- associate_public_ip_address = true -> null
- availability_zone = "us-west-2c" -> null
- cpu_core_count = 1 -> null
- cpu_threads_per_core = 1 -> null
- disable_api_termination = false -> null
- ebs_optimized = false -> null
- get_password_data = false -> null
- hibernation = false -> null
- id = "i-0eec33286ea4b0740" -> null
- instance_state = "running" -> null
- instance_type = "t2.micro" -> null
- ipv6_address_count = 0 -> null
- ipv6_addresses = [] -> null
- monitoring = false -> null
- primary_network_interface_id = "eni-02a46f2802fd15634" -> null
- private_dns = "ip-172-31-13-160.us-west-2.compute.internal" -> null
- private_ip = "172.31.13.160" -> null
- public_dns = "ec2-34-221-77-94.us-west-2.compute.amazonaws.com" -> null
- public_ip = "34.221.77.94" -> null
- secondary_private_ips = [] -> null
- security_groups = [
- "default",
] -> null
- source_dest_check = true -> null
- subnet_id = "subnet-5551200c" -> null
- tags = {} -> null
- tenancy = "default" -> null
- volume_tags = {} -> null
- vpc_security_group_ids = [
- "sg-b5b480d1",
] -> null

- credit_specification {
- cpu_credits = "standard" -> null
}

- metadata_options {
- http_endpoint = "enabled" -> null
- http_put_response_hop_limit = 1 -> null
- http_tokens = "optional" -> null
}

- root_block_device {
- delete_on_termination = true -> null
- device_name = "/dev/sda1" -> null
- encrypted = false -> null
- iops = 100 -> null
- volume_id = "vol-0be2673afff6b1a86" -> null
- volume_size = 8 -> null
- volume_type = "gp2" -> null
}
}

Plan: 0 to add, 0 to change, 1 to destroy.

Do you really want to destroy all resources?
Terraform will destroy all your managed infrastructure, as shown above.
There is no undo. Only 'yes' will be accepted to confirm.

Enter a value: yes

aws_instance.terraform_demo: Destroying... [id=i-0eec33286ea4b0740]
aws_instance.terraform_demo: Still destroying... [id=i-0eec33286ea4b0740, 10s elapsed]
aws_instance.terraform_demo: Still destroying... [id=i-0eec33286ea4b0740, 20s elapsed]
aws_instance.terraform_demo: Still destroying... [id=i-0eec33286ea4b0740, 30s elapsed]
aws_instance.terraform_demo: Destruction complete after 34s

Destroy complete! Resources: 1 destroyed.
```

If you recheck the EC2 dashboard, you will see the instance got terminated.

![[terraform-aws-ec2-destroyed.png|terraform aws ec2 destroyed - geekflare]]

### Conclusion

I believe the above gives you an idea to get it started with Terraform. Go ahead and try out the example I have just shown.

If you are interested in learning more, then I would suggest checking [Learning DevOps with Terraform course](https://click.linksynergy.com/deeplink?id=jf7w44yEft4&mid=39197&murl=https%3A%2F%2Fwww.udemy.com%2Fcourse%2Flearn-devops-infrastructure-automation-with-terraform%2F).