# Terraform Output Values
- Output values have several uses:
1. A root module can use outputs to print certain values in the CLI output after running terraform apply.
2. A child module can use outputs to expose a subset of its resource attributes to a parent module.
3. When using remote state, root module outputs can be accessed by other configurations via a terraform_remote_state data source.

- Resource instances managed by Terraform each export attributes whose values can be used elsewhere in configuration. 
- Outputs are only rendered when Terraform applies your plan. Running terraform plan will not render outputs.

## Step-01: Introduction
- Understand about Output Values and implement them
- Query outputs using `terraform output`
- Understand about redacting secure attributes in output values
- Generate machine-readable output

## Step-02: Basics of Output Values
- **Reference Sub folder:** terraform-manifests
- Understand Output Values
- You can export both Argument & Attribute References
- [Terraform AWS EC2 Instance](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/instance)

```t
output "attribute/argument name" {
  value = <resource_type>.<resource_local.name>.<attribute/argument referance>
}
```
- The label immediately after the output keyword is the name, which must be a valid identifier. In a root module, this name is displayed to the user; in a child module, it can be used to access the output's value.

```t
# Initialize Terraform
terraform init

# Validate Terraform configuration files
terraform validate

# Format Terraform configuration files
terraform fmt

# Review the terraform plan
terraform plan 

# Create Resources
terraform apply -auto-approve

# Access Application
http://<Public-IP>
http://<Public-DNS>
```

## Step-03: Query Terraform Outputs
- Terraform will load the project state in state file, so that using `terraform output` command we can query the state file. 
```t
# Terraform Output Commands
terraform output
terraform output ec2_publicdns
```


## Step-04: Output Values - Suppressing Sensitive Values in Output
- We can redact the sensitive outputs using `sensitve = true` in output block
- This will only redact the cli output for terraform plan and apply
- When you query using `terraform output`, you will be able to fetch exact values from `terraform.tfstate` files
- Add `sensitve = true` for output `ec2_publicdns`
```t
# Attribute Reference - Create Public DNS URL with http:// appended
output "ec2_publicdns" {
  description = "Public DNS URL of an EC2 Instance"
  value = "http://${aws_instance.my-ec2-vm.public_dns}"
  sensitive = true
}
```
- Test the flow
```t
# Terraform Apply
terraform apply -auto-approve
Observation: You should see the value as sensitive i.e., ec2_publicdns = <sensitive>

# Query using terraform output
terraform output ec2_publicdns
Observation: You should get original value from terraform.tfstate file
```

## Step-05: Generate machine-readable output
```t
# Generate machine-readable output
terraform output -json
```

## Extra-1: Accessing Child Module Outputs
In a parent module, outputs of child modules are available in expressions as module.<MODULE NAME>.<OUTPUT NAME>. 
For example, if a child module named web_server declared an output named instance_ip_addr, you could access that value as module.web_server.instance_ip_addr

## Extra-2: Custom Condition Checks
You can use precondition blocks to specify guarantees about output data.
The following examples creates a precondition that checks whether the EC2 instance has an encrypted root volume.

```t
precondition {
    condition     = data.<resource_name>.<resource_local_name>.encrypted
    error_message = "The server's root volume is not encrypted."
  }
```

Custom conditions can help capture assumptions, helping future maintainers understand the configuration design and intent. They also return useful information about errors earlier and in context, helping consumers more easily diagnose issues in their configurations.

## Extra-3: Explicit Output Dependencies - depends_on
- When a parent module accesses an output value exported by one of its child modules, the dependencies of that output value allow Terraform to correctly determine the dependencies between resources defined in different modules.

```t
depends_on = [
    # Security group rule must be created before this IP address could actually be used, otherwise the services will be unreachable.
    aws_security_group_rule.local_access,
  ]
```

## Step-06: Destroy Resources
```t
# Destroy Resources
terraform destroy -auto-approve

# Clean-Up
rm -rf .terraform*
rm -rf terraform.tfstate*
```


## References
- [Terraform Output Values](https://www.terraform.io/docs/language/values/outputs.html)
