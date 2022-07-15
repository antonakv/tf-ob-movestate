# tf-ob-movestate
Terraform - move state

This manual is dedicated to move Terraform resource from main module
to the local module.

## Requirements

- Hashicorp terraform recent version installed
[Terraform installation manual](https://learn.hashicorp.com/tutorials/terraform/install-cli)

- git installed
[Git installation manual](https://git-scm.com/download/mac)

- Clone git repository

```bash
git clone https://github.com/antonakv/tf-ob-movestate.git
```

Expected command output looks like this:

```bash
Cloning into 'tf-ob-movestate'...
remote: Enumerating objects: 12, done.
remote: Counting objects: 100% (12/12), done.
remote: Compressing objects: 100% (12/12), done.
remote: Total 12 (delta 1), reused 3 (delta 0), pack-reused 0
Receiving objects: 100% (12/12), done.
Resolving deltas: 100% (1/1), done.
```

- Change folder to tf-ob-movestate

```bash
cd tf-ob-movestate
```

- Run the `terraform init`

Expected result:

```
% terraform init

Initializing the backend...

Initializing provider plugins...
- Finding hashicorp/random versions matching "~> 3.3.2"...
- Finding hashicorp/null versions matching "~> 3.1.1"...
- Installing hashicorp/random v3.3.2...
- Installed hashicorp/random v3.3.2 (signed by HashiCorp)
- Installing hashicorp/null v3.1.1...
- Installed hashicorp/null v3.1.1 (signed by HashiCorp)

Terraform has created a lock file .terraform.lock.hcl to record the provider
selections it made above. Include this file in your version control repository
so that Terraform can guarantee to make the same selections by default when
you run "terraform init" in the future.

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
```

- Run the `terraform plan`

Expected result:

```
% terraform plan 

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # null_resource.hello will be created
  + resource "null_resource" "hello" {
      + id = (known after apply)
    }

  # random_pet.name will be created
  + resource "random_pet" "name" {
      + id        = (known after apply)
      + length    = 4
      + separator = "-"
    }

Plan: 2 to add, 0 to change, 0 to destroy.

──────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────

Note: You didn't use the -out option to save this plan, so Terraform can't guarantee to take exactly these actions if you run "terraform apply" now.
```

- Run the `terraform apply`

```
% terraform apply

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # null_resource.hello will be created
  + resource "null_resource" "hello" {
      + id = (known after apply)
    }

  # random_pet.name will be created
  + resource "random_pet" "name" {
      + id        = (known after apply)
      + length    = 4
      + separator = "-"
    }

Plan: 2 to add, 0 to change, 0 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

random_pet.name: Creating...
random_pet.name: Creation complete after 0s [id=directly-loosely-mature-husky]
null_resource.hello: Creating...
null_resource.hello: Provisioning with 'local-exec'...
null_resource.hello (local-exec): Executing: ["/bin/sh" "-c" "echo Hello directly-loosely-mature-husky"]
null_resource.hello (local-exec): Hello directly-loosely-mature-husky
null_resource.hello: Creation complete after 0s [id=6978338004362525484]

Apply complete! Resources: 2 added, 0 changed, 0 destroyed.
```

- Run the `terraform state list` command

Expected result:

```
% terraform state list
null_resource.hello
random_pet.name
```

- Replace `main.tf` contents with

```
module "randompet" {
    source = "./randompet"
}
```

- Run the `terraform init`

```
% terraform init      
Initializing modules...

Initializing the backend...

Initializing provider plugins...
- Reusing previous version of hashicorp/null from the dependency lock file
- Reusing previous version of hashicorp/random from the dependency lock file
- Using previously-installed hashicorp/null v3.1.1
- Using previously-installed hashicorp/random v3.3.2

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
```

- Move resource `null_resource.hello` to the module called `randompet`. Run `terraform state mv null_resource.hello module.randompet.null_resource.hello`

Expected result:

```
% terraform state mv null_resource.hello module.randompet.null_resource.hello
Move "null_resource.hello" to "module.randompet.null_resource.hello"
Successfully moved 1 object(s).

```

- Move resource `random_pet.name` to the module called `randompet`. Run `terraform state mv random_pet.name module.randompet.random_pet.name`

Expected result:

```
% terraform state mv random_pet.name module.randompet.random_pet.name
Move "random_pet.name" to "module.randompet.random_pet.name"
Successfully moved 1 object(s).

```

- Run the `terraform apply`

Expected result:

```
% terraform apply                                                    
module.randompet.random_pet.name: Refreshing state... [id=directly-loosely-mature-husky]
module.randompet.null_resource.hello: Refreshing state... [id=6978338004362525484]

No changes. Your infrastructure matches the configuration.

Terraform has compared your real infrastructure against your configuration and found no differences, so no changes are needed.

Apply complete! Resources: 0 added, 0 changed, 0 destroyed.
```
