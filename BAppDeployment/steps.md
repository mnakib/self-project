# Azure App Deployment

## Environment Preparation

### Setting the Resources Names

```bash

```

### Set the Default Subscription

```bash
# Variables
Subscription_ID=23d2cf3f-c98e-44c5-8cca-52cc13acad13

# Set the default subscription
az account set --subscription $Subscription_ID
```

### Creating the Resource Group
```bash
# Variables			
Random_Number=$(echo $((100 + RANDOM % 1)))
ENV=nonprod
RGName=rg-bea-$ENV-beyn-$ENV-ne-$Random_Number
AzureRegion=westeurope
Subscription_ID=23d2cf3f-c98e-44c5-8cca-52cc13acad13

az group create --name $RGName --location $AzureRegion --subscription $Subscription_ID
```

### Creating the Virtual Network and Subnets

#### Create the Virtual Network

```bash
# Variables			
VNet_Name=vnet-bea-$ENV-beyn-$ENV-ne-$Random_Number
VNet_AddressPrefix=10.5.0.0/20

# Register the Network Provider
az provider register --namespace Microsoft.Network


az network vnet create \
--resource-group $RGName \
--name $VNet_Name \
--address-prefix $VNet_AddressPrefix
```

#### Creating the Subnets

##### Create the Application Gateway subnet

```bash
# Variables			
APPGW_Subnet_Name=sub-agw-bea-$ENV-beyn-$ENV-ne-$Random_Number
APPGW_Subnet_AddressPrefix=10.5.1.0/24

az network vnet subnet create \
--resource-group $RGName \
--vnet-name $VNet_Name \
--name $APPGW_Subnet_Name \
--address-prefix $APPGW_Subnet_AddressPrefix
```


##### Create the WEB Subnet
```bash
# Variables			
WEB_Subnet_Name=sub-web-bea-$ENV-beyn-$ENV-ne-$Random_Number
WEB_Subnet_AddressPrefix=10.5.2.0/24

az network vnet subnet create \
--resource-group $RGName \
--vnet-name $VNet_Name \
--name $WEB_Subnet_Name \
--address-prefix $WEB_Subnet_AddressPrefix
```


##### Create the APP Subnet
```bash
# Variables			
APP_Subnet_Name=sub-app-bea-$ENV-beyn-$ENV-ne-$Random_Number
APP_Subnet_AddressPrefix=10.5.3.0/24

az network vnet subnet create \
--resource-group $RGName \
--vnet-name $VNet_Name \
--name $APP_Subnet_Name \
--address-prefix $APP_Subnet_AddressPrefix
```


##### Create the DB Subnet
```bash
# Variables			
DB_Subnet_Name=sub-db-bea-$ENV-beyn-$ENV-ne-$Random_Number
DB_Subnet_AddressPrefix=10.5.4.0/24

az network vnet subnet create \
--resource-group $RGName \
--vnet-name $VNet_Name \
--name $DB_Subnet_Name \
--address-prefix $DB_Subnet_AddressPrefix
```

##### Create the SFTP Subnet
```bash
# Variables			
SFTP_Subnet_Name=sub-sftp-bea-$ENV-beyn-$ENV-ne-$Random_Number
SFTP_Subnet_AddressPrefix=10.5.5.0/24

az network vnet subnet create \
--resource-group $RGName \
--vnet-name $VNet_Name \
--name $SFTP_Subnet_Name \
--address-prefix $SFTP_Subnet_AddressPrefix
```

##### Create the Bastion Host Subnet
```bash
# Variables		
BASTION_Subnet_AddressPrefix=10.5.10.0/24

az network vnet subnet create \
--resource-group $RGName \
--vnet-name $VNet_Name \
--name AzureBastionSubnet \
--address-prefix $BASTION_Subnet_AddressPrefix
```

#### Creating the NSGs

##### Create the App GW Subnet NSG ==> SKIP <==

```bash
# Variables			
NSG_AppGW_Name=nsg-azg-bea-$ENV-beyn-$ENV-ne-$Random_Number

# Create the NSG
az network nsg create \
--resource-group $RGName \
--name $NSG_AppGW_Name

# Associate the NSG with the subnet
az network vnet subnet update \
--resource-group $RGName \
--vnet-name $VNet_Name \
--name $APPGW_Subnet_Name \
--network-security-group $NSG_AppGW_Name
```

##### Create the App GW NSG Rules - Manual at a later stage

| Rule Type | Direction | Source | Destination | Protocol | Port(s) | Purpose |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Azure Management** | Inbound | `GatewayManager` Service Tag | Any | TCP | **v1: 65503-65534**<br>**v2: 65200-65535** | Required for the Azure platform to manage the gateway. |
| **Client Traffic** | Inbound | `Internet` (or restricted IPs) | Any | TCP | **80, 443** (Your Listeners) | Allows web clients to reach the Application Gateway. |
| **Backend Communication** | Outbound | Any | **Backend Subnet CIDR** | TCP | **80, 443** (or custom port) | Allows the gateway to send traffic to the backend VMs. |
| **Internet Egress** | Outbound | Any | `Internet` Service Tag | Any | Any | **Required for v2 SKU** for various operational needs (e.g., CRL checks, updates). You **cannot** block all outbound internet access. |
| **Health Probes** | Inbound | `AzureLoadBalancer` Service Tag | Any | Any | Any | Ensures internal load balancer traffic (like health probes) is allowed. |


##### Create the WEB Subnet NSG

```bash
NSG_Web_Name=nsg-web-bea-$ENV-beyn-$ENV-ne-$Random_Number
WEB_Subnet_Name=sub-web-bea-$ENV-beyn-$ENV-ne-$Random_Number


# Create the NSG
az network nsg create \
--resource-group $RGName \
--name $NSG_Web_Name

# Associate the NSG with the subnet
az network vnet subnet update \
--resource-group $RGName \
--vnet-name $VNet_Name \
--name $WEB_Subnet_Name \
--network-security-group $NSG_Web_Name
```

##### Create the Web NSG Rules - Manual at a later stage

| Field | Configuration | Notes |
| :--- | :--- | :--- |
| **Source** | **IP Addresses** or **CIDR range** of the Application Gateway Subnet | This is the most critical step. The traffic originates from the Application Gateway's **internal (private) IP addresses**, which are within the range of its dedicated subnet. |
| **Source Port Ranges** | `*` (Any) | The Application Gateway uses dynamic source ports. |
| **Destination** | `Any` or **Application Security Group (ASG)** | If you use an ASG that contains your Web VM's NIC, it's a more flexible and robust solution than specifying individual VM IPs. |
| **Destination Port Ranges** | **`80`** (for HTTP) and/or **`443`** (for HTTPS) | This is the port your web application is listening on. If using end-to-end TLS, this will typically be `443`. |
| **Protocol** | **`TCP`** | Web traffic (HTTP/HTTPS) uses TCP. |
| **Action** | **`Allow`** | |
| **Priority** | A lower number (e.g., `100` or `110`) | Ensure this rule has a higher priority than the default "Deny all inbound" rule you should also have. |


##### Create the APP Subnet NSG
```bash
# Variables			
NSG_App_Name=nsg-app-bea-$ENV-beyn-$ENV-ne-$Random_Number
APP_Subnet_Name=sub-app-bea-$ENV-beyn-$ENV-ne-$Random_Number


# Create the NSG
az network nsg create \
--resource-group $RGName \
--name $NSG_App_Name

# Associate the NSG with the subnet
az network vnet subnet update \
--resource-group $RGName \
--vnet-name $VNet_Name \
--name $APP_Subnet_Name \
--network-security-group $NSG_App_Name
```


##### Create the APP NSG Rules - Manual at a later stage

| Setting | Value | Explanation |
| :--- | :--- | :--- |
| **Direction** | **Inbound** | The traffic is coming *into* the App Subnet. |
| **Source** | **IP Addresses / CIDR ranges** (or **VirtualNetwork** and then a specific subnet) | This should be the **CIDR range of the Web Subnet** (e.g., `10.0.1.0/24`). Specifying the Web Subnet's range limits access only to those web servers. |
| **Source port ranges** | `*` (Any) | The web scale set instances will use ephemeral (random high) ports as their source port. |
| **Destination** | `*` (Any) or **IP Addresses / CIDR ranges** | Since the rule is on the **subnet NSG**, the destination is the subnet itself, which contains the ILB and the scale set instances. You can use the **CIDR range of the App Subnet** (e.g., `10.0.2.0/24`). |
| **Destination port ranges** | **8090** | This is the specific port the application is listening on, as required. |
| **Protocol** | **TCP** (or `*` if the application uses UDP) | Most web/app traffic uses TCP. Use TCP unless the application is specifically running on UDP. |
| **Action** | **Allow** | To explicitly permit the traffic. |
| **Priority** | A number **lower than 65500** | A lower priority number (e.g., 100-300) ensures this rule is processed before any default "Deny" rules that might block inter-subnet traffic. |


##### Create the SFTP Subnet NSG
```bash
# Variables			
NSG_Sftp_Name=nsg-sftp-bea-$ENV-beyn-$ENV-ne-$Random_Number
SFTP_Subnet_Name=sub-sftp-bea-$ENV-beyn-$ENV-ne-$Random_Number

# Create the NSG
az network nsg create \
--resource-group $RGName \
--name $NSG_Sftp_Name

# Associate the NSG with the subnet
az network vnet subnet update \
--resource-group $RGName \
--vnet-name $VNet_Name \
--name $SFTP_Subnet_Name \
--network-security-group $NSG_Sftp_Name
```

##### Create the SFTP NSG Rules - Manual at a later stage

| Traffic | Direction | Port | Source | Destination | Action | Purpose |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **App-to-DB** | Inbound | 5432 (default) | Subnet/IP range of your **Application Servers** | Subnet/IP range of the **DB Subnet** | Allow | Allows the application to connect. |
| **HA/Internal** | Inbound & Outbound | 5432, 6432 | Subnet/IP range of the **DB Subnet** | Subnet/IP range of the **DB Subnet** | Allow | Required for internal server communication, including HA. |
| **Azure Services** | Outbound | 443 | Subnet/IP range of the **DB Subnet** | **Service Tag: `AzureStorage`** | Allow | For automated backups and log archival. |
| **Azure AD/Entra ID**| Outbound | 443 | Subnet/IP range of the **DB Subnet** | **Service Tag: `AzureActiveDirectory`** | Allow | Required for Microsoft Entra authentication. |
| **General Internet**| Outbound | * | * | **Service Tag: `Internet`** | Deny | **Best practice** to prevent unauthorized data exfiltration. |


##### Create the DB Subnet NSG
```bash
# Variables			
NSG_DB_Name=nsg-db-bea-$ENV-partenaire-$ENV-ne-$Random_Number
DB_Subnet_Name=sub-db-bea-$ENV-beyn-$ENV-ne-$Random_Number

# Create the NSG
az network nsg create \
--resource-group $RGName \
--name $NSG_DB_Name

# Associate the NSG with the subnet
az network vnet subnet update \
--resource-group $RGName \
--vnet-name $VNet_Name \
--name $DB_Subnet_Name \
--network-security-group $NSG_DB_Name
```

#### Creating the Load Balancers

#### Create APP Internal Load Balancer

```bash
# Variables
APP_LB_Name=lb-app-bea-$ENV-beyn-$ENV-ne-$Random_Number
APP_LB_SKU=Standard

az network lb create \
  --resource-group $RGName \
  --name  $APP_LB_Name\
  --sku $APP_LB_SKU \
  --vnet-name $VNet_Name \
  --subnet $APP_Subnet_Name \
  --location $AzureRegion \
  --public-ip-address ""
```


### Deploying the Bastion Host 

#### Create the Bastion Public IP

```bash
# Variables
BastionPublicIP_Name=bst-pub-ip-bea-$ENV-beyn-$ENV-ne-$Random_Number
Bastion_PublicIP_SKU=Standard
Bastion_PublicIP_Tier=Regional
Bastion_PublicIP_AllocationMethod=Static

az network public-ip create \
  --resource-group $RGName \
  --name $BastionPublicIP_Name \
  --location $AzureRegion \
  --sku $Bastion_PublicIP_SKU \
  --tier $Bastion_PublicIP_Tier \
  --allocation-method $Bastion_PublicIP_AllocationMethod
```


#### Create the Bastion Host

```bash
# Variables
BastionHostName=bst-host-bea-$ENV-beyn-$ENV-ne-$Random_Number
BastionSKU=Standard

az network bastion create \
  --name $BastionHostName \
  --resource-group $RGName \
  --vnet-name $VNet_Name \
  --location $AzureRegion \
  --public-ip-address $BastionPublicIP_Name \
  --sku $BastionSKU
```



### Application Gateway Deployment

#### Create the Application Gateway Public IP
```bash
# Variables
APPGW_PublicIP_Name=app-agw-ip-bea-$ENV-beyn-$ENV-ne-$Random_Number
APPGW_PublicIP_SKU=Standard
APPGW_PublicIP_Tier=Regional
APPGW_PublicIP_AllocationMethod="Static"

az network public-ip create \
  --resource-group $RGName \
  --name $APPGW_PublicIP_Name \
  --location $AzureRegion \
  --sku $APPGW_PublicIP_SKU \
  --tier $APPGW_PublicIP_Tier \
  --allocation-method $APPGW_PublicIP_AllocationMethod
```

#### Creating the Web Frontend Application Gateway - Manual

```bash
# Variables
APPGW_Name="app_gw-bea-non_prod_partenaire-nonprod-ne-$(echo $((100 + RANDOM % 1)))"
APPGW_SKU="WAF_v2"
APPGW_CAPACITY=2
APPGW_FE_PORT=80
APPGW_HTTP_SETTINGS_COOKIE_BASED_AFFINITY="Disabled"
APPGW_SETTINGS_PORT=80
APPGW_SETTINGS_PROTOCOL="Http"

az network application-gateway create \
  --name $APPGW_Name \
  --location $AzureRegion \
  --resource-group $RGName \
  --sku $APPGW_SKU \
  --capacity $APPGW_CAPACITY \
  --frontend-port $APPGW_FE_PORT \
  --http-settings-cookie-based-affinity $APPGW_HTTP_SET_COOKIE_AFFINITY \
  --http-settings-port $APPGW_SET-PORT \
  --http-settings-protocol $APPGW_SET_PROTOCOL \
  --vnet-name $VNet_Name \
  --subnet $APPGW_Subnet_Name \
  --public-ip-address $APPGWPublic_IPName
```


## SSH Keys Creation - Create manually

```bash
# Variables
WEB_SSH_Key_Name=web-ssh-key-bea-prod-beyn-prod-ne-100
APP_SSH_Key_Name=app-ssh-key-bea-prod-beyn-prod-ne-100
SFTP_SSH_Key_Name=sftp-ssh-key-bea-prod-beyn-prod-ne-100
```



## Key Vault Creation
```bash

## Create the Key Vault
# Variables		
KeyVaultName=kvault-bea-nonprod-beyn1
AzureRegion=westeurope

az keyvault create --name $KeyVaultName \
--resource-group $RGName \
--location $AzureRegion

## Configure Key Vault RBAC
# Variables	
Vault_Role	=	"Key Vault Secrets Officer"
AzureADObjectId	=	m.nakib-ext@bea-international.fr
SubID	=	23d2cf3f-c98e-44c5-8cca-52cc13acad13
RGName	=	$RGName
Vault_Name	=	bea-key-vault-non-prod
My_Public_IP_Address=		$(echo $(curl -s -4 ipinfo.io/ip))

az role assignment create --role $Vault_Role \
--assignee $AzureADObjectId \
--scope /subscriptions/$SubID/resourceGroups/$RGName/providers/Microsoft.KeyVault/vaults/$Vault_Name
```


## NAT Gateway Creation
#### Create the NAT Gateway Public IP
```bash
# Variables
NAT_GW_PublicIP_Name=natgw-pub-ip-bea-$ENV-beyn-$ENV-ne-$Random_Number
NAT_GW_PublicIP_SKU=Standard
NAT_GW_PublicIP_Tier=Regional
NAT_GW_PublicIP_AllocationMethod="Static"

# Create the NAT Gateway Public IP
az network public-ip create \
  --resource-group $RGName \
  --name $NAT_GW_PublicIP_Name \
  --location $AzureRegion \
  --sku $NAT_GW_PublicIP_SKU \
  --tier $NAT_GW_PublicIP_Tier \
  --allocation-method $NAT_GW_PublicIP_AllocationMethod
```

#### Create the NAT Gateway
```bash

# Variables
NAT_GW_Name=natgw-bea-$ENV-beyn-$ENV-ne-$Random_Number
NAT_GW_Name_SKU=Standard

az network nat gateway create \
  --resource-group $RGName \
  --name $NAT_GW_Name \
  --location $AzureRegion \
  --public-ip-addresses $NAT_GW_PublicIP_Name \
  --sku Standard
```



## Resources Deployment

### Frontend WEB VMs Scale Set Deployment - Manual

#### Create the VM Scale Set - Manual

```bash
# Variables
VMSS_Name="vmss-bea-non_prod_partenaire-nonprod-ne-$(echo $((100 + RANDOM % 1)))"
VMSS_Image=Ubuntu2204
VMSS_Upgrade_Policy=automatic
VMSS_Admin_UserName=azureuser
VMSS_Instance_Count=2
VMSS_SKU=B2als_v2
VMSS_Bakend_PoolName=APPGW_Backend_Pool
VMSS_APPGW=$APPGW_Name
```


#### Link the the Application GW to the the VMSS backend pool - Manual

```bash
az network application-gateway address-pool create \
  --gateway-name $APPGW_Name \
  --resource-group $RGName \
  --name APPGW_Backend_Pool
  --servers X.X.X.X X.X.X.X
```

```bash
az network application-gateway rule create \
  --gateway-name APPGW_Name \
  --resource-group $RGName \
  --name AppGW_RoutingRule \
  --address-pool APPGW_Backend_Pool \
  --http-listener appGatewayHttpListener \
  --rule-type Basic \
  --http-settings appGatewayBackendHttpSettings
```




## Application VMs Scale Set Deployment - Manual


#### Create the VM Scale Set - Manual

#### Create the APP Internal Load Balancer - Manual




## Testing Connectivity to the Application VMs - Manual

## Testing Connectivity to Internal Load Balancer and Checking Proper Routing

## Azure Database for PostgresSQL Deployment - Manual










