# Azure App Deployment

## Environment Preparation

### Set the Resources Names

```bash
$RGName = "RGName"
$AzureRegion = "westfrance"

$VNetName = "VNetName"
$VNetAddressPrefix = "10.0.0.0/16"
$WEB_Subnet_1 = "WEB_Subnet_1"
$APP_Subnet_2 = "APP_Subnet_2"
$DB_Subnet_3 = "DB_Subnet_3"
$WEB_Subnet_1_AddressPrefix = "10.1.0.0/24"
$APP_Subnet_2_AddressPrefix = "10.2.0.0/24"
$DB_Subnet_3_AddressPrefix = "10.3.0.0/24"

$BastionSubnet_AddressPrefix="10.0.10.0/27"
$BastionHostName = "BastionHostName"
$BastionPublicIPName = "BastionHost-IP"
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

#### Create the VMs Subnets

```bash
az network vnet subnet create \
  --resource-group $RGName \
  --vnet-name $VNetName \
  --name $WEB_Subnet_1 \
  --address-prefix $WEB_Subnet_1_AddressPrefix
```

```bash
    az network vnet subnet create \
  --resource-group $RGName \
  --vnet-name $VNetName \
  --name $APP_Subnet_2 \
  --address-prefix $APP_Subnet_1_AddressPrefix
```

```bash
    az network vnet subnet create \
  --resource-group $RGName \
  --vnet-name $VNetName \
  --name $DB_Subnet_3 \
  --address-prefix $DB_Subnet_1_AddressPrefix
```

#### Create the Bastion Subnets and Bastion Public IP

```bash
az network vnet subnet create \
  --resource-group $RGName \
  --vnet-name $VNetName \
  --name AzureBastionSubnet \
  --address-prefixes $BastionSubnet_AddressPrefix
```

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




