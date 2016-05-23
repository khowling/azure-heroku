Login-AzureRmAccount

Get-AzureRmSubscription

Select-AzureRmSubscription -subscriptionid 95efa97a-9b5d-4f74-9f75-a3396e23344d

[Test|New]-AzureRmResourceGroupDeployment -ResourceGroupName deisv2 -TemplateFile .\armtemplate.json -TemplateParameterFile  .\params.json

