# Azure App Deployment

## Environment Preparation

### Setting the Resources Names

```bash
RGName="rg-bea-non_prod_partenaire-non_prod-ne-$(echo $((100 + RANDOM % 1)))"
AzureRegion="westeurope"

VNet_Name="vnet-bea-non_prod_partenaire-nonprod-ne-$(echo $((100 + RANDOM % 1)))"
VNet_AddressPrefix="10.5.0.0/20"

APPGW_Subnet_Name="app_gw_snet-bea-non_prod_partenaire-nonprod-ne-$(echo $((100 + RANDOM % 1)))"
BASTION_Subnet_Name="AzureBastionSubnet"
WEB_Subnet_Name="snet_web-bea-non_prod_partenaire-nonprod-ne-$(echo $((100 + RANDOM % 1)))"
APP_Subnet_Name="snet_app-bea-non_prod_partenaire-nonprod-ne-$(echo $((100 + RANDOM % 1)))"
DB_Subnet_Name="snet_db-bea-non_prod_partenaire-nonprod-ne-$(echo $((100 + RANDOM % 1)))"

APPGW_Subnet_AddressPrefix="10.5.1.0/24"
WEB_Subnet_AddressPrefix="10.5.2.0/24"
APP_Subnet_AddressPrefix="10.5.3.0/24"
DB_Subnet_AddressPrefix="10.5.4.0/24"
BASTION_Subnet_AddressPrefix="10.5.5.0/24"



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
az group create --name $RGName --location $AzureRegion
```

### Creating the Virtual Network and Subnets

#### Create the Virtual Network
```bash
az network vnet create \
  --resource-group $RGName \
  --name $VNet_Name \
  --address-prefix $VNet_AddressPrefix
```

#### Creating the Subnets

##### Create the Application Gateway subnet
```bash
az network vnet subnet create \
  --resource-group $RGName \
  --vnet-name $VNet_Name \
  --name $APPGW_Subnet_Name \
  --address-prefix $APPGW_Subnet_AddressPrefix
```

##### Create the Bastion Host Subnet
```bash
az network vnet subnet create \
  --resource-group $RGName \
  --vnet-name $VNet_Name \
  --name AzureBastionSubnet \
  --address-prefix $BASTION_Subnet_AddressPrefix
```

##### Create the WEB Subnet
```bash
az network vnet subnet create \
  --resource-group $RGName \
  --vnet-name $VNet_Name \
  --name $WEB_Subnet_Name \
  --address-prefix $WEB_Subnet_AddressPrefix
```

##### Create the APP Subnet
```bash
az network vnet subnet create \
  --resource-group $RGName \
  --vnet-name $VNet_Name \
  --name $APP_Subnet_Name \
  --address-prefix $APP_Subnet_AddressPrefix
```

##### Create the DB Subnet
```bash
az network vnet subnet create \
  --resource-group $RGName \
  --vnet-name $VNet_Name \
  --name $DB_Subnet_Name \
  --address-prefix $DB_Subnet_AddressPrefix
```


### Deploying the Bastion Host 

#### Create the Bastion Public IP

```bash
az network public-ip create \
  --resource-group $RGName \
  --name $BastionPublicIPName \
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
  --location $AzureRegion \

```
#### Attach a Managed Disk to Each VM in the Scale Set
```bash
az vmss disk attach \
  --resource-group $RGName \
  --vmss-name $VMSS_Name \
  --size-gb 500 \
  --sku StandardSSD_ZRS
```


## Testing Connectivity via Bastion to the Frontend VMs

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










