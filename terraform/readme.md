## The Backend Configuration ##

This implementation of Terraform uses a backend state file stored in a secure location with a Storage Account.

Rather than stating the access details to this location inline, a set of environment variables should be defined and passed through the initialisation, plan and apply stages.

### Setting-Up Backend - with Set-TF module ###

* Import the Module
> Import-Module ./set-tf.psm1

* Run the setup.  Syntax is as follows: Set-TF -env *your_environment* -keyvault *your_keyvault_name*
>Set-TF -env dev -keyvault markpatton-cloud-keyvault

### Environment Variables ###
Assigning the environment variables can be carried-out as follows:

``` powershell
$ENV:access_key=(az keyvault secret show --vault-name <<<ENTER_VAULT_NAME>>> --name tfstateaccesskey --query value --output tsv) 

$ENV:key=(az keyvault secret show --vault-name <<<ENTER_VAULT_NAME>>> --name tfstatefile --query value --output tsv)

$ENV:storage_account_name=(az keyvault secret show --vault-name <<<ENTER_VAULT_NAME>>> --name tfstatestorageaccountname --query value --output tsv)
```

Note that the state.tf file has been assigned default values rather than including the sensitive credentials inline:

```terraform
terraform {
  backend "azurerm" {
    storage_account_name = "default"
    container_name       = "default"
    key                  = "default"
    access_key           = "default"
  }
}
```

### Module Reference ###
In order to assign variables in the ***Backend*** block, they have to be declared in the root module.  In this case, you will find these referenced in the ***main.tf*** located at:
> environments/master


```terraform
variable "storage_account_name" {
    default = "default"
}

variable "container_name" {
    default = "dev"
}

variable "key" {
    default = "default"
}

variable "access_key" {
    default = "default"
}
```

### Terraform - Initialising with Environment Variables ###

In order to initialise the backend, in addition to the modules and the provider plugins, the following command needs to be run by passing the Environment variables:
```powershell
terraform init -backend-config="access_key=$ENV:access_key" -backend-config="storage_account_name=$ENV:storage_account_name" -backend-config="key=$ENV:key" -backend-config=“$ENV:container_name”
 ```

### Terraform - Running a plan or apply with Environment Variables ###

```powershell
terraform plan -var="access_key=$ENV:access_key,key=$ENV:key,storage_account_name=$ENV:storage_account_name,container_name=$ENV:container_name"
```