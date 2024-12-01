# contoso-air

## Prerequisites

- Node.js 22.0.0 or later

## Getting Started

Create an Azure CosmosDB account and export the account name and access key as environment variables:

```bash
# create random resource identifier
RAND=$RANDOM
export RAND
echo "Random resource identifier will be: ${RAND}"

# set variables
AZURE_SUBSCRIPTION_ID=$(az account show --query id -o tsv)
AZURE_RESOURCE_GROUP_NAME=rg-aks-auto-swc-20241129 # rg-mita-contosoair$RAND
AZURE_COSMOS_ACCOUNT_NAME=db-contosoair$RAND
AZURE_REGION=swedencentral

# create resource group
az group create \
--name $AZURE_RESOURCE_GROUP_NAME \
--location $AZURE_REGION

# create cosmosdb account
AZURE_COSMOS_ACCOUNT_ID=$(az cosmosdb create \
--name $AZURE_COSMOS_ACCOUNT_NAME \
--resource-group $AZURE_RESOURCE_GROUP_NAME \
--kind MongoDB \
--query id -o tsv)

# create managed identity
AZURE_COSMOS_IDENTITY_ID=$(az identity create \
--name db-contosoair$RAND-id \
--resource-group $AZURE_RESOURCE_GROUP_NAME \
--query id -o tsv)

# get managed identity principal id
AZURE_COSMOS_IDENTITY_PRINCIPAL_ID=$(az identity show \
--ids $AZURE_COSMOS_IDENTITY_ID \
--query principalId \
-o tsv)

# assign role to managed identity
az role assignment create \
--role "DocumentDB Account Contributor" \
--assignee $AZURE_COSMOS_IDENTITY_PRINCIPAL_ID \
--scope $AZURE_COSMOS_ACCOUNT_ID

# export variables for azure identity auth
export AZURE_COSMOS_CLIENTID=$(az identity show \
--ids $AZURE_COSMOS_IDENTITY_ID \
--query clientId \
-o tsv)
export AZURE_COSMOS_LISTCONNECTIONSTRINGURL=https://management.azure.com/subscriptions/$AZURE_SUBSCRIPTION_ID/resourceGroups/$AZURE_RESOURCE_GROUP_NAME/providers/Microsoft.DocumentDB/databaseAccounts/$AZURE_COSMOS_ACCOUNT_NAME/listConnectionStrings?api-version=2021-04-15
export AZURE_COSMOS_SCOPE=https://management.azure.com/.default
```

Clone the repository then run the following commands:

```bash
# change directory
cd src/web

# install dependencies
npm install

# run the app
npm start
```

Browse to `http://localhost:3000` to see the app.

## Cleanup

```bash
az group delete --name $AZURE_RESOURCE_GROUP_NAME --yes --no-wait
```
