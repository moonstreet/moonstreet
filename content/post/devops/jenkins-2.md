---
title: "Jenkins shared library notes"
date: 2019-06-30T13:08:28+02:00
draft: false
image: ""
author: "Jacqueline"
---
# Resources

* KeyVault plugin
* az cli, jq, Terraform installed on the agent
* Pipeline utility steps plugin
* https://jenkins.io/doc/pipeline/steps/pipeline-utility-steps/
* https://jenkins.io/doc/book/pipeline/syntax/


# Create Azure storage with a Shared Library

This piece of code exposes some of the secrets of Jenkins pipeline and shared library.

* It logs in with an Azure Service Principal
* Creates a storage account
* Creates a container
* returns the access key

Make a file named vars/azureCreateStatefileStorage.groovy

```
def call(Map config) {
    echo "Creating storage for ${config.customerName)"
    script {
        withCredentials([azureServicePrincipal("${config.azureServicePrincipal}")]) {
            sh "az login --service-principal -u ${AZURE_CLIENT_ID} -p ${AZURE_CLIENT_SECRET} --tenant ${AZURE_TENANT_ID}"
            sh "az account set --subscription ${AZURE_SUBSCRIPTION_ID} "
            sh "az group create --name deploymentartifacts --location ${config.location}"
            sh "az storage account create --name ${config.deploymentStorageAccount} --resource-group deploymentartifacts --location ${config.location} --sku Standard_LRS --kind StorageV2"
            sh "az storage container create --account-name ${config.deploymentStorageAccount} --name terraformstate"
            accessKey = sh(returnStdout: true, script: "az storage account keys list -n ${config.deploymentStorageAccount} --query \"[0].value\" -o tsv")
            return accessKey
          }   
    }
}
```

Then call it like so

```
stage('Prepare statefile storage') {
    steps {
        script {
            accessKey = azureCreateStorage azureServicePrincipal: env.AZURE_SERVICE_PRINCIPAL,
                deploymentStorageAccount: "${env.CUSTOMER_NAME}artifacts",
                location: env.LOCATION
            echo accessKey
        }
    }
}
```

You need to set the environment variables as a string parameter in your pipeline definition.


# Replace tokens
