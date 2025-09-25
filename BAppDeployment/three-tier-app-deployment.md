# Azure App Deployment

## Environment Preparation

### Set the Resources Names

```bash
RGName="rg-bea-non_prod_partenaire-non_prod-ne-$((001 + RANDOM % 700))"
AzureRegion="northeurope"

VNetName="vnet-bea-non_prod_partenaire-nonprod-ne-$((001 + RANDOM % 700))"
VNetAddressPrefix="10.5.0.0/20"

APPGW_Subnet_100_Name="app_gw_snet-bea-non_prod_partenaire-nonprod-ne-echo $((1 + RANDOM % 100))"
BASTION_Subnet_110_Name="bst_snet-bea-non_prod_partenaire-nonprod-ne-echo $((1 + RANDOM % 100))"
WEB_Subnet_10_Name="snet_web-bea-non_prod_partenaire-nonprod-ne-echo $((1 + RANDOM % 100))"
APP_Subnet_20_Name="snet_app-bea-non_prod_partenaire-nonprod-ne-echo $((1 + RANDOM % 100))"
DB_Subnet_30_Name="snet_db-bea-non_prod_partenaire-nonprod-ne-echo $((1 + RANDOM % 100))"

APPGW_Subnet_100_AddressPrefix="10.5.100.0/24"
BASTION_Subnet_110_AddressPrefix="10.5.110.0/24"
WEB_Subnet_10_AddressPrefix="10.5.10.0/24"
APP_Subnet_20_AddressPrefix="10.5.20.0/24"
DB_Subnet_30_AddressPrefix="10.5.30.0/24"

APPGWName = "app_gw-bea-non_prod_partenaire-nonprod-ne-echo $((1 + RANDOM % 100))"
APPGWPublicIPName = "app_gw_ip-bea-non_prod_partenaire-nonprod-ne-echo $((1 + RANDOM % 100))"



BastionHostName = "bst_host-bea-non_prod_partenaire-nonprod-ne-echo $((1 + RANDOM % 100))"
$BastionPublicIPName = "bst_ip-bea-non_prod_partenaire-nonprod-ne-echo $((1 + RANDOM % 100))"
```


### Create the Resource Group

```bash
az group create --name $RGName --location $AzureRegion
```

### Create the Virtual Network and Subnets

#### Create the Virtual Network
```bash
az network vnet create \
  --resource-group $RGName \
  --name $VNetName \
  --address-prefix $VNetAddressPrefix
```

#### Creating the VMs Subnets

##### Create the Application Gateway subnet
```bash
az network vnet subnet create \
  --resource-group $RGName \
  --vnet-name $VNetName \
  --name $APPGW_Subnet_100_Name \
  --address-prefix $APPGW_Subnet_100_AddressPrefix
```

##### Create the Bastion Host Subnet
```bash
az network vnet subnet create \
  --resource-group $RGName \
  --vnet-name $VNetName \
  --name BASTION_Subnet_10_Name \
  --address-prefix $BASTION_Subnet_110_AddressPrefix
```

##### Create the WEB Subnet
```bash
az network vnet subnet create \
  --resource-group $RGName \
  --vnet-name $VNetName \
  --name $WEB_Subnet_10_Name \
  --address-prefix $WEB_Subnet_10_AddressPrefix
```

##### Create the APP Subnet
```bash
    az network vnet subnet create \
  --resource-group $RGName \
  --vnet-name $VNetName \
  --name $APP_Subnet_20_Name \
  --address-prefix $APP_Subnet_20_AddressPrefix
```

##### Create the DB Subnet
```bash
    az network vnet subnet create \
  --resource-group $RGName \
  --vnet-name $VNetName \
  --name $DB_Subnet_30_Name \
  --address-prefix $DB_Subnet_30_AddressPrefix
```


### Creating the Public IPs

#### Create the Application Gateway Public IP
```bash
az network public-ip create \
  --resource-group $RGName \
  --name BastionPublicIPName \
  --location $AzureRegion \
  --sku Standard \
  --allocation-method Static
```

#### Create the Bastion Bastion Public IP
```bash
az network public-ip create \
  --resource-group $RGName \
  --name BastionPublicIPName \
  --location $AzureRegion \
  --sku Standard \
  --allocation-method Static
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




