# Azure App Deployment

## Environment Preparation

### Setting the Resources Names

```bash
RGName="rg-bea-non_prod_partenaire-non_prod-ne-$(echo $((100 + RANDOM % 1)))"
AzureRegion="westeurope"

VNetName="vnet-bea-non_prod_partenaire-nonprod-ne-$(echo $((100 + RANDOM % 1)))"
VNetAddressPrefix="10.5.0.0/20"

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



APPGW_PublicIP_Name="app_gw_ip-bea-non_prod_partenaire-nonprod-ne-$(echo $((100 + RANDOM % 1)))"
APPGW_PublicIP_SKU="Standard"
APPGW_PublicIP_Tier="Regional"
APPGW_PublicIP_AllocationMethod="Static"

APPGW_Name="app_gw-bea-non_prod_partenaire-nonprod-ne-$(echo $((100 + RANDOM % 1)))"
APPGW_SKU="WAF_v2"
APPGW_CAPACITY=2
APPGW_FE_PORT=80
APPGW_HTTP_SETTINGS_COOKIE_BASED_AFFINITY="Disabled"
APPGW_SETTINGS-PORT=80
APPGW_SETTINGS_PROTOCOL="Http"


BastionPublicIP_Name="bst_host_ip-bea-non_prod_partenaire-nonprod-ne-$(echo $((100 + RANDOM % 1)))"
Bastion_PublicIP_SKU="Standard"
Bastion_PublicIP_Tier="Regional"
Bastion_PublicIP_AllocationMethod="Static"


BastionName="bst_host-bea-non_prod_partenaire-nonprod-ne-$(echo $((100 + RANDOM % 1)))"
BastionSKU="Standard"

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
  --name $VNetName \
  --address-prefix $VNetAddressPrefix
```

#### Creating the Subnets

##### Create the Application Gateway subnet
```bash
az network vnet subnet create \
  --resource-group $RGName \
  --vnet-name $VNetName \
  --name $APPGW_Subnet_Name \
  --address-prefix $APPGW_Subnet_AddressPrefix
```

##### Create the Bastion Host Subnet
```bash
az network vnet subnet create \
  --resource-group $RGName \
  --vnet-name $VNetName \
  --name AzureBastionSubnet \
  --address-prefix $BASTION_Subnet_AddressPrefix
```

##### Create the WEB Subnet
```bash
az network vnet subnet create \
  --resource-group $RGName \
  --vnet-name $VNetName \
  --name $WEB_Subnet_Name \
  --address-prefix $WEB_Subnet_AddressPrefix
```

##### Create the APP Subnet
```bash
az network vnet subnet create \
  --resource-group $RGName \
  --vnet-name $VNetName \
  --name $APP_Subnet_Name \
  --address-prefix $APP_Subnet_AddressPrefix
```

##### Create the DB Subnet
```bash
az network vnet subnet create \
  --resource-group $RGName \
  --vnet-name $VNetName \
  --name $DB_Subnet_Name \
  --address-prefix $DB_Subnet_AddressPrefix
```


### Creating the Public IPs

#### Create the Application Gateway Public IP
```bash
az network public-ip create \
  --resource-group $RGName \
  --name $APPGWPublicIPName \
  --location $AzureRegion \
  --sku Standard \
  --tier Regional \
  --allocation-method Static
```

#### Create the Bastion Bastion Public IP
```bash
az network public-ip create \
  --resource-group $RGName \
  --name $BastionPublicIPName \
  --location $AzureRegion \
  --sku Standard \
  --tier Regional \
  --allocation-method Static
```

### Creating the Web Frontend Application Gateway
```bash
az network application-gateway create \
  --name $APPGW_Name \
  --location AzureRegion \
  --resource-group RGName \
  --sku $APPGW_SKU \
  --capacity $APPGW_CAPACITY \
  --frontend-port $APPGW_FE_PORT \
  --http-settings-cookie-based-affinity $APPGW_HTTP_SET_COOKIE_AFFINITY \
  --http-settings-port $APPGW_SET-PORT \
  --http-settings-protocol $APPGW_SET_PROTOCOL \
  --vnet-name $VNetName \
  --subnet APPGW_Subnet_Name \
  --public-ip-address APPGWPublicIPName
```





### Creating the Bastion Host
```bash
az network bastion create \
  --name $BastionHostName \
  --resource-group $RGName \
  --vnet-name $VNetName \
  --location $AzureRegion \
  --public-ip-address $BastionPublicIPName \
  --sku Standard
```




## Frontend Scale Set VMs Deployment

## Testing Connectivity to the Frontend VMs

## Application Gateway Deployment

## Testing Connectivity to Application Gateway and Checking Proper Routing



## Application Virtual Machines Deployment

## Testing Connectivity to the Application VMs

## Internal Load Balancer Deployment

## Testing Connectivity to Internal Load Balancer and Checking Proper Routing



## Azure Database for PostgresSQL Deployment 




## Bastion Host Deployment

```bash
az network bastion create \
  --name $BastionName \
  --resource-group $RGName \
  --location $Location \
  --vnet-name $VNetName \
  --public-ip-address $PublicIPName
```






