---
title: "Github actions with Manual Approval Jobs"
date: 2021-07-21T08:05:15+05:30
# draft: true
---

Lets say we use terraform and github actions for ci/cd.

We will have plan and apply steps like below

```yaml
name: 'Terraform'

on: [push, pull_request]

jobs:
  check_lint:
    #....
  check_vulnerability:
    needs: check_lint
    #...
  plan_and_apply:
    name: 'Plan and Apply'
    runs-on: ubuntu-latest
    needs: check_vulnerability

    steps:
    - name: Checkout
      uses: actions/checkout@v2
    
    - name: Setup Terraform
      uses: hashicorp/setup-terraform@v1

    - name: Terraform Init
      run: terraform init

    - name: Terraform Plan
      run: terraform plan

    - name: Terraform Apply
      run:  terraform apply -auto-approve 
```

### How to make it approval based ?

Using github [environments](https://docs.github.com/en/actions/reference/environments) feature

Steps:
- Create new environment `your repo > Settings > Environments`
- Add the Required reviewers and save protection rule (refer below image)
- [Optionally] Control from which branch the deployments can happen using `Deployment branches` section

> Refer the image below: Creating a `prod-env` with required reviewers to be `dineshba`

![github-env.png](/github-env.png)


#### Split single job into two inside same workflow:

```yaml
name: 'Terraform'

on: [push, pull_request]

jobs:
  check_lint:
    #....
  check_vulnerability:
    needs: check_lint
    #...
  plan:
    name: 'Plan'
    runs-on: ubuntu-latest
    needs: check_vulnerability

    steps:
    - name: Checkout
      uses: actions/checkout@v2
    
    - name: Setup Terraform
      uses: hashicorp/setup-terraform@v1

    - name: Terraform Init
      run: terraform init

    - name: Terraform Plan
      run: terraform plan

  apply:
    name: 'Apply'
    runs-on: ubuntu-latest
    needs: plan
    environment:
        name: prod-env
        # As prod-env required reviewers is `dineshba`,
        # it will wait for approval from dineshba
        

    steps:
    #... checkout, setup, init terraform steps like plan
    - name: Terraform Apply
      run: terraform apply -auto-approve
```

After the plan stage, github actions will send notification/email to configured reviewers.
After it is **approved**, apply stage will run
