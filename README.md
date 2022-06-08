# Equinix Metal Virtual Routing and Forwarding (VRF)
[![Experimental](https://img.shields.io/badge/Stability-Experimental-red.svg)](https://github.com/equinix-labs/standards#about-uniform-standards)
[![terraform](https://github.com/equinix-labs/terraform-metal-vrf/actions/workflows/integration.yaml/badge.svg)](https://github.com/equinix-labs/terraform-metal-vrf/actions/workflows/integration.yaml)


# VRF deployment on Equinix Platform

This Terraform script provides VRF deployments on Equinix Metal platform where a Metal Gateway, a VRF and a number of metal nodes are deployed. The metal VRF is connected to a pair of far-end (or colo) edge devices via a pair of redundant Virtual Connections (VC) created in a pair of dedicated fabric ports (see high-level diagram below). The VRF is used to establish BGP sessions with the colo network devices and advertise the specified IP routes to the colo.

<img width="1202" alt="Screen Shot 2022-05-28 at 4 33 37 PM" src="https://user-images.githubusercontent.com/46980377/170843873-bdd78ee1-4778-435b-be18-08b31ecc6f1b.png">

For information regarding Metal Gateway and VRF, please see the following document - https://metal.equinix.com/developers/docs/networking/metal-gateway/, https://metal.equinix.com/developers/api/vrfs/ For the Layer-2 bonded mode, please see the following Equinix Metal document - https://metal.equinix.com/developers/docs/layer2-networking/layer2-mode/#pure-layer-2-modes

The Metal Gateway and the Metal nodes shared a single VLAN: the VLAN is hardcoded using 192.168.100.0/24 for IP assignments with Metal Gateway being assigned with 192.168.100.1, the first metal node being assigned with 192.168.100.2, the second metal node being assigned with 192.168.100.3 etc. 

In order to establish the BGP sessions, you'll need to setup the fabric virtual connections on your colo network devices and perform the BGP configurations too.

We recommend the following steps to be performed BEFORE runing this script:

Step 1 - Plan your setup. Include your BGP neighbor IPs, IP subnet for metal gateway and metal nodes, Metal VRF ASN (use private ASN space, your network ASN, UUID of your dedicated Metal fabric port (from Metal's portal)
Step 2 - In Equini fabric portal, create a pair of redundant virtual connections using your dedicated fabric port to your colo network devices
Step 3 - Perform BGP setups on your colo network edge devices

Please note, DO NOT manually setup vritual connections (VC) using your Metal's dedicted fabric port via Metal's portal. This script will setup the VCs and BGP sessions etc. on Metal side.

After the nodes are sucessfully deployed, the following behaviors are expected:
1. A Metal node can reach to the metal gateway via the gateway's IP 192.168.100.1
2. Metal nodes can reach to each anoter via their IPs (192.168.100.x)
3. A Metal node can reach to the VRF's BGP IP (for example, 169.254.100.1)
4. A Metal node can reach to the colo device's BGP IP (for example, 169.254.100.2)
5. Metal nodes and your colo servers can reah to reach other, if you have servers setup behind your colo network devices and advertise routes via the BGP sessions established between your network devices and Metal VRF.

This repository is [Experimental](https://github.com/packethost/standards/blob/master/experimental-statement.md) meaning that it's based on untested ideas or techniques and not yet established or finalized or involves a radically new and innovative style! This means that support is best effort (at best!) and we strongly encourage you to NOT use this in production.

## Install Terraform

Terraform is just a single binary.  Visit their [download page](https://www.terraform.io/downloads.html), choose your operating system, make the binary executable, and move it into your path.

Here is an example for **macOS**:

```bash
curl -LO https://releases.hashicorp.com/terraform/0.12.18/terraform_0.12.18_darwin_amd64.zip
unzip terraform_0.12.18_darwin_amd64.zip
chmod +x terraform
sudo mv terraform /usr/local/bin/
```

## Download this project

To download this project, run the following command:

```bash
git clone https://github.com/equinix-labs/terraform-metal-vrf.git
cd terraform-metal-vrf
```

## Initialize Terraform

Terraform uses modules to deploy infrastructure. In order to initialize the modules you simply run: `terraform init`. This should download modules into a hidden directory `.terraform`

## Modify your variables

See `variables.tf` for a description of each variable. You will need to set the `auth_token` and `project_id` variables at a minimum:

```
cp terraform.tfvars.example terraform.tfvars
vim terraform.tfvars
```

#### Note - Currently only Ubuntu is supported; Only Gen# 3 server plans support hybrid bonded mode.

## Deploy terraform template

```bash
terraform apply --auto-approve
```

Once this is complete you should get output similar to this:

```console
Apply complete! Resources: 31 added, 0 changed, 0 destroyed.

Outputs:

```
