# Vault and Consul AMI

You can use this AMI to deploy a [Vault cluster](https://www.vaultproject.io/) by using the *vault-cluster
module*. This Vault cluster will use Consul as its storage backend, so you can also use the
same AMI to deploy a separate [Consul server cluster](https://www.consul.io/) by using the *consul-cluster
module*.


## Quick start

To build the Vault and Consul AMI:

1. `git clone` this repo to your computer.

1. Install [Packer](https://www.packer.io/).

1. Get certificate variables.

1. Update the `variables` section, with the outputs, on the `vault-consul.json` Packer template.

1. Run `packer build vault-consul.json`.

When the build finishes, it will output the IDs of the new AMIs. 