---
title: "Access management for git at scale"
date: 2020-12-18T17:49:40+05:30
---

In order to manage multiple users in a github organization, **it is best to create multiple teams with different access and add users to the necessary teams**.

```yaml
Example:
  OrgName: example
  Teams: # each team will have different repo access
  - frontend
  - backend
  - devops 
  Members:
  - name: dineshba
    teams: devops, backend
  - name: another-dev
    teams: frontend
  - name: some-tech-lead
    teams: frontend, backend,devops
```
Hope the above yaml explains the concept of **organization, teams and team-memberships** at high level

So the steps will be:

1. Add user to the organization (1 time per user)
2. Create a team if needed or reuse the team (only 1 time)
3. Give repo access to newly created team (only 1 time)
4. Add user to the team (1 time per user)

So, number of steps for 10 users will be 10 + 1 + 1 + 10. 

To be generic, `2 + 2n` steps

#### At Scale:

- How easy it is to add around 100 users to your github organization and manage easily ?
    - Read further if you think it is not easy
    - Read further if you think it is super boring and repeating task
    - Read further if you will be tired of clicking mouse buttons again and again
- How long will it take it do ?
    - Read further if you think it will take around 50min (Assume: 2 users can be added in 1 min)
    - Read further if you think the automation will work but writing automation may take more time than adding manually


We had an usecase, where we have to add more than 100 users to group of repos in our github organization immediately and also add users in future on demand basis.

So we created a automation to do this for us using **terraform** (yes, terraform is not only for infra management, we can do lot more with terraform). And it took less than 50 min (but this blog took more than 50 min 🤪)

Lets talk using terraform. Even if you dont know about terraform, you should be able to understand things a high level.

- Declare the [github terraform provider](https://www.terraform.io/docs/providers/github/)

```tf
provider "github" {
  token        = var.github_token
  organization = var.github_organization
}
```
with this, terraform knows how to interact with github apis.

- Specify the version

```tf
terraform {
  required_version = ">=0.12.6"
  required_providers {
    github = "~> 2.9.2"
  }
}
```
> Note: Please use the latest version if you copying from here

- Define list of variables

```tf
variable "github_token" { # var used in first step
  type = string
}

variable "github_organization" { # var used in first step
  type    = string
  default = "some-org"
}

variable "github_members" { # var used in next steps
  type = list(string)

  default = ["dineshba"]
}
```

- Invite user to your organization

```tf
resource "github_membership" "membership_for_users" {
  for_each = toset(var.github_members)
  username = each.key
  role     = "member"
}
```
>Note: organization name and token to interact with github apis is given in first step

If we run terraform plan and apply with above content, terraform will invite `dineshba` to join `some-org` github organisation

- Use a team in your organization if it is already present

```tf
# use data-block to read team information, as team is already present
data "github_team" "some_team" { 
  slug = "some-team"
}
```

- Create a team if you want to create some-team for this use-case

```tf
# use this resource-block to create/update team as team is already not present
resource "github_team" "some_team" { 
  name        = "some-team"
  description = "Some cool team"
  privacy     = "closed"
}

# provide necessary repository access to the team
resource "github_team_repository" "some_team_repo" {
  team_id    = "${github_team.some_team.id}"
  repository = "some-repo"
  permission = "pull"
}
```
If we run terraform plan and apply with above content, terraform will create a team called `some-team` and provide pull access to this team members for the repo `some-repo`

- Add newly added person to this team

```tf
resource "github_team_membership" "some_team_team_membership" {
  for_each = toset(var.github_members)
  team_id  = data.github_team.some_team.id  # use resource instead of data if team is created by terraform
  username = each.key
  role     = "member"
}
```

- Thats all, do
> Create github token and provide to terraform, so that terraform can do things on our behalf
```sh
export TF_VAR_github_token=<your-personal-github-token-created-in-UI>
```
Run terraform
```sh
terraform init
terraform plan
terraform apply
```

- Add this steps to CI/CD to make things much more simple. I am not covering this as part of this blog

Please try and share your feedbacks below