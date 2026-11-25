# Terraform Q&A

Questions are segregated by level. Answers are hidden by default — click the arrow to reveal each answer.

---

## Basic Level

<details>
<summary>1. What is Terraform?</summary>
- Automates and manages the infrastructure, the platform & services that run on that platform.
- It's open-source and uses declarative language.
</details>

<details>
<summary>2. What are the main Terraform commands?</summary>
- **Init**: Initializes the working directory for other Terraform commands (.terraform hidden file will be created).
- **Fmt**: Formats the configuration properly.
- **Validate**: Checks whether the configuration is valid or not.
- **Plan**: Shows changes required by the current configuration.
- **Apply**: Creates or updates the infrastructure.
- **Destroy**: Destroys previously created infrastructure.
- **Output**: Output variable file will show some important info so other members in the team can know the states easily.
- **Show**: Shows the current state of the saved plan.
</details>

<details>
<summary>3. How to lock/unlock Terraform state?</summary>
- Azure Storage blobs are automatically locked before any operation that writes state. This pattern prevents concurrent state operations, which can cause corruption.
- Terraform automatically locks the state file when you run commands that modify your infrastructure (like `terraform apply`, `terraform plan`, etc.). This prevents other Terraform processes from running simultaneously, ensuring consistency.
- Manual locking: `terraform lock` and `terraform unlock [LOCK_ID]`.
- Reference: https://learn.microsoft.com/en-us/azure/developer/terraform/store-state-in-azure-storage?tabs=azure-cli
</details>

<details>
<summary>4. What is a dynamic block in Terraform?</summary>
- A dynamic block creates repeatable nested blocks within a resource.
- It is similar to the `for` expression.
- Particularly useful when you need to create multiple instances of a block but don't know the exact number or properties in advance.
- The `dynamic` block simplifies configuration by enabling you to loop over a collection and create blocks conditionally.
</details>

<details>
<summary>5. How do you create 10 Azure VMs in Terraform?</summary>
- Use `count = 10` and name the VMs dynamically using `${count.index}`.
- Do the same for subnet and other modules as per the requirement.
</details>

<details>
<summary>6. How to create an AKS cluster using Terraform?</summary>
- Refer to HashiCorp's official AKS tutorial: https://developer.hashicorp.com/terraform/tutorials/kubernetes/aks
</details>

<details>
<summary>7. How can we export data from one module to another?</summary>
- In Terraform, Outputs are used to define values that can be accessed by other modules.
- Define the output in the module using the `output` block.
- Reference it in the calling module using the syntax `module.MODULE_NAME.OUTPUT_NAME`.

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
- An output variable is defined in resource configuration.
- Declare the output variable of module_A.
- Create a file variable.tf for module_B.
- Establish the input variable inside this file having the same name as the key defined in module_B.
- Replicate the process for making variables available to other modules.
</details>

<details>
<summary>9. If I have created EC2 and VPC using Terraform and unfortunately the tfstate file got deleted, can you recover it? (File is only on the local machine, not on S3 or DynamoDB)</summary>
- You can import the resources that are created by Terraform using the `terraform import` command, and then they will come into the state file.
```
terraform import <resource_type.resource_name> <real_resource_id>
```
</details>

<details>
<summary>10. I want to put all the Terraform files in Git, from there all the infrastructure has to be managed. How do you achieve this?</summary>
- Create a dedicated Git repository (e.g., `infrastructure`) to hold all your Terraform configuration files.
- Organize your files and directories for modularity and environment-specific configurations.
- Ensure sensitive data like secrets or private keys is not committed directly. Use environment variables or secret management tools (like HashiCorp Vault or cloud provider-specific secret services).
</details>

<details>
<summary>11. Explain how you run Terraform in the pipeline?</summary>
</details>

<details>
<summary>12. Have you heard of HashiCorp Vault?</summary>
</details>

<details>
<summary>13. What kind of changes will you make to the Terraform code to use it in different regions and environments?</summary>
</details>

---

## Intermediate Level

<details>
<summary>1. What is the difference between terraform import and terraform taint?</summary>
- **Terraform Taint** marks a resource as degraded or damaged, indicating that this resource will be destroyed and recreated during the next apply operation.
- **Terraform import** is used when you want to bring manually created resources under the Terraform umbrella.
</details>

<details>
<summary>2. How to get manually created AKS instances into Terraform?</summary>
- Using the `terraform import` command we can bring manually created AKS instances into Terraform.
- **Important**: Before running `terraform import`, we must manually write a resource configuration block for the resource. The resource block describes where Terraform should map the imported object.

**Steps:**
1. Get the resource group name & ID from Azure.
2. Create import.tf file and mention (provider-specific identifier) & id (resource type and name).
```
import {
  id = "full_resource_id"
  to = resource_type.name
}
```
3. Run `terraform plan` and `terraform apply`.
4. After applying, comment out the import block.
</details>

<details>
<summary>3. How do you manage secrets in Terraform without hardcoding them?</summary>
- Secrets in Terraform are managed using Azure Key Vault, fetching them dynamically at runtime instead of hardcoding them in the code or state files.
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
- **`count`** is used when you want to create multiple identical resources.
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
- **`for_each`** is used when you want to create multiple resources that are similar but need unique configurations or keys.
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
- Use `count` for identical resources and `for_each` when each resource needs unique attributes or identifiers.
</details>

<details>
<summary>5. How do you handle drift detection in Terraform?</summary>
- Drift detection in Terraform is done using the `terraform plan` command, which compares the real infrastructure in Azure with the state file.
- If Terraform detects differences (e.g., someone manually changed a resource in Azure), it shows those drifts in the plan output.
- You can then decide whether to fix it by re-applying the configuration (`terraform apply`) or updating the code and state to match the current setup.
</details>

<details>
<summary>6. What is a Terraform remote backend, and why is it important?</summary>
- A Terraform remote backend is where Terraform stores its state file remotely instead of locally.
- It's important because it allows **collaboration**, **consistency**, and **state locking** to prevent multiple people or pipelines from changing the same infrastructure at once.

**Example (Azure):**
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
<summary>7. Where do you store the Terraform state files?</summary>
- In Azure Storage Account using a remote backend (azurerm).
</details>

<details>
<summary>8. How do you manage multiple environments (dev, staging, prod) in Terraform?</summary>
- It depends on the requirement. If the team is big and wants strict isolation, keep separate statefiles for each environment. If the team is small, use a workspace to manage multiple environments.

**Terraform workspaces:**
- Terraform workspaces provide isolation for different environments (e.g., dev, staging & prod).
- List: `$ terraform workspace list`
- Create: `$ terraform workspace new workspace_name`
- Switch: `$ terraform workspace select workspace_name`
- Apply with var files: `$ terraform apply -var-file='dev.tfvars' / -var-file='stag.tfvars'`

**Defining workspace main.tf contents:**
```
resource "azurerm_instance" "azure-vm" {
  ami           = var.ami
  instance_type = var.instance_type

  tags = {
    "Name" = "myazurevm-${terraform.workspace}"
  }
}
```
- Note: `terraform.tfstate.d` folder gets created whenever a workspace is created.
</details>

<details>
<summary>9. Difference between local-exec and remote-exec provisioners.</summary>
- **`local-exec`**: Runs commands locally on the Terraform host (e.g., Azure DevOps agent).
```
resource "azurerm_resource_group" "example" {
  name     = "demo-rg"
  location = "eastus"

  provisioner "local-exec" {
    command = "echo Resource Group ${self.name} created successfully"
  }
}
```
- **`remote-exec`**: Runs commands remotely inside the created resource, such as an Azure VM.
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
<summary>10. How do you safely roll back infrastructure changes after a failed deployment?</summary>
- Revert to a previous Git version and re-apply it using the remote state stored in Azure Storage, ensuring consistent and safe recovery.
- Example:
  1. Check the Terraform state file in Azure Storage (remote backend).
  2. Roll back your code repository to the previous Git tag or commit (e.g., `git checkout release-v1.2`).
  3. Run: `terraform init`, `terraform plan`, `terraform apply`.
- This reapplies the last stable configuration, restoring the previous infrastructure setup.
</details>

<details>
<summary>11. Explain terraform refresh vs terraform plan.</summary>
- `Terraform refresh` updates the state file to match real infrastructure.
- `Terraform plan` compares your code with the state to show what changes Terraform will make.
</details>

<details>
<summary>12. How do you write reusable Terraform modules?</summary>
- Create reusable Terraform modules by encapsulating resources with variables and outputs, so the same module can be used across Azure environments just by changing inputs.
</details>

<details>
<summary>13. What is a null resource in Terraform?</summary>
- A null resource in Terraform does not create or manage any infrastructure, and it is used as a placeholder or a trigger for other resources.
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

---

## Advanced / Production Troubleshooting

<details>
<summary>1. What is Terraform and how does it manage infrastructure state?</summary>
</details>

<details>
<summary>2. Terraform state file is accidentally deleted — how would you recover the infrastructure state?</summary>
</details>

<details>
<summary>3. Multiple team members are working on the same Terraform code and state — how do you prevent conflicts?</summary>
</details>

<details>
<summary>4. Terraform apply failed in the middle of execution — how do you recover from a partial deployment?</summary>
</details>

<details>
<summary>5. Terraform shows drift between infrastructure and state — how do you identify and fix it?</summary>
</details>

<details>
<summary>6. Terraform plan shows unexpected resource changes — how do you troubleshoot it?</summary>
</details>

<details>
<summary>7. Remote backend (like S3) is not accessible — how do you debug backend issues?</summary>
</details>

<details>
<summary>8. Terraform state file is locked — how do you resolve state lock issues?</summary>
</details>

<details>
<summary>9. Terraform resources are getting recreated unexpectedly — what could be the cause?</summary>
</details>

<details>
<summary>10. Sensitive values are visible in Terraform state — how do you secure them?</summary>
</details>

<details>
<summary>11. Terraform modules are not working as expected — how do you debug module issues?</summary>
</details>

<details>
<summary>12. A resource dependency is not respected — how do you handle dependency issues?</summary>
</details>

<details>
<summary>13. Terraform variables are not being picked correctly — how do you debug variable precedence?</summary>
</details>

<details>
<summary>14. Terraform apply takes too long — how do you optimize performance?</summary>
</details>

<details>
<summary>15. Infrastructure created by Terraform is not accessible after deployment — how do you troubleshoot?</summary>
</details>

<details>
<summary>16. Terraform workspace is not switching properly — how do you fix workspace-related issues?</summary>
</details>

<details>
<summary>17. Terraform provider version conflict occurs — how do you resolve provider dependency issues?</summary>
</details>

<details>
<summary>18. Terraform apply fails due to API rate limits — how do you handle throttling issues?</summary>
</details>

<details>
<summary>19. Terraform cannot authenticate to cloud provider — how do you debug authentication failures?</summary>
</details>

<details>
<summary>20. Terraform plan works locally but fails in CI/CD pipeline — how do you troubleshoot?</summary>
</details>

<details>
<summary>21. Resources are manually modified outside Terraform — how do you detect and fix configuration drift?</summary>
</details>

<details>
<summary>22. Terraform destroy fails due to dependencies — how do you resolve it safely?</summary>
</details>

<details>
<summary>23. Remote state corruption occurs — how do you recover state safely?</summary>
</details>

<details>
<summary>24. Multiple environments (dev, stage, prod) need isolation — how do you design Terraform structure?</summary>
</details>

<details>
<summary>25. Terraform outputs are not available after deployment — how do you debug?</summary>
</details>

<details>
<summary>26. Infrastructure deployment fails due to incorrect IAM permissions — how do you troubleshoot?</summary>
</details>

<details>
<summary>27. Terraform cannot find required modules — how do you fix module source issues?</summary>
</details>

<details>
<summary>28. Changes are applied to wrong environment — how do you prevent such mistakes?</summary>
</details>

<details>
<summary>29. Terraform rollback is required after a failed deployment — how do you handle rollback strategy?</summary>
</details>

<details>
<summary>30. How do you troubleshoot a complete infrastructure deployment failure using Terraform?</summary>
</details>

---

*Maintained for interview preparation. Answers are hidden by default — click any question to reveal.*