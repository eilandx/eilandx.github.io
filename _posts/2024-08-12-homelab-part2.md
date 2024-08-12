--- 
title: Configuring a homelab using Terraform and Ansible on Proxmox and Kubernetes - Part 2
date: 2024-08-12
categories: [Guide, Tutorial]
tags: [proxmox, terraform, homelab, dns]
author: jvekeman
pin: true
---

In this short guide, we will continue setting up our homelab using Terraform and Ansible. In the previous part, we set up a Proxy and DNS server on a RPI. In this part, we will provision the DNS records for our homelab using Terraform.

# Part 2: Provisioning DNS records using Terraform

## Prerequisites

Before starting, make sure you have the following:
- A workstation with Terraform installed
- Access to the RPI with the DNS server from part 1

## The Terraform environment

Terraform allows you to define your infrastructure as code. This means that you can define your infrastructure in a declarative way, and Terraform will make sure that the infrastructure matches the desired state. Since there is no good or bad way to structure your Terraform code, we will use following structure.

### Project structure

```txt
backend/
├── plan
├── terraform.tfstate
└── terraform.tfstate.backup
variables/
└── adguard.tfvars
dns.tf
provider.tf
variables.tf
```

## Provider configuration

First, we need to configure the provider. In this case, we will use the [AdGuard provider](https://registry.terraform.io/providers/gmichels/adguard/latest/docs) to manage our DNS records. 

```hcl
terraform {
  required_providers {
    adguard = {
      source = "gmichels/adguard"
      version = "1.3.0"
    }
  }
  backend "local" {
    path = "backend/terraform.tfstate"
  }
}

provider "adguard" {
  host     = var.adguard_host
  username = var.adguard_username
  password = var.adguard_password
  scheme   = "http"
  timeout  = 5 
}
```

The provider configuration allows us to connect to the AdGuard API. The backend configuration specifies that the state file should be stored locally in the `backend/` directory. The `terraform.tfstate` file contains the current state of the infrastructure, this is the only source of truth for Terraform. Terraform will match the current configuration with the state file and make the necessary changes to the infrastructure. This file should be stored in a secure location, as it contains sensitive information and should never be deleted or modified manually.

## Variables

### Variables.tf file

Start by creating a `variables.tf` file in the root directory. This file contains all the variables that are used in our Terraform configuration.

```hcl
variable "adguard_host" {
  type        = string
  description = "The URL of the AdGuard server"
}

variable "adguard_username" {
  type        = string
  description = "The username for the AdGuard server"
}

variable "adguard_password" {
  type        = string
  description = "The password for the AdGuard server"
}
```

As you can see, we define three variables: `adguard_host`, `adguard_username`, and `adguard_password`. These variables are used to configure the AdGuard provider but don't contain any sensitive information. The sensitive information is stored in a separate file, `adguard.tfvars`, which is not checked into version control.

### Adguard.tfvars file

Create a `adguard.tfvars` file in the `variables/` directory. This file contains the sensitive information that is used to configure the AdGuard provider.

```hcl
adguard_host=<YOUR_ADGUARD_HOST>
adguard_username=<YOUR_ADGUARD_USERNAME>
adguard_password=<YOUR_ADGUARD_PASSWORD>
```

## Defining the DNS records

Now that we can connect to the AdGuard API, we can define the DNS records for our homelab. Create a `dns.tf` file in the root directory of the project. Since we use a Proxy server in our homelab, we only need to define one DNS record. 

```hcl
# manage a DNS rewrite rule
resource "adguard_rewrite" "wildcard_domain" {
  domain = "*.<YOUR_DOMAIN>"
  answer = "<PROXY_IP>"
}
```

This configuration will create a DNS rewrite rule that redirects all requests to the Proxy server. The proxy server will then forward the requests to the correct service in the homelab.

> Note: Replace `<YOUR_DOMAIN>` with your domain and `<PROXY_IP>` with the IP address of your Proxy server.
{: .prompt-tip }

## Applying the configuration

Now that we have defined the DNS records, we can apply the configuration. Run the following commands in the root directory of the project.

```bash
terraform init 
```

This command initializes the Terraform project and downloads the necessary providers.

```bash
terraform plan -var-file=variables/adguard.tfvars -out=backend/plan
```

This command generates an execution plan that shows what Terraform will do when you apply the configuration. Review the plan to make sure that it matches your expectations. It also saves the plan to a file so that you can apply it later.

```bash
terraform apply backend/plan
```

This command applies the configuration and creates the DNS records in the AdGuard server. Terraform will update the state file with the new configuration.

### Changing ~/.bashrc

To make it easier to run the Terraform commands, you can add the following aliases to your `~/.bashrc` file.

```bash
alias tp='terraform plan -var-file="variables/adguard.tfvars" -out=backend/plan'
alias ta='terraform apply backend/plan'
```

> Note: Don't forget to run `source ~/.bashrc` after adding the aliases to apply the changes.
{: .prompt-tip }

### Verifying the DNS records

You can verify that the DNS records have been created by logging into the AdGuard web interface. You should see a new DNS rewrite rule that redirects all requests to the Proxy server.

### Testing the infrastructure

To test the infrastructure, you can try to access adguard in your homelab using the domain name. Remember to set the DNS server in your network configuration to the IP address of the RPI with the DNS server.

> Note: the connection should be using https, if you don't have a valid certificate, your browser will show a warning. You can ignore this warning. 
> If your infrastructure is not working as expected, review part 1 and make sure that the Proxy server is correctly configured.
{: .prompt-tip }

## Conclusion

In this part, we provisioned the DNS records for our homelab using Terraform. We defined the DNS rewrite rule that redirects all requests to the Proxy server. In the next part, we will set up VMs on the Proxmox server using Terraform.
