# Terraform Module Basics
- Modules are containers for multiple resources that are used together. A module consists of a collection of .tf and/or .tf.json files kept together in a directory.
- Modules are the main way to package and reuse resource configurations with Terraform.

## Step-01: Introduction
1. Introduction - Module Basics  
  - **Root Module=** Every Terraform configuration has at least one module, known as its root module, which consists of the resources defined in the .tf files in the main working directory.
  - **Child Modules=** A Terraform module (usually the root module of a configuration) can call other modules to include their resources into the configuration. A module that has been called by another module is often referred to as a child module.
Child modules can be called multiple times within the same configuration, and multiple configurations can use the same child module.
  - **Published Modules=** In addition to modules from the local filesystem, Terraform can load modules from a public or private registry. This makes it possible to publish modules for others to use, and to use modules that others have published.
- The Terraform Registry hosts a broad collection of publicly available Terraform modules for configuring many kinds of common infrastructure. Terraform can download them automatically if you specify the appropriate source and version in a module call block. 
- Also, members of your organization might produce modules specifically crafted for your own infrastructure needs. Terraform Cloud and Terraform Enterprise both include a private module registry for sharing modules internally within your organization.

2. Module Basics 
  - Defining a Child Module = To call a module means to include the contents of that module into the configuration with specific values for its input variables. 
    - Source (Mandatory) = Its value is either the path to a local directory containing the module's configuration files, or a remote module source that Terraform should download and use. Source=Local paths, Terraform Registry, GitHub, Bitbucket, Generic Git, Mercurial repositories, HTTP URLs, S3 buckets, GCS buckets, Modules in Package Sub-directories
    - Version = When using modules installed from a module registry, we recommend explicitly constraining the acceptable version numbers to avoid unexpected or unwanted changes.
    - Meta-arguments (count, for_each, providers, depends_on, )
    - Accessing Module Output Values = The resources defined in a module are encapsulated, so the calling module cannot access their attributes directly. However, the child module can declare output values to selectively export certain values to be accessed by the calling module. e.g., module.<module_local_name>.<param>
    - Transferring Resource State Into Modules = Moving resource blocks from one module into several child modules causes Terraform to see the new location as an entirely different resource. As a result, Terraform plans to destroy all resource instances at the old address and create new instances at the new address. To preserve existing objects, you can use refactoring blocks to record the old and new addresses for each resource instance. This directs Terraform to treat existing objects at the old addresses as if they had originally been created at the corresponding new addresses.
    - Replacing resources within a module = You may have an object that needs to be replaced with a new object for a reason that isn't automatically visible to Terraform, such as if a particular virtual machine is running on degraded underlying hardware.
    ```t
    terraform plan -replace=module.example.aws_instance.example
    ```
    The above selects a resource "aws_instance" "example" declared inside a module "example" child module declared inside your root module.
    Because replacing is a very disruptive action, Terraform only allows selecting individual resource instances. 
    - Tainting resources within a module = tained resouces will get deployed in the next terraform apply 
    ```t
    terraform taint <RESOURCE-NAME>
    ```

## Step-02: Defining a Child Module
- We need to understand about the following
  - Module Source (Mandatory): To start with we will use Terraform Registry
  - Module Version (Optional): Recommended to use module version
- Create a simple EC2 Instance module
  - c1-versions.tf: standard
  - c2-variables.tf: standard
  - c3-ami-datasource.tf: standard
  - c4-ec2instance-module.tf: We will focus on building this template  
```t
# AWS EC2 Instance Module

module "ec2_cluster" {
  source                 = "terraform-aws-modules/ec2-instance/aws"
  version                = "~> 2.0"

  name                   = "my-modules-demo"
  instance_count         = 2

  ami                    = data.aws_ami.amzlinux.id
  instance_type          = "t2.micro"
  key_name               = "terraform-key"
  monitoring             = true
  vpc_security_group_ids = ["sg-08b25c5a5bf489ffa"]  # Get Default VPC Security Group ID and replace
  subnet_id              = "subnet-4ee95470" # Get one public subnet id from default vpc and replace
  user_data               = file("apache-install.sh")

  tags = {
    Name        = "Modules-Demo"
    Terraform   = "true"
    Environment = "dev"
  }
}
```

## Step-03: Define Outputs from a EC2 Instance Module
- c5-outputs.tf: We will output the EC2 Instance Module attributes (Public DNS and Public IP)
```t
# Output variable definitions

output "ec2_instance_public_ip" {
  description = "Public IP addresses of EC2 instances"
  value       = module.ec2_cluster.*.public_ip
}

output "ec2_instance_public_dns" {
  description = "Public IP addresses of EC2 instances"
  value       = module.ec2_cluster.*.public_dns
}
```

## Step-04: Execute Terraform Commands
```t
# Terraform Init
terraform init

# Terraform Validate
terraform validate

# Terraform Format
terraform fmt

# Terraform Plan
terraform plan

# Terraform Apply
terraform apply -auto-apporve

# Verify 
1) Verify in AWS management console , if EC2 Instances got created.
2) Update default security group of a VPC to allow port 80 from internet
3) Access Apache Webserver
http://<Public-IP-VM1>
http://<Public-IP-VM2>
```

## Step-05: Tainting Resources in a Module
- The **taint command** can be used to taint specific resources within a module
- **Very Very Important Note:** It is not possible to taint an entire module. Instead, each resource within the module must be tainted separately.
```t
# List Resources from State
terraform state list

# Taint a Resource
terraform taint <RESOURCE-NAME>
terraform taint module.ec2_cluster.aws_instance.this[0]

# Terraform Plan
terraform plan
Observation: First VM will be destroyed and re-created

# Terraform Apply
terraform apply -auto-approve
```

## Step-06: Clean-Up Resources & local working directory
```t
# Terraform Destroy
terraform destroy -auto-approve

# Delete Terraform files 
rm -rf .terraform*
rm -rf terraform.tfstate*
```

## Step-07: Meta-Arguments for Modules
- Meta-Argument concepts are going to be same as how we learned during Resources section.
  - count
  - for_each
  - providers
  - depends_on
- [Meta-Arguments for Modules](https://www.terraform.io/docs/language/modules/syntax.html#meta-arguments)


## References
- [Terraform EC2 Instance Module](https://registry.terraform.io/modules/terraform-aws-modules/ec2-instance/aws/latest)
