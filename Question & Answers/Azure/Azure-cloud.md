# Azure / Troubleshooting

<details>
<summary>1. How will you use Azure Key Vault in Kubernetes?</summary>

→ Store secrets in Azure Key Vault.

→ Install the **Secrets Store CSI Driver** + **Azure Key Vault Provider** in AKS.

<pre>
helm repo add csi-secrets-store-provider-azure https://azure.github.io/secrets-store-csi-driver-provider-azure/charts
helm install csi-secrets-store-provider-azure csi-secrets-store-provider-azure/csi-secrets-store-provider-azure
</pre>

→ Create a `SecretProviderClass` mapped to Key Vault secrets.

→ Mount it as a volume in the pod, or sync it to a Kubernetes Secret.

→ Use **workload identity / managed identity** (or pod identity) for the CSI driver to authenticate to Key Vault — no credentials in the cluster.

</details>

<details>
<summary>2. How will you troubleshoot a VM if it stopped working in Azure?</summary>

→ **Connectivity**: Check NSGs, effective routes, public IP, Azure Bastion/Serial Console.

→ **Health**: Azure Monitor metrics — CPU, memory, disk. Boot diagnostics screenshot.

→ **OS-level**: Serial Console for kernel/OS issues; redeploy if the host is broken.

→ **Guest agent**: Check `AzureGuestAgent` logs / extension status.

→ **Performance**: Check for disk full, runaway processes, memory pressure.

→ **Network Security / IP flow verify**: `az network watcher test-ip-flow`.

→ **Last resort**: Attach the disk to another VM to recover data.

</details>

<details>
<summary>3. How will you troubleshoot if an AKS node stopped responding or working?</summary>

→ **Step 1**: Check node status.
<pre>
kubectl get nodes
kubectl describe node &lt;node&gt;
</pre>

→ **Step 2**: Check node conditions (Ready, MemoryPressure, DiskPressure, PIDPressure).

→ **Step 3**: Inspect kubelet and container runtime logs on the node (via `kubectl debug node/&lt;node&gt;`).

→ **Step 4**: Check Azure resource health for the VMSS instance; check boot diagnostics.

→ **Step 5**: Cordon the node, drain it if recoverable; if not, `az vmss delete-instances` so the VMSS replaces it.

→ **Step 6**: Review Azure Monitor for resource saturation (CPU/memory/I/O).

→ **Step 7**: If systemic, check control-plane status, API server, cluster autoscaler, and node pool upgrade status.

</details>

<details>
<summary>4. How are permissions for groups and users managed?</summary>

→ **Azure**: Entra ID groups → RBAC role assignments at subscription/RG/resource levels.

→ Groups per function (dev, ops, security, read-only).

→ JIT for privileged access (Entra ID PIM).

→ **Kubernetes**: Azure AD groups bound to K8s RBAC RoleBindings for cluster access.

→ **Repos/pipelines**: Code owners, branch permissions, service connections scoped to specific pipelines.

</details>

<details>
<summary>5. In a pipeline, how are stages defined and how are environments defined?</summary>

→ **Stages** are logical phases: Build → Test → Deploy.

→ **Environments** are named targets (dev, staging, prod) with associated approval gates.

→ Example:
<pre>
stages:
- stage: Build
  jobs: [ buildJob ]
- stage: DeployProd
  dependsOn: Build
  jobs: [ deployProdJob ]
</pre>

→ Environments configured under **Pipelines → Environments** with Approval & Checks configured.

</details>

<details>
<summary>6. What are the AWS services we are going to use?</summary>

→ Compute: EC2, EKS, Lambda, ECS/Fargate.

→ Networking: VPC, ALB/NLB, Route 53, CloudFront.

→ Storage: S3, EBS, EFS.

→ Database: RDS, DynamoDB, ElastiCache.

→ Security: IAM, KMS, Secrets Manager, WAF, GuardDuty.

→ Observability: CloudWatch, X-Ray.

→ IaC/CI: CodePipeline/CodeBuild, or third-party (GitHub Actions, Terraform).

</details>

<details>
<summary>7. What are the tool stacks we are going to use for CI/CD?</summary>

→ **SCM**: GitHub / Azure Repos.

→ **CI/CD**: Azure DevOps / GitHub Actions / Jenkins.

→ **IaC**: Terraform (+ Terragrunt optional), Ansible for config.

→ **Image build**: Packer, Docker/Buildpacks.

→ **Registry**: ACR / ECR.

→ **Orchestration**: AKS / EKS.

→ **Secrets**: Key Vault / Secrets Manager / Vault.

→ **Security**: SonarQube, Trivy, tfsec, Snyk.

→ **Observability**: Azure Monitor, Grafana, Prometheus.

</details>
