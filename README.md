# # Terraform Infrastructure with CI/CD Pipeline

This repository automates the deployment of an EC2 instance using Terraform, with a CI/CD pipeline powered by GitHub Actions. When changes are pushed to the `main` branch, Terraform is used to plan and apply the infrastructure changes.

## Infrastructure

The Terraform configuration file `main.tf` provisions an EC2 instance in AWS using a specified Amazon Machine Image (AMI) ID (`ami-071226ecf16aa7d96` in this example). It provisions the EC2 instance using the `t2.micro` instance type, which is suitable for testing and development purposes under the AWS Free Tier.

### GitHub Actions CI/CD Pipeline

The CI/CD pipeline automates the process of deploying Terraform changes to AWS whenever changes are made to the `main` branch. The pipeline performs the following steps:

1. **Checkout the repository** - This step checks out the code to the GitHub runner.
2. **Set up Terraform** - It installs the required version of Terraform on the runner.
3. **Initialize Terraform** - This step initializes the Terraform environment and downloads the required provider plugins.
4. **Plan Terraform Deployment** - It generates an execution plan that shows the infrastructure changes that will be made.
5. **Apply Terraform Changes** - It automatically applies the Terraform plan to provision or update the infrastructure.

## Requirements

- **Terraform**: This configuration uses Terraform 1.3.6, which can be installed by following the instructions [here](https://learn.hashicorp.com/tutorials/terraform/install-cli).
- **AWS Account**: Ensure you have AWS credentials (Access Key ID and Secret Access Key) set up in GitHub Secrets to authenticate the CI/CD pipeline with AWS.

## Setup

1. **Set up AWS credentials in GitHub Secrets**:
   - `AWS_ACCESS_KEY`: Your AWS Access Key ID.
   - `AWS_SECRET_ACCESS_KEY`: Your AWS Secret Access Key.
   - `AWS_DEFAULT_REGION`: The AWS region where you want the EC2 instance deployed.

2. **Create the EC2 instance**:
   - Ensure that the AMI (`ami-071226ecf16aa7d96`) is valid for your AWS region. You may need to replace it with an AMI ID for your region.
   - Once the pipeline runs successfully, you will have an EC2 instance provisioned with the tag `Name: Terraform-CICD`.

## Pipeline Workflow

This GitHub Actions workflow runs on every push to the `main` branch. The workflow includes:

- Checkout the repository
- Set up Terraform
- Initialize Terraform
- Plan Terraform deployment
- Apply Terraform changes

### Example of `main.tf`

```hcl
provider "aws" {
  region = "us-east-1"
}

resource "aws_instance" "my_ec2" {
  ami           = "ami-071226ecf16aa7d96"  # Replace with a valid AMI for your region
  instance_type = "t2.micro"

  tags = {
    Name = "Terraform-CICD"
  }
}
```
### Troubleshooting

Ensure your AWS Access Key ID and Secret Access Key are correctly configured in GitHub Secrets.

If the EC2 instance isn't created, check the Terraform logs for any error messages and verify that the AMI ID is correct.

1. **AMI Validation**:
   - The AMI `ami-071226ecf16aa7d96` is likely specific to a region. Ensure it's a valid AMI for the region you want to deploy in. You can find the AMI IDs for different regions in the AWS Console or use the [AWS CLI](https://docs.aws.amazon.com/cli/latest/reference/ec2/describes-images.html).

2. **Security Groups and Networking**:
   - Consider adding a security group to your EC2 instance to allow inbound traffic (e.g., SSH or HTTP) if you want to interact with it. You can add it like this:
   ```hcl
   resource "aws_security_group" "allow_ssh" {
     name        = "allow_ssh"
     description = "Allow SSH inbound traffic"
     ingress {
       from_port   = 22
       to_port     = 22
       protocol    = "tcp"
       cidr_blocks = ["0.0.0.0/0"]
     }
   }

   resource "aws_instance" "my_ec2" {
     ami           = "ami-071226ecf16aa7d96"
     instance_type = "t2.micro"
     security_groups = [aws_security_group.allow_ssh.name]
     tags = {
       Name = "Terraform-CICD"
     }
   }

### Outputs

It would be helpful to output some values, like the public IP or the instance ID, after deployment. This can be added in main.tf:
```hcl
output "instance_public_ip" {
  value = aws_instance.my_ec2.public_ip
}

output "instance_id" {
  value = aws_instance.my_ec2.id
}
```
### Error Handling
In your CI/CD pipeline, make sure you handle errors gracefully by setting appropriate exit codes for Terraform operations or by adding additional logging in the GitHub Actions workflow. This ensures you can easily debug issues in the pipeline.