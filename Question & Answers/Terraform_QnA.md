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
* **Init**: Initializes the working directory for other Terraform commands (.terraform hidden file will be created).
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
<pre>
output "vnet_id" {
  value = azurerm_virtual_network.main.id
}
</pre>

main.tf:
<pre>
module "vnet" {
  source = "./modules/vnet"
}
</pre>
</details>

<details>
<summary>8. How will you make an object of one module available for the other module at a high level?</summary>
* Define an output variable in module_A.
* Create a `variable.tf` file for module_B with an input variable of the same name.
* Pass the output from module_A as input to module_B.
* Replicate the process to chain variables across modules.
</details>

<details>
<summary>9. If I have created EC2 and VPC using Terraform and the tfstate file got deleted, can you recover it? (File is only on the local machine, not on S3 or DynamoDB)</summary>
* Import the existing resources back into the state file using the `terraform import` command.
<pre>
terraform import &lt;resource_type.resource_name&gt; &lt;real_resource_id&gt;
</pre>
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
<pre>
import {
  id = "full_resource_id"
  to = azurerm_kubernetes_cluster.example
}
</pre>
3. Run `terraform plan` and `terraform apply`.
4. After applying, comment out or remove the import block.
</details>

<details>
<summary>3. How do you manage secrets in Terraform without hardcoding them?</summary>
* Manage secrets using Azure Key Vault and fetch them dynamically at runtime instead of hardcoding them in code or state files.

**Example:**
<pre>
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
</pre>
</details>

<details>
<summary>4. What's the difference between count and for_each? Give a real-world use case.</summary>
* **`count`**: Used to create multiple identical resources.
<pre>
resource "azurerm_storage_account" "example" {
  count                    = 3
  name                     = "storage${count.index}"
  resource_group_name      = "rg-demo"
  location                 = "eastus"
  account_tier             = "Standard"
  account_replication_type = "LRS"
}
</pre>
* **`for_each`**: Used to create multiple resources with unique configurations or keys.
<pre>
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
</pre>
* Use `count` for identical resources; use `for_each` when each resource needs unique attributes or identifiers.
</details>

<details>
<summary>5. How do you handle drift detection in Terraform?</summary>
* Run `terraform plan` to compare the real infrastructure in Azure with the state file.
* Terraform shows drifts in the plan output if resources were manually changed outside Terraform.
* Fix drift by re-applying the configuration (`terraform apply`) to enforce the desired state, or update code/state to match the intended setup.
</details>

<details>
<summary>6. What is a Terraform remote backend, and why is it important?</summary>
* A Terraform remote backend stores the state file remotely instead of locally.
* It enables collaboration, consistency, and state locking to prevent multiple people or pipelines from modifying the same infrastructure simultaneously.

**Example (Azure):**
<pre>
terraform {
  backend "azurerm" {
    resource_group_name   = "terraform-storage-rg"
    storage_account_name  = "terraformstatexlrwdrzs"
    container_name        = "tfstatefiles"
    key                   = "terraform.tfstate"
  }
}
</pre>
</details>

<details>
<summary>7. Where do you store the Terraform state files?</summary>
* In an Azure Storage Account using a remote backend (azurerm).
</details>

<details>
<summary>8. How do you manage multiple environments (dev, staging, prod) in Terraform?</summary>
* Use separate state files for each environment if strict isolation is needed (recommended for large teams).
* Use Terraform workspaces for smaller teams to manage multiple environments within the same configuration.

**Workspace commands:**
<pre>
$ terraform workspace list
$ terraform workspace new workspace_name
$ terraform workspace select workspace_name
$ terraform apply -var-file='dev.tfvars' / -var-file='stag.tfvars'
</pre>
* The `terraform.tfstate.d` folder contains isolated state files per workspace.
</details>

<details>
<summary>9. Difference between local-exec and remote-exec provisioners.</summary>
* **`local-exec`**: Runs commands locally on the machine where Terraform is executed (e.g., Azure DevOps agent).
<pre>
resource "azurerm_resource_group" "example" {
  name     = "demo-rg"
  location = "eastus"

  provisioner "local-exec" {
    command = "echo Resource Group ${self.name} created successfully"
  }
}
</pre>
* **`remote-exec`**: Runs commands remotely inside the created resource, such as an Azure VM.
<pre>
resource "azurerm_linux_virtual_machine" "vm" {
  name                = "demo-vm"
  resource_group_name = "demo-rg"
  location            = "eastus"
  size                = "Standard_B1s"
  admin_username      = "azureuser"
  network_interface_ids = [azurerm_network_interface.example.id]

  admin_ssh_key {
    username   = "azureuser"
    public_key = file("~/.ssh/id_rsa.pub")
  }

  provisioner "remote-exec" {
    inline = [
      "sudo apt-get update -y",
      "sudo apt-get install nginx -y"
    ]
  }
}
</pre>
</details>

<details>
<summary>10. How do you safely roll back infrastructure changes after a failed deployment?</summary>
* Revert the Terraform code to the previous stable Git commit/tag.
* Re-apply using the remote state stored in Azure Storage.

**Steps:**
1. Check the Terraform state file in Azure Storage (remote backend).
2. Roll back the code repository: `git checkout release-v1.2`.
3. Run: `terraform init`, `terraform plan`, `terraform apply`.
* This reapplies the last stable configuration and restores the previous infrastructure setup.
</details>

<details>
<summary>11. Explain terraform refresh vs terraform plan.</summary>
* `terraform refresh` updates the state file to match the real infrastructure.
* `terraform plan` compares your code with the state to show what changes Terraform will make.
</details>

<details>
<summary>12. How do you write reusable Terraform modules?</summary>
* Encapsulate resources with input variables and output values.
* Design modules so they can be used across environments by changing only the inputs.
* Version modules and store them in a module registry or Git repository.
</details>

<details>
<summary>13. What is a null resource in Terraform?</summary>
* A null resource does not create or manage any infrastructure.
* It is used as a placeholder or a trigger for other resources (e.g., running scripts).

**Example:**
<pre>
resource "null_resource" "example" {
  provisioner "local-exec" {
    command = "echo 'This is a test script'"
  }

  triggers = {
    always_run = "${timestamp()}"
  }
}
</pre>
</details>

---

## Advanced / Production Troubleshooting

<details>
<summary>1. Multiple team members are working on the same Terraform code and state — how do you prevent conflicts?</summary>
* Use a remote backend with state locking (e.g., Azure Storage) so only one operation can modify state at a time.
* Use a CI/CD pipeline with code reviews and pull request approvals.
* Use separate workspaces or state files per environment/team to isolate changes.
* Communicate planned changes and avoid running `terraform apply` simultaneously.
</details>

<details>
<summary>2. Terraform apply failed in the middle of execution — how do you recover from a partial deployment?</summary>
* Run `terraform plan` to see what was applied and what remains.
* Fix the issue in code or configuration.
* Re-run `terraform apply` — Terraform is idempotent and will only apply remaining changes.
* If a resource is stuck, use `terraform state rm` or `terraform state mv` to clean up state, then re-apply.
</details>

<details>
<summary>3. Terraform plan shows unexpected resource changes — how do you troubleshoot it?</summary>
* Review the diff in `terraform plan` output.
* Check if provider versions changed, `count`/`for_each` changed, or tags/labels were modified.
* Look for `lifecycle` blocks like `create_before_destroy`.
* Compare against state using `terraform show`.
</details>

<details>
<summary>4. Remote backend is not accessible — how do you debug backend issues?</summary>
* Verify cloud credentials (Azure CLI, service principal, or managed identity).
* Check network connectivity to the storage endpoint.
* Validate container name, storage account name, and access keys in the backend block.
* Ensure the backend configuration matches across all team members.
* Run `terraform init -reconfigure` if backend settings changed.
</details>

<details>
<summary>5. Terraform resources are getting recreated unexpectedly — what could be the cause?</summary>
* Changes in `count` or `for_each` can cause resource replacement.
* `lifecycle { create_before_destroy = true }` triggers replacement.
* Provider version upgrades can change resource behavior.
* Changes to tags, computed values, or drift can make Terraform think the resource doesn't match config.
</details>

<details>
<summary>6. Sensitive values are visible in Terraform state — how do you secure them?</summary>
* Mark variables as `sensitive = true` so they are not shown in CLI output.
* Use a remote backend with encryption at rest (e.g., Azure Storage encryption).
* Fetch secrets from Azure Key Vault or HashiCorp Vault at runtime.
* Avoid printing sensitive outputs in CI/CD logs.
</details>

<details>
<summary>7. Terraform modules are not working as expected — how do you debug module issues?</summary>
* Verify module source paths are correct and accessible.
* Check variable names, types, and default values match between caller and module.
* Confirm output references are correctly named.
* Run `terraform init` to re-download modules.
* Use `terraform console` to test expressions and variable values.
</details>

<details>
<summary>8. A resource dependency is not respected — how do you handle dependency issues?</summary>
* Use `depends_on` to explicitly declare dependencies when Terraform cannot infer them implicitly.
* Check output references between resources to ensure proper ordering in the dependency graph.
* Review resource creation order in `terraform plan` output.
</details>

<details>
<summary>9. Terraform variables are not being picked correctly — how do you debug variable precedence?</summary>
* Check variable precedence order: command-line flags > tfvars files > environment variables > variable definition files > defaults.
* Use `terraform console` to evaluate variable values.
* Run `terraform plan -var-file` to override with a specific tfvars file.
* Verify there are no conflicting values in multiple sources.
</details>

<details>
<summary>10. Terraform apply takes too long — how do you optimize performance?</summary>
* Break monolithic configs into smaller modules with separate state files.
* Use `-target` to apply only specific resources.
* Adjust the `-parallelism` flag to control concurrent operations.
* Avoid unnecessary provisioners and long-running inline scripts.
* Split environments into isolated state files to reduce state size.
</details>

<details>
<summary>11. Infrastructure created by Terraform is not accessible after deployment — how do you troubleshoot?</summary>
* Check Network Security Groups (NSGs) and firewall rules.
* Verify public IP assignment and load balancer configuration.
* Confirm subnet routing and DNS settings.
* Check Terraform outputs to ensure expected endpoints and IPs were created.
* Test connectivity from within the virtual network.
</details>

<details>
<summary>12. Terraform workspace is not switching properly — how do you fix workspace-related issues?</summary>
* Run `terraform workspace list` to confirm available workspaces.
* Use `terraform workspace select <name>` to switch to the desired workspace.
* Ensure state files exist under `terraform.tfstate.d/`.
* Check that no local `terraform.tfstate` is conflicting with workspace state.
</details>

<details>
<summary>13. Terraform provider version conflict occurs — how do you resolve provider dependency issues?</summary>
* Align provider versions in the `required_providers` block across all modules.
* Pin provider versions to avoid unexpected breaking changes.
* Run `terraform init -upgrade` to update the provider lock file.
* Verify all modules use compatible provider constraints.
</details>

<details>
<summary>14. Terraform apply fails due to API rate limits — how do you handle throttling issues?</summary>
* Reduce parallelism using `-parallelism=1` to slow down API calls.
* Implement retries with exponential backoff in your CI/CD pipeline.
* Split the apply into smaller batches using `-target`.
* Contact the cloud provider to request higher API rate limits if needed.
</details>

<details>
<summary>15. Terraform cannot authenticate to cloud provider — how do you debug authentication failures?</summary>
* Verify service principal credentials (client ID, client secret, tenant ID, subscription ID).
* Check that the service principal has the required RBAC roles assigned.
* Confirm client secrets are not expired.
* Check environment variables (`ARM_CLIENT_ID`, `ARM_CLIENT_SECRET`, etc.) or pipeline secrets.
* Review Azure Activity Logs for authentication denial errors.
</details>

<details>
<summary>16. Terraform plan works locally but fails in CI/CD pipeline — how do you troubleshoot?</summary>
* Compare backend configuration, provider versions, and credentials between local and CI environments.
* Ensure secrets are injected as pipeline variables.
* Verify the service principal has required permissions in CI.
* Confirm the working directory and module paths match your local setup.
* Check pipeline logs for specific authentication or network errors.
</details>

<details>
<summary>17. Terraform destroy fails due to dependencies — how do you resolve it safely?</summary>
* Identify dependent resources using `terraform graph` or `terraform state list`.
* Remove dependent resources manually or use `terraform state rm` to remove them from state before destroying.
* Use `-target` to destroy resources in the correct dependency order.
</details>

<details>
<summary>18. Remote state corruption occurs — how do you recover state safely?</summary>
* Restore from a backup of the state file if versioning is enabled on Azure Storage.
* If no backup exists, create a fresh state file and use `terraform import` to bring existing resources back under management.
* Validate the recovered state with `terraform plan` before applying changes.
</details>

<details>
<summary>19. Terraform outputs are not available after deployment — how do you debug?</summary>
* Check if output blocks are defined in the root module.
* Run `terraform output` to list available outputs.
* Verify outputs are not marked `sensitive = true` (which hides values in some CI tools).
* Confirm `terraform apply` completed successfully without errors.
</details>

<details>
<summary>20. Infrastructure deployment fails due to incorrect IAM permissions — how do you troubleshoot?</summary>
* Check the service principal's RBAC role assignments on the resource group or subscription.
* Verify required permissions (e.g., Contributor, User Access Administrator).
* Review Azure Activity Logs for authorization denial errors.
* Grant missing roles and re-run `terraform apply`.
</details>

<details>
<summary>21. Terraform cannot find required modules — how do you fix module source issues?</summary>
* Verify the `source` path in module blocks is correct.
* Run `terraform init` to re-install modules.
* Check the `.terraform/modules` cache for downloaded modules.
* Confirm module registry paths or Git URLs are accessible and valid.
</details>

<details>
<summary>22. Changes are applied to wrong environment — how do you prevent such mistakes?</summary>
* Use separate state files and backends per environment.
* Require explicit `-var-file` selection in pipelines.
* Add manual approval gates in CI/CD before applying to production.
* Use clear naming conventions and tags to visually distinguish environments.
</details>

<details>
<summary>23. How do you troubleshoot a complete infrastructure deployment failure using Terraform?</summary>
* Check `terraform plan` output for errors or unexpected changes.
* Review cloud provider activity logs and error messages.
* Verify credentials, quotas, and API rate limits.
* Check dependent resources and networking configuration.
* Fix the code, then run `terraform init`, `terraform plan`, and `terraform apply`.
* If needed, import existing resources or clean state with `terraform state rm`.
</details>

---

*Maintained for interview preparation. Answers are hidden by default — click any question to reveal.*
