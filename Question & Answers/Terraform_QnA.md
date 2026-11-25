# Terraform Q&A

Questions are segregated by level. Answers are hidden by default — click the question to reveal the answer.

---

## Basic Level

<details>
<summary>1. What is Terraform?</summary>
* Automates and manages the infrastructure, the platform & services that run on that platform.
* It's open-source and uses declarative language.
</details>

<details>
<summary>2. What are the main Terraform commands?</summary>
* **Init**: Initializes the working directory for other Terraform commands (.terraform hidden file is created).
* **Fmt**: Formats the configuration properly.
* **Validate**: Checks whether the configuration is valid or not.
* **Plan**: Shows changes required by the current configuration.
* **Apply**: Creates or updates the infrastructure.
* **Destroy**: Destroys previously created infrastructure.
* **Output**: Shows output variables so team members can know the states easily.
* **Show**: Shows the current state of the saved plan.
</details>

<details>
<summary>3. How to lock/unlock Terraform state?</summary>
* Azure Storage blobs are automatically locked before any operation that writes state. This prevents concurrent state operations and corruption.
* Terraform automatically locks the state file when you run commands that modify infrastructure (like `terraform apply`, `terraform plan`).
* Manual locking: `terraform lock` and `terraform unlock [LOCK_ID]`.
</details>

<details>
<summary>4. What is a dynamic block in Terraform?</summary>
* A dynamic block creates repeatable nested blocks within a resource.
* It is similar to the `for` expression.
* Useful when you need to create multiple instances of a block but don't know the exact number or properties in advance.
* It loops over a collection and creates blocks conditionally.
</details>

<details>
<summary>5. How do you create 10 Azure VMs in Terraform?</summary>
* Use `count = 10` and name the VMs dynamically using `${count.index}`.
* Apply the same pattern for subnet and other modules as per requirement.
</details>

<details>
<summary>6. How to create an AKS cluster using Terraform?</summary>
* Refer to HashiCorp's official AKS tutorial: https://developer.hashicorp.com/terraform/tutorials/kubernetes/aks
</details>

<details>
<summary>7. How can we export data from one module to another?</summary>
* In Terraform, Outputs are used to define values accessible by other modules.
* Define an output in the source module using the `output` block.
* Reference it in the calling module using `module.MODULE_NAME.OUTPUT_NAME`.

**Example:**

modules/vnet/outputs.tf:
```
output "vnet_id" {
  value = azurerm_virtual_network.main.id
}
```

main.tf:
```
module "vnet" {
  source = "./modules/vnet"
}
```
</details>

<details>
<summary>8. How will you make an object of one module available for the other module at a high level?</summary>
* Define an output variable in module_A.
* Create a `variable.tf` file for module_B with an input variable of the same name.
* Pass the output from module_A as input to module_B.
* Replicate the process to chain variables across modules.
</details>

<details>
<summary>9. If I have created EC2 and VPC using Terraform and the tfstate file got deleted, can I recover it? (File is only on the local machine, not on S3 or DynamoDB)</summary>
* Import the existing resources back into the state file using the `terraform import` command.
```
terraform import <resource_type.resource_name> <real_resource_id>
```
</details>

<details>
<summary>10. I want to put all the Terraform files in Git, from there all the infrastructure has to be managed. How do you achieve this?</summary>
* Create a dedicated Git repository (e.g., `infrastructure`) to hold all Terraform configuration files.
* Organize files and directories for modularity and environment-specific configurations.
* Do not commit secrets or private keys directly. Use environment variables or secret management tools (like HashiCorp Vault or cloud provider secret services).
</details>

<details>
<summary>11. Explain how you run Terraform in the pipeline?</summary>
* Store Terraform code in a Git repository.
* Use a CI/CD tool (Azure DevOps, GitHub Actions, Jenkins) to trigger the pipeline on code changes.
* The pipeline authenticates to the cloud provider using a service principal or managed identity.
* Run `terraform init` to initialize providers and backend.
* Run `terraform plan` to generate an execution plan for review.
* Run `terraform apply -auto-approve` to apply changes after approval.
* Store the state file in a remote backend (e.g., Azure Storage) with state locking enabled.
</details>

<details>
<summary>12. Have you heard of HashiCorp Vault?</summary>
* HashiCorp Vault is a secrets management tool used to securely store, access, and distribute secrets like passwords, API keys, and certificates.
* It provides encryption as a service, dynamic secrets, and lease management.
* Terraform can integrate with Vault to fetch secrets dynamically instead of hardcoding them in code or state files.
</details>

<details>
<summary>13. What kind of changes will you make to the Terraform code to use it in different regions and environments?</summary>
* Use variables for region, environment name, resource names, and tags so the same code can be reused.
* Use separate `.tfvars` files for each environment (e.g., `dev.tfvars`, `prod.tfvars`).
* Use Terraform workspaces or separate state files per environment to isolate state.
* Use conditional logic or modules to handle environment-specific differences (e.g., different VM sizes for dev vs prod).
</details>

---

## Intermediate Level

<details>
<summary>1. What is the difference between terraform import and terraform taint?</summary>
* **Terraform Taint** marks a resource as degraded or damaged, indicating it will be destroyed and recreated during the next apply operation.
* **Terraform import** is used to bring manually created resources under Terraform management.
</details>

<details>
<summary>2. How to get manually created AKS instances into Terraform?</summary>
* Use the `terraform import` command to bring manually created AKS instances into Terraform.
* Before importing, manually write a resource configuration block so Terraform knows where to map the imported object.

**Steps:**
1. Get the resource group name & ID from Azure.
2. Create `import.tf` and specify the ID and target resource.
```
import {
  id = "full_resource_id"
  to = azurerm_kubernetes_cluster.example
}
```
3. Run `terraform plan` and `terraform apply`.
4. After applying, comment out or remove the import block.
</details>

<details>
<summary>3. How do you manage secrets in Terraform without hardcoding them?</summary>
* Manage secrets using Azure Key Vault and fetch them dynamically at runtime instead of hardcoding them in code or state files.

**Example:**
```
data "azurerm_key_vault" "example" {
  name                = "my-keyvault"
  resource_group_name = "my-resource-group"
}

data "azurerm_key_vault_secret" "db_password" {
  name         = "db-password"
  key_vault_id = data.azurerm_key_vault.example.id
}

resource "azurerm_sql_server" "example" {
  name                         = "my-sql-server"
  resource_group_name          = "my-resource-group"
  location                     = "eastus"
  version                      = "12.0"
  administrator_login            = "adminuser"
  administrator_login_password   = data.azurerm_key_vault_secret.db_password.value
}
```
</details>

<details>
<summary>4. What's the difference between count and for_each? Give a real-world use case.</summary>
* **`count`**: Used to create multiple identical resources.
```
resource "azurerm_storage_account" "example" {
  count                    = 3
  name                     = "storage${count.index}"
  resource_group_name      = "rg-demo"
  location                 = "eastus"
  account_tier             = "Standard"
  account_replication_type = "LRS"
}
```
* **`for_each`**: Used to create multiple resources with unique configurations or keys.
```
variable "storage_accounts" {
  type = map(string)
  default = {
    sa1 = "eastus"
    sa2 = "westeurope"
  }
}

resource "azurerm_storage_account" "example" {
  for_each                 = var.storage_accounts
  name                     = each.key
  location                 = each.value
  resource_group_name      = "rg-demo"
  account_tier             = "Standard"
  account_replication_type = "LRS"
}
```
* Use `count` for identical resources; use `for_each` when each resource needs unique attributes or identifiers.
</details>

<details>
<summary>5. How do you handle drift detection in Terraform?</summary>
* Run `terraform plan` to compare the real infrastructure in Azure with the state file.
* Terraform shows drifts in the plan output if resources were manually changed outside Terraform.
* Fix drift by re-applying the configuration (`terraform apply`) to enforce the desired state, or update code/state to match the intended setup.
</details>
