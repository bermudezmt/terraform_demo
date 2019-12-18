# How to Create and Destroy an EC2 Instance with Terraform 0.12

Terraform is a well-loved devops tool which lets you create and manage infrastructure using code. In this tutorial, we will show you how to create an Amazon EC2 instance and destroy it using Terraform.

This guide covers a basic use case for infrastructure as code and assumes you have the following:
- An AWS account
- An IAM user with sufficient rights to provision resources

## 1. Download and install Terraform

To get started, visit the [Terraform downloads page](https://www.terraform.io/downloads.html). Click on the download link for your operating system and architecture.

Start a command shell session. You will need to run the remaining commands in the same session.

To install Terraform, unzip the binary file and move it to a directory included in your system's `PATH`. This example works on both Linux and Mac operating systems.

[![asciicast](https://asciinema.org/a/Ra8UnDdQhCbFYjkzkcb4ikrrG.svg)](https://asciinema.org/a/Ra8UnDdQhCbFYjkzkcb4ikrrG?t=5)

```
# Change directory and unzip the Terraform binary
cd ~/Downloads
unzip terraform_0.12.6.linux_amd64.zip

# Move to a location in your system $PATH
sudo mv terraform /usr/local/bin/
```

## 2. Configure your AWS credentials

You should have an  `AWS_ACCESS_KEY_ID` and an `AWS_SECRET_ACCESS_KEY`. If you do not have these, contact your AWS administrator.

To do this directly in the shell, replace `xxxxxx` with your `AWS_ACCESS_KEY_ID` and `yyyyyyyyyy` with your `AWS_SECRET_ACCESS_KEY`.

```
# Set your AWS credentials as environment variables
export AWS_ACCESS_KEY_ID=xxxxxx
export AWS_SECRET_ACCESS_KEY=yyyyyyyyyy
```

## 3. Define your AWS resources using Terraform

Create an empty directory and change into it. 

`mkdir quickstart && cd quickstart`

Add _terraform.tf_ using your preferred text editor.

**terraform.tf**
```
provider "aws" {
  region = "us-west-2"
}
```

_terraform.tf_ configures the [provider](https://www.terraform.io/docs/providers/index.html) which maps the [Terraform Configuration Language](https://www.terraform.io/docs/configuration/index.html) to the cloud provider API. In this example, we are using the AWS provider and configuring it to use the `us-west-2` region.

Next, it is time to add our resources. 


**security.tf**
```

resource "aws_security_group" "test_sg" {
  ingress {
    from_port                 = 80
    to_port                   = 80
    protocol                  = "tcp"
    cidr_blocks               = ["0.0.0.0/0"]
  }

  egress {
    from_port                 = 0
    to_port                   = 0
    protocol                  = "-1"
    cidr_blocks               = ["0.0.0.0/0"]
  }
}
```

The first resource is the [AWS security group](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-security-groups.html) called `test_sg`. In this example, we’ve defined that inbound traffic (ingress) is only allowed over TCP protocol using port 80, while all outbound traffic (egress) is allowed.

Now that we have a security group defined, it’s time to add our [Amazon EC2 instance](https://aws.amazon.com/ec2/). 

**instance.tf**
```
resource "aws_instance" "test_server" {
  ami                         = "ami-0c5b2ae5bbaf55b22"
  instance_type               = "t2.micro"
  associate_public_ip_address = true
  security_groups             = [ aws_security_group.test_sg.name ]
}

output "test_server_ip" {
  value                       = aws_instance.test_server.public_ip
}
```

Our example instance, `test_server`, uses an AMI that has been prepared for this tutorial, a t2.micro size, and references the security group that we’ve previously defined. In the `security_groups` line, you can see that the `name` attribute of `test_sg` is referenced, connecting the two resources. The output block will display the public IP address of the EC2 instance once it is created.

## 4. Create your resources

Run `terraform init` to initialize Terraform and the AWS provider. This downloads the provider plugins and runs a syntax check on your `.tf` files.

```

Initializing the backend...

Initializing provider plugins...
- Checking for available provider plugins...
- Downloading plugin for provider "aws" (hashicorp/aws) 2.42.0...

The following providers do not have any version constraints in configuration,
so the latest version was installed.

To prevent automatic upgrades to new major versions that may contain breaking
changes, it is recommended to add version = "..." constraints to the
corresponding provider blocks in configuration, with the constraint strings
suggested below.

* provider.aws: version = "~> 2.42"

Terraform has been successfully initialized!
...
```

Run `terraform apply`. This runs a dependency check and creates the necessary security group first, followed by the EC2 instance as described in `instance.tf`.

```

Do you want to perform these actions?
 Terraform will perform the actions described above.
 Only 'yes' will be accepted to approve.

 Enter a value: 
```
The Terraform CLI displays the plan based on your state file and prompts you for confirmation. Enter "yes" to create the resources. You will see some output similar to the following:

```
aws_security_group.test_sg: Creating...
aws_security_group.test_sg: Creation complete after 3s [id=sg-019bbcf39fb77683c]
aws_instance.test_server: Creating...
aws_instance.test_server: Still creating... [10s elapsed]
aws_instance.test_server: Creation complete after 12s [id=i-06f58f95ea68b6fb7]

Apply complete! Resources: 2 added, 0 changed, 0 destroyed.

Outputs:

test_server_ip = 34.208.138.29
```

In a browser, navigate to `http://x.x.x.x` where ***x.x.x.x*** is the value of `test_server_ip`.
Prepare yourself for a visual treat. 🐻

## 5. Destroy your resources

To destroy the resources you have created, run `terraform destroy`.

```
Plan: 0 to add, 0 to change, 2 to destroy.

Do you really want to destroy all resources?
 Terraform will destroy all your managed infrastructure, as shown above.
 There is no undo. Only 'yes' will be accepted to confirm.

 Enter a value: 
```

Enter "yes" to confirm that you want to destroy the specified resources.

You’ve just created and destroyed your first set of resources using Terraform. Well done! 😎
