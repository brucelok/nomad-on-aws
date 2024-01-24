# Provision a Nomad cluster on AWS

## Pre-requisites

To get started, create the following:

- AWS account
- [API access keys](http://aws.amazon.com/developers/access-keys/)
- [SSH key pair](http://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-key-pairs.html)

## Set the AWS environment variables

```bash
$ export AWS_ACCESS_KEY_ID=[AWS_ACCESS_KEY_ID]
$ export AWS_SECRET_ACCESS_KEY=[AWS_SECRET_ACCESS_KEY]
```

## Build an AWS machine image with Packer

[Packer](https://www.packer.io/intro/index.html) is HashiCorp's open source tool 
for creating identical machine images for multiple platforms from a single 
source configuration. The Terraform templates included in this repo reference a 
publicly available Amazon machine image (AMI) by default. The AMI can be customized 
through modifications to the [build configuration script](../shared/scripts/setup.sh) 
and [packer.json](packer.json).

Use the following command to build the AMI:

```bash
$ packer build packer.json
```

## Prepare variables for Terraform build

Update `terraform.tfvars` with your SSH key name and your AMI ID (if you created 
a custom AMI), `region`, `instance_type`, `server_count`, and `client_count` variables
as appropriate. At least one client and one server are required. You can 
optionally replace the Nomad binary at runtime by adding the `nomad_binary` 
variable like so:

```bash
name = "nomad"
key_name = "myMacbook"
nomad_binary = "https://releases.hashicorp.com/nomad/1.5.1/nomad_1.5.1_linux_amd64.zip"
region = "ap-southeast-2"
ami = "ami-0a2a058201b0c3486"
server_instance_type = "t2.small"
server_count         = "3"
client_instance_type = "t2.medium"
client_count         = "2"
```

By default, the infrastructure that is provisioned for this test environment is configured to 
allow all traffic over port 22. You can add the `whitelist_ip` variable in the terraform.tfvars which only allows your IP address. For example
```
whitelist_ip = "120.200.10.6/32"
```

## Provision the cluster

```bash
$ terraform init
$ terraform get
$ terraform plan
$ terraform apply
```

## Access the cluster

SSH to one of the servers using its public IP:

```bash
$ ssh -i /path/to/private/key ubuntu@PUBLIC_IP
```

## Next Steps

Verify the Nomad and Consul clusters with the following commands
```bash
nomad server members
nomad node status
consul members
consul operator raft list-peers
```

Then you can start to run sample job
```bash
nomad job init -connect
```
