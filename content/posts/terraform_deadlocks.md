---
title: "Avoid Deadlocks in Terraform"
date: 2021-07-29T16:12:28+05:30
---

Recently I encountered a problem with the **terraform code that I have written**. It created a **deadlock between resources**.

- [Example](#example)
- [Problem](#problem)
- [Solutions: (I came cross)](#solutions-i-came-cross)
  * [Best/Easy/Final Solution](#besteasyfinal-solution)
  * [Okay Solution](#okay-solution)
  * [Worst Solution](#worst-solution)
- [References](#references)

#### Example

```hcl
resource "google_compute_instance_template" "instance_template" {
}

resource "google_compute_instance_group_manager" "igm" {
  version {
    instance_template = google_compute_instance_template.instance_template.id
  }
}
```
> Note: resource `google_compute_instance_group_manager` dependson resource `google_compute_instance_template`

#### Problem
Whenever there is a change in config of `google_compute_instance_template`, terraform will delete and create it freshly (as `google_compute_instance_template` is immutable resource)

So plan will show something like this
```sh
Plan: 1 to add, 1 to change, 1 to destroy.
```

- 1 to add is new `google_compute_instance_template`
- 1 to delete is old/current `google_compute_instance_template`
- 1 to change is `google_compute_instance_group_manager`. It will be updated with new `google_compute_instance_template` id

During apply, it will **first delete** google_compute_instance_template, then create it and then change google_compute_instance_group_manager

While trying to delete, it will fail saying it is already in use by `google_compute_instance_group_manager`

#### Solutions: (I came cross in below order)
- [Worst Solution](#worst-solution)
- [Okay Solution](#okay-solution)
- [Best/Easy/Final Solution](#besteasyfinal-solution)

> Note: Use best solution only. Other solutions are just for learning purpose

##### Worst Solution

- Append the `google_compute_instance_template` name to depend resource's name (in this case, `google_compute_instance_group_manager`)

```hcl
resource "google_compute_instance_template" "instance_template" {
  name_prefix = "my-template"
  # while create it will append some random numbers to the end
  # In this way, whenever it is recreated, name will be changed.
}

resource "google_compute_instance_group_manager" "igm" {
  name = "igm-${google_compute_instance_template.instance_template.id}"
  # whenever google_compute_instance_template changes, it will
  # recreate, which means name will change, which means name of
  # this resource will change. This way, we are forcing to
  # recreate this resource, whenever google_compute_instance_template
  # recreated
  version {
    instance_template = google_compute_instance_template.instance_template.id
  }
}
```
- This will recreate the all dependent resources whenever there is a change in `google_compute_instance_template`. Very destructive solution 🤮🤯😵‍💫

##### Okay Solution

Whenever there is a change in `google_compute_instance_template` resource in terraform, follow below steps:
- Remove `google_compute_instance_template` state from tfstate
- `terraform apply` (it doesn't know about existing template as we deleted it, it will create new one and change the google_compute_instance_group_manager)
- remove the old template manually or using `curl/gcloud commands`

**Sample code:**
```sh
# content of ./terraform-apply.sh

# get plan as json
terraform plan -out=tf-state
terraform show -json tf-state > tf-state.json
# detech re-create change in google_compute_instance_template
result=$(cat tf-state.json \
| jq '.resource_changes[] | select( .type | contains("google_compute_instance_template"))' \
| jq '. | select(.change.actions[] | contains("create")) | select(.change.actions[] | contains("delete"))' \
| jq -r .address)

# delete recreating google_compute_instance_template state entry in tf-state
if [[ $result ]]; then
IFS=" "
for line in $result; do
    printf "Deleting State for: $line\n" && terraform state rm $line || exit 1;
done
fi
# terraform apply
terraform apply $@
# curl/gcloud command to remove the old google_compute_instance_template
```

##### Best/Easy/Final Solution

Use `create_before_destroy` lifecycle meta-argument for `google_compute_instance_template` like
```hcl
resource "google_compute_instance_template" "instance_template" {
    lifecycle {
        create_before_destroy = true
    }
}
```

During apply, it will **first create** new google_compute_instance_template, then change google_compute_instance_group_manager (and all other resources which depends on it) and then finally delete the old google_compute_instance_template

#### References
- https://www.terraform.io/docs/language/meta-arguments/lifecycle.html#create_before_destroy
- https://github.com/hashicorp/terraform/issues/6234#event-2551997957
- https://registry.terraform.io/providers/hashicorp/google/latest/docs/resources/compute_instance_template#using-with-instance-group-manager
