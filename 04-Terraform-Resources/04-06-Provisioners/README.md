# Terraform Resource Meta-Argument Provider
For selecting a non-default provider configuration (Refer section-3)

# Provisioners & Connections

- You can use provisioners to model specific actions on the local machine or on a remote machine in order to prepare servers or other infrastructure objects for service.
- For taking extra actions after the resource creation (Example: Install some app on server or do something on local desktop after resource is created at remote destination using user_data, metadata or custom_data)
- Provisioners are a Last Resort (use them only if there is no other option)
- Running configuration management tool like HashiCorp Packer, chef, ansible to deploy required software and reboot 
- First-class Terraform provider functionality may be available = Provisioners are used to execute scripts on a local or remote machine as part of resource creation or destruction. Provisioners can be used to bootstrap a resource, cleanup before destroy, run configuration management, etc

## Creation-Time Provisioners
- By default, provisioners run when the resource they are defined within is created. Creation-time provisioners are only run during creation, not during updating or any other lifecycle. They are meant as a means to perform bootstrapping of a system.
- If a creation-time provisioner fails, the resource is marked as tainted. A tainted resource will be planned for destruction and recreation upon the next terraform apply.

## Destroy-Time Provisioners
- Destroy provisioners are run before the resource is destroyed.
- 

-  We are going to learn Terraform Provisioners in detail in [Section-09](https://github.com/stacksimplify/hashicorp-certified-terraform-associate/tree/master/09-Terraform-Provisioners) of this course. 
