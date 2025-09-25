# Azure App Deployment

## Environment Preparation

### Setting the Resources Names

```bash
RGName="rg-bea-non_prod_partenaire-non_prod-ne-$(echo $((100 + RANDOM % 1)))"
AzureRegion="northeurope"

VNetName="vnet-bea-non_prod_partenaire-nonprod-ne-$(echo $((100 + RANDOM % 1)))"
VNetAddressPrefix="10.5.0.0/20"

APPGW_Subnet_Name="app_gw_snet-bea-non_prod_partenaire-nonprod-ne-$(echo $((100 + RANDOM % 1)))"
BASTION_Subnet_Name="bst_snet-bea-non_prod_partenaire-nonprod-ne-$(echo $((100 + RANDOM % 1)))"
WEB_Subnet_Name="snet_web-bea-non_prod_partenaire-nonprod-ne-$(echo $((100 + RANDOM % 1)))"
APP_Subnet_Name="snet_app-bea-non_prod_partenaire-nonprod-ne-$(echo $((100 + RANDOM % 1)))"
DB_Subnet_Name="snet_db-bea-non_prod_partenaire-nonprod-ne-$(echo $((100 + RANDOM % 1)))"

APPGW_Subnet_AddressPrefix="10.5.1.0/24"
WEB_Subnet_AddressPrefix="10.5.2.0/24"
APP_Subnet_AddressPrefix="10.5.3.0/24"
DB_Subnet_AddressPrefix="10.5.4.0/24"
BASTION_Subnet_AddressPrefix="10.5.5.0/24"

APPGWName="app_gw-bea-non_prod_partenaire-nonprod-ne-$(echo $((100 + RANDOM % 1)))"
APPGWPublicIPName="app_gw_ip-bea-non_prod_partenaire-nonprod-ne-$(echo $((100 + RANDOM % 1)))"

BastionHostName="bst_host-bea-non_prod_partenaire-nonprod-ne-$(echo $((100 + RANDOM % 1)))"
BastionPublicIPName="bst_host_ip-bea-non_prod_partenaire-nonprod-ne-$(echo $((100 + RANDOM % 1)))"
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
  --name $BASTION_Subnet_Name \
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
  --name $DB_Subnet_30_Name \
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


## Frontend Virtual Machines Deployment

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




