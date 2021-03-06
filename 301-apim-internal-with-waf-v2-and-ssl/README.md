# ARM Template to deploy WAF (v2) and Internal APIM

<IMG SRC="https://azurequickstartsservice.blob.core.windows.net/badges/301-apim-internal-with-waf-v2-and-ssl/PublicLastTestDate.svg" />&nbsp;
<IMG SRC="https://azurequickstartsservice.blob.core.windows.net/badges/301-apim-internal-with-waf-v2-and-ssl/PublicDeployment.svg" />&nbsp;

<IMG SRC="https://azurequickstartsservice.blob.core.windows.net/badges/301-apim-internal-with-waf-v2-and-ssl/FairfaxLastTestDate.svg" />&nbsp;
<IMG SRC="https://azurequickstartsservice.blob.core.windows.net/badges/301-apim-internal-with-waf-v2-and-ssl/FairfaxDeployment.svg" />&nbsp;

<IMG SRC="https://azurequickstartsservice.blob.core.windows.net/badges/301-apim-internal-with-waf-v2-and-ssl/BestPracticeResult.svg" />&nbsp;
<IMG SRC="https://azurequickstartsservice.blob.core.windows.net/badges/301-apim-internal-with-waf-v2-and-ssl/CredScanResult.svg" />&nbsp;

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F301-apim-internal-with-waf-v2-and-ssl%2Fazuredeploy.json" target="_blank">
<img src="https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/1-CONTRIBUTION-GUIDE/images/deploytoazure.png"/>
</a>
<a href="http://armviz.io/#/?load=https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F301-apim-internal-with-waf-v2-and-ssl%2Fazuredeploy.json" target="_blank">
<img src="https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/1-CONTRIBUTION-GUIDE/images/visualizebutton.png"/>
</a>

## Summary
With this ARM template, you will be able to deploy these resources:

![Architecture](docs/result.png)


## How to deploy it?
### Prerequisites

* A DNS Zone. In `azuredeploy.parameters.json`, you have to define `dns_zone` and `dns_resource_group` with DNS Zone information

* A Keyvault with a wildcard certificate for the DNS zone.
To create the wilcard certificate associated to the DNS zone, please follow [this tutorial](https://docs.microsoft.com/en-us/azure/app-service/configure-ssl-certificate#import-an-app-service-certificate).

Inside KeyVault, there are 2 configurations to do:
- Allow `Azure Resource Manager for template deployment`
```
az keyvault update  --name ExampleVault --enabled-for-template-deployment true
```

- Activate Azure Key Vault soft-delete with this command
```
az resource update --id $(az keyvault show --name ExampleVault -o tsv | awk '{print $1}') --set properties.enableSoftDelete=true
```

Once it's done, in `azuredeploy.parameters.json`, you can define this value:
```
"base64_encoded_pfx_wildcard_certificate": {
    "reference": {
    "keyVault": {
        "id": "/subscriptions/<subscriptionID>/resourceGroups/<resourceGroupName>/providers/Microsoft.KeyVault/vaults/<KeyVaultName>"
    },
    "secretName": "<SecretNameContainingCertificate>"
    }
}
```

### Run the script
Select your subscription:
```
az login
az account set --subscription 3edd151d-7da5-4bdc-bab2-c90a8da1c6ff
````

Create a resource group:
```
az group create --location westeurope --name sandbox-gateway --tags environment=sandbox provider="ARM Template"
```
Based on [Azure ARM templates best practices](https://docs.microsoft.com/en-gb/archive/blogs/mvpawardprogram/azure-resource-manager), it is handy to provision resources with the same lifecycle grouped into the same resource group.
Tags defined at Resource group level will be propagated to all resources.

Validate ARM template:
```
az group deployment validate -g sandbox-gateway --template-file azuredeploy.json --parameters @azuredeploy.parameters.json --handle-extended-json-format
```

Deploy ARM template:
```
az group deployment create -g sandbox-gateway --template-file azuredeploy.json --parameters @azuredeploy.parameters.json --handle-extended-json-format
```
