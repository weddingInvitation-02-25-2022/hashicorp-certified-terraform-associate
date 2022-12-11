# Terraform Datasources

## Step-01: Introduction
- Data source allow data to be fetched or computed for use elsewhere in terraform configuration.
- It allows Terraform to use information defined outside of Terraform, defined by another separate Terraform configuration, or modified by functions.
- A data source is accessed via a special kind of resource known as a data resource, declared using a data block
- Each data resource is associated with a single data source, which determines the kind of object (or objects) it reads and what query constraint arguments are available.
- Get the latest Amazon Linux 2 AMI ID using datasources and reference that value when creating EC2 Instance resource `ami = data.aws_ami.amzlinux.id`
- The data source and name together serve as an identifier for a given resource and so must be unique within a module.
- Resources cause Terraform to create, update, and delete infrastructure objects, data resources cause Terraform only to read objects.
- Data resources have the same dependency resolution behavior as defined for managed resources. Setting the depends_on meta-argument within data blocks defers reading of the data source until after all changes to the dependencies have been applied.
- 

**Local-only Data Sources**
- While many data sources correspond to an infrastructure object type that is accessed via a remote network API, some specialized data sources operate only within Terraform itself, calculating some results and exposing them for use elsewhere.
- For example, local-only data sources exist for rendering templates, reading local files, and rendering AWS IAM policies.
- The behavior of local-only data sources is the same as all other data sources, but their result data exists only temporarily during a Terraform operation, and is re-calculated each time a new plan is created.

**Custom Condition Checks**
- You can use precondition and postcondition blocks to specify assumptions and guarantees about how the data source operates.
```t
lifecycle {
    # The AMI ID must refer to an existing AMI that has the tag "nomad-server".
    postcondition {
      condition     = self.tags["Component"] == "nomad-server"
      error_message = "tags[\"Component\"] must be \"nomad-server\"."
    }
  }
```
**Meta-argument support**
- Data resources support count and for_each meta-arguments as defined for managed resources, with the same syntax and behavior. Each instance will separately read from its data source with its own variant of the constraint arguments, producing an indexed result.
- Data resources do not have any customization settings available for their lifecycle. However, the lifecycle block is reserved for future versions.
- When a module has multiple configurations for the same provider you can specify which configuration to use with the provider meta-argument. e.g., provider = aws.west

**Datasource Lifecycle**
- If the arguments of a data instance contain no references to computed values, such as attributes of resources that have not yet been created, then the data instance will be read and its state updated during Terraform's "refresh" phase, which by default runs prior to creating a plan. This ensures that the retrieved data is available for use during planning and the diff will show the real values obtained.
- Data instance arguments may refer to computed values, in which case the attributes of the instance itself cannot be resolved until all of its arguments are defined. In this case, refreshing the data instance will be deferred until the "apply" phase, and all interpolations of the data instance attributes will show as "computed" in the plan since the values are not yet known.

## Step-02: Create a Datasource to fetch latest AMI ID
- Create or review manifest `c6-ami-datasource.tf`
- Go to AWS Mgmt Console -> Services -> EC2 -> Images -> AMI 
- Search for "Public Images" -> Provide AMI ID
  - We can get AMI Name format
  - We can get Owner Name
  - Visibility
  - Platform
  - Root Device Type
  - and many more info here. 
- Accordingly using this information build your filters in datasource

- A data block requests that Terraform read from a given data source ("aws_ami") and export the result under the given local name. The name is used to refer to this resource from elsewhere in the same Terraform module, but has no significance outside of the scope of a module.
- Each data instance will export one or more attributes, which can be used in other resources as reference expressions of the form data.<data_resource_name>.<data_resource_local_name>.<ATTRIBUTE>. e.g., data.aws_ami.web.id


## Step-03: Reference the datasource in ec2-instance.tf
```
  ami           = data.aws_ami.amzlinux.id 
```

## Step-04: Test using Terraform commands
```
# Initialize Terraform
terraform init

# Validate Terraform configuration files
terraform validate

# Format Terraform configuration files
terraform fmt

# Review the terraform plan
terraform plan 

# Create Resources (Optional)
terraform apply -auto-approve

# Access Application
http://<Public-DNS>

# Destroy Resources
terraform destroy -auto-approve
```


## References
- [AWS EC2 AMI Datasource](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/data-sources/ami)
