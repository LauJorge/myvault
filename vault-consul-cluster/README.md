# Private Vault Cluster Example

This folder shows an example of Terraform code to deploy a [Vault](https://www.vaultproject.io/) cluster in
[AWS](https://aws.amazon.com/) using the [vault-cluster module]. The Vault cluster uses
[Consul](https://www.consul.io/) as a storage backend, so this example also deploys a separate Consul server cluster
using the [consul-cluster module]
from the Consul AWS Module.

Each of the servers in this example has [Dnsmasq](http://www.thekelleys.org.uk/dnsmasq/doc.html) installed 
which allows it to use the Consul server cluster for service discovery and thereby access Vault via DNS using the
domain name `vault.service.consul`.

![Vault architecture](https://github.com/hashicorp/terraform-aws-vault/blob/master/_docs/architecture.png?raw=true)


## Quick start

To deploy a Vault Cluster:

1. `git clone` this repo to your computer.
1. Run `terraform init`.
1. Run `terraform apply`.
1. Run the [vault-examples-helper.sh script] to
   print out the IP addresses of the Vault servers and some example commands you can run to interact with the cluster:
   `../helper.sh`.
