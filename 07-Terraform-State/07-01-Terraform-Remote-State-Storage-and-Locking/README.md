# Terraform Remote State Storage & Locking

## What is Terraform state ?
The primary purpose of Terraform state is to store bindings between objects in a remote system and resource instances declared in your configuration. When Terraform creates a remote object in response to a change of configuration, it will record the identity of that remote object against a particular resource instance, and then potentially update or delete that object in response to future configuration changes.
- Terraform state file gets generated after 'terraform apply' command 

**Purpose of Terraform State**
1. Mappings between resources and remote objects.
2. To track metadata such as resource dependencies.
3. Terraform stores a cache of the attribute values for all resources in the state. This is the most optional feature of Terraform state and is done only as a performance improvement.
4. In the default configuration, Terraform stores the state in a file in the current working directory where Terraform was run and get synced.

## What is Terraform backend ?
- Backends are responsible for storing state and providing an API for state locking. (State storage + State Lock)
- Backend configuration is only used by Terraform CLI.
- Terraform cloud and Terraform enterprise always use their own state storage when performing Terraform run, so they ignore any backend block in the configuartion. 
- Backend types = 

| Enhanced Backends | Standard Backends    |
| :---:   | :---: | 
| It can both store state & perform operations. There are only two enhanced backend: local & remote | It only store state and rely on the local backend for performing operations |
| E.g., of remote backend = TF cloud, TF Enterprise | E.g., AWS S3, Azure RM, Consul, etcd, gcs http ... |

| Local State | Remote State    |
| :---:   | :---: | 
| By default, Terraform stores state locally in a file named terraform.tfstate. | Terraform writes the state data to a remote data store. Terraform supports storing state in Terraform Cloud, HashiCorp Consul, Amazon S3, Azure Blob Storage, Google Cloud Storage, Alibaba Cloud OSS, and more. |
| When working with Terraform in a team, use of a local file makes Terraform usage complicated because each user must make sure they always have the latest state data before running Terraform and make sure that nobody else runs Terraform at the same time. | Terraform remote state file can then be shared between all members of a team. |

## Problem with remote state =
If two team members are running Terraform at the same time, you may run into race condition as multiple terraform processes make concurrent updates to the state files, leading to conflicts, data loss and state file corruption. To avoid such implement state locking.

## State Locking 
- Not all backend supports state locking. AWS S3 does support.
- Terraform state lock started with 'terraform plan' and 'terraform apply'. Once complete state lock get automatically released.
- Terraform will lock your state for all operations that could write state. This prevents others from acquiring the lock and potentially corrupting your state.
- State locking happens automatically on all operations that could write state. You won't see any message that it is happening. 
- If state locking fails, Terraform will not continue.
- If acquiring the lock is taking longer than expected, Terraform will output a status message.
- If Terraform doesn't output a message, state locking is still occurring if your backend supports it.
- Terraform has a force-unlock command to manually unlock the state if unlocking failed. If you unlock the state when someone else is holding the lock it could cause multiple writers. Force unlock should only be used to unlock your own lock in the situation where automatic unlocking failed. To protect you, the force-unlock command requires a unique lock ID. Terraform will output this lock ID if unlocking fails. This lock ID acts as a nonce, ensuring that locks and unlocks target the correct lock.
- Locking takes place by using terraform plan, terraform apply, terraform refresh etc. commands, where .statetf file get update.

## Step-01: Introduction
- Understand Terraform Backends
- Understand about Remote State Storage and its advantages
- This state is stored by default in a local file named "terraform.tfstate", but it can also be stored remotely, which works better in a team environment.
- Create AWS S3 bucket to store `terraform.tfstate` file and enable backend configurations in terraform settings block
- Understand about **State Locking** and its advantages
- Create DynamoDB Table and  implement State Locking by enabling the same in Terraform backend configuration

## Step-02: Create S3 Bucket
- Go to Services -> S3 -> Create Bucket
- **Bucket name:** terraform-stacksimplify
- **Region:** US-East (N.Virginia)
- **Bucket settings for Block Public Access:** leave to defaults
- **Bucket Versioning:** Enable
- Rest all leave to **defaults**
- Click on **Create Bucket**
- **Create Folder**
  - **Folder Name:** dev
  - Click on **Create Folder**


## Step-03: Terraform Backend Configuration
- **Reference Sub-folder:** terraform-manifests
- [Terraform Backend as S3](https://www.terraform.io/docs/language/settings/backends/s3.html)
- Add the below listed Terraform backend block in `Terrafrom Settings` block in `main.tf`
```
# Terraform Backend Block
  backend "s3" {
    bucket = "terraform-stacksimplify"
    key    = "dev/terraform.tfstate"
    region = "us-east-1"    
  }
```

## Step-04: Test with Remote State Storage Backend
```t
# Initialize Terraform
terraform init

Observation: 
Successfully configured the backend "s3"! Terraform will automatically
use this backend unless the backend configuration changes.

**Note: In case, change in the backend configuration then use below commands - 
If you wish to attempt automatic migration of the state, use "terraform init -migrate-state".
If you wish to store the current configuration with no changes to the state, use "terraform init -reconfigure".**

# Verify S3 Bucket for terraform.tfstate file
bucket-name/dev/terraform.tfstate

# Validate Terraform configuration files
terraform validate

# Verify S3 Bucket for terraform.tfstate file
bucket-name/dev/terraform.tfstate

# Format Terraform configuration files
terraform fmt

# Verify S3 Bucket for terraform.tfstate file
bucket-name/dev/terraform.tfstate

# Review the terraform plan
terraform plan 

# Verify S3 Bucket for terraform.tfstate file
bucket-name/dev/terraform.tfstate

# Create Resources 
terraform apply -auto-approve

# Verify S3 Bucket for terraform.tfstate file
bucket-name/dev/terraform.tfstate
Observation: Finally at this point you should see the terraform.tfstate file in s3 bucket

# Access Application
http://<Public-DNS>
```

## Step-05: Bucket Versioning - Test
- Update in `c2-variables.tf` 
```t
variable "instance_type" {
  description = "EC2 Instance Type - Instance Sizing"
  type = string
  #default = "t2.micro"
  default = "t2.small"
}
```
- Execute Terraform Commands
```t
# Review the terraform plan
terraform plan 

# Create Resources 
terraform apply -auto-approve

# Verify S3 Bucket for terraform.tfstate file
bucket-name/dev/terraform.tfstate
Observation: New version of terraform.tfstate file will be created
```


## Step-06: Destroy Resources
- Destroy Resources and Verify Bucket Versioning
```t
# Destroy Resources
terraform destroy -auto-approve
```
## Step-07: Terraform State Locking Introduction
- Understand about Terraform State Locking Advantages

## Step-08: Add State Locking Feature using DynamoDB Table
- Create Dynamo DB Table
  - **Table Name:** terraform-dev-state-table
  - **Partition key (Primary Key):** LockID (Type as String)
  - **Table settings:** Use default settings (checked)
  - Click on **Create**

## Step-09: Update DynamoDB Table entry in backend in c1-versions.tf
- Enable State Locking in `c1-versions.tf`
```t
  # Adding Backend as S3 for Remote State Storage with State Locking
  backend "s3" {
    bucket = "terraform-stacksimplify"
    key    = "dev2/terraform.tfstate"
    region = "us-east-1"  

    # For State Locking
    dynamodb_table = "terraform-dev-state-table"
  }
```

## Step-10: Execute Terraform Commands
```t
# Initialize Terraform 
terraform init

# Review the terraform plan
terraform plan 
Observation: 
1) Below messages displayed at start and end of command
Acquiring state lock. This may take a few moments...
Releasing state lock. This may take a few moments...
2) Verify DynamoDB Table -> Items tab

# Create Resources 
terraform apply -auto-approve

# Verify S3 Bucket for terraform.tfstate file
bucket-name/dev2/terraform.tfstate
Observation: New version of terraform.tfstate file will be created in dev2 folder
```

## Step-11: Destroy Resources
- Destroy Resources and Verify Bucket Versioning
```t
# Destroy Resources
terraform destroy -auto-approve

# Clean-Up Files
rm -rf .terraform*
rm -rf terraform.tfstate*  # This step not needed as e are using remote state storage here
```

## References 
- [AWS S3 Backend](https://www.terraform.io/docs/language/settings/backends/s3.html)
- [Terraform Backends](https://www.terraform.io/docs/language/settings/backends/index.html)
- [Terraform State Storage](https://www.terraform.io/docs/language/state/backends.html)
- [Terraform State Locking](https://www.terraform.io/docs/language/state/locking.html)
- [Remote Backends - Enhanced](https://www.terraform.io/docs/language/settings/backends/remote.html)
