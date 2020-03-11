---
title: "Fix corrupted Terraform state"
date: 2020-03-11T20:12:28+05:30
---

Lets say you corrupte your terraform state. There are multiple ways in which you
can corrupte it. So I am not gonna tell how we corrupted it.

In order to fix it,
1) You have to remove unneccassary resources from the state.

```bash
terraform state rm resource.resource_id
#eg: terraform state rm google_container_node_pool.node_pool
#eg: terraform state rm google_container_cluster.cluster
```

2) You have to import the actual state into the state file (Make sure import is available for your resource)

```bash
terraform import resource.resource_id resource_specification
#eg: terraform import google_container_node_pool.node_pool <gcp-project>/<region>/<cluster-name>/<node-pool-name>
#eg: terraform import module.gke_cluster.google_container_cluster.cluster <gcp-project>/<region>/<cluster-name>

```

> Note: We can manually edit the state file and fix it. You can do that only
when you have permission to the remote backend(if you are having one) and when you are expert in understanding the state file.