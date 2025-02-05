# Terraform Resource Meta-Argument for_each
- To create multiple instances according to a map, or set of string 
- If a resource or module block includes a for_each argument whose value is a map or a set of strings, Terraform creates one instance for each member of that map or set.
- Each instance has a distinct infrastructure object associated with it, and each is separately created, updated, or destroyed when the configuration is applied.
- In blocks where for_each is set, an additional each object is available in expressions, so you can modify the configuration of each instance. This object has two attributes:
each.key — The map key (or set member) corresponding to this instance.
each.value — The map value corresponding to this instance. (If a set was provided, this is the same as each.key.)

**For set of string, each.key=each.value**
for_each = toset(["one","two"]) => each.key = one 

**For Maps, we use each.key & each.value**
for_each = {
    region = "eastus"
  }
each.key = region
each.value = eastus

## Step-01: Introduction
- Understand about Meta-Argument `for_each`
- Implement `for_each` with **Maps**
- Implement `for_each` with **Set of Strings**

## Step-02: Implement for_each with Maps
- **Reference Folder:** v1-for_each-maps
- **Use case:** Create four S3 buckets using for_each maps 
- **c2-s3bucket.tf**
```t
# Create S3 Bucket per environment with for_each and maps
resource "aws_s3_bucket" "mys3bucket" {

  for_each = {
    dev   = "my-dapp-bucket"
    qa    = "my-qapp-bucket"
    stag  = "my-sapp-bucket"    
    prod  = "my-papp-bucket"        
  }  

  bucket = "${each.key}-${each.value}"
  acl    = "private"

  tags = {
    eachvalue   = each.value
    Environment = each.key
    bucketname  = "${each.key}-${each.value}"
  }
}
```

## Step-03: Execute Terraform Commands
```t
# Switch to Working Directory
cd v1-for_each-maps

# Initialize Terraform
terraform init

# Validate Terraform Configuration Files
terraform validate

# Format Terraform Configuration Files
terraform fmt

# Generate Terraform Plan
terraform plan
Observation: 
1) Four buckets creation will be generated in plan
2) Review Resource Names ResourceType.ResourceLocalName[each.key]
2) Review bucket name (each.key+each.value)
3) Review bucket tags

# Create Resources
terraform apply
Observation: 
1) 4 S3 buckets should be created
2) Review bucket names and tags in AWS Management console

# Destroy Resources
terraform destroy

# Clean-Up 
rm -rf .terraform*
rm -rf terraform.tfstate*
```


## Step-04: Implement for_each with toset "Strings"
- **Reference Folder:** v2-for_each-toset
- **Use case:** Create four IAM Users using for_each toset strings 
- **c2-iamuser.tf**
```t
# Create 4 IAM Users
resource "aws_iam_user" "myuser" {
  for_each = toset( ["Jack", "James", "Madhu", "Dave"] )
  name     = each.key
}
```

## Step-05: Execute Terraform Commands
```t
# Switch to Working Directory
cd v2-for_each-toset

# Initialize Terraform
terraform init

# Validate Terraform Configuration Files
terraform validate

# Format Terraform Configuration Files
terraform fmt

# Generate Terraform Plan
terraform plan
Observation: 
1) Four IAM users creation will be generated in plan
2) Review Resource Names ResourceType.ResourceLocalName[each.key]
2) Review IAM User name (each.key)

# Create Resources
terraform apply
Observation: 
1) 4 IAM users should be created
2) Review IAM users in AWS Management console

# Destroy Resources
terraform destroy

# Clean-Up 
rm -rf .terraform*
rm -rf terraform.tfstate*
```

## Reference
- [Resource Meta-Argument: for_each](https://www.terraform.io/docs/language/meta-arguments/for_each.html)
- [Resource: AWS S3 Bucket](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/s3_bucket)
- [Resource: AWS IAM User](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/iam_user)
