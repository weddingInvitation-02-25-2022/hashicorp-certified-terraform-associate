# Terraform Block/ Settings Block/ Configuration Blocks

## Step-01: Introduction
- Understand about Terraform Block and its importance
1. Terraform block used to specify a required terraform version, provider requirement and configure terraform backend (Terraform state)
2. Within a Terraform block, only constant values can be used; arguments may not refer to named objects such as resources, input variables etc and may not use any of the terraform built-in functions.

- Understand how to handle version constraints for Terraform Version and Provider Version in Terraform Block

### Step-02: Understand about Terraform Settings Block
- Required Terraform Version (`required_version`) = Terraform will first check local TF CLI version is matching with the provided required version. It will lock the current configuration version. 
- Provider Requirements (`required_providers`) = Give the provider details. The provided version will get downloaded while terraform init 
- Terraform backends
- Experimental Language Features
- Passing Metadata to Providers
- Review the file named **sample-terraform-settings.tf** for more understading

## Step-03: Create a simple terraform block and play with required_version
- `required_version` focuses on underlying Terraform CLI installed on your desktop
- If the running version of Terraform on your local desktop doesn't match the constraints specified in your terraform block, Terraform will produce an error and exit without taking any further actions.
- By changing the versions try `terraform init` and observe whats happening
```
Play with Terraform Version
  required_version = "~> 0.14.3" 
  required_version = "= 0.14.3"    
  required_version = "= 0.14.4"  
  required_version = ">= 0.13"   
  required_version = "= 0.13"    
  required_version = "~> 0.13"   
 

# Terraform Block
terraform {
  required_version = "~> 0.14"
}

# To view my Terraform CLI Version installed on my desktop
terraform version

# Initialize Terraform
terraform init
```
## Step-04: Add Provider and play with Provider version
- `required_providers` block specifies all of the providers required by the current module, mapping each local provider name to a source address and a version constraint.

```
Play with Provider Version
      version = "~> 3.0"            
      version = ">= 3.0.0, < 3.1.0"
      version = ">= 3.0.0, <= 3.1.0"
      version = "~> 2.0"
      version = "~> 3.0"   
```

```
# Terraform Init with upgrade option to change provider version
terraform init -upgrade
```


## Step-05: Clean-Up
```
# Delete Terraform Folders & Files
rm -rf .terraform*
```

## References
- [Terraform Version Constraints](https://www.terraform.io/docs/configuration/version-constraints.html)
- [Terraform Versions - Best Practices](https://www.terraform.io/docs/configuration/version-constraints.html#best-practices)

