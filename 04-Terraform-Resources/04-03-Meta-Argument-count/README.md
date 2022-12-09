# Terraform Resource Meta-Argument count
- For creating multiple resource instances according to a count 
- If a resource or module block includes a count argument whose value is a whole number, Terraform will create that many instances.
- Each instance has a distinct infrastructure object associated with it, and each is separately created, updated, or destroyed when the configuration is applied.
- It accepts numeric expressions only. The count value must be known before Terraform performs any remote resource actions. 
- count.index — The distinct index number (starting with 0) corresponding to this instance.
- When count is set, Terraform distinguishes between the block itself and the multiple resource or module instances associated with it. Instances are identified by an index number, starting with 0. <TYPE>.<NAME>[<INDEX>] or module.<NAME>[<INDEX>] (for example, aws_instance.server[0], aws_instance.server[1], etc.) refers to individual instances.
- **A given resource or module block cannot use both count and for_each**
- **Problem with Count** = The resource instances were still identified by their index instead of the string values in the list. If an element was removed from the middle of the list, every instance after that element would see its index value change, resulting in more remote object changes than intended. Using for_each gives the same flexibility without the extra churn.

## Step-01: Introduction
- Understand Resource Meta-Argument `count`
- Also implement count and count index practically 

## Step-02: Create 5 EC2 Instances using Terraform
- In general, 1 EC2 Instance Resource in Terraform equals to 1 EC2 Instance in Real AWS Cloud
- 5 EC2 Instance Resources = 5 EC2 Instances in AWS Cloud
- With `Meta-Argument count` this is going to become super simple. 
- Lets see how. 
```t
# Create EC2 Instance
resource "aws_instance" "web" {
  ami = "ami-047a51fa27710816e" # Amazon Linux
  instance_type = "t2.micro"
  count = 5
  tags = {
    "Name" = "web"
  }
}
```
- **Execute Terraform Commands**
```t
# Initialize Terraform
terraform init

# Terraform Validate
terraform validate

# Terraform Plan to Verify what it is going to create / update / destroy
terraform plan

# Terraform Apply to Create EC2 Instance
terraform apply 
```
- Verify EC2 Instances and its Name


## Step-03: Understand about count index
- If we currently see all our EC2 Instances has the same name `web`
- Lets name them by using count index `web-0, web-1, web-2, web-3, web-4`
```t
# Create EC2 Instance
resource "aws_instance" "web" {
  ami = "ami-047a51fa27710816e" # Amazon Linux
  instance_type = "t2.micro"
  count = 5
  tags = {
    #"Name" = "web"
    "Name" = "web-${count.index}"
  }
}
```
- **Execute Terraform Commands**
```t
# Terraform Validate
terraform validate

# Terraform Plan to Verify what it is going to create / update / destroy
terraform plan

# Terraform Apply to Create EC2 Instance
terraform apply 
```
- Verify EC2 Instances


## Step-04: Destroy Terraform Resources
```
# Destroy Terraform Resources
terraform destroy

# Remove Terraform Files
rm -rf .terraform*
rm -rf terraform.tfstate*
```

## References
- [Resources: Count Meta-Argument](https://www.terraform.io/docs/language/meta-arguments/count.html)
