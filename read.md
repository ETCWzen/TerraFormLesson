This file is used to learn terraform, and to set up AWS infrastructure as code. 

https://www.youtube.com/watch?v=P4A62b1dkJE&ab_channel=RishabinCloud

using : HashiCorp Terraform v2.34.0 extension (formatter) 

*Configuration File*
main.tf file is the configuration file
Basic structure:

provider "aws" {
  region = "us-west-2"
}

resource "aws_instance" "example" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
}

*Commands* to know:
terraform init - Initializes a working directory.
terraform plan - Creates an execution plan.
terraform apply - Applies the configuration.
terraform destroy - Destroys the managed infrastructure.

*State Management*
Terraform uses a state file (terraform.tfstate) to track resources.
Learn about remote state to store the state file securely (e.g., in S3, Azure Blob Storage).

*Variables and Outputs*
Variables: Parameterize your configurations for flexibility.
Example:

variable "instance_type" {
  default = "t2.micro"
}

resource "aws_instance" "example" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = var.instance_type
}
Outputs: Share data between modules or make it available after terraform apply.
Example:

output "instance_id" {
  value = aws_instance.example.id
}

*Providers and Modules*
Providers: Learn how to configure providers like AWS, Azure, or GCP.
Modules: Create reusable modules or use public modules from Terraform Registry.

*Terraform Workflow*
Version Control: Always commit .tf files to Git, but exclude terraform.tfstate.
CI/CD Integration: Automate Terraform with pipelines using tools like GitHub Actions or Jenkins.

*Terraform Best Practices*
Use version pinning for providers.
Separate environments (e.g., dev, staging, prod) using workspaces or directory structures.
Keep configurations DRY (Don’t Repeat Yourself) by using modules and variables.

*Debugging and Troubleshooting*
Use terraform fmt, terraform validate, and terraform graph for debugging.
Learn to read and interpret Terraform logs.





#############
*Providers*
What Are Providers?
Providers are plugins that let Terraform interact with APIs for cloud platforms, SaaS providers, and other services.
Examples: AWS, Azure, Google Cloud, Kubernetes, GitHub, etc.
Key Points About Providers:
Provider Configuration

You define providers in your configuration to specify how Terraform should interact with a particular platform.
Example (AWS):

provider "aws" {
  region = "us-west-2"
}

Here, the aws provider is used, and the region is set to us-west-2.
Authentication

Most providers require credentials for authentication.
Example (AWS):
Use environment variables like AWS_ACCESS_KEY_ID and AWS_SECRET_ACCESS_KEY.
Alternatively, pass them directly in the provider block (not recommended for security):

provider "aws" {
  region     = "us-west-2"
  access_key = "your-access-key"
  secret_key = "your-secret-key"
}
*Provider Versioning*

Always pin provider versions to ensure stability.
Example:

terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}
*Multiple Providers*

You can use multiple providers in one configuration, e.g., AWS and Azure:

provider "aws" {
  region = "us-west-2"
}

provider "azurerm" {
  features {}
}

##############################

*Modules*
What Are Modules?
Modules are containers for multiple resources that are used together.
Think of them as "functions" in Terraform, allowing you to reuse code across different configurations.
Why Use Modules?
Reusability: Avoid rewriting similar configurations.
Simplification: Abstract complex configurations into reusable blocks.
Consistency: Enforce standardized practices across projects.
Structure of a Module

A module typically consists of:

Input Variables: Parameters to customize the module.
hcl
Copy code
variable "instance_type" {
  default = "t2.micro"
}


Resources: The infrastructure being provisioned.

resource "aws_instance" "example" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = var.instance_type
}


Output Variables: Expose useful data from the module.

output "instance_id" {
  value = aws_instance.example.id
}

*Using a Module*
You reference modules in your main configuration file:


module "ec2_instance" {
  source        = "./modules/ec2_instance"  # Local module path
  instance_type = "t3.micro"
}


*Types of Modules*
Root Module: The main working directory where Terraform is executed.
Child Modules: Any module called from another module.
Using Public Modules
Terraform Registry has many pre-built modules for common use cases.
Example: Using a public VPC module from the registry:

module "vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "3.5.0"

  name = "my-vpc"
  cidr = "10.0.0.0/16"

  azs             = ["us-west-2a", "us-west-2b", "us-west-2c"]
  public_subnets  = ["10.0.1.0/24", "10.0.2.0/24", "10.0.3.0/24"]
  private_subnets = ["10.0.101.0/24", "10.0.102.0/24", "10.0.103.0/24"]
}


*Creating Your Own Module*
Create a folder for your module.
Add .tf files for resources, variables, and outputs.
Call the module in your main configuration.

*Putting It Together: Example of Providers and Modules*
Here’s a complete example using both providers and modules:

*Folder Structure*
css
Copy code
terraform-project/
├── main.tf
├── modules/
│   └── ec2_instance/
│       ├── main.tf
│       ├── variables.tf
│       └── outputs.tf

*Root Module*(main.tf)
 
provider "aws" {
  region = "us-west-2"
}

module "web_server" {
  source        = "./modules/ec2_instance"
  instance_type = "t3.micro"
}


*Child Module* (modules/ec2_instance/main.tf)

resource "aws_instance" "web" {
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = var.instance_type
}

*Variables* (modules/ec2_instance/variables.tf)

variable "instance_type" {
  default = "t2.micro"
}

*Outputs* (modules/ec2_instance/outputs.tf)

output "instance_id" {
  value = aws_instance.web.id
}

This setup shows how you can use providers to connect to a cloud service and modules to encapsulate reusable infrastructure configurations.