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
    withCredentials([azureServicePrincipal("${config.azureServicePrincipal}")]) {
        sh "az login --service-principal -u ${AZURE_CLIENT_ID} -p ${AZURE_CLIENT_SECRET} --tenant ${AZURE_TENANT_ID}"
        sh "az account set --subscription ${AZURE_SUBSCRIPTION_ID} "
        sh "az group create --name deploymentartifacts --location ${config.location}"
        sh "az storage account create --name ${config.deploymentStorageAccount} --resource-group deploymentartifacts --location ${config.location} --sku Standard_LRS --kind StorageV2"
        sh "az storage container create --account-name ${config.deploymentStorageAccount} --name terraformstate"
        accessKey = sh(returnStdout: true, script: "az storage account keys list -n ${config.deploymentStorageAccount} --query \"[0].value\" -o tsv | tr -d '\n'")
        return accessKey
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

This is a function that replaces tokens with the StrSubstitutor. The SimpleTemplateEngine doesn't work (https://issues.jenkins-ci.org/browse/JENKINS-38432)

```
def call(String text, Map binding) {
    _tokenize(text, binding)
}

@NonCPS
def _tokenize(String text, Map binding){
    def engine = new org.apache.commons.lang3.text.StrSubstitutor(binding)
    def s = engine.replace(text)
    engine = null
    return s
}
```

Then in your Pipeline:

```
stage('Replace tokens for backend') {
steps {
    script {
        text = readFile("${FOLDER}/backend.txt")
        def binding = [:]
        binding.StorageAccountName = "${env.CUSTOMER_NAME}artifacts"                         
        binding.StorageAccessKey = accessKey
        binding.StateFileName = "${env.CUSTOMER_NAME}${env.TELEPHONY_PLATFORM}${env.ENVIRONMENT_TO_BUILD}artifacts"
        writeFile(file: "${FOLDER}/backend.tf", text: tokenize(text, binding)
        )
        sh "cat ${FOLDER}/backend.tf"  
      }
   }
}
```


# Credentials

Example of credentials

```
stage('Prepare') {
  steps {
      script {
          withCredentials([file(credentialsId: env.TFVARS, variable: 'tfvars')]) {
              sh 'cp $tfvars terraform.tfvars.json'
          }
          withCredentials([azureServicePrincipal("${env.AZURE_PRINCIPAL}")]) {
              vars = readJSON file: 'terraform.tfvars.json'
              vars['az_subscription_id'] = $AZURE_SUBSCRIPTION_ID
              vars['az_client_id'] = $AZURE_CLIENT_ID
              vars['az_secret'] = $AZURE_CLIENT_SECRET
              vars['az_tenant_id'] = $AZURE_TENANT_ID
              writeJSON file: 'terraform.tfvars.json', json: vars
         }
         withCredentials([file(credentialsId: "${params.CONFIG}", variable: 'Configuration')]) {
              sh 'cp $Configuration configuration.json'
              vars = readJSON file: 'configuration.json'
              STORAGE = "${vars['az_storage_account_name']}"
      }
    }
  }
}
```

# From Azure KeyVault

```
stage('Fetch Terraform vars from KeyVault') {
    steps {
        script {
            def secrets = [
                [ secretType: 'Secret', name: "Secret001", version: '', envVariable: 'SECRET' ]
            ]
            withAzureKeyvault(secrets) {
                sh "echo $SECRET > ${FOLDER}/terraform.tfvars.json"
                vars = readJSON file:"${FOLDER}/terraform.tfvars.json"
                //sh "cat ${FOLDER}/terraform.tfvars.json"
            }
        }
    }
}
```
