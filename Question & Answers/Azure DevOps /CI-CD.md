# Azure DevOps / CI-CD

<details>
<summary>1. You have a .NET application that needs to be deployed to Azure. Describe the steps to set up a CI/CD pipeline in Azure DevOps.</summary>

→ **Step 1**: Create a project and repo in Azure DevOps; push code.

→ **Step 2**: Create a Service Connection to Azure (service principal or managed identity).

→ **Step 3**: Build pipeline (`azure-pipelines.yml`):
<pre>
# azure-pipelines.yml (CI)
trigger:
- main

pool:
  vmImage: ubuntu-latest

steps:
- task: UseDotNet@2
  inputs:
    packageType: 'sdk'
    version: '8.x'
- script: dotnet restore
- script: dotnet build --configuration Release
- script: dotnet test
- task: PublishBuildArtifacts@1
  inputs:
    PathtoPublish: '$(Build.ArtifactStagingDirectory)'
    ArtifactName: 'drop'
</pre>

→ **Step 4**: Release pipeline (`azure-pipelines-cd.yml`) with stages for dev → staging → prod, using Azure App Service deploy task.

→ **Step 5**: Gate prod with approvals; use variable groups linked to Key Vault.

</details>

<details>
<summary>2. Your team is working on a project with multiple microservices. How would you structure your Azure DevOps repository to manage these microservices efficiently?</summary>

→ **Option 1 — Mono-repo**: One repo, one folder per microservice. Path-filtered pipelines.
<pre>
/services/api
/services/web
/services/worker
/shared
/services/*/azure-pipelines.yml
</pre>

→ **Option 2 — Poly-repo**: One repo per microservice, each with its own pipeline.

→ Use a shared templates repo for common build/deploy logic.

→ Each microservice owns its versioning and release cadence.

</details>

<details>
<summary>3. What are Azure DevOps service connections, and how would you use them in a CI/CD pipeline?</summary>

→ A service connection stores credentials/config for Azure DevOps to talk to external systems (Azure subscription, GitHub, Docker registry, etc.).

→ You create them in **Project Settings → Service Connections**.

→ Use them in pipeline tasks via the connection name.

<pre>
- task: AzureWebApp@1
  inputs:
    azureSubscription: 'my-azure-connection'
    appName: 'my-app'
</pre>

→ Best practice: use workload identity federation (OIDC) instead of secrets.

</details>

<details>
<summary>4. Explain the purpose of an ARM template and how you can incorporate it into your Azure DevOps pipeline.</summary>

→ ARM (Azure Resource Manager) templates are declarative JSON files that define Azure infrastructure.

→ Key benefits over manual CLI commands: repeatability, idempotency, and version control.

→ Use them in pipelines via the `AzureResourceManagerTemplateDeployment` task:
<pre>
- task: AzureResourceManagerTemplateDeployment@3
  inputs:
    deploymentScope: 'Resource Group'
    azureResourceManagerConnection: 'my-connection'
    subscriptionId: '$(subId)'
    resourceGroupName: 'my-rg'
    location: 'eastus'
    templateLocation: 'Linked artifact'
    csmFile: 'arm/main.json'
    csmParametersFile: 'arm/parameters.json'
</pre>

</details>

<details>
<summary>5. You want to automate testing of a web application across different browsers in Azure DevOps. How can you achieve this?</summary>

→ Use Selenium/Playwright with the **BrowserStack** or **Sauce Labs** extensions.

→ Or run Selenium Grid in Docker containers as part of the pipeline.

→ Store browser configs as parameters; publish test results via the `PublishTestResults` task.

</details>

<details>
<summary>6. Difference between Azure DevOps YAML pipelines and classic (UI-based) pipelines.</summary>

→ **YAML pipelines**: Code (in Git) → PR review → version control, reusable templates, easier to share, better for IaC.

→ **Classic pipelines**: UI-driven, task-by-task, no code review or branch diff; easier for quick one-offs but harder to maintain at scale.

→ Recommendation: YAML for any team that grows past 2-3 pipelines.

</details>

<details>
<summary>7. Your application depends on a database hosted in Azure. How can you ensure the CI/CD pipeline updates the database schema as needed during deployment?</summary>

→ Use a migration tool in the pipeline: Flyway, Liquibase, EF Core migrations, or DACPAC (SQL Server).

→ Add a migration step before the app deployment:
<pre>
- script: flyway -url=$(DB_URL) -user=$(DB_USER) -password=$(DB_PASS) migrate
</pre>

→ Keep migrations idempotent and versioned. Test migrations on a non-prod DB first.

</details>

<details>
<summary>8. How do you implement security and compliance checks in your Azure DevOps pipeline for infrastructure as code deployments?</summary>

→ **Static analysis**: `tfsec`, `checkov`, `tflint` for Terraform/IaC.

→ **Security scanning**: Microsoft Defender for DevOps, SonarQube.

→ **Policy enforcement**: Azure Policy at subscription level; pipeline gates via `az policy`.

→ **Secret scanning**: GitHub Secret Scanning / Azure DevOps Credential Scanner.

→ Enforce via PR policies and mandatory task outputs.

</details>

<details>
<summary>9. Your team uses Terraform for infrastructure provisioning. Explain how to integrate Terraform with Azure DevOps.</summary>

→ **Step 1**: Store Terraform code in a repo.

→ **Step 2**: Create an Azure service connection.

→ **Step 3**: Add pipeline stages: `terraform init` → `terraform plan` → `terraform apply`.

<pre>
- task: TerraformInstaller@1
  inputs:
    terraformVersion: '1.6.0'
- script: |
    terraform init -backend-config="resource_group_name=my-rg" \
                   -backend-config="storage_account_name=mytfstate" \
                   -backend-config="container_name=tfstate" \
                   -backend-config="key=prod.tfstate"
- script: terraform plan -out=tfplan
- script: terraform apply -auto-approve tfplan
</pre>

→ Store state in Azure Storage with locking enabled. Gate `apply` behind manual approval for production.

</details>

<details>
<summary>10. What is Blue-Green deployment, and how can you set it up using Azure DevOps for an Azure-based application?</summary>

→ Blue-green runs two identical production environments (blue = current, green = new version), and traffic is switched after validation.

→ Setup in Azure DevOps:
  - Two **deployment slots** on Azure App Service (or two sets of AKS deployments).
  - Deploy new version to the staging/green slot.
  - Warm up and smoke-test on green.
  - Swap slots (`AzureAppServiceManage` task with `Swap Slots`).
  - Keep blue available for instant rollback (swap back).

</details>

<details>
<summary>11. Describe how you can implement a canary deployment strategy in Azure DevOps.</summary>

→ Deploy the new version to a small subset of users (e.g., 5%) before full rollout.

→ Azure options:
  - **App Service deployment slots** with traffic routing (5% to staging slot).
  - **AKS**: split traffic between service versions with an Ingress (NGINX canary annotations) or a service mesh (Istio).
  - **Azure Traffic Manager** weighted routing.

→ Automate: in the pipeline, deploy canary → monitor error rate/latency → expand traffic → promote fully or roll back.

</details>

<details>
<summary>12. You need to build a containerized application and deploy it to AKS. Explain the steps involved in setting up a CI/CD pipeline for this scenario.</summary>

→ **Step 1**: Build the Docker image in the pipeline.

→ **Step 2**: Push to Azure Container Registry (ACR).
<pre>
- task: Docker@2
  inputs:
    containerRegistry: 'acr-connection'
    repository: 'myapp'
    command: 'buildAndPush'
    tags: '$(Build.BuildId)'
</pre>

→ **Step 3**: Apply Kubernetes manifests.
<pre>
- task: KubernetesManifest@1
  inputs:
    action: 'deploy'
    connectionType: 'azureResourceManager'
    azureSubscriptionConnection: 'my-az-connection'
    kubernetesCluster: 'myAKS'
    manifests: 'k8s/deployment.yaml'
</pre>

→ **Step 4**: Update image tag via `kubectl set image` or Kustomize.

→ **Step 5**: Post-deployment verification (`kubectl rollout status`).

</details>

<details>
<summary>13. Your team follows a GitFlow branching strategy. How would you automate the CI/CD pipeline in Azure DevOps to build and deploy feature branches?</summary>

→ **On feature branch push** → run CI only (build, unit tests) — no deploy.

→ **On `develop` merge** → build + deploy to dev.

→ **On `release/*` branch** → build + deploy to staging.

→ **On `main` (or a tag)** → deploy to production.

→ Implement via branch filters in YAML triggers:
<pre>
trigger:
  branches:
    include:
    - main
    - develop
    - release/*
    - feature/*
</pre>

→ Add stage conditions like `condition: and(succeeded(), eq(variables['Build.SourceBranch'], 'refs/heads/main'))`.

</details>

<details>
<summary>14. How are your current CI/CD pipelines structured?</summary>

→ Pipeline consists of stages: **Build** → **Test** → **Security Scan** → **Deploy Dev** → **Deploy Staging** → **Deploy Prod**.

→ Each stage uses reusable templates (`build-template.yml`, `deploy-template.yml`).

→ Variables come from variable groups linked to Key Vault.

→ Prod gate includes a manual approval + validation step.

</details>

<details>
<summary>15. How do you handle secrets management and environment variables in your CI/CD workflows?</summary>

→ **Azure DevOps**: Variable groups linked to Azure Key Vault.

→ **GitHub Actions**: Encrypted secrets and environment secrets, scoped per environment.

→ Avoid printing secrets in logs — mask them.

→ Pull secrets at runtime (never bake into images). Use OIDC/workload identity where possible.

</details>

<details>
<summary>16. How often are your pipelines triggered, and what are the typical triggers?</summary>

→ **On every push** to feature branches → CI.

→ **On merge to `main`** → CD to staging.

→ **On release tags** → deploy to production.

→ **Scheduled** (nightly) for end-to-end regression and dependency scans.

→ **Webhook/PR triggers** for validation builds on PRs.

</details>

<details>
<summary>17. What deployment strategies are currently employed?</summary>

→ **Rolling updates** for most stateless services (default in Kubernetes).

→ **Blue-green** for critical services needing instant rollback.

→ **Canary** for high-risk features — small traffic slice first.

→ **Feature flags** for risky changes so they can be turned off without a deploy.

</details>

<details>
<summary>18. How do you handle rollback scenarios in case of deployment failures?</summary>

→ Prefer reverted image/tag deploys over rollbacks:
  - `kubectl rollout undo deployment/&lt;name&gt;`
  - Re-deploy the previous known-good artifact from the pipeline.

→ Keep the last N images in ACR; retain the previous build artifact for quick revert.

→ Use canary/blue-green so a bad version only affects a controlled slice.

→ Automated alerts trigger rollback gates in the pipeline.

</details>

<details>
<summary>19. Do you integrate security scans or vulnerability assessments within your CI/CD pipelines?</summary>

→ **SAST**: SonarQube / Fortify / CodeQL for source analysis.

→ **Dependency scanning**: Snyk / Dependabot / OWASP Dependency-Check.

→ **Container scanning**: Trivy / Defender for Containers for image CVEs.

→ **IaC scanning**: tfsec / checkov for Terraform and Bicep.

→ **Secret scanning**: GitGuardian / Azure DevOps CredScan.

→ Gates fail the build on **High/Critical** findings.

</details>

<details>
<summary>20. How are logs generated during CI/CD processes, and how are they analysed?</summary>

→ Pipelines stream logs to the console and store them per-run (Azure DevOps / GitHub Actions both retain builds for a period).

→ Integration with Application Insights, Log Analytics, or Splunk for aggregation.

→ Metrics ingestion by Grafana / Azure Monitor.

→ Common analyses: failure rates, MTTR, slowest stages, flaky jobs.

</details>

<details>
<summary>21. Are there any alerting mechanisms for pipeline failures or anomalies?</summary>

→ **Azure DevOps**: Service hooks → Teams/Slack/Email on build failed.

→ **GitHub Actions**: Notification workflows; Slack/GitHub integrations.

→ Alert on **consecutive failures**, **increasing build duration**, **pipeline drift**, and **deployment error rates** (via Azure Monitor).

→ Route to on-call (PagerDuty) for production deploys.

</details>

---
