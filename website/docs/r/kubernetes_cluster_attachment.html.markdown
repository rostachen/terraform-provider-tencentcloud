---
subcategory: "Tencent Kubernetes Engine(TKE)"
layout: "tencentcloud"
page_title: "TencentCloud: tencentcloud_kubernetes_cluster_attachment"
sidebar_current: "docs-tencentcloud-resource-kubernetes_cluster_attachment"
description: |-
  Provide a resource to attach an existing  cvm to kubernetes cluster.
---

# tencentcloud_kubernetes_cluster_attachment

Provide a resource to attach an existing  cvm to kubernetes cluster.

## Example Usage

```hcl
variable "availability_zone" {
  default = "ap-guangzhou-3"
}

variable "cluster_cidr" {
  default = "172.16.0.0/16"
}

variable "default_instance_type" {
  default = "S1.SMALL1"
}

data "tencentcloud_images" "default" {
  image_type = ["PUBLIC_IMAGE"]
  os_name    = "centos"
}

data "tencentcloud_vpc_subnets" "vpc" {
  is_default        = true
  availability_zone = var.availability_zone
}

data "tencentcloud_instance_types" "default" {
  filter {
    name   = "instance-family"
    values = ["SA2"]
  }

  cpu_core_count = 8
  memory_size    = 16
}

resource "tencentcloud_instance" "foo" {
  instance_name     = "tf-auto-test-1-1"
  availability_zone = var.availability_zone
  image_id          = data.tencentcloud_images.default.images.0.image_id
  instance_type     = var.default_instance_type
  system_disk_type  = "CLOUD_PREMIUM"
  system_disk_size  = 50
}

resource "tencentcloud_kubernetes_cluster" "managed_cluster" {
  vpc_id                  = data.tencentcloud_vpc_subnets.vpc.instance_list.0.vpc_id
  cluster_cidr            = "10.1.0.0/16"
  cluster_max_pod_num     = 32
  cluster_name            = "keep"
  cluster_desc            = "test cluster desc"
  cluster_max_service_num = 32

  worker_config {
    count                      = 1
    availability_zone          = var.availability_zone
    instance_type              = var.default_instance_type
    system_disk_type           = "CLOUD_SSD"
    system_disk_size           = 60
    internet_charge_type       = "TRAFFIC_POSTPAID_BY_HOUR"
    internet_max_bandwidth_out = 100
    public_ip_assigned         = true
    subnet_id                  = data.tencentcloud_vpc_subnets.vpc.instance_list.0.subnet_id

    data_disk {
      disk_type = "CLOUD_PREMIUM"
      disk_size = 50
    }

    enhanced_security_service = false
    enhanced_monitor_service  = false
    user_data                 = "dGVzdA=="
    password                  = "ZZXXccvv1212"
  }

  cluster_deploy_type = "MANAGED_CLUSTER"
}

resource "tencentcloud_kubernetes_cluster_attachment" "test_attach" {
  cluster_id  = tencentcloud_kubernetes_cluster.managed_cluster.id
  instance_id = tencentcloud_instance.foo.id
  password    = "Lo4wbdit"

  labels = {
    "test1" = "test1",
    "test2" = "test2",
  }

  worker_config_overrides {
    desired_pod_num = 8
  }
}
```

## Argument Reference

The following arguments are supported:

* `cluster_id` - (Required, ForceNew) ID of the cluster.
* `instance_id` - (Required, ForceNew) ID of the CVM instance, this cvm will reinstall the system.
* `hostname` - (Optional, ForceNew) The host name of the attached instance. Dot (.) and dash (-) cannot be used as the first and last characters of HostName and cannot be used consecutively. Windows example: The length of the name character is [2, 15], letters (capitalization is not restricted), numbers and dashes (-) are allowed, dots (.) are not supported, and not all numbers are allowed. Examples of other types (Linux, etc.): The character length is [2, 60], and multiple dots are allowed. There is a segment between the dots. Each segment allows letters (with no limitation on capitalization), numbers and dashes (-).
* `key_ids` - (Optional, ForceNew) The key pair to use for the instance, it looks like skey-16jig7tx, it should be set if `password` not set.
* `labels` - (Optional, ForceNew) Labels of tke attachment exits CVM.
* `password` - (Optional, ForceNew) Password to access, should be set if `key_ids` not set.
* `unschedulable` - (Optional, ForceNew) Sets whether the joining node participates in the schedule. Default is '0'. Participate in scheduling.
* `worker_config_overrides` - (Optional, ForceNew) Override variable worker_config, commonly used to attach existing instances.
* `worker_config` - (Optional, ForceNew) Deploy the machine configuration information of the 'WORKER', commonly used to attach existing instances.

The `data_disk` object supports the following:

* `auto_format_and_mount` - (Optional, ForceNew) Indicate whether to auto format and mount or not. Default is `false`.
* `disk_size` - (Optional, ForceNew) Volume of disk in GB. Default is `0`.
* `disk_type` - (Optional, ForceNew) Types of disk, available values: `CLOUD_PREMIUM` and `CLOUD_SSD`.
* `file_system` - (Optional, ForceNew) File system, e.g. `ext3/ext4/xfs`.
* `mount_target` - (Optional, ForceNew) Mount target.

The `worker_config_overrides` object supports the following:

* `data_disk` - (Optional, ForceNew) Configurations of data disk.
* `desired_pod_num` - (Optional, ForceNew) Indicate to set desired pod number in node. valid when the cluster is podCIDR.
* `docker_graph_path` - (Optional, ForceNew) Docker graph path. Default is `/var/lib/docker`.
* `extra_args` - (Optional, ForceNew) Custom parameter information related to the node. This is a white-list parameter.
* `is_schedule` - (Optional, ForceNew) Indicate to schedule the adding node or not. Default is true.
* `mount_target` - (Optional, ForceNew) Mount target. Default is not mounting.
* `user_data` - (Optional, ForceNew) Base64-encoded User Data text, the length limit is 16KB.

The `worker_config` object supports the following:

* `data_disk` - (Optional, ForceNew) Configurations of data disk.
* `desired_pod_num` - (Optional, ForceNew) Indicate to set desired pod number in node. valid when the cluster is podCIDR.
* `docker_graph_path` - (Optional, ForceNew) Docker graph path. Default is `/var/lib/docker`.
* `extra_args` - (Optional, ForceNew) Custom parameter information related to the node. This is a white-list parameter.
* `is_schedule` - (Optional, ForceNew) Indicate to schedule the adding node or not. Default is true.
* `mount_target` - (Optional, ForceNew) Mount target. Default is not mounting.
* `user_data` - (Optional, ForceNew) Base64-encoded User Data text, the length limit is 16KB.

## Attributes Reference

In addition to all arguments above, the following attributes are exported:

* `id` - ID of the resource.
* `security_groups` - A list of security group IDs after attach to cluster.
* `state` - State of the node.


