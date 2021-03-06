# What is Infrastructure as Code?
The idea behind infrastructure as code (IAC) is that you write and execute code to define, deploy, update, and destroy your infrastructure.
## There are five broad categories of IAC tools:
1. Ad Hoc Scripts (Bash script)
- Ad hoc scripts are great for small, one-off tasks.
2. Configuration Management Tools (Ansible)
- Coding conventions
- Idempotence: Code that works correctly no matter how many times you run it is called `idempotent code`.
3. Server Templating Tools (Docker, Packer, and Vagrant)
- Packer can build an AMI from template (packer build webserver.json) to run in your production AWS account.
- Vagrant is typically used to create images that you run on your development computers (VirtualBox image that you run on your Mac or Windows laptop).
- Docker is typically used to create images of individual applications. You can run Docker images on production or development computers.
- For example, a common pattern is to use Packer to create an AMI that has the Docker Engine installed, deploy that AMI on a cluster of servers in your AWS account, and then deploy individual
Docker containers across that cluster to run your applications.
- Server templating is a key component of the shift to immutable infrastructure. If you need to update something, such as deploy a new version of your code, you create a new image from your server template and you deploy it on a new server.
4. Orchestration Tools
- Server templating tools are great for creating VMs and containers, but how do you actually manage them?
- Orchestration tools: Kubernetes, Marathon/Mesos, Amazon Elastic Container Service (Amazon ECS), Docker Swarm, and Nomad.
5. Provisioning Tools
- Whereas `configuration management`, `server templating`, and `orchestration tools` define the code that runs on each server, `provisioning` tools such as Terraform, CloudFormation, and
OpenStack Heat are responsible for creating the servers themselves 
# The Benefits of Infrastructure as Code
- Move to code improves some aspects: deploy more frequently, recover from failures faster, reduce lead time.
1. Self service
- Developers now can write code and kick off their own deployments whenever necessary, do not depend on sysadmins
2. Speed and safety
- Computer can carry out the deployment steps far faster than a person; and safer, given that an automated process will be more consistent, more repeatable, and not prone to manual error.
3. Documentation
- Instead of the state of your infrastructure being locked away in a single sysadmin???s head, you can represent the state of your infrastructure in source files that anyone can read.
4. Version control
- Keep the entire history of IaC source files in version control. Might be resolve issue by simply reverting back to previous working version.
5. Validation
- By code review, automated test, and analysis tools.
6. Reuse
- Package your code into reusable modules.
7. Happiness
- Deploying code and managing infrastructure manually is repetitive and tedious. IaC offers a better alternative that allows computers to do what they do best (automation) and developers to do what they do best (coding).

# How Terrafom Works
- Terraform is open source and written in Go programming language.
- The Go code compiles down into a single binary, called terraform.
- You can use this binary to deploy infrastructure from your laptop or build server.
- Under the hood, the `terraform` binary makes API calls to `providers` (such as AWS, Azure, GCP). Terraform gets to leverage the infrastructure those providers are already running for their API servers, as well as the authentication mechanisms you???re already using
with those providers (e.g. `%userprofile%\.aws\credentials` file).
- How does Terraform know what API calls to make? The answer is that you create `Terraform configurations`, which are text files that specify what infrastructure you want to create.
- Example of Terraform Configuration code:
```Terraform
resource "aws_instance" "example" {
  ami = "ami-408c7f28"
  instance_type = "t2.micro"
}
```
- The terraform binary parses your code, translates it into a series of API calls to the cloud providers specified in the code, and makes those API calls as efficiently as possible on your behalf.
- This snippet instructs Terraform to make API calls to AWS to deploy a server. 
- When someone needs to make changes in their infrastructure, instead of updating the infrastructure manually and directly on the servers, they make their changes in the `Terraform configuration files`, validare their changes, code reviews, commit code, run `terraform apply` command to have Terraform make the necessary API calls to deploy the changes.
# How Terraform Compares to Other IaC Tools
## Configuration Management Versus Provisioning
- Chef, Puppet, Ansible, and SaltStack are all configuration management tools, whereas CloudFormation, Terraform, and OpenStack Heat are all provisioning tools.
- Although the distinction is not entirely clear cut, configuration management tools can typically do some degree of provisioning (e.g., you can deploy a server with Ansible) and provisioning tools can typically do some degree of configuration (e.g., you can run configuration scripts on each server you provision with Terraform).
- If you already have images (created by Docker or Packer), all that is left is provision the infrastructure for running those images --> provisioning tool is your best choice.
- if you???re not using server templating tools, a good alternative is to use a configuration management and provisioning tool together ( For example, you might use Terraform to provision your servers and run Chef to configure each one.)
## Mutable Infrastructure Versus Immutable Infrastructure
- Chef, Puppet, Ansible, and SaltStack typically default to a mutable infrastructure paradigm
- If you???re using a provisioning tool such as Terraform to deploy machine images (immutable images) created by Docker or Packer, most ???changes??? are actually deployments of a completely new server.
## Procedural Language Versus Declarative Language
- Imagine that you want to deploy 10 servers using Ansible
```Shell
- ec2:
  count: 10
  image: ami-0c55b159cbfafe1f0
  instance_type: t2.micro
```
And using Terraform:
```Terraform
resource "aws_instance" "example" {
  count = 10
  ami = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
}
```
For example, imagine traffic has gone up and you want to increase the number of servers to 15. With Ansible, you need to be aware of what is already deployed and write a
totally new procedural script to add the five new servers:
```Shell
- ec2:
  count: 5
  image: ami-0c55b159cbfafe1f0
  instance_type: t2.micro
```
With declarative code, because all you do is declare the end state that you want, and Terraform figures out how to get to that end state, Terraform will also be aware of any state it created in the past. 
```Terraform
resource "aws_instance" "example" {
  count = 15
  ami = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
}
```
- Two major problems with procedural IAC tools:
  - Procedural code does not fully capture the state of the infrastructure
  - Procedural code limits reusability
## Master Versus Masterless
- By default, Chef, Puppet, and SaltStack all require that you run a master server for storing the state of your infrastructure and distributing updates.
- Ansible, CloudFormation, Heat, and Terraform are all masterless by default. 
## Agent Versus Agentless
- Chef, Puppet, and SaltStack all require you to install agent software (e.g., Chef Client, Puppet Agent, Salt Minion) on each server that you want to configure. 
- Ansible, CloudFormation, Heat, and Terraform do not require you to install any extra agents.



