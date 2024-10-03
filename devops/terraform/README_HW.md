# Prepare install
## Open WSL (Ubuntu) terminal on Windows of VS Code. 
![alt text](image-1.png)

## Install Terraform
```
$ curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo gpg --dearmor -o /usr/share/keyrings/hashicorp-archive-keyring.gpg
$ sudo apt update && sudo apt install terraform
$ terraform --version
Terraform v1.9.7
on linux_amd64
...
```

## Install Azure CLI
```
$ curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
```

##  Azure CLI logon
```
$ az login
```
Copy printed URL Link and open it on browser for auth in Azure Portal. After success login, 'az login' should finish work in console automatically.

## Install required packages for scripts.

```
$ sudo apt install jq
```
## Prepare default terraform.tfvars
```
$ cd devops\terraform
$ cp terraform.tfvars.template terraform.tfvars
```

# Prepare data for fill devops\terraform\terraform.tfvars

## ./setup_grafana_datasources.sh
```
$ cd devops/terraform
$ chmod +x ./setup_grafana_datasources.sh
$ source ./setup_grafana_datasources.sh

Using Subscription ID: ......
Infinity datasource credentials not found. Creating new ones.
...
TF_VAR_azure_client_secret: [Hidden for security. Use the TF_VAR_azure_client_secret environment variable]
TF_VAR_infinity_client_secret: [Hidden for security. Use the TF_VAR_infinity_client_secret environment variable]

$ echo $TF_VAR_azure_client_secret
vGt......................

echo $TF_VAR_infinity_client_secret
Ez-8.....................

```


## ssh keygen

Only RSA supported (bastion):

    Error: - the provided ssh-ed25519 SSH key is not supported. Only RSA SSH keys are supported by Azure

```
$ ssh-keygen
Generating public/private rsa key pair
...
The key's randomart image is:
+---[RSA 3072]----+
|       o+*+ +.  .|
|       +o+*o o ..|
|      + O+=+. o. |
|     . XoE.=.oo  |
|      =.S.. +o . |
|     . o.o .  .  |
|      o +        |
|     . =         |
|      o          |
+----[SHA256]-----+
```
$ cat ~/.ssh/id_rsa.pub
```
ssh-rsa AAAAB3NzaC1yc2EAAAADAQA....
```

## Fill data
Open in editor `terraform.tfvars` file and fill previously printed data or environments.
For print environment variables can use:
```
$ echo $ENVIRONMENT_VAR
```
ATTENTION! STORAGE NAME HAS LIMIT up to 24 chars

`project_name` must be not longer than:
`st${var.project_name}${var.environment}` without dash char.

So, for example, my error was:

`project_name = itmarathon-epam-2024-lexxai-prod`

```
Error: name ("stitmarathonepam2024lexxaiprod") can only consist of lowercase letters and numbers, and must be between 3 and 24 characters long
│
│   with module.storage.azurerm_storage_account.storage,
│   on modules/05_storage/main.tf line 3, in resource "azurerm_storage_account" "storage":
│    3:   name                     = lower(replace("st${var.project_name}${var.environment}", "-", ""))
```

# terraform 
## terraform init
```
$ terraform init
Initializing the backend...
Initializing modules...
- app_dotnet in modules/07_app_dotnet
- app_python in modules/09_app_python
- bastion in modules/03_bastion
- database in modules/04_database
- email in modules/08_email
....
```
![terraform installed](image.png)
## Plan and Apply the Infrastructure by Modules
### 1. Network (01_network)

```
$ terraform plan -var-file=terraform.tfvars -target=module.network
```
<details>
  <summary>Click to expand result of command</summary>

```
Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # module.network.azurerm_private_dns_zone.mysql will be created
  + resource "azurerm_private_dns_zone" "mysql" {
      + id                                                    = (known after apply)
      + max_number_of_record_sets                             = (known after apply)
      + max_number_of_virtual_network_links                   = (known after apply)
      + max_number_of_virtual_network_links_with_registration = (known after apply)
      + name                                                  = "privatelink.mysql.database.azure.com"
      + number_of_record_sets                                 = (known after apply)
      + resource_group_name                                   = "rg-itmarathon-epam-2024-lexxai-prod"

      + soa_record (known after apply)
    }

  # module.network.azurerm_private_dns_zone_virtual_network_link.mysql will be created
  + resource "azurerm_private_dns_zone_virtual_network_link" "mysql" {
      + id                    = (known after apply)
      + name                  = "mysqldnslink"
      + private_dns_zone_name = "privatelink.mysql.database.azure.com"
      + registration_enabled  = false
      + resource_group_name   = "rg-itmarathon-epam-2024-lexxai-prod"
      + virtual_network_id    = (known after apply)
    }

  # module.network.azurerm_public_ip.public_ips["bastion"] will be created
  + resource "azurerm_public_ip" "public_ips" {
      + allocation_method       = "Dynamic"
      + ddos_protection_mode    = "VirtualNetworkInherited"
      + fqdn                    = (known after apply)
      + id                      = (known after apply)
      + idle_timeout_in_minutes = 4
      + ip_address              = (known after apply)
      + ip_version              = "IPv4"
      + location                = "northeurope"
      + name                    = "pip-bastion-itmarathon-epam-2024-lexxai-prod"
      + resource_group_name     = "rg-itmarathon-epam-2024-lexxai-prod"
      + sku                     = "Basic"
      + sku_tier                = "Regional"
    }

  # module.network.azurerm_public_ip.public_ips["monitoring"] will be created
  + resource "azurerm_public_ip" "public_ips" {
      + allocation_method       = "Dynamic"
      + ddos_protection_mode    = "VirtualNetworkInherited"
      + fqdn                    = (known after apply)
      + id                      = (known after apply)
      + idle_timeout_in_minutes = 4
      + ip_address              = (known after apply)
      + ip_version              = "IPv4"
      + location                = "northeurope"
      + name                    = "pip-monitoring-itmarathon-epam-2024-lexxai-prod"
      + resource_group_name     = "rg-itmarathon-epam-2024-lexxai-prod"
      + sku                     = "Basic"
      + sku_tier                = "Regional"
    }

  # module.network.azurerm_resource_group.rg will be created
  + resource "azurerm_resource_group" "rg" {
      + id       = (known after apply)
      + location = "northeurope"
      + name     = "rg-itmarathon-epam-2024-lexxai-prod"
    }

  # module.network.azurerm_subnet.bastion_subnet will be created
  + resource "azurerm_subnet" "bastion_subnet" {
      + address_prefixes                               = [
          + "10.0.6.0/24",
        ]
      + default_outbound_access_enabled                = true
      + enforce_private_link_endpoint_network_policies = (known after apply)
      + enforce_private_link_service_network_policies  = (known after apply)
      + id                                             = (known after apply)
      + name                                           = "bastion-subnet-itmarathon-epam-2024-lexxai-prod"
      + private_endpoint_network_policies              = (known after apply)
      + private_endpoint_network_policies_enabled      = (known after apply)
      + private_link_service_network_policies_enabled  = (known after apply)
      + resource_group_name                            = "rg-itmarathon-epam-2024-lexxai-prod"
      + virtual_network_name                           = "vnet-itmarathon-epam-2024-lexxai-prod"
    }

  # module.network.azurerm_subnet.monitoring_subnet will be created
  + resource "azurerm_subnet" "monitoring_subnet" {
      + address_prefixes                               = [
          + "10.0.7.0/24",
        ]
      + default_outbound_access_enabled                = true
      + enforce_private_link_endpoint_network_policies = (known after apply)
      + enforce_private_link_service_network_policies  = (known after apply)
      + id                                             = (known after apply)
      + name                                           = "monitoring-subnet-itmarathon-epam-2024-lexxai-prod"
      + private_endpoint_network_policies              = (known after apply)
      + private_endpoint_network_policies_enabled      = (known after apply)
      + private_link_service_network_policies_enabled  = (known after apply)
      + resource_group_name                            = "rg-itmarathon-epam-2024-lexxai-prod"
      + virtual_network_name                           = "vnet-itmarathon-epam-2024-lexxai-prod"
    }

  # module.network.azurerm_subnet.mysql_subnet will be created
  + resource "azurerm_subnet" "mysql_subnet" {
      + address_prefixes                               = [
          + "10.0.3.0/24",
        ]
      + default_outbound_access_enabled                = true
      + enforce_private_link_endpoint_network_policies = (known after apply)
      + enforce_private_link_service_network_policies  = (known after apply)
      + id                                             = (known after apply)
      + name                                           = "mysql-subnet-itmarathon-epam-2024-lexxai-prod"
      + private_endpoint_network_policies              = (known after apply)
      + private_endpoint_network_policies_enabled      = (known after apply)
      + private_link_service_network_policies_enabled  = (known after apply)
      + resource_group_name                            = "rg-itmarathon-epam-2024-lexxai-prod"
      + service_endpoints                              = [
          + "Microsoft.Storage",
        ]
      + virtual_network_name                           = "vnet-itmarathon-epam-2024-lexxai-prod"

      + delegation {
          + name = "fs"

          + service_delegation {
              + actions = [
                  + "Microsoft.Network/virtualNetworks/subnets/join/action",
                ]
              + name    = "Microsoft.DBforMySQL/flexibleServers"
            }
        }
    }

  # module.network.azurerm_subnet.private_subnet will be created
  + resource "azurerm_subnet" "private_subnet" {
      + address_prefixes                               = [
          + "10.0.5.0/24",
        ]
      + default_outbound_access_enabled                = true
      + enforce_private_link_endpoint_network_policies = (known after apply)
      + enforce_private_link_service_network_policies  = (known after apply)
      + id                                             = (known after apply)
      + name                                           = "private-subnet-itmarathon-epam-2024-lexxai-prod"
      + private_endpoint_network_policies              = (known after apply)
      + private_endpoint_network_policies_enabled      = (known after apply)
      + private_link_service_network_policies_enabled  = (known after apply)
      + resource_group_name                            = "rg-itmarathon-epam-2024-lexxai-prod"
      + virtual_network_name                           = "vnet-itmarathon-epam-2024-lexxai-prod"

      + delegation {
          + name = "app-service-delegation"

          + service_delegation {
              + actions = [
                  + "Microsoft.Network/virtualNetworks/subnets/action",
                ]
              + name    = "Microsoft.Web/serverFarms"
            }
        }
    }

  # module.network.azurerm_subnet.public_subnet will be created
  + resource "azurerm_subnet" "public_subnet" {
      + address_prefixes                               = [
          + "10.0.4.0/24",
        ]
      + default_outbound_access_enabled                = true
      + enforce_private_link_endpoint_network_policies = (known after apply)
      + enforce_private_link_service_network_policies  = (known after apply)
      + id                                             = (known after apply)
      + name                                           = "public-subnet-itmarathon-epam-2024-lexxai-prod"
      + private_endpoint_network_policies              = (known after apply)
      + private_endpoint_network_policies_enabled      = (known after apply)
      + private_link_service_network_policies_enabled  = (known after apply)
      + resource_group_name                            = "rg-itmarathon-epam-2024-lexxai-prod"
      + virtual_network_name                           = "vnet-itmarathon-epam-2024-lexxai-prod"

      + delegation {
          + name = "app-service-delegation"

          + service_delegation {
              + actions = [
                  + "Microsoft.Network/virtualNetworks/subnets/action",
                ]
              + name    = "Microsoft.Web/serverFarms"
            }
        }
    }

  # module.network.azurerm_virtual_network.marathon_virtual_network will be created
  + resource "azurerm_virtual_network" "marathon_virtual_network" {
      + address_space       = [
          + "10.0.0.0/16",
        ]
      + dns_servers         = (known after apply)
      + guid                = (known after apply)
      + id                  = (known after apply)
      + location            = "northeurope"
      + name                = "vnet-itmarathon-epam-2024-lexxai-prod"
      + resource_group_name = "rg-itmarathon-epam-2024-lexxai-prod"
      + subnet              = (known after apply)
    }

Plan: 11 to add, 0 to change, 0 to destroy.

Changes to Outputs:
  + monitoring_vm_public_ip = (known after apply)
  + mysql_subnet_id         = (known after apply)
  + resource_group_name     = "rg-itmarathon-epam-2024-lexxai-prod"
  + vnet_id                 = (known after apply)
╷
│ Warning: Resource targeting is in effect
│
│ You are creating a plan with the -target option, which means that the result of this plan may not represent all of the changes requested by the current configuration.
│
│ The -target option is not for routine use, and is provided only for exceptional situations such as recovering from errors or mistakes, or when Terraform specifically suggests to use it as part 
│ of an error message.
╵

────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── 

Note: You didn't use the -out option to save this plan, so Terraform can't guarantee to take exactly these actions if you run "terraform apply" now.
```
</details>


```
$ terraform apply -var-file=terraform.tfvars -target=module.network
```
<details>
  <summary>Click to expand result of command</summary>


```
Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # module.network.azurerm_private_dns_zone.mysql will be created
  + resource "azurerm_private_dns_zone" "mysql" {
      + id                                                    = (known after apply)
      + max_number_of_record_sets                             = (known after apply)
      + max_number_of_virtual_network_links                   = (known after apply)
      + max_number_of_virtual_network_links_with_registration = (known after apply)

    .....

    Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

module.network.azurerm_resource_group.rg: Creating...
module.network.azurerm_resource_group.rg: Creation complete after 9s [id=/subscriptions/................/resourceGroups/rg-itmarathon-epam-2024-lexxai-prod]
module.network.azurerm_private_dns_zone.mysql: Creating...
module.network.azurerm_public_ip.public_ips["monitoring"]: Creating...
module.network.azurerm_public_ip.public_ips["bastion"]: Creating...
module.network.azurerm_virtual_network.marathon_virtual_network: Creating...
module.network.azurerm_public_ip.public_ips["monitoring"]: Creation complete after 3s [id=/subscriptions/................/resourceGroups/rg-itmarathon-epam-2024-lexxai-prod/providers/Microsoft.Network/publicIPAddresses/pip-monitoring-itmarathon-epam-2024-lexxai-prod]
module.network.azurerm_public_ip.public_ips["bastion"]: Creation complete after 4s [id=/subscriptions/................/resourceGroups/rg-itmarathon-epam-2024-lexxai-prod/providers/Microsoft.Network/publicIPAddresses/pip-bastion-itmarathon-epam-2024-lexxai-prod]
module.network.azurerm_virtual_network.marathon_virtual_network: Creation complete after 6s [id=/subscriptions/................/resourceGroups/rg-itmarathon-epam-2024-lexxai-prod/providers/Microsoft.Network/virtualNetworks/vnet-itmarathon-epam-2024-lexxai-prod]

...


Apply complete! Resources: 11 added, 0 changed, 0 destroyed.

Outputs:

monitoring_vm_public_ip = ""
mysql_subnet_id = "/subscriptions/......./resourceGroups/rg-itmarathon-epam-2024-lexxai-prod/providers/Microsoft.Network/virtualNetworks/vnet-itmarathon-epam-2024-lexxai-prod/subnets/mysql-subnet-itmarathon-epam-2024-lexxai-prod"
resource_group_name = "rg-itmarathon-epam-2024-lexxai-prod"
vnet_id = "/subscriptions/......../resourceGroups/rg-itmarathon-epam-2024-lexxai-prod/providers/Microsoft.Network/virtualNetworks/vnet-itmarathon-epam-2024-lexxai-prod"
```
</details>

### 2. Security (02_security)
```
$ terraform plan -var-file=terraform.tfvars -target=module.security
```
<details>
  <summary>Click to expand result of command</summary>


```
module.network.azurerm_resource_group.rg: Refreshing state... [id=/subscriptions/................/resourceGroups/rg-itmarathon-epam-2024-lexxai-prod]
module.network.azurerm_virtual_network.marathon_virtual_network: Refreshing state... [id=/subscriptions/................/resourceGroups/rg-itmarathon-epam-2024-lexxai-prod/providers/Microsoft.Network/virtualNetworks/vnet-itmarathon-epam-2024-lexxai-prod]
module.network.azurerm_subnet.monitoring_subnet: Refreshing state... [id=/subscriptions/................/resourceGroups/rg-itmarathon-epam-2024-lexxai-prod/providers/Microsoft.Network/virtualNetworks/vnet-itmarathon-epam-2024-lexxai-prod/subnets/monitoring-subnet-itmarathon-epam-2024-lexxai-prod]
module.network.azurerm_subnet.mysql_subnet: Refreshing state... [id=/subscriptions/................/resourceGroups/rg-itmarathon-epam-2024-lexxai-prod/providers/Microsoft.Network/virtualNetworks/vnet-itmarathon-epam-2024-lexxai-prod/subnets/mysql-subnet-itmarathon-epam-2024-lexxai-prod]
module.network.azurerm_subnet.private_subnet: Refreshing state... [id=/subscriptions/................/resourceGroups/rg-itmarathon-epam-2024-lexxai-prod/providers/Microsoft.Network/virtualNetworks/vnet-itmarathon-epam-2024-lexxai-prod/subnets/private-subnet-itmarathon-epam-2024-lexxai-prod]
module.network.azurerm_subnet.bastion_subnet: Refreshing state... [id=/subscriptions/................/resourceGroups/rg-itmarathon-epam-2024-lexxai-prod/providers/Microsoft.Network/virtualNetworks/vnet-itmarathon-epam-2024-lexxai-prod/subnets/bastion-subnet-itmarathon-epam-2024-lexxai-prod]
module.network.azurerm_subnet.public_subnet: Refreshing state... [id=/subscriptions/................/resourceGroups/rg-itmarathon-epam-2024-lexxai-prod/providers/Microsoft.Network/virtualNetworks/vnet-itmarathon-epam-2024-lexxai-prod/subnets/public-subnet-itmarathon-epam-2024-lexxai-prod]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # module.security.azurerm_network_security_group.bastion_subnet_sg will be created
  + resource "azurerm_network_security_group" "bastion_subnet_sg" {
      + id                  = (known after apply)
      + location            = "northeurope"
      + name                = "bastion-services-itmarathon-epam-2024-lexxai-prod"
      + resource_group_name = "rg-itmarathon-epam-2024-lexxai-prod"
      + security_rule       = (known after apply)
    }

  # module.security.azurerm_network_security_group.monitoring_subnet_sg will be created
  + resource "azurerm_network_security_group" "monitoring_subnet_sg" {
      + id                  = (known after apply)
      + location            = "northeurope"
      + name                = "monitoring-services-itmarathon-epam-2024-lexxai-prod"
      + resource_group_name = "rg-itmarathon-epam-2024-lexxai-prod"
      + security_rule       = (known after apply)
    }

  # module.security.azurerm_network_security_group.mysql_subnet_sg will be created
  + resource "azurerm_network_security_group" "mysql_subnet_sg" {
      + id                  = (known after apply)
      + location            = "northeurope"
      + name                = "mysql-services-itmarathon-epam-2024-lexxai-prod"
      + resource_group_name = "rg-itmarathon-epam-2024-lexxai-prod"
      + security_rule       = (known after apply)
    }

  # module.security.azurerm_network_security_group.private_subnet_sg will be created
  + resource "azurerm_network_security_group" "private_subnet_sg" {
      + id                  = (known after apply)
      + location            = "northeurope"
      + name                = "dotnet-app-services-itmarathon-epam-2024-lexxai-prod"
      + resource_group_name = "rg-itmarathon-epam-2024-lexxai-prod"
      + security_rule       = (known after apply)
    }

  # module.security.azurerm_network_security_group.public_subnet_sg will be created
  + resource "azurerm_network_security_group" "public_subnet_sg" {
      + id                  = (known after apply)
      + location            = "northeurope"
      + name                = "dotnet-lb-services-itmarathon-epam-2024-lexxai-prod"
      + resource_group_name = "rg-itmarathon-epam-2024-lexxai-prod"
      + security_rule       = (known after apply)
    }

  # module.security.azurerm_network_security_rule.allow_gateway_to_app will be created
  + resource "azurerm_network_security_rule" "allow_gateway_to_app" {
      + access                      = "Allow"
      + destination_address_prefix  = "10.0.5.0/24"
      + destination_port_range      = "443"
      + direction                   = "Inbound"
      + id                          = (known after apply)
      + name                        = "AllowGatewayToApp"
      + network_security_group_name = "dotnet-app-services-itmarathon-epam-2024-lexxai-prod"
      + priority                    = 1001
      + protocol                    = "Tcp"
      + resource_group_name         = "rg-itmarathon-epam-2024-lexxai-prod"
      + source_address_prefix       = "10.0.4.0/24"
      + source_port_range           = "*"
    }

  # module.security.azurerm_network_security_rule.allow_http_from_internet_monitoring will be created
  + resource "azurerm_network_security_rule" "allow_http_from_internet_monitoring" {
      + access                      = "Allow"
      + destination_address_prefix  = "*"
      + destination_port_range      = "80"
      + direction                   = "Inbound"
      + id                          = (known after apply)
      + name                        = "AllowHTTP"
      + network_security_group_name = "monitoring-services-itmarathon-epam-2024-lexxai-prod"
      + priority                    = 1002
      + protocol                    = "Tcp"
      + resource_group_name         = "rg-itmarathon-epam-2024-lexxai-prod"
      + source_address_prefixes     = [
          + "0.0.0.0/0",
        ]
      + source_port_range           = "*"
    }

  # module.security.azurerm_network_security_rule.allow_http_https_from_allowed_ips will be created
  + resource "azurerm_network_security_rule" "allow_http_https_from_allowed_ips" {
      + access                      = "Allow"
      + destination_address_prefix  = "*"
      + destination_port_ranges     = [
          + "443",
          + "80",
        ]
      + direction                   = "Inbound"
      + id                          = (known after apply)
      + name                        = "AllowHTTPandHTTPSFromAllowedIPs"
      + network_security_group_name = "dotnet-lb-services-itmarathon-epam-2024-lexxai-prod"
      + priority                    = 1002
      + protocol                    = "Tcp"
      + resource_group_name         = "rg-itmarathon-epam-2024-lexxai-prod"
      + source_address_prefixes     = [
          + "0.0.0.0/0",
        ]
      + source_port_range           = "*"
    }

  # module.security.azurerm_network_security_rule.allow_https_from_internet_monitoring will be created
  + resource "azurerm_network_security_rule" "allow_https_from_internet_monitoring" {
      + access                      = "Allow"
      + destination_address_prefix  = "*"
      + destination_port_range      = "443"
      + direction                   = "Inbound"
      + id                          = (known after apply)
      + name                        = "AllowHTTPS"
      + network_security_group_name = "monitoring-services-itmarathon-epam-2024-lexxai-prod"
      + priority                    = 1003
      + protocol                    = "Tcp"
      + resource_group_name         = "rg-itmarathon-epam-2024-lexxai-prod"
      + source_address_prefixes     = [
          + "0.0.0.0/0",
        ]
      + source_port_range           = "*"
    }

  # module.security.azurerm_network_security_rule.allow_ssh_from_internet will be created
  + resource "azurerm_network_security_rule" "allow_ssh_from_internet" {
      + access                      = "Allow"
      + destination_address_prefix  = "*"
      + destination_port_range      = "22"
      + direction                   = "Inbound"
      + id                          = (known after apply)
      + name                        = "AllowSSH"
      + network_security_group_name = "bastion-services-itmarathon-epam-2024-lexxai-prod"
      + priority                    = 1001
      + protocol                    = "Tcp"
      + resource_group_name         = "rg-itmarathon-epam-2024-lexxai-prod"
      + source_address_prefixes     = [
          + "0.0.0.0/0",
        ]
      + source_port_range           = "*"
    }

  # module.security.azurerm_network_security_rule.allow_ssh_from_internet_monitoring will be created
  + resource "azurerm_network_security_rule" "allow_ssh_from_internet_monitoring" {
      + access                      = "Allow"
      + destination_address_prefix  = "*"
      + destination_port_range      = "22"
      + direction                   = "Inbound"
      + id                          = (known after apply)
      + name                        = "AllowSSH"
      + network_security_group_name = "monitoring-services-itmarathon-epam-2024-lexxai-prod"
      + priority                    = 1001
      + protocol                    = "Tcp"
      + resource_group_name         = "rg-itmarathon-epam-2024-lexxai-prod"
      + source_address_prefixes     = [
          + "0.0.0.0/0",
        ]
      + source_port_range           = "*"
    }

  # module.security.azurerm_network_security_rule.deny_direct_access_to_app will be created
  + resource "azurerm_network_security_rule" "deny_direct_access_to_app" {
      + access                      = "Deny"
      + destination_address_prefix  = "10.0.5.0/24"
      + destination_port_range      = "*"
      + direction                   = "Inbound"
      + id                          = (known after apply)
      + name                        = "DenyDirectAccessToApp"
      + network_security_group_name = "dotnet-app-services-itmarathon-epam-2024-lexxai-prod"
      + priority                    = 1000
      + protocol                    = "*"
      + resource_group_name         = "rg-itmarathon-epam-2024-lexxai-prod"
      + source_address_prefix       = "Internet"
      + source_port_range           = "*"
    }

  # module.security.azurerm_subnet_network_security_group_association.bastion_subnet_sg_assoc will be created
  + resource "azurerm_subnet_network_security_group_association" "bastion_subnet_sg_assoc" {
      + id                        = (known after apply)
      + network_security_group_id = (known after apply)
      + subnet_id                 = "/subscriptions/................/resourceGroups/rg-itmarathon-epam-2024-lexxai-prod/providers/Microsoft.Network/virtualNetworks/vnet-itmarathon-epam-2024-lexxai-prod/subnets/bastion-subnet-itmarathon-epam-2024-lexxai-prod"
    }

  # module.security.azurerm_subnet_network_security_group_association.monitoring_subnet_sg_assoc will be created
  + resource "azurerm_subnet_network_security_group_association" "monitoring_subnet_sg_assoc" {
      + id                        = (known after apply)
      + network_security_group_id = (known after apply)
      + subnet_id                 = "/subscriptions/................/resourceGroups/rg-itmarathon-epam-2024-lexxai-prod/providers/Microsoft.Network/virtualNetworks/vnet-itmarathon-epam-2024-lexxai-prod/subnets/monitoring-subnet-itmarathon-epam-2024-lexxai-prod"
    }

  # module.security.azurerm_subnet_network_security_group_association.mysql_subnet_sg_assoc will be created
  + resource "azurerm_subnet_network_security_group_association" "mysql_subnet_sg_assoc" {
      + id                        = (known after apply)
      + network_security_group_id = (known after apply)
      + subnet_id                 = "/subscriptions/................/resourceGroups/rg-itmarathon-epam-2024-lexxai-prod/providers/Microsoft.Network/virtualNetworks/vnet-itmarathon-epam-2024-lexxai-prod/subnets/mysql-subnet-itmarathon-epam-2024-lexxai-prod"
    }

  # module.security.azurerm_subnet_network_security_group_association.private_subnet_sg_assoc will be created
  + resource "azurerm_subnet_network_security_group_association" "private_subnet_sg_assoc" {
      + id                        = (known after apply)
      + network_security_group_id = (known after apply)
      + subnet_id                 = "/subscriptions/................/resourceGroups/rg-itmarathon-epam-2024-lexxai-prod/providers/Microsoft.Network/virtualNetworks/vnet-itmarathon-epam-2024-lexxai-prod/subnets/private-subnet-itmarathon-epam-2024-lexxai-prod"
    }

  # module.security.azurerm_subnet_network_security_group_association.public_subnet_sg_assoc will be created
  + resource "azurerm_subnet_network_security_group_association" "public_subnet_sg_assoc" {
      + id                        = (known after apply)
      + network_security_group_id = (known after apply)
      + subnet_id                 = "/subscriptions/................/resourceGroups/rg-itmarathon-epam-2024-lexxai-prod/providers/Microsoft.Network/virtualNetworks/vnet-itmarathon-epam-2024-lexxai-prod/subnets/public-subnet-itmarathon-epam-2024-lexxai-prod"
    }

Plan: 17 to add, 0 to change, 0 to destroy.
╷
│ Warning: Resource targeting is in effect
│
│ You are creating a plan with the -target option, which means that the result of this plan may not represent all of the changes requested by the current configuration.
│
│ The -target option is not for routine use, and is provided only for exceptional situations such as recovering from errors or mistakes, or when Terraform specifically suggests to use it as part 
│ of an error message.
╵

────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── 

Note: You didn't use the -out option to save this plan, so Terraform can't guarantee to take exactly these actions if you run "terraform apply" now.
```
</details>

```
$ terraform apply -var-file=terraform.tfvars -target=module.security
```
<details>
  <summary>Click to expand result of command</summary>

```
...

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

module.security.azurerm_network_security_group.bastion_subnet_sg: Creating...
module.security.azurerm_network_security_group.private_subnet_sg: Creating...
module.security.azurerm_network_security_group.mysql_subnet_sg: Creating...
module.security.azurerm_network_security_group.public_subnet_sg: Creating...
module.security.azurerm_network_security_group.monitoring_subnet_sg: Creating...
module.security.azurerm_network_security_group.public_subnet_sg: Creation complete after 4s [id=/subscriptions/................/resourceGroups/rg-itmarathon-epam-2024-lexxai-prod/providers/Microsoft.Network/networkSecurityGroups/dotnet-lb-services-itmarathon-epam-2024-lexxai-prod]
module.security.azurerm_network_security_group.mysql_subnet_sg: Creation complete after 4s [id=/subscriptions/................/resourceGroups/rg-itmarathon-epam-2024-lexxai-prod/providers/Microsoft.Network/networkSecurityGroups/mysql-services-itmarathon-epam-2024-lexxai-prod]
module.security.azurerm_network_security_group.bastion_subnet_sg: Creation complete after 4s [id=/subscriptions/................/resourceGroups/rg-itmarathon-epam-2024-lexxai-prod/providers/Microsoft.Network/networkSecurityGroups/bastion-services-itmarathon-epam-2024-lexxai-prod]
module.security.azurerm_network_security_group.monitoring_subnet_sg: Creation complete after 4s [id=/subscriptions/................/resourceGroups/rg-itmarathon-epam-2024-lexxai-prod/providers/Microsoft.Network/networkSecurityGroups/monitoring-services-itmarathon-epam-2024-lexxai-prod]
module.security.azurerm_subnet_network_security_group_association.monitoring_subnet_sg_assoc: Creating...
module.security.azurerm_network_security_rule.allow_https_from_internet_monitoring: Creating...
module.security.azurerm_subnet_network_security_group_association.public_subnet_sg_assoc: Creating...
module.security.azurerm_network_security_rule.allow_ssh_from_internet: Creating...
module.security.azurerm_network_security_rule.allow_http_from_internet_monitoring: Creating...
module.security.azurerm_subnet_network_security_group_association.mysql_subnet_sg_assoc: Creating...
module.security.azurerm_network_security_rule.allow_ssh_from_internet_monitoring: Creating...
module.security.azurerm_network_security_rule.allow_http_https_from_allowed_ips: Creating...
module.security.azurerm_subnet_network_security_group_association.bastion_subnet_sg_assoc: Creating...
module.security.azurerm_network_security_group.private_subnet_sg: Creation complete after 4s [id=/subscriptions/................/resourceGroups/rg-itmarathon-epam-2024-lexxai-prod/providers/Microsoft.Network/networkSecurityGroups/dotnet-app-services-itmarathon-epam-2024-lexxai-prod]
module.security.azurerm_network_security_rule.deny_direct_access_to_app: Creating...
module.security.azurerm_network_security_rule.allow_ssh_from_internet: Creation complete after 3s [id=/subscriptions/................/resourceGroups/rg-itmarathon-epam-2024-lexxai-prod/providers/Microsoft.Network/networkSecurityGroups/bastion-services-itmarathon-epam-2024-lexxai-prod/securityRules/AllowSSH]
module.security.azurerm_subnet_network_security_group_association.private_subnet_sg_assoc: Creating...
module.security.azurerm_network_security_rule.allow_http_https_from_allowed_ips: Creation complete after 3s [id=/subscriptions/................/resourceGroups/rg-itmarathon-epam-2024-lexxai-prod/providers/Microsoft.Network/networkSecurityGroups/dotnet-lb-services-itmarathon-epam-2024-lexxai-prod/securityRules/AllowHTTPandHTTPSFromAllowedIPs]
module.security.azurerm_network_security_rule.allow_gateway_to_app: Creating...
module.security.azurerm_network_security_rule.allow_ssh_from_internet_monitoring: Creation complete after 3s [id=/subscriptions/................/resourceGroups/rg-itmarathon-epam-2024-lexxai-prod/providers/Microsoft.Network/networkSecurityGroups/monitoring-services-itmarathon-epam-2024-lexxai-prod/securityRules/AllowSSH]
module.security.azurerm_network_security_rule.deny_direct_access_to_app: Creation complete after 4s [id=/subscriptions/................/resourceGroups/rg-itmarathon-epam-2024-lexxai-prod/providers/Microsoft.Network/networkSecurityGroups/dotnet-app-services-itmarathon-epam-2024-lexxai-prod/securityRules/DenyDirectAccessToApp]
module.security.azurerm_network_security_rule.allow_https_from_internet_monitoring: Creation complete after 4s [id=/subscriptions/................/resourceGroups/rg-itmarathon-epam-2024-lexxai-prod/providers/Microsoft.Network/networkSecurityGroups/monitoring-services-itmarathon-epam-2024-lexxai-prod/securityRules/AllowHTTPS]
module.security.azurerm_network_security_rule.allow_http_from_internet_monitoring: Creation complete after 4s [id=/subscriptions/................/resourceGroups/rg-itmarathon-epam-2024-lexxai-prod/providers/Microsoft.Network/networkSecurityGroups/monitoring-services-itmarathon-epam-2024-lexxai-prod/securityRules/AllowHTTP]
module.security.azurerm_network_security_rule.allow_gateway_to_app: Creation complete after 3s [id=/subscriptions/................/resourceGroups/rg-itmarathon-epam-2024-lexxai-prod/providers/Microsoft.Network/networkSecurityGroups/dotnet-app-services-itmarathon-epam-2024-lexxai-prod/securityRules/AllowGatewayToApp]
module.security.azurerm_subnet_network_security_group_association.public_subnet_sg_assoc: Creation complete after 7s [id=/subscriptions/................/resourceGroups/rg-itmarathon-epam-2024-lexxai-prod/providers/Microsoft.Network/virtualNetworks/vnet-itmarathon-epam-2024-lexxai-prod/subnets/public-subnet-itmarathon-epam-2024-lexxai-prod]
module.security.azurerm_subnet_network_security_group_association.monitoring_subnet_sg_assoc: Still creating... [10s elapsed]
module.security.azurerm_subnet_network_security_group_association.mysql_subnet_sg_assoc: Still creating... [10s elapsed]
module.security.azurerm_subnet_network_security_group_association.bastion_subnet_sg_assoc: Still creating... [10s elapsed]
module.security.azurerm_subnet_network_security_group_association.private_subnet_sg_assoc: Still creating... [10s elapsed]
module.security.azurerm_subnet_network_security_group_association.monitoring_subnet_sg_assoc: Creation complete after 13s [id=/subscriptions/................/resourceGroups/rg-itmarathon-epam-2024-lexxai-prod/providers/Microsoft.Network/virtualNetworks/vnet-itmarathon-epam-2024-lexxai-prod/subnets/monitoring-subnet-itmarathon-epam-2024-lexxai-prod]
module.security.azurerm_subnet_network_security_group_association.mysql_subnet_sg_assoc: Creation complete after 20s [id=/subscriptions/................/resourceGroups/rg-itmarathon-epam-2024-lexxai-prod/providers/Microsoft.Network/virtualNetworks/vnet-itmarathon-epam-2024-lexxai-prod/subnets/mysql-subnet-itmarathon-epam-2024-lexxai-prod]
module.security.azurerm_subnet_network_security_group_association.bastion_subnet_sg_assoc: Still creating... [20s elapsed]
module.security.azurerm_subnet_network_security_group_association.private_subnet_sg_assoc: Still creating... [20s elapsed]
module.security.azurerm_subnet_network_security_group_association.bastion_subnet_sg_assoc: Creation complete after 26s [id=/subscriptions/................/resourceGroups/rg-itmarathon-epam-2024-lexxai-prod/providers/Microsoft.Network/virtualNetworks/vnet-itmarathon-epam-2024-lexxai-prod/subnets/bastion-subnet-itmarathon-epam-2024-lexxai-prod]
module.security.azurerm_subnet_network_security_group_association.private_subnet_sg_assoc: Creation complete after 29s [id=/subscriptions/................/resourceGroups/rg-itmarathon-epam-2024-lexxai-prod/providers/Microsoft.Network/virtualNetworks/vnet-itmarathon-epam-2024-lexxai-prod/subnets/private-subnet-itmarathon-epam-2024-lexxai-prod]
╷
│ Warning: Applied changes may be incomplete
│
│ The plan was created with the -target option in effect, so some changes requested in the configuration may have been ignored and the output values may not be fully updated. Run the following   
│ command to verify that no other changes are pending:
│     terraform plan
│
│ Note that the -target option is not suitable for routine use, and is provided only for exceptional situations such as recovering from errors or mistakes, or when Terraform specifically
│ suggests to use it as part of an error message.
╵

Apply complete! Resources: 17 added, 0 changed, 0 destroyed.

Outputs:

monitoring_vm_public_ip = ""
mysql_subnet_id = "/subscriptions/................/resourceGroups/rg-itmarathon-epam-2024-lexxai-prod/providers/Microsoft.Network/virtualNetworks/vnet-itmarathon-epam-2024-lexxai-prod/subnets/mysql-subnet-itmarathon-epam-2024-lexxai-prod"
resource_group_name = "rg-itmarathon-epam-2024-lexxai-prod"
vnet_id = "/subscriptions/................/resourceGroups/rg-itmarathon-epam-2024-lexxai-prod/providers/Microsoft.Network/virtualNetworks/vnet-itmarathon-epam-2024-lexxai-prod"
```
</details>

### 3. Bastion (03_bastion)

```
$ terraform plan -var-file=terraform.tfvars -target=module.bastion
```
<details>
  <summary>Click to expand result of command</summary>


```
module.network.azurerm_resource_group.rg: Refreshing state... [id=/subscriptions/................/resourceGroups/rg-itmarathon-epam-2024-lexxai-prod]
module.network.azurerm_public_ip.public_ips["monitoring"]: Refreshing state... [id=/subscriptions/................/resourceGroups/rg-itmarathon-epam-2024-lexxai-prod/providers/Microsoft.Network/publicIPAddresses/pip-monitoring-itmarathon-epam-2024-lexxai-prod]
module.network.azurerm_public_ip.public_ips["bastion"]: Refreshing state... [id=/subscriptions/................/resourceGroups/rg-itmarathon-epam-2024-lexxai-prod/providers/Microsoft.Network/publicIPAddresses/pip-bastion-itmarathon-epam-2024-lexxai-prod]
module.network.azurerm_virtual_network.marathon_virtual_network: Refreshing state... [id=/subscriptions/................/resourceGroups/rg-itmarathon-epam-2024-lexxai-prod/providers/Microsoft.Network/virtualNetworks/vnet-itmarathon-epam-2024-lexxai-prod]
module.network.azurerm_subnet.bastion_subnet: Refreshing state... [id=/subscriptions/................/resourceGroups/rg-itmarathon-epam-2024-lexxai-prod/providers/Microsoft.Network/virtualNetworks/vnet-itmarathon-epam-2024-lexxai-prod/subnets/bastion-subnet-itmarathon-epam-2024-lexxai-prod]

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # module.bastion.azurerm_linux_virtual_machine.bastion will be created
  + resource "azurerm_linux_virtual_machine" "bastion" {
      + admin_username                                         = "azureuser"
      + allow_extension_operations                             = true
      + bypass_platform_safety_checks_on_user_schedule_enabled = false
      + computer_name                                          = (known after apply)
      + custom_data                                            = (sensitive value)
      + disable_password_authentication                        = true
      + disk_controller_type                                   = (known after apply)
      + extensions_time_budget                                 = "PT1H30M"
      + id                                                     = (known after apply)
      + location                                               = "northeurope"
      + max_bid_price                                          = -1
      + name                                                   = "bastion-itmarathon-epam-2024-lexxai-prod"
      + network_interface_ids                                  = (known after apply)
      + patch_assessment_mode                                  = "ImageDefault"
      + patch_mode                                             = "ImageDefault"
      + platform_fault_domain                                  = -1
      + priority                                               = "Regular"
      + private_ip_address                                     = (known after apply)
      + private_ip_addresses                                   = (known after apply)
      + provision_vm_agent                                     = true
      + public_ip_address                                      = (known after apply)
      + public_ip_addresses                                    = (known after apply)
      + resource_group_name                                    = "rg-itmarathon-epam-2024-lexxai-prod"
      + size                                                   = "Standard_B1s"
      + virtual_machine_id                                     = (known after apply)
      + vm_agent_platform_updates_enabled                      = false

      + admin_ssh_key {
          + public_key = "ssh-rsa AAAAB3NzaC1yc2.........."
          + username   = "azureuser"
        }

      + identity {
          + principal_id = (known after apply)
          + tenant_id    = (known after apply)
          + type         = "SystemAssigned"
        }

      + os_disk {
          + caching                   = "ReadWrite"
          + disk_size_gb              = (known after apply)
          + name                      = "osdisk-itmarathon-epam-2024-lexxai-prod"
          + storage_account_type      = "Standard_LRS"
          + write_accelerator_enabled = false
        }

      + source_image_reference {
          + offer     = "debian-11"
          + publisher = "Debian"
          + sku       = "11"
          + version   = "latest"
        }

      + termination_notification (known after apply)
    }

  # module.bastion.azurerm_network_interface.bastion_nic will be created
  + resource "azurerm_network_interface" "bastion_nic" {
      + accelerated_networking_enabled = (known after apply)
      + applied_dns_servers            = (known after apply)
      + dns_servers                    = (known after apply)
      + enable_accelerated_networking  = (known after apply)
      + enable_ip_forwarding           = (known after apply)
      + id                             = (known after apply)
      + internal_domain_name_suffix    = (known after apply)
      + ip_forwarding_enabled          = (known after apply)
      + location                       = "northeurope"
      + mac_address                    = (known after apply)
      + name                           = "nic-bastion-itmarathon-epam-2024-lexxai-prod"
      + private_ip_address             = (known after apply)
      + private_ip_addresses           = (known after apply)
      + resource_group_name            = "rg-itmarathon-epam-2024-lexxai-prod"
      + virtual_machine_id             = (known after apply)

      + ip_configuration {
          + gateway_load_balancer_frontend_ip_configuration_id = (known after apply)
          + name                                               = "internal"
          + primary                                            = (known after apply)
          + private_ip_address                                 = (known after apply)
          + private_ip_address_allocation                      = "Dynamic"
          + private_ip_address_version                         = "IPv4"
          + public_ip_address_id                               = "/subscriptions/................/resourceGroups/rg-itmarathon-epam-2024-lexxai-prod/providers/Microsoft.Network/publicIPAddresses/pip-bastion-itmarathon-epam-2024-lexxai-prod"
          + subnet_id                                          = "/subscriptions/................/resourceGroups/rg-itmarathon-epam-2024-lexxai-prod/providers/Microsoft.Network/virtualNetworks/vnet-itmarathon-epam-2024-lexxai-prod/subnets/bastion-subnet-itmarathon-epam-2024-lexxai-prod"
        }
    }

Plan: 2 to add, 0 to change, 0 to destroy.
╷
│ Warning: Resource targeting is in effect
│
│ You are creating a plan with the -target option, which means that the result of this plan may not represent all of the changes requested by the current configuration.
│
│ The -target option is not for routine use, and is provided only for exceptional situations such as recovering from errors or mistakes, or when Terraform specifically suggests to use it as part 
│ of an error message.
╵

────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── 

Note: You didn't use the -out option to save this plan, so Terraform can't guarantee to take exactly these actions if you run "terraform apply" now.
```
</details>

```
$ terraform apply -var-file=terraform.tfvars -target=module.bastion
```
<details>
  <summary>Click to expand result of command</summary>

```
...
Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes
module.bastion.azurerm_network_interface.bastion_nic: Creating...
module.bastion.azurerm_network_interface.bastion_nic: Still creating... [10s elapsed]
module.bastion.azurerm_network_interface.bastion_nic: Creation complete after 17s [id=/subscriptions/................/resourceGroups/rg-itmarathon-epam-2024-lexxai-prod/providers/Microsoft.Network/networkInterfaces/nic-bastion-itmarathon-epam-2024-lexxai-prod]
module.bastion.azurerm_linux_virtual_machine.bastion: Creating...
module.bastion.azurerm_linux_virtual_machine.bastion: Still creating... [10s elapsed]
module.bastion.azurerm_linux_virtual_machine.bastion: Still creating... [20s elapsed]
module.bastion.azurerm_linux_virtual_machine.bastion: Still creating... [30s elapsed]
module.bastion.azurerm_linux_virtual_machine.bastion: Still creating... [40s elapsed]
module.bastion.azurerm_linux_virtual_machine.bastion: Still creating... [50s elapsed]
module.bastion.azurerm_linux_virtual_machine.bastion: Still creating... [1m0s elapsed]
module.bastion.azurerm_linux_virtual_machine.bastion: Creation complete after 1m7s [id=/subscriptions/................/resourceGroups/rg-itmarathon-epam-2024-lexxai-prod/providers/Microsoft.Compute/virtualMachines/bastion-itmarathon-epam-2024-lexxai-prod]
╷
│ Warning: Applied changes may be incomplete
│
│ The plan was created with the -target option in effect, so some changes requested in the configuration may have been ignored and the output values may not be fully updated. Run the following   
│ command to verify that no other changes are pending:
│     terraform plan
│
│ Note that the -target option is not suitable for routine use, and is provided only for exceptional situations such as recovering from errors or mistakes, or when Terraform specifically
│ suggests to use it as part of an error message.
╵

Apply complete! Resources: 2 added, 0 changed, 0 destroyed.

Outputs:

monitoring_vm_public_ip = ""
mysql_subnet_id = "/subscriptions/................/resourceGroups/rg-itmarathon-epam-2024-lexxai-prod/providers/Microsoft.Network/virtualNetworks/vnet-itmarathon-epam-2024-lexxai-prod/subnets/mysql-subnet-itmarathon-epam-2024-lexxai-prod"
resource_group_name = "rg-itmarathon-epam-2024-lexxai-prod"
vnet_id = "/subscriptions/................/resourceGroups/rg-itmarathon-epam-2024-lexxai-prod/providers/Microsoft.Network/virtualNetworks/vnet-itmarathon-epam-2024-lexxai-prod"
```
</details>

![vm bastion](image-2.png)

### 4. Database (04_database)

```
$ terraform plan -var-file=terraform.tfvars -target=module.database
```
<details>
  <summary>Click to expand result of command</summary>

```
module.network.azurerm_resource_group.rg: Refreshing state... [id=/subscriptions/................/resourceGroups/rg-itmarathon-epam-2024-lexxai-prod]
module.network.azurerm_private_dns_zone.mysql: Refreshing state... [id=/subscriptions/................/resourceGroups/rg-itmarathon-epam-2024-lexxai-prod/providers/Microsoft.Network/privateDnsZones/privatelink.mysql.database.azure.com]
module.network.azurerm_virtual_network.marathon_virtual_network: Refreshing state... [id=/subscriptions/................/resourceGroups/rg-itmarathon-epam-2024-lexxai-prod/providers/Microsoft.Network/virtualNetworks/vnet-itmarathon-epam-2024-lexxai-prod]
module.network.azurerm_subnet.mysql_subnet: Refreshing state... [id=/subscriptions/................/resourceGroups/rg-itmarathon-epam-2024-lexxai-prod/providers/Microsoft.Network/virtualNetworks/vnet-itmarathon-epam-2024-lexxai-prod/subnets/mysql-subnet-itmarathon-epam-2024-lexxai-prod]   
module.network.azurerm_private_dns_zone_virtual_network_link.mysql: Refreshing state... [id=/subscriptions/................/resourceGroups/rg-itmarathon-epam-2024-lexxai-prod/providers/Microsoft.Network/privateDnsZones/privatelink.mysql.database.azure.com/virtualNetworkLinks/mysqldnslink] 

Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # module.database.azurerm_mysql_flexible_database.marathon_mysql will be created
  + resource "azurerm_mysql_flexible_database" "marathon_mysql" {
      + charset             = "utf8mb4"
      + collation           = "utf8mb4_0900_ai_ci"
      + id                  = (known after apply)
      + name                = "itmarathon-epam-2024-lexxai-prod"
      + resource_group_name = "rg-itmarathon-epam-2024-lexxai-prod"
      + server_name         = "marathon-itmarathon-epam-2024-lexxai-prod"
    }

  # module.database.azurerm_mysql_flexible_server.marathon_mysql will be created
  + resource "azurerm_mysql_flexible_server" "marathon_mysql" {
      + administrator_login           = (sensitive value)
      + administrator_password        = (sensitive value)
      + backup_retention_days         = 7
      + delegated_subnet_id           = "/subscriptions/................/resourceGroups/rg-itmarathon-epam-2024-lexxai-prod/providers/Microsoft.Network/virtualNetworks/vnet-itmarathon-epam-2024-lexxai-prod/subnets/mysql-subnet-itmarathon-epam-2024-lexxai-prod"
      + fqdn                          = (known after apply)
      + geo_redundant_backup_enabled  = false
      + id                            = (known after apply)
      + location                      = "northeurope"
      + name                          = "marathon-itmarathon-epam-2024-lexxai-prod"
      + private_dns_zone_id           = "/subscriptions/................/resourceGroups/rg-itmarathon-epam-2024-lexxai-prod/providers/Microsoft.Network/privateDnsZones/privatelink.mysql.database.azure.com"
      + public_network_access_enabled = (known after apply)
      + replica_capacity              = (known after apply)
      + replication_role              = (known after apply)
      + resource_group_name           = "rg-itmarathon-epam-2024-lexxai-prod"
      + sku_name                      = (sensitive value)
      + version                       = (sensitive value)
      + zone                          = "3"

      + storage (known after apply)
    }

  # module.database.azurerm_mysql_flexible_server_configuration.event_scheduler will be created
  + resource "azurerm_mysql_flexible_server_configuration" "event_scheduler" {
      + id                  = (known after apply)
      + name                = "event_scheduler"
      + resource_group_name = "rg-itmarathon-epam-2024-lexxai-prod"
      + server_name         = "marathon-itmarathon-epam-2024-lexxai-prod"
      + value               = "OFF"
    }

  # module.database.azurerm_mysql_flexible_server_configuration.require_secure_transport will be created
  + resource "azurerm_mysql_flexible_server_configuration" "require_secure_transport" {
      + id                  = (known after apply)
      + name                = "require_secure_transport"
      + resource_group_name = "rg-itmarathon-epam-2024-lexxai-prod"
      + server_name         = "marathon-itmarathon-epam-2024-lexxai-prod"
      + value               = "OFF"
    }

  # module.database.azurerm_mysql_flexible_server_configuration.sql_generate_invisible_primary_key will be created
  + resource "azurerm_mysql_flexible_server_configuration" "sql_generate_invisible_primary_key" {
      + id                  = (known after apply)
      + name                = "sql_generate_invisible_primary_key"
      + resource_group_name = "rg-itmarathon-epam-2024-lexxai-prod"
      + server_name         = "marathon-itmarathon-epam-2024-lexxai-prod"
      + value               = "OFF"
    }

Plan: 5 to add, 0 to change, 0 to destroy.

Changes to Outputs:
  + mysql_server_fqdn   = (known after apply)
╷
│ Warning: Resource targeting is in effect
│
│ You are creating a plan with the -target option, which means that the result of this plan may not represent all of the changes requested by the current  
│ configuration.
│
│ The -target option is not for routine use, and is provided only for exceptional situations such as recovering from errors or mistakes, or when Terraform 
│ specifically suggests to use it as part of an error message.
╵

────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────────── 

Note: You didn't use the -out option to save this plan, so Terraform can't guarantee to take exactly these actions if you run "terraform apply" now.       
```
</details>


```
$ terraform apply -var-file=terraform.tfvars -target=module.database
```
<details>
  <summary>Click to expand result of command</summary>

```
...
Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes
module.database.azurerm_mysql_flexible_server.marathon_mysql: Creating...
module.database.azurerm_mysql_flexible_server.marathon_mysql: Still creating... [10s elapsed]
module.database.azurerm_mysql_flexible_server.marathon_mysql: Still creating... [20s elapsed]
module.database.azurerm_mysql_flexible_server.marathon_mysql: Still creating... [30s elapsed]
module.database.azurerm_mysql_flexible_server.marathon_mysql: Still creating... [40s elapsed]
...
module.database.azurerm_mysql_flexible_server.marathon_mysql: Still creating... [6m40s elapsed]
module.database.azurerm_mysql_flexible_server.marathon_mysql: Still creating... [6m50s elapsed]
module.database.azurerm_mysql_flexible_server.marathon_mysql: Still creating... [7m0s elapsed]
module.database.azurerm_mysql_flexible_server.marathon_mysql: Creation complete after 7m8s [id=/subscriptions/................/resourceGroups/rg-itmarathon-epam-2024-lexxai-prod/providers/Microsoft.DBforMySQL/flexibleServers/marathon-itmarathon-epam-2024-lexxai-prod]
module.database.azurerm_mysql_flexible_server_configuration.sql_generate_invisible_primary_key: Creating...
module.database.azurerm_mysql_flexible_database.marathon_mysql: Creating...
module.database.azurerm_mysql_flexible_server_configuration.require_secure_transport: Creating...
module.database.azurerm_mysql_flexible_server_configuration.event_scheduler: Creating...
module.database.azurerm_mysql_flexible_database.marathon_mysql: Still creating... [10s elapsed]
module.database.azurerm_mysql_flexible_server_configuration.sql_generate_invisible_primary_key: Still creating... [10s elapsed]
module.database.azurerm_mysql_flexible_server_configuration.require_secure_transport: Still creating... [10s elapsed]
module.database.azurerm_mysql_flexible_server_configuration.event_scheduler: Still creating... [10s elapsed]
module.database.azurerm_mysql_flexible_server_configuration.sql_generate_invisible_primary_key: Creation complete after 19s [id=/subscriptions/................/resourceGroups/rg-itmarathon-epam-2024-lexxai-prod/providers/Microsoft.DBforMySQL/flexibleServers/marathon-itmarathon-epam-2024-lexxai-prod/configurations/sql_generate_invisible_primary_key]
module.database.azurerm_mysql_flexible_database.marathon_mysql: Still creating... [20s elapsed]
module.database.azurerm_mysql_flexible_server_configuration.event_scheduler: Still creating... [20s elapsed]
module.database.azurerm_mysql_flexible_server_configuration.require_secure_transport: Still creating... [20s elapsed]
module.database.azurerm_mysql_flexible_database.marathon_mysql: Still creating... [30s elapsed]
module.database.azurerm_mysql_flexible_server_configuration.require_secure_transport: Still creating... [30s elapsed]
module.database.azurerm_mysql_flexible_server_configuration.event_scheduler: Still creating... [30s elapsed]
module.database.azurerm_mysql_flexible_database.marathon_mysql: Still creating... [40s elapsed]
module.database.azurerm_mysql_flexible_server_configuration.event_scheduler: Still creating... [40s elapsed]
module.database.azurerm_mysql_flexible_server_configuration.require_secure_transport: Still creating... [40s elapsed]
module.database.azurerm_mysql_flexible_database.marathon_mysql: Still creating... [50s elapsed]
module.database.azurerm_mysql_flexible_server_configuration.require_secure_transport: Still creating... [50s elapsed]
module.database.azurerm_mysql_flexible_server_configuration.event_scheduler: Still creating... [50s elapsed]
module.database.azurerm_mysql_flexible_server_configuration.event_scheduler: Still creating... [1m0s elapsed]
module.database.azurerm_mysql_flexible_database.marathon_mysql: Still creating... [1m0s elapsed]
module.database.azurerm_mysql_flexible_server_configuration.require_secure_transport: Still creating... [1m0s elapsed]
module.database.azurerm_mysql_flexible_database.marathon_mysql: Creation complete after 1m5s [id=/subscriptions/................/resourceGroups/rg-itmarathon-epam-2024-lexxai-prod/providers/Microsoft.DBforMySQL/flexibleServers/marathon-itmarathon-epam-2024-lexxai-prod/databases/itmarathon-epam-2024-lexxai-prod]
module.database.azurerm_mysql_flexible_server_configuration.event_scheduler: Still creating... [1m10s elapsed]
module.database.azurerm_mysql_flexible_server_configuration.require_secure_transport: Still creating... [1m10s elapsed]
module.database.azurerm_mysql_flexible_server_configuration.require_secure_transport: Still creating... [1m20s elapsed]
module.database.azurerm_mysql_flexible_server_configuration.event_scheduler: Still creating... [1m20s elapsed]
module.database.azurerm_mysql_flexible_server_configuration.require_secure_transport: Creation complete after 1m23s [id=/subscriptions/................/resourceGroups/rg-itmarathon-epam-2024-lexxai-prod/providers/Microsoft.DBforMySQL/flexibleServers/marathon-itmarathon-epam-2024-lexxai-prod/configurations/require_secure_transport]
module.database.azurerm_mysql_flexible_server_configuration.event_scheduler: Still creating... [1m30s elapsed]
module.database.azurerm_mysql_flexible_server_configuration.event_scheduler: Creation complete after 1m40s [id=/subscriptions/................/resourceGroups/rg-itmarathon-epam-2024-lexxai-prod/providers/Microsoft.DBforMySQL/flexibleServers/marathon-itmarathon-epam-2024-lexxai-prod/configurations/event_scheduler]
╷
│ Warning: Applied changes may be incomplete
│
│ The plan was created with the -target option in effect, so some changes requested in the configuration may have been ignored and the output values may not be fully updated. Run the following   
│ command to verify that no other changes are pending:
│     terraform plan
│
│ Note that the -target option is not suitable for routine use, and is provided only for exceptional situations such as recovering from errors or mistakes, or when Terraform specifically
│ suggests to use it as part of an error message.
╵

Apply complete! Resources: 5 added, 0 changed, 0 destroyed.

Outputs:

monitoring_vm_public_ip = ""
mysql_server_fqdn = "marathon-itmarathon-epam-2024-lexxai-prod.mysql.database.azure.com"
mysql_subnet_id = "/subscriptions/................/resourceGroups/rg-itmarathon-epam-2024-lexxai-prod/providers/Microsoft.Network/virtualNetworks/vnet-itmarathon-epam-2024-lexxai-prod/subnets/mysql-subnet-itmarathon-epam-2024-lexxai-prod"
resource_group_name = "rg-itmarathon-epam-2024-lexxai-prod"
vnet_id = "/subscriptions/................/resourceGroups/rg-itmarathon-epam-2024-lexxai-prod/providers/Microsoft.Network/virtualNetworks/vnet-itmarathon-epam-2024-lexxai-prod"
```
</details>

![mysql](image-3.png)


### 5. Storage (05_storage)
```
terraform plan -var-file=terraform.tfvars -target=module.storage
```
Was replaced project name to shorten:
<details>
  <summary>Click to expand result of command</summary>

So after change project name, need delete all previous resources and start again, since total number of "PUBLIC IP" limited on this subscription, and names of resources will be mixed then.
![delete resources](image-4.png)
</details>

```
terraform apply -var-file=terraform.tfvars -target=module.storage
```
<details>
  <summary>Click to expand result of command</summary>

```

```
</details>
