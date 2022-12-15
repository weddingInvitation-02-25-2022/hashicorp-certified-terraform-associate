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
- If they fail, Terraform will error and rerun the provisioners again on the next terraform apply. Due to this behavior, care should be taken for destroy provisioners to be safe to run multiple times.
- If "when = destroy" is specified, the provisioner will run when the resource it is defined within is destroyed.
- Destroy-time provisioners can only run if they remain in the configuration at the time a resource is destroyed. If a resource block with a destroy-time provisioner is removed entirely from the configuration, its provisioner configurations are removed along with it and thus the destroy provisioner won't run. 

## Failure Behavior
By default, provisioners that fail will also cause the Terraform apply itself to fail. The on_failure setting can be used to change this. The allowed values are:
continue - Ignore the error and continue with creation or destruction.
fail - Raise an error and stop applying (the default behavior). If this is a creation provisioner, taint the resource.

## Multiple Provisioners
- Multiple provisioners can be specified within a resource. Multiple provisioners are executed in the order they're defined in the configuration file.
- You may also mix and match creation and destruction provisioners. Only the provisioners that are valid for a given operation will be run. 

```t
resource "aws_instance" "web" {
  # ...

  provisioner "local-exec" {
    command = "echo first"
  }

  provisioner "local-exec" {
    command = "echo second"
  }
}
```

## Connection Block
- Most provisioners require access to the remote resource via SSH or WinRM and expect a nested connection block with details about how to connect.
- Connection blocks don't take a block label and can be nested within either a resource or a provisioner = 
1. A connection block nested directly within a resource affects all of that resource's provisioners.
2. A connection block nested in a provisioner block only affects that provisioner and overrides any resource-level connection settings.

**Self object**
Expressions in connection blocks cannot refer to their parent resource by name. Instead, expressions can use the self object, which represents the connection's parent resource and has all of that resource's attributes.
```t
connection {
    type     = "ssh"
    user     = "ec2-user"
    password = ""
    host     = self.public_ip
    private_key = file("<path>")
  }
```

- SSH connection also supports the connection indirectly with a bastion host, facilitate connections by SSH over HTTP proxy.

**Executing Scripts using SSH/SCP=** 
- When using the SSH protocol, provisioners upload their script files using the Secure Copy Protocol (SCP), which requires that the remote system have the scp service program installed to act as the server for that protocol.
- Provisioners will pass the chosen script path (after %RAND% expansion) directly to the remote scp process, which is responsible for interpreting it. With the default configuration of scp as distributed with OpenSSH, you can place temporary scripts in the home directory of the remote user by specifying a relative path
```
 script_path = "terraform_provisioner_%RAND%.sh"
 ```
 

-  We are going to learn Terraform Provisioners in detail in [Section-09](https://github.com/stacksimplify/hashicorp-certified-terraform-associate/tree/master/09-Terraform-Provisioners) of this course. 
