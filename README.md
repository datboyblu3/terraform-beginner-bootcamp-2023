# Terraform Beginner Bootcamp 2023

## Semantic Versioning :mage:

This project is going to utilize the semantic versioning for its tagging.

[semver.org](https://semver.org/)

The general format: 

**MAJOR.MINOR.PATCH**, eg `1.0.1`

- **MAJOR** version when you make incompatible API changes
- **MINOR** version when you add functionality in a backward compatible manner
- **PATCH** version when you make backward compatible bug fixes
Additional labels for pre-release and build metadata are available as extensions to the MAJOR.MINOR.PATCH format.

## Install the Terraform CLI

### Considerations with the Terraform CLI changes
The Terraform CLI installation instructions have changed due to gpg keyring changes. So we needed to refer to the latest install CLI instructions via Terraform Documentation and change the scripting for install.

[Install the Terraform CLI](https://developer.hashicorp.com/terraform/tutorials/aws-get-started/install-cli)


### Considerations for Linux Distribution

This project is built against Ubuntu. Please consider verifying your Linux Distribution and change according to your distribution needs

[How to Check OS Version in Linux](
https://www.cyberciti.biz/faq/how-to-check-os-version-in-linux-command-line/
)

Example of checking OS version:
```bash
$ cat /etc/os-release

PRETTY_NAME="Ubuntu 22.04.3 LTS"
NAME="Ubuntu"
VERSION_ID="22.04"
VERSION="22.04.3 LTS (Jammy Jellyfish)"
VERSION_CODENAME=jammy
ID=ubuntu
ID_LIKE=debian
HOME_URL="https://www.ubuntu.com/"
SUPPORT_URL="https://help.ubuntu.com/"
BUG_REPORT_URL="https://bugs.launchpad.net/ubuntu/"
PRIVACY_POLICY_URL="https://www.ubuntu.com/legal/terms-and-policies/privacy-policy"
UBUNTU_CODENAME=jammy
```


### Refactoring into Bash Scripts

While fixing the Terraform CLI gpg deprication issues we noticed noticed the bash scripts steps were a considerable amount of more code. So we decided to create a bash script to install the Terraform CLI.

This bash script is located here: [./bin/install_terraform_cli](./bin/install_terraform_cli)

- This will keep the Gitpod Task File ([.gitpod.yml](.gitpod.yml)) tidy.
- This will allow us an easier to debug and execute manually Terraform CLI install
- This will allow better protability for other projects that need to install Terraform CLI.


#### Shebang Considerations

A Shebang (prounanced Sha-bang) tells the bash script what program that will interpret the script. eg. `#!/bin/bash` 

ChatGPT recommended this format for bash: `#!/usr/bin/env bash`
- for portability for different OS distributions
- will search the user's PATH for the bash executable

https://en.wikipedia.org/wiki/Shebang_(Unix)

#### Execution Considerations

When executing the bash script we can use the `./` shortnad notation to execute the bash script.

eg. `./bin/install_terraform_cli`

If we are using a script in .gitpod.yml we need to point the script to a program to intrepret it.

eg. `source ./bin/install_terraform_cli`

#### Linux Permissions Considerations

In order to make our bash scripts executable we need to change linux permission for the fix to be executable at the user mode

```sh
chmod u+x ./bin/install_terraform_cli
```

alternatively:

```sh
chmod 744 ./bin/install_terraform_cli
```

https://en.wikipedia.org/wiki/Chmod

### GitHub Lifecycle (Before, Init, Command)

We need to be careful when using the Init because it will not rerun if we restart an existing workspace.

https://www.gitpod.io/docs/configure/workspace/tasks

### Working with Env Vars

We can list out all Environment Variables (Env Vars) using the `env` command

We can filter specific env vars using grep eg. `env | grep AWS_`

#### Setting and Unsetting Env Vars

In the terminal we can set using `export HELLO=`world`

In the terminal we unset using `unset HELLO`

We can set an env var temporarily when just running a command

```sh
HELLO=`world` ./bin/print_message
```

Within a bash script we can set env without writing export eg.

```sh
#!/usr/bin/env bash
HELLO=`world`

echo $HELLO
```

### Printing Vars

We can print an env var using echo eg. `echo $HELLO`

#### Scoping of Env Vars

When you open new bash terminals in VSCode it will not be aware of env vars that you have set in another window.

If you want env vars to persist across all future bash terminals, you need to set env vars in your bash profile. eg. `bash_profile`

#### Persisting Env Vars in Gitpod

We can persist env vars into gitpod by storing them in Gitpod Secrets Storage

```
gp env HELLO=`world`
```

All future workspaces launched will set the env vars for all bash terminals opened in those workspaces.

You can also set env vars in the `.gitpod.yml` but this can only contain non-sensitive env.

#### AWS CLI Installation

AWS CLI is installed for this project via the bash script [`./bin/install_aws_cli`](./bin/install_aws_clli)

[Getting Started](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)

[AWS CLI Env Vars](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-envvars.html)

We can check if our AWS credentials is configured correctly by running the following AWS CLI command:
```sh
aws sts get-caller-identity
```

If it is successful you should see a json payload return that looks like this:

```json
{
    "UserId": "AILAZVEJ4YJI9QLWBARUS",
    "Account": "123456789012",
    "Arn": "arn:aws:iam::123456789012:user/terraform-beginner-bootcamp"
}
```

We'll need to generate AWS CLI credentials from IAM User in order for the user to use AWS CLI

## Terraform Basics

### Terraform Registry

Terraform sources their providers and modules from the Terraform registry which is located at [registry.terraform.io](https://registry.terraform.io)

- **Providers** is an interface to APIs that will allow to create resources in terraform
- **Modules** are a way to make large amount of terraform code modular, portable and sharable

[Random Terraform Provider](https://registry.terraform.io/providers/hashicorp/random/)
### Terraform Console

We can see a list of all Terraform commands by simply typing `terraform`

#### Terraform init

At the start of a new Terraform project we will run `terraform init` to download the binaries for the terraform providers
that we'll use in this project

#### Terraform Plan

This will generate out a changeset, about the state of our infrastructure and what will be changed.

We can output this changeset ie. "plan" to be passed to an apply, but often you can just ignore outputting.

#### Terraform Apply

`terraform apply`

This will run a plan and pass the changeset to be executed by terraform. Apply should prompt yes or no.

Apply the `--auto-approve` flag to automatically approve a terraform transaction

#### Terraform Destroy

This will destroy resources. This can use the `--auto-approve` flag as well
```
terraform destroy --auto-approve
```

If you receive an Access Denied message when deleting, verify your permissions, groups and ensure you have the proper group name.

Troubleshooting tips:
- https://saturncloud.io/blog/troubleshooting-s3-error-access-denied-when-calling-the-listbuckets-operation/
- https://docs.aws.amazon.com/AmazonS3/latest/userguide/troubleshoot-403-errors.html

### Terraform Lock Files

`.terraform.lock.hcl` contains the locked versioning for the providers or modules that should be used
with this project.

The Terraform Lock File should be committed to your Version Control System - i.e GitHub

### Terraform State File

`.terraform.tfstate` contain information about the current state of your infrastructure .

This file **should not be committed** to your VCS. This file can contain sensitive information.
If you lose this file, you will lose the knowledge of your infrastructure. 

`.terraform.tfstate.backup` is the previous state file state.

### Terraform Directory

`.terraform` directory contains binaries of terraform providers 

### Terraform AWS Configuration

#### AWS Bucket Naming Rules

AWS has specific bucket nameing rules you should read carefully through before deploying your resources:

[AWS Bucket Naming Rules](https://docs.aws.amazon.com/AmazonS3/latest/userguide/bucketnamingrules.html)

#### Terraform Random String

To ensure your bucket names adhere to AWS's bucket naming schema, set the following variables using [Terraforms Random String resource](https://registry.terraform.io/providers/hashicorp/random/latest/docs/resources/string):

 - lower            = true
 - upper            = false
 - length           = 32 #Must be between 3 and 63 characters long
 - special          = false

