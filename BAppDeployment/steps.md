# Azure App Deployment

## Environment Preparation

### Setting the Resources Names

```bash
BastionPublicIP_Name="bst_host_ip-bea-non_prod_partenaire-nonprod-ne-$(echo $((100 + RANDOM % 1)))"
Bastion_PublicIP_SKU="Standard"
Bastion_PublicIP_Tier="Regional"
Bastion_PublicIP_AllocationMethod="Static"

BastionName="bst_host-bea-non_prod_partenaire-nonprod-ne-$(echo $((100 + RANDOM % 1)))"
BastionSKU="Standard"


VMSS_Name="vmss-bea-non_prod_partenaire-nonprod-ne-$(echo $((100 + RANDOM % 1)))"
VMSS_Image=Ubuntu2204
VMSS_Upgrade_Policy=automatic
VMSS_Admin_UserName=azureuser
VMSS_Instance_Count=2
VMSS_SKU=B2als_v2
VMSS_Bakend_PoolName=APPGW_Backend_Pool
VMSS_APPGW=$APPGW_Name


APPGW_PublicIP_Name="app_gw_ip-bea-non_prod_partenaire-nonprod-ne-$(echo $((100 + RANDOM % 1)))"
APPGW_PublicIP_SKU="Standard"
APPGW_PublicIP_Tier="Regional"
APPGW_PublicIP_AllocationMethod="Static"

APPGW_Name="app_gw-bea-non_prod_partenaire-nonprod-ne-$(echo $((100 + RANDOM % 1)))"
APPGW_SKU="WAF_v2"
APPGW_CAPACITY=2
APPGW_FE_PORT=80
APPGW_HTTP_SETTINGS_COOKIE_BASED_AFFINITY="Disabled"
APPGW_SETTINGS_PORT=80
APPGW_SETTINGS_PROTOCOL="Http"
```


### Creating the Resource Group
```bash
# Variables
RGName="rg-bea-non_prod_partenaire-non_prod-ne-$(echo $((100 + RANDOM % 1)))"
AzureRegion="westeurope"

az group create --name $RGName --location $AzureRegion
```

### Creating the Virtual Network and Subnets

#### Create the Virtual Network

```bash
# Variables
VNet_Name="vnet-bea-non_prod_partenaire-nonprod-ne-$(echo $((100 + RANDOM % 1)))"
VNet_AddressPrefix="10.5.0.0/20"

az network vnet create \
  --resource-group $RGName \
  --name $VNet_Name \
  --address-prefix $VNet_AddressPrefix
```

#### Creating the Subnets

##### Create the Application Gateway subnet

```bash
# Variables
APPGW_Subnet_Name="sub_agw_snet-bea-non_prod_partenaire-nonprod-ne-$(echo $((100 + RANDOM % 1)))"
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
WEB_Subnet_Name="sub_web-bea-non_prod_partenaire-nonprod-ne-$(echo $((100 + RANDOM % 1)))"
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
APP_Subnet_Name="sub_app-bea-non_prod_partenaire-nonprod-ne-$(echo $((100 + RANDOM % 1)))"
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
DB_Subnet_Name="sub_db-bea-non_prod_partenaire-nonprod-ne-$(echo $((100 + RANDOM % 1)))"
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
SFTP_Subnet_Name="sub_sftp-bea-non_prod_partenaire-nonprod-ne-$(echo $((100 + RANDOM % 1)))"
SFTP_Subnet_AddressPrefix=10.5.5.0/24

az network vnet subnet create \
  --resource-group $RGName \
  --vnet-name $VNet_Name \
  --name $DB_Subnet_Name \
  --address-prefix $DB_Subnet_AddressPrefix
```

##### Create the Bastion Host Subnet
```bash
# Variables
BASTION_Subnet_AddressPrefix="10.5.10.0/24"

az network vnet subnet create \
  --resource-group $RGName \
  --vnet-name $VNet_Name \
  --name AzureBastionSubnet \
  --address-prefix $BASTION_Subnet_AddressPrefix
```

#### Creating the NSGs

##### Create the App GW Subnet NSG

```bash
# Variables
NSG_AppGW_Name=nsg_azg-bea-non_prod_partenaire-nonprod-ne-$(echo $((100 + RANDOM % 1)))

# Create the NSG
az network nsg create \
  --resource-group $NSG_AppGW_Name \
  --name $NSG_Name

# Associate the NSG with the subnet
az network vnet subnet update \
  --resource-group $RGName \
  --vnet-name $VNet_name \
  --name $WEB_Subnet_Name \
  --network-security-group $NSG_Name
```

##### Create the App GW NSG Rules

| Rule Type | Direction | Source | Destination | Protocol | Port(s) | Purpose |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Azure Management** | Inbound | `GatewayManager` Service Tag | Any | TCP | **v1: 65503-65534**<br>**v2: 65200-65535** | Required for the Azure platform to manage the gateway. |
| **Client Traffic** | Inbound | `Internet` (or restricted IPs) | Any | TCP | **80, 443** (Your Listeners) | Allows web clients to reach the Application Gateway. |
| **Backend Communication** | Outbound | Any | **Backend Subnet CIDR** | TCP | **80, 443** (or custom port) | Allows the gateway to send traffic to the backend VMs. |
| **Internet Egress** | Outbound | Any | `Internet` Service Tag | Any | Any | **Required for v2 SKU** for various operational needs (e.g., CRL checks, updates). You **cannot** block all outbound internet access. |
| **Health Probes** | Inbound | `AzureLoadBalancer` Service Tag | Any | Any | Any | Ensures internal load balancer traffic (like health probes) is allowed. |


##### Create the WEB Subnet NSG

```bash
# Variables
NSG_Web_Name=nsg_web-bea-non_prod_partenaire-nonprod-ne-$(echo $((100 + RANDOM % 1)))

# Create the NSG
az network nsg create \
  --resource-group $RGName \
  --name $NSG_Web_Name

# Associate the NSG with the subnet
az network vnet subnet update \
  --resource-group $RGName \
  --vnet-name $VNet_name \
  --name $WEB_Subnet_Name \
  --network-security-group $NSG_Name
```

##### Create the Web NSG Rules

| Field | Configuration | Notes |
| :--- | :--- | :--- |
| **Source** | **IP Addresses** or **CIDR range** of the Application Gateway Subnet | This is the most critical step. The traffic originates from the Application Gateway's **internal (private) IP addresses**, which are within the range of its dedicated subnet. |
| **Source Port Ranges** | `*` (Any) | The Application Gateway uses dynamic source ports. |
| **Destination** | `Any` or **Application Security Group (ASG)** | If you use an ASG that contains your Web VM's NIC, it's a more flexible and robust solution than specifying individual VM IPs. |
| **Destination Port Ranges** | **`80`** (for HTTP) and/or **`443`** (for HTTPS) | This is the port your web application is listening on. If using end-to-end TLS, this will typically be `443`. |
| **Protocol** | **`TCP`** | Web traffic (HTTP/HTTPS) uses TCP. |
| **Action** | **`Allow`** | |
| **Priority** | A lower number (e.g., `100` or `110`) | Ensure this rule has a higher priority than the default "Deny all inbound" rule you should also have. |






### Deploying the Bastion Host 

#### Create the Bastion Public IP

```bash
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
az network bastion create \
  --name $BastionHostName \
  --resource-group $RGName \
  --vnet-name $VNet_Name \
  --location $AzureRegion \
  --public-ip-address $BastionPublicIPName \
  --sku $BastionSKU
```



### Application Gateway Deployment

#### Create the Application Gateway Public IP
```bash
az network public-ip create \
  --resource-group $RGName \
  --name $APPGWPublicIPName \
  --location $AzureRegion \
  --sku $APPGW_PublicIP_SKU \
  --tier $APPGW_PublicIP_Tier \
  --allocation-method $APPGW_PublicIP_AllocationMethod
```

#### Creating the Web Frontend Application Gateway
```bash
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



## Frontend Scale Set VMs Deployment

#### Create the VM Scale Set

```bash
az vmss create \
  --resource-group $RGName \
  --name $VMSS_Name \
  --image $VMSS_Image \
  --upgrade-policy-mode $VMSS_Upgrade_Policy \
  --admin-username $VMSS_Admin_UserName \
  --generate-ssh-keys \
  --vnet-name $VNet_Name \
  --subnet $WEB_Subnet_Name \
  --instance-count $VMSS_Instance_Count \
  --vm-sku $VMSS_SKU \
  --location $AzureRegion
```
#### Attach a Managed Disk to Each VM in the Scale Set
```bash
az vmss disk attach \
  --resource-group $RGName \
  --vmss-name $VMSS_Name \
  --size-gb 500 \
  --sku StandardSSD_ZRS
```


#### Link the the Application GW to the the VMSS backend pool:

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




## Testing Connectivity to Application Gateway and Checking Proper Routing



## Application Virtual Machines Deployment

## Testing Connectivity to the Application VMs

## Internal Load Balancer Deployment

## Testing Connectivity to Internal Load Balancer and Checking Proper Routing



## Azure Database for PostgresSQL Deployment 










