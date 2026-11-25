# Terraform Q&A

Questions are segregated by level. Answers are hidden by default — click the arrow to reveal each answer.

---

## Basic Level

<details>
<summary>1. What is Terraform?</summary>
Automates and manages the infrastructure, the platform & services that run on that platform. It's open-source and uses declarative language.
</details>

<details>
<summary>2. What are the main Terraform commands?</summary>
**Init**: Initializes the working directory for other Terraform commands (.terraform hidden file is created).  
**Fmt**: Formats the configuration properly.  
**Validate**: Checks whether the configuration is valid or not.  
**Plan**: Shows changes required by the current configuration.  
**Apply**: Creates or updates the infrastructure.  
**Destroy**: Destroys previously created infrastructure.  
**Output**: Shows important output variable info so team members can know the states easily.  
**Show**: Shows the current state of the saved plan.
</details>

<details>
<summary>3. How to lock/unlock Terraform state?</summary>
Azure Storage blobs are automatically locked before any operation that writes state. Terraform automatically locks the state file when you run commands that modify infrastructure (like `terraform apply`, `terraform plan`), preventing concurrent state operations. Manual locking: `terraform lock` and `terraform unlock [LOCK_ID]`.
</details>

<details>
<summary>4. What is a dynamic block in Terraform?</summary>
A dynamic block creates repeatable nested blocks within a resource. It is similar to the `for` expression and is useful when you need to create multiple instances of a block but don't know the exact number or properties in advance.
</details>

<details>
<summary>5. How do you create 10 Azure VMs in Terraform?</summary>
Use `count = 10` and name the VMs dynamically using `${count.index}`. Do the same for subnet and other modules as per requirement.
</details>

<details>
<summary>6. How to create an AKS cluster using Terraform?</summary>
Refer to HashiCorp's official AKS tutorial: https://developer.hashicorp.com/terraform/tutorials/kubernetes/aks
</details>

<details>
<summary>7. How can we export data from one module to another?</summary>
In Terraform, Outputs are used to define values that can be accessed by other modules. Define it in the module using the "output" block and reference it in the calling module using `module.MODULE_NAME.OUTPUT_NAME`.

Example:
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
An output variable is defined in resource configuration. Declare the output variable of module_A. Create a file variable.tf for module B. Establish the input variable inside this file having the same name as the key defined in module_B. Replicate the process for making variables available to other modules.
</details>

<details>
<summary>9. If I have created EC2 and VPC using Terraform and unfortunately the tfstate file got deleted, can you recover it? (File is only on the local machine, not on S3 or DynamoDB)</summary>
You can import the resources that are created by Terraform using the `terraform import` command, and then they will come into the state file:
```
terraform import <resource_type.resource_name> <real_resource_id>
```
</details>

<details>
<summary>10. What is the difference between terraform import and terraform taint?</summary>
**Terraform Taint** marks a resource as degraded or damaged, indicating that this resource will be destroyed and recreated during the next apply operation.  
**Terraform import** is used when you want to bring manually created resources under the Terraform umbrella.
</details>

<details>
<summary>11. How to get manually created AKS instances into Terraform?</summary>
Using the `terraform import` command we can bring manually created AKS instances into Terraform.  
Important: before running `terraform import`, we must manually write a resource configuration block for the resource.

Step 1: Get the resource group name & ID from Azure  
Step 2: Create import.tf file and mention (provider-specific identifier) & id (resource type and name)
```
import {
  id = "full_resource_id"
  to = resource_type.name
}
```
Step 3: Run `terraform plan` and `terraform apply`.  
Step 4: After applying, comment out the import block.
</details>

<details>
<summary>12. How do you manage secrets in Terraform without hardcoding them?</summary>
Secrets in Terraform are managed using Azure Key Vault, fetching them dynamically at runtime instead of hardcoding them in code or state files.
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
<summary>13. What's the difference between count and for_each? Give a real-world use case.</summary>
`count` is used when you want to create multiple identical resources.  
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
`for_each` is used when you want to create multiple resources that are similar but need unique configurations or keys.  
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
Use `count` for identical resources and `for_each` when each resource needs unique attributes or identifiers.
</details>

<details>
<summary>14. How do you handle drift detection in Terraform?</summary>
Drift detection in Terraform is done using the `terraform plan` command, which compares the real infrastructure in Azure with the state file. If Terraform detects differences (e.g., someone manually changed a resource in Azure), it shows those drifts in the plan output. You can then decide whether to fix it by re-applying the configuration (`terraform apply`) or updating the code and state to match the current setup.
</details>

<details>
<summary>15. What is a Terraform remote backend, and why is it important?</summary>
A Terraform remote backend is where Terraform stores its state file remotely instead of locally. It's important because it allows collaboration, consistency, and state locking to prevent multiple people or pipelines from changing the same infrastructure at once.

Example (Azure):
```
terraform {
  backend "azurerm" {
    resource_group_name   = "terraform-storage-rg"
    storage_account_name  = "terraformstatexlrwdrzs"
    container_name        = "tfstatefiles"
    key                   = "terraform.tfstate"
  }
}
```
</details>

<details>
<summary>16. Where do you store the Terraform state files?</summary>
In Azure Storage Account using a remote backend (azurerm).
</details>

<details>
<summary>17. How do you manage multiple environments (dev, staging, prod) in Terraform?</summary>
It depends on the requirement. If the team is big and wants strict isolation, keep separate statefiles for each environment. If the team is small, use a workspace to manage multiple environments.

Terraform workspaces provide isolation for different environments (dev, staging & prod).
- List: `$ terraform workspace list`
- Create: `$ terraform workspace new workspace_name`
- Switch: `$ terraform workspace select workspace_name`
- Apply with var files: `$ terraform apply -var-file='dev.tfvars' / -var-file='stag.tfvars'`

Note: terraform.tfstate.d folder gets created whenever a workspace is created.
</details>

<details>
<summary>18. Difference between local-exec and remote-exec provisioners.</summary>
`local-exec` runs commands locally on the Terraform host (e.g., Azure DevOps agent).
```
resource "azurerm_resource_group" "example" {
  name     = "demo-rg"
  location = "eastus"

  provisioner "local-exec" {
    command = "echo Resource Group ${self.name} created successfully"
  }
}
```
`remote-exec` runs commands remotely inside the created resource, such as an Azure VM.
```
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
```
</details>

<details>
<summary>19. How do you safely roll back infrastructure changes after a failed deployment?</summary>
Revert to a previous Git version and re-apply it using the remote state stored in Azure Storage, ensuring consistent and safe recovery. Example:
1. Check the Terraform state file in Azure Storage.
2. Roll back your code repository to the previous Git tag/commit (e.g., `git checkout release-v1.2`).
3. Run: `terraform init`, `terraform plan`, `terraform apply`.

This reapplies the last stable configuration, restoring the previous infrastructure setup.
</details>

<details>
<summary>20. Explain terraform refresh vs terraform plan.</summary>
`Terraform refresh` updates the state file to match real infrastructure.  
`Terraform plan` compares your code with the state to show what changes Terraform will make.
</details>

<details>
<summary>21. How do you write reusable Terraform modules?</summary>
Create reusable Terraform modules by encapsulating resources with variables and outputs, so the same module can be used across Azure environments just by changing inputs.
</details>

<details>
<summary>22. What is a null resource in Terraform?</summary>
A null resource in Terraform does not create or manage any infrastructure, and it is used as a placeholder or a trigger for other resources.
```
resource "null_resource" "example" {
  provisioner "local-exec" {
    command = "echo 'This is a test script'"
  }

  triggers = {
    always_run = "${timestamp()}"
  }
}
```
</details>

<details>
<summary>23. I want to put all the Terraform files in Git, from there all the infrastructure has to be managed. How do you achieve this?</summary>
Create a dedicated Git repository (e.g., `infrastructure`) to hold all your Terraform configuration files. Organize your files and directories for modularity and environment-specific configurations. Ensure sensitive data like secrets or private keys is not committed directly — use environment variables or secret management tools (like HashiCorp Vault or cloud provider-specific secret services).
</details>

<details>
<summary>24. Explain how you run Terraform in the pipeline?</summary>
</details>

<details>
<summary>25. Have you heard of HashiCorp Vault?</summary>
</details>

<details>
<summary>26. What kind of changes will you make to the Terraform code to use it in different regions and environments?</summary>
</details>

---

## Intermediate Level

<details>
<summary>27. Multiple team members are working on the same Terraform code and state — how do you prevent conflicts?</summary>
Use a remote backend with state locking (e.g., Azure Storage with blob locking) so only one Terraform operation can modify the state at a time. Combine with a CI/CD pipeline, code reviews, and separate workspaces/state files per environment.
</details>

<details>
<summary>28. Terraform apply failed in the middle of execution — how do you recover from a partial deployment?</summary>
Run `terraform plan` to see what changed and what didn't. Fix the issue in code, then run `terraform apply` again. Terraform is idempotent, so it will only apply the remaining changes. If a resource is stuck, use `terraform import` or `terraform state rm`/`terraform state mv` to clean up state, then re-apply.
</details>

<details>
<summary>29. Terraform shows drift between infrastructure and state — how do you identify and fix it?</summary>
Run `terraform plan` to identify drift — it will show resources that differ from the state. Fix by either re-applying the configuration (`terraform apply`) to bring infra back to code, or `terraform import` / state updates if the manual change is intended.
</details>

<details>
<summary>30. Terraform plan shows unexpected resource changes — how do you troubleshoot it?</summary>
Review the diff in `terraform plan`. Check if provider versions changed, if `count`/`for_each` changed, if tags/labels were modified, or if `lifecycle` blocks are set. Compare against the state file using `terraform show`.
</details>

<details>
<summary>31. Remote backend (like S3) is not accessible — how do you debug backend issues?</summary>
Check credentials (Azure CLI / service principal), network connectivity to the storage endpoint, container name, storage account key, and whether the backend block matches across all team members. Run `terraform init -reconfigure` if needed.
</details>

<details>
<summary>32. Terraform state file is locked — how do you resolve state lock issues?</summary>
Wait for the lock to release if another operation is running. If it's a stale lock, use `terraform unlock [LOCK_ID]` (Azure) or force-unlock depending on backend. Identify who holds the lock and coordinate before forcing.
</details>

<details>
<summary>33. Terraform resources are getting recreated unexpectedly — what could be the cause?</summary>
Causes include: changes in `count`/`for_each`, `lifecycle { create_before_destroy = true }`, provider version upgrades, changes in tags or computed values, or drift causing Terraform to think the resource doesn't match the config.
</details>

<details>
<summary>34. Sensitive values are visible in Terraform state — how do you secure them?</summary>
Mark variables as `sensitive = true`, use a remote backend with encryption at rest (Azure Storage encryption), use Azure Key Vault for secrets, and avoid printing sensitive outputs in CI/CD logs.
</details>

<details>
<summary>35. Terraform modules are not working as expected — how do you debug module issues?</summary>
Check module source paths, variable names and types, output references, and version constraints. Run `terraform init` to re-download modules, use `terraform console` to test expressions, and check module documentation.
</details>

<details>
<summary>36. A resource dependency is not respected — how do you handle dependency issues?</summary>
Use `depends_on` to explicitly declare dependencies when Terraform can't infer them implicitly. Check output references between resources and ensure proper ordering in the graph.
</details>

<details>
<summary>37. Terraform variables are not being picked correctly — how do you debug variable precedence?</summary>
Check variable precedence: command-line flags > tfvars files > environment variables > variable definition files > defaults. Use `terraform console` to evaluate variable values and `terraform plan -var-file` to override.
</details>

<details>
<summary>38. Terraform apply takes too long — how do you optimize performance?</summary>
Use `-target` to apply only specific resources, break monolithic configs into smaller modules/state files, use `parallelism` flag, and avoid unnecessary provisioners. Split environments into separate state files.
</details>

<details>
<summary>39. Infrastructure created by Terraform is not accessible after deployment — how do you troubleshoot?</summary>
Check NSG/firewall rules, public IP assignment, load balancer configuration, subnet routing, and DNS. Verify outputs match expectations and test connectivity from within the VNet.
</details>

<details>
<summary>40. Terraform workspace is not switching properly — how do you fix workspace-related issues?</summary>
Run `terraform workspace list` to confirm available workspaces. Use `terraform workspace select <name>` to switch. Ensure state files exist under `terraform.tfstate.d/` and there's no local `terraform.tfstate` conflicting.
</details>

<details>
<summary>41. Terraform provider version conflict occurs — how do you resolve provider dependency issues?</summary>
Align provider versions in the `required_providers` block across all modules. Run `terraform init -upgrade` to update the lock file, and pin versions to avoid unexpected breaking changes.
</details>

<details>
<summary>42. Terraform apply fails due to API rate limits — how do you handle throttling issues?</summary>
Reduce parallelism (`-parallelism=1`), add retries, implement exponential backoff, or split the apply into smaller batches using `-target`. Contact cloud provider support if limits need increasing.
</details>

<details>
<summary>43. Terraform cannot authenticate to cloud provider — how do you debug authentication failures?</summary>
Verify service principal credentials, client secret expiration, tenant ID, subscription ID, and assigned RBAC roles. Run `az login` or check environment variables (`ARM_CLIENT_ID`, `ARM_CLIENT_SECRET`, etc.).
</details>

<details>
<summary>44. Terraform plan works locally but fails in CI/CD pipeline — how do you troubleshoot?</summary>
Compare backend config, provider versions, and credentials between local and CI. Ensure secrets are injected as pipeline variables, the service principal has required permissions, and the working directory matches.
</details>

---

## Advanced / Production Troubleshooting

<details>
<summary>45. Resources are manually modified outside Terraform — how do you detect and fix configuration drift?</summary>
Run `terraform plan` to detect drift — it will show differences between real infrastructure and state. Fix by re-applying (`terraform apply`) to enforce desired state, or `terraform import` / state updates if manual changes are intended.
</details>

<details>
<summary>46. Terraform destroy fails due to dependencies — how do you resolve it safely?</summary>
Identify dependent resources using `terraform graph` or `terraform state list`. Remove dependencies manually or use `terraform state rm` to remove resources from state before destroying. Use `-target` to destroy in the correct order.
</details>

<details>
<summary>47. Remote state corruption occurs — how do you recover state safely?</summary>
Restore from a backup of the state file (if versioning is enabled on Azure Storage). Otherwise, use `terraform import` to bring existing resources back into a fresh state file.
</details>

<details>
<summary>48. Multiple environments (dev, stage, prod) need isolation — how do you design Terraform structure?</summary>
Use separate state files per environment with separate directories (e.g., `env/dev`, `env/stage`, `env/prod`) or Terraform workspaces. Use a remote backend per environment to prevent cross-environment state access.
</details>

<details>
<summary>49. Terraform outputs are not available after deployment — how do you debug?</summary>
Check if output blocks are defined in the root module, run `terraform output` to list them, and verify they aren't marked sensitive (which hides values in some CI tools). Ensure `terraform apply` completed successfully.
</details>

<details>
<summary>50. Infrastructure deployment fails due to incorrect IAM permissions — how do you troubleshoot?</summary>
Check the service principal's RBAC role assignments on the resource group/subscription. Verify required permissions (e.g., Contributor, User Access Administrator) and check Azure Activity Logs for denial errors.
</details>

<details>
<summary>51. Terraform cannot find required modules — how do you fix module source issues?</summary>
Verify the `source` path in module blocks, run `terraform init` to re-install modules, check `.terraform/modules` cache, and confirm module registry paths or Git URLs are accessible.
</details>

<details>
<summary>52. Changes are applied to wrong environment — how do you prevent such mistakes?</summary>
Use separate state files/backends per environment, require explicit `-var-file` selection, add pipeline approval gates, and use naming conventions/tags to visually distinguish environments. Avoid running apply from the wrong workspace directory.
</details>

<details>
<summary>53. Terraform rollback is required after a failed deployment — how do you handle rollback strategy?</summary>
Revert code to the last known good Git commit/tag and re-apply using the remote state. Alternatively, use `terraform state push` to restore a previous state snapshot, then `terraform apply` to match.
</details>

<details>
<summary>54. How do you troubleshoot a complete infrastructure deployment failure using Terraform?</summary>
1. Check `terraform plan` output for errors.  
2. Review cloud provider activity logs and error messages.  
3. Verify credentials, quotas, and API limits.  
4. Check dependent resources and networking.  
5. Fix code, re-run `terraform init`, `terraform plan`, then `terraform apply`.  
6. If needed, import existing resources or clean state with `terraform state rm`.
</details>

---

*Maintained for interview preparation. Answers are hidden by default — click any question to reveal.*