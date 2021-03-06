# Rancher vSphere Terraform Module

Terraform module for provisioning Kubernetes clusters on vSphere from VM templates using [Rancher](https://rancher.com/).

## Usage

```hcl
module "rancher_cluster_separate_control_plane_etcd_global_spec" {
  source = "../.."

  cloud_credential_name     = "MyVsphereCredentials"
  cluster_name              = "tf_test"
  cluster_description       = "Terraform test Rancher K8s cluster"
  enable_monitoring         = true
  enable_alerting           = false
  enable_istio              = false
  kubernetes_network_plugin = "canal"
  kubernetes_version        = "v1.17.4-rancher1-1"

  private_registries_spec = {
    privreg1 = {
      url        = "myreg1.mydomain.com"
      user       = "myuser"
      password   = "mysecretpass"
      is_default = true
    }
    privreg2 = {
      url = "myreg2.mydomain.com"
    }
  }

  cloud_provider_spec = {
    global_insecure_flag = false
    virtual_center_spec = {
      myvcenter = {
        name                 = "myvcenter.mydomain.com"
        user                 = "myvcuser"
        password             = "mysecretpass"
        datacenters          = "/MyDC"
        port                 = 443
        soap_roundtrip_count = 0
      }
    }
    workspace_server            = "myvcenter.mydomain.com"
    workspace_datacenter        = "/MyDC"
    workspace_folder            = "/MyDC/vm/MyFolder"
    workspace_default_datastore = "MyDatastore"
    workspace_resourcepool_path = "/MyDC/host/MyCluster/Resources/MyResourcePool"
    disk_scsi_controller_type   = "pvscsi"
    network_public_network      = "MyPortgroup"
  }

  node_datacenter              = "MyDC"
  node_datastore               = "MyDatastore"
  node_cluster                 = "MyCluster"
  node_resource_pool           = "MyResourcePool"
  node_folder                  = "MyFolder"
  node_portgroup               = "MyPortgroup"
  node_template_ssh_user       = "root"
  node_template_ssh_password   = "MySecretPass"
  node_template_ssh_user_group = "root"
  node_cloud_config            = file("cloud-config.yml")

  node_spec = {
    control_plane = {
      vsphere_template = "MyFolder/k8s-control_plane"
      num_vcpu         = 2
      memory_gb        = 4
      disk_gb          = 20
    }
    etcd = {
      vsphere_template = "MyFolder/k8s-etcd"
      num_vcpu         = 1
      memory_gb        = 2
      disk_gb          = 20
    }
    worker = {
      vsphere_template = "MyFolder/k8s-worker"
      num_vcpu         = 4
      memory_gb        = 8
      disk_gb          = 20
    }
  }

  control_plane_node_pool_name = "tf-control-plane"
  control_plane_node_prefix    = "tf-control-plane-"
  control_plane_node_quantity  = 2

  etcd_node_pool_name = "tf-etcd"
  etcd_node_prefix    = "tf-etcd-"
  etcd_node_quantity  = 3

  worker_node_pool_name = "tf-worker"
  worker_node_prefix    = "tf-worker-"
  worker_node_quantity  = 3
}
```


See `examples` directory for more usage examples.

## Requirements

| Name | Version |
|------|---------|
| terraform | >= 0.12.12 |
| rancher2 | >= 1.7.2 |

## Providers

| Name | Version |
|------|---------|
| rancher2 | >= 1.7.2 |

## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| all\_in\_one\_node\_pool\_name | Name of the all-in-one node pool | `string` | `"all-in-one"` | no |
| all\_in\_one\_node\_prefix | Prefix for node created in the all-in-one node pool | `string` | `"all-in-one-"` | no |
| cloud\_credential\_name | Name of vSphere cloud credential | `string` | n/a | yes |
| cloud\_provider\_spec | Specification of vSphere cloud provider, which is necessary to allow dynamic provisioning of volumes. Take a look at the `examples` directory for synthax and Rancher vSphere Cloud Provider documentation for explanation of parameters | `map` | `{}` | no |
| cluster\_description | Cluster description | `string` | n/a | yes |
| cluster\_name | Cluster name | `string` | n/a | yes |
| control\_plane\_node\_pool\_name | Name of the control plane node pool | `string` | `"control-plane"` | no |
| control\_plane\_node\_prefix | Prefix for nodes created in control plane node pool | `string` | `"control-plane-"` | no |
| control\_plane\_node\_quantity | Number of nodes in control plane node pool | `number` | `null` | no |
| enable\_alerting | Whether to enable cluster alerting | `bool` | `false` | no |
| enable\_istio | Whether to enable Istio for the cluster | `bool` | `false` | no |
| enable\_monitoring | Whether to enable cluster monitoring | `bool` | `true` | no |
| etcd\_node\_pool\_name | Name of the etcd node pool | `string` | `"etcd"` | no |
| etcd\_node\_prefix | Prefix for nodes created in etcd node pool | `string` | `"etcd-"` | no |
| etcd\_node\_quantity | Number of nodes in etcd node pool | `number` | `null` | no |
| kubernetes\_network\_plugin | Kubernetes network plugin to use, one of `canal` (default), `flannel`, `calico`, `weave` | `string` | `"canal"` | no |
| kubernetes\_version | Kubernetes version to deploy | `string` | `null` | no |
| master\_node\_pool\_name | Name of the master (consolidated control plane and etcd) node pool | `string` | `"master"` | no |
| master\_node\_prefix | Prefix for nodes created in master (consolidated control plane and etcd) node pool | `string` | `"master-"` | no |
| master\_node\_quantity | Number of nodes in master (consolidated control plane and etcd) node pool | `number` | `null` | no |
| node\_cloud\_config | Global setting for the contents of cloud-config YAML file which will be applied on all cluster nodes; these contents should be defined in the root module either inline using the `heredoc` synthax, or loaded from a file using Terraform's `file()` function. `cloud_config` parameter inside `node_spec` overrides this global setting on a per node role basis; if neither are set, cloud-config file for the node will have content of `#cloud-config` (which effectively makes it empty) | `string` | `null` | no |
| node\_cluster | Global setting for vSphere cluster in which to create all cluster nodes; either this or `cluster` parameter inside `node_spec` (which overrides it) need to be set | `string` | `null` | no |
| node\_datacenter | Global setting for vSphere datacenter in which to create all cluster nodes; either this or `datacenter` parameter inside `node_spec` (which overrides it) need to be set | `string` | `null` | no |
| node\_datastore | Global setting for vSphere datastore in which to create all cluster nodes; either this or `datastore` parameter inside `node_spec` (which overrides it) need to be set | `string` | `null` | no |
| node\_folder | Global setting for vSphere VM and template folder in which to create all cluster nodes; `folder` parameter inside `node_spec` overrides this global setting; if neither are set, nodes will be created in datacenter root | `string` | `null` | no |
| node\_network\_protocol\_profile\_addressing | (Requires minimum Rancher v2.3.6) Whether to use node portgroup's network protocol profile to transfer network properties such as IP address, subnet mask, default gateway, DNS servers, DNS search path and DNS domain to the node through its vApp properties. In order for the transferred properties to be actually configured in the OS, `cloud_config` can be used to read the vApp properties through VMware Tools and configure the OS network stack. The following vApp properties are read from the network protocol profile and transferred to node VMs: `guestinfo.dns.servers` (DNS servers specified in the network protocol profile), `guestinfo.dns.domain` (domain name), `guestinfo.dns.searchpath` (DNS search path), `guestinfo.interface.0.ip.0.address` (assigned IP address from the IP pool), `guestinfo.interface.0.ip.0.netmask` (subnet mask of the assigned IP address) and `guestinfo.interface.0.route.0.gateway`(default gateway). If set to `false` (default) no vApp properties are configured and cluster nodes will use DHCP assigned addresses | `bool` | `false` | no |
| node\_portgroup | Global setting for vSphere portgroup to which to connect all cluster nodes; either this or `portgroup` parameter inside `node_spec` (which overrides it) need to be set | `string` | `null` | no |
| node\_resource\_pool | Global setting for vSphere resource pool in which to create all cluster nodes; `resource_pool` parameter inside `node_spec` overrides this global setting; if neither are set, nodes will be created in cluster root | `string` | `null` | no |
| node\_spec | Specification of node templates for each of the node roles. Available roles are `control_plane`, `etcd`, `master` (consolidated `control_plane` and `etcd`), `worker` and `all_in_one` (`control_plane`, `etcd` and `worker` consolidate on one node, used for creating single node clusters). `node_spec` allows for specifying parameters such as vSphere template, datacenter, cluster etc. on a node role basis. If these parameters are set both through `node_spec` and globally through `node_*`, `node_spec` values will have precedence. As a minimum, each node role needs to have the following inputs set in `node_spec`: `num_vcpu` (VM number of vCPUs), `memory_gb` (VM memory in GB) and `disk_gb` (VM disk size in GB) - all other values can be inherited from global variables. Take a look at the `examples` directory for detailed synthax | `any` | n/a | yes |
| node\_template\_ssh\_password | Global setting for the SSH password for Rancher to connect to all cluster nodes after deployment from template; either this or `template_ssh_password` parameter inside `node_spec` (which overrides it) need to be set | `string` | `null` | no |
| node\_template\_ssh\_user | Global setting for the SSH user for Rancher to connect to all cluster nodes after deployment from template; either this or `template_ssh_user` parameter inside `node_spec` (which overrides it) need to be set | `string` | `null` | no |
| node\_template\_ssh\_user\_group | Global setting for the user group to which Rancher will chown the uploaded keys on all cluster nodes; either this or `template_ssh_user_group` parameter inside `node_spec` (which overrides it) need to be set | `string` | `null` | no |
| node\_vsphere\_template | Global setting for vSphere template from which to create all cluster nodes; either this or `vsphere_template` parameter inside `node_spec` (which overrides it) need to be set | `string` | `null` | no |
| private\_registries\_spec | Specification of private registries for Docker images. Multiple registries can be specified, take a look at the `examples` directory for synthax. Only the `url` parameter is mandatory. If you set password access to registry, for future Terraform runs have in mind that Rancher API doesn't return registry password, so every Terraform operation will offer to change the cluster resource by adding the registry password field even if you haven't done any changes in your spec | `map` | `{}` | no |
| single\_node\_cluster | Whether to create a single node cluster with all roles consolidated on one node | `bool` | `false` | no |
| worker\_node\_pool\_name | Name of the worker node pool | `string` | `"worker"` | no |
| worker\_node\_prefix | Prefix for nodes created in worker node pool | `string` | `"worker-"` | no |
| worker\_node\_quantity | Number of nodes in worker node pool | `number` | `null` | no |

## Outputs

| Name | Description |
|------|-------------|
| id | ID of the created cluster |
| kube\_config | kubeconfig of the created cluster |




