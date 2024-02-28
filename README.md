# Project based Infrastructure as Code

## You're doing it wrong.
Software development best practices applied to Infrastructure as *Code*

### Version: 0.1 
### Revision History
| Date | Version |	Description |	Author |
| ------  | ----- | ----- | ----- |
| 29-02-24 | 0.1 | HackGreenville presentation | Brian Kennedy |








### Introduction










### Software Development best practices
#### Documentation.
haha.
#### Keep code simple, DRY.
Modules
#### Clear definition of requirements.
Modules
#### Agile, Set small milestones.
#### Effective tools.
Atlantis, github
#### Code review and Quality Control
Environment gates.  github, Atlantis
#### Install and Deploy Automation
DON'T Copy Paste $#!~
#### Appropriate Design and stick to it.
Microservice

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;


### The wrong way
```
terraform % find . -not -path '*/\.*'
.
./prod
./prod/account.tf
./qa
./qa/account.tf
./qa/user_db.tf
./dev
./dev/account.tf
./dev/user_db.tf
./modules
./modules/sso
./modules/vpc
./modules/rds
```

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

### The right way

```
terraform % find . -not -path '*/\.*'
.
./iac-project
./iac-global-module
./iac-rds-module
./iac-account
```

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

### Remote States
```
data "terraform_remote_state" "state_file" {
 backend = "s3"

 config = {
   bucket   = "companycode-terraform-state-storage"
   key      = "${var.account_id}/us-east-1/account.tfstate"
   role_arn = "arn:aws:iam::${master-account}:role/atlantis"
   region   = "us-east-1"
 }
}

locals {
  vpc_arn                  = lookup(data.terraform_remote_state.state_file.outputs, "vpc_id", null)
  subnets              = lookup(data.terraform_remote_state.state_file.outputs, "private_subnets", null)
  eks_cluster_name                  = lookup(data.terraform_remote_state.state_file.outputs, "eks_cluster_name", null)
  eks_cluster_endpoint              = lookup(data.terraform_remote_state.state_file.outputs, "eks_cluster_endpoint", null)
  eks_cluster_certificate_authority = lookup(data.terraform_remote_state.state_file.outputs, "eks_cluster_certificate_authority", null)
}
```

```
account_id = "123456789012"
atlantis_role = "arn:aws:iam::${account_id}:role/atlantis"
env = "dev"
```

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

&nbsp;

#### use it
```
module "account" {
  source        = "git@github.com:sirbeep/iac-global-module.git?ref=v1.0.0
  environment   = var.env
  account_id       = var.account_id
}
module "tags" {
  source          = "git@github.com:sirbeep/iac-module-tags.git"
  name            = "user-db-${var.env}"
  project         = "Make monay!!!"
  terraform       = true
  environment     = var.env
  cost_center     = "700"
  cost_department = "online"
  sub_department  = "Brian"
}
module "user-db" {
  source = "git@github.com:sirbeep/iac-module-aurora.git//serverless-cluster:ref=v1.0.0"
  cluster_name = "user-db-${var.env}"
  tags = module.tags.tags

  region = "us-east-1"
  env = var.env
  vpc_id = tags.outputs.vpc_id
  subnet_id = var.subnet_id
}
module "signup-queueue" { ..... }
module "microservice-serviceaccount" { ...... }
etc...
```
