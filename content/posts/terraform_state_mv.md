---
title: "Rename terraform module name or resource name without recreating it"
date: 2021-09-01T01:12:28+05:30
---

Lets say you want to **rename your** module after terraform is applied *atleast once*.

### Example:
Lets say you have a module `access`

```hcl
module "access" {
  source     = "..."
  variables  = "..."
}
```

and you want to rename it to `iam` like
```hcl
module "iam" {
  source     = "..."
  variables  = "..."
}
```

#### Normal ways:
##### By destroying and recreating
- `Option1:` Comment the module and apply (which will destroy it). And then rename and apply it (which will create it with new name)
- `Option2:` Using target destroy and apply like below 
```sh
terraform apply -destroy -target module.access
terraform apply -target=module.iam
```

#### Best way:
Using 
```sh
terraform state mv module.access module.iam
```

> Note: We can manually edit the state file and fix it. You can do that only
when you have permission to the remote backend(if you are having one) and when you are expert in understanding the state file.