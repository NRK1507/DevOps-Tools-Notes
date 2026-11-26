# DevOps Tools Q&A

Answers are hidden by default — click the question to reveal the answer.

---

# Kubernetes

## Basic Level

<details>
<summary>1. What is the kube-apiserver?</summary>

→ The Kubernetes API server is the front end of the Kubernetes control plane.

→ It is how users (and other components) interact with their Kubernetes cluster.

</details>

<details>
<summary>2. How do you create a namespace in Kubernetes?</summary>

→ Use the `kubectl create namespace` command.

<pre>
kubectl create namespace &lt;namespace_name&gt;
</pre>

</details>

<details>
<summary>3. How do you list the pods in a particular namespace?</summary>

→ Use the `--namespace` (or `-n`) flag.

<pre>
kubectl get pods -n &lt;namespace_name&gt;
</pre>

</details>

<details>
<summary>4. How many worker nodes are running in your Kubernetes cluster?</summary>

→ In our environment, we run 10–12 worker nodes.

</details>

<details>
<summary>5. Is your Kubernetes cluster on-prem or cloud?</summary>

→ Cloud. We use Azure Kubernetes Service (AKS).

</details>

<details>
<summary>6. What are the various types of Services in Kubernetes?</summary>

→ **ClusterIP**: Exposes the service on an internal IP within the cluster (default).

→ **NodePort**: Exposes the service on each node's IP at a static port.

→ **LoadBalancer**: Exposes the service externally using a cloud provider's load balancer.

</details>

<details>
<summary>7. If you apply a deployment file, what all things will get created?</summary>

→ A Deployment resource is created.

→ A ReplicaSet is created automatically by the Deployment.

→ The required number of Pods are created by the ReplicaSet.

</details>

<details>
<summary>8. How do you troubleshoot a pod that is not working?</summary>

→ Check events for error messages:
<pre>
kubectl describe pod &lt;pod_name&gt;
</pre>

→ Check container logs for application errors:
<pre>
kubectl logs &lt;pod_name&gt;
</pre>

→ Check the pod status and restart count:
<pre>
kubectl get pods
</pre>

</details>

<details>
<summary>9. What is kubeconfig?</summary>

→ `kubeconfig` is a configuration file used by `kubectl` to connect to a cluster.

→ It stores cluster endpoints, user credentials, and context information.

→ Default location is `~/.kube/config`. It is used to switch between clusters and namespaces using contexts.

</details>

<details>
<summary>10. Where do you see the logs of your applications?</summary>

→ Use `kubectl logs` for a specific pod/container.
<pre>
kubectl logs &lt;pod_name&gt; -c &lt;container_name&gt;
</pre>

→ For live streaming, add the `-f` flag.
<pre>
kubectl logs -f &lt;pod_name&gt;
</pre>

→ For a centralized view in AKS, use Azure Monitor / Log Analytics / Container Insights.

</details>

<details>
<summary>11. What is a Service? How does service discovery work?</summary>

→ A Service is an abstraction that exposes a set of pods as a network service with a stable IP and DNS name.

→ **Service discovery** is the process of connecting pods and services.

→ Kubernetes provides CoreDNS as an add-on. Every Service registers with the DNS server, so pods can communicate using service names instead of pod IPs.

</details>

---

## Intermediate Level

<details>
<summary>1. How do you upgrade the Kubernetes cluster?</summary>

→ Upgrade happens one patch version at a time.

→ A new node pool is created and the old node pool is cordoned and drained one node at a time.

→ Pods from drained nodes are recreated on the new node pool.

→ Once pods are running on the new pool, the old node pool is deleted.

→ Reference: https://github.com/MicrosoftDocs/azure-docs/blob/main/articles/aks/tutorial-kubernetes-upgrade-cluster.md

</details>

<details>
<summary>2. If monitoring tools are to be installed, who does that?</summary>

→ We use Azure monitoring tools.

→ Installation and ownership sit with the client-side Cyber Support team.

</details>

<details>
<summary>3. If a new Kubernetes cluster has to be created, who will do it?</summary>

→ The DevOps team creates the cluster using Terraform.

</details>

<details>
<summary>4. What is a DaemonSet in Kubernetes?</summary>

→ A DaemonSet ensures that all (or some) nodes run a copy of a pod.

→ As nodes are added to the cluster, pods are automatically added to them.

</details>

<details>
<summary>5. What are the use cases for a DaemonSet?</summary>

→ Log collection agents (e.g., Fluentd/Fluent Bit) on every node.

→ Monitoring agents (e.g., Prometheus node exporter, Datadog agent) on every node.

→ Network plugins (e.g., Calico, kube-proxy) that must run on all nodes.

→ Storage daemons (e.g., Ceph, GlusterFS) that need a copy on each node.

</details>

<details>
<summary>6. What is a StatefulSet in Kubernetes?</summary>

→ A StatefulSet manages stateful applications by providing stable storage, stable network identity, and ordered deployments.

→ Each pod gets a stable hostname and its own PersistentVolume.

</details>

<details>
<summary>7. Any example of a StatefulSet?</summary>

→ A Cassandra cluster where each node maintains access to its own data is a classic example.

→ Other examples: MySQL, PostgreSQL, MongoDB, Elasticsearch, Kafka, ZooKeeper — anything that needs persistent per-replica identity.

</details>

<details>
<summary>8. What is the difference between a Deployment and a ReplicaSet?</summary>

→ **Deployment**: A higher-level object providing rolling updates, rollbacks, and self-healing.

→ **ReplicaSet**: A lower-level object whose only job is to ensure a specified number of replicas are always running.

</details>

<details>
<summary>9. You want to have 5 replicas of a pod, where do you define it?</summary>

→ Define it under `spec.replicas` in either a `deployment.yaml` or a `replicaset.yaml`.

<pre>
spec:
  replicas: 5
</pre>

</details>

<details>
<summary>10. How do you restart a node without downtime?</summary>

→ Drain the node gracefully so all pods are evicted and rescheduled onto other nodes.

<pre>
kubectl drain &lt;node_name&gt; --ignore-daemonsets
</pre>

→ Restart the node.

→ Uncordon the node so it can accept pods again.
<pre>
kubectl uncordon &lt;node_name&gt;
</pre>

</details>

<details>
<summary>11. How do you drain the traffic from a Pod during maintenance?</summary>

→ Use `kubectl drain` on the node hosting the pod, which evicts pods and honors PodDisruptionBudgets.

<pre>
kubectl drain &lt;node_name&gt; --ignore-daemonsets --delete-emptydir-data
</pre>

→ For application-level draining, use a readiness probe failure or scale the deployment down gracefully behind a Service.

</details>

<details>
<summary>12. What is the Pending state in Kubernetes?</summary>

→ The pod has been accepted by the Kubernetes system, but one or more container images have not been created.

→ This includes time spent scheduling and downloading images over the network, which can take a while.

</details>

<details>
<summary>13. What are the general reasons for a pod going into Pending state?</summary>

→ Insufficient resources (CPU/memory) available on any node.

→ Unsatisfiable node selectors, affinity, or anti-affinity rules.

→ Taints on nodes without matching tolerations.

→ Unbound PersistentVolumeClaims.

→ Image pull still in progress.

</details>

<details>
<summary>14. How do you manage secrets in Kubernetes?</summary>

→ **Option 1: Kubernetes Secrets** – create a secret and reference it as env variables.
<pre>
kubectl create secret generic my-secret \
  --from-literal=username=my-user \
  --from-literal=password=my-password
</pre>

→ **Option 2: Azure Key Vault** – use the Secrets Store CSI Driver with the Azure Key Vault provider to mount secrets directly into pods.

</details>

<details>
<summary>15. How will you implement Secrets in Kubernetes?</summary>

→ **Step 1**: Create the secret.
<pre>
kubectl create secret generic my-secret \
  --from-literal=username=my-user \
  --from-literal=password=my-password
</pre>

→ **Step 2**: Reference it in the pod/deployment YAML as environment variables.
<pre>
env:
- name: USERNAME
  valueFrom:
    secretKeyRef:
      name: my-secret
      key: username
- name: PASSWORD
  valueFrom:
    secretKeyRef:
      name: my-secret
      key: password
</pre>

→ For Azure Key Vault:
<pre>
helm repo add secrets-store-csi-driver https://kubernetes-sigs.github.io/secrets-store-csi-driver/charts
helm install csi-secrets-store secrets-store-csi-driver/secrets-store-csi-driver

helm repo add csi-secrets-store-provider-azure https://azure.github.io/secrets-store-csi-driver-provider-azure/charts
helm install csi-secrets-store-provider-azure csi-secrets-store-provider-azure/csi-secrets-store-provider-azure
</pre>

→ Then create a `SecretProviderClass` resource, and reference it from the pod.

</details>

<details>
<summary>16. What are the methods of implementing secrets, and where do we use which one?</summary>

→ **Native Kubernetes Secret**: Simple key-value secrets for non-critical config. Base64 encoded, not encrypted at rest by default — enable encryption at rest.

→ **Sealed Secrets / SOPS**: Encrypt secrets in Git so they can be safely committed alongside code.

→ **External secret stores (Azure Key Vault, AWS Secrets Manager, HashiCorp Vault)**: For production, rotation, audit, and centralized governance — consumed via CSI driver or External Secrets Operator.

</details>

<details>
<summary>17. Does the container restart when applying/updating a Secret object? If not, how is the new password applied to the database?</summary>

→ No — updating a Secret does not restart pods automatically.

→ For secrets mounted as **volumes**, Kubernetes eventually syncs the updated value into the mounted file (kubelet refresh, typically up to a couple of minutes), but the app must re-read the file.

→ For secrets injected as **environment variables**, the value is fixed at container start — a pod restart is required.

→ For database passwords, the typical pattern is: update the secret → restart the deployment (rolling restart) → the app reconnects with the new credentials.

<pre>
kubectl rollout restart deployment/&lt;deployment-name&gt;
</pre>

</details>

<details>
<summary>18. How should you connect an app pod with a database pod?</summary>

→ Create a Service in front of the database pod.

→ The app pod connects using the service DNS name (e.g., `mydb.default.svc.cluster.local`) and port, not the pod IP.

→ Store credentials in a Secret.

→ Add a NetworkPolicy so only the app pods can reach the database service.

→ For a stateful database, use a StatefulSet with a headless Service for stable identity.

</details>

<details>
<summary>19. How do you configure a default ImagePullSecret for any deployment?</summary>

→ Attach the pull secret to the ServiceAccount used by the pods.
<pre>
kubectl create secret docker-registry regcred \
  --docker-server=&lt;registry&gt; \
  --docker-username=&lt;user&gt; \
  --docker-password=&lt;pass&gt;
</pre>

<pre>
kubectl patch serviceaccount default \
  -p '{"imagePullSecrets": [{"name": "regcred"}]}'
</pre>

→ All pods using that ServiceAccount inherit the pull secret.

→ In AKS, you can also attach the ACR to the cluster with `az aks update --attach-acr`.

</details>

<details>
<summary>20. If you have a pod that is using a ConfigMap which you updated, and you want the container to be updated with those changes, what should you do?</summary>

→ ConfigMaps mounted as volumes are eventually updated in the container, but the application must re-read the file.

→ ConfigMaps used as environment variables require a pod restart.

→ Safest approach: trigger a rolling restart.
<pre>
kubectl rollout restart deployment/&lt;deployment-name&gt;
</pre>

→ For automation, use a checksum annotation on the pod template so any ConfigMap change rolls the deployment.

</details>

<details>
<summary>21. What is the difference between node affinity and pod affinity?</summary>

→ **Node affinity**: Constrains which nodes a pod can be scheduled on, based on node labels. Two types:
  - `requiredDuringSchedulingIgnoredDuringExecution` — hard rule, pod will not schedule if no node matches.
  - `preferredDuringSchedulingIgnoredDuringExecution` — soft rule, scheduler tries but will not fail if it can't.

→ **Pod affinity**: Constrains scheduling relative to other pods. Used to co-locate related pods on the same node (e.g., for reduced latency).

</details>

<details>
<summary>22. Affinity and Anti-affinity?</summary>

→ Used for scheduling of pods.

→ **Affinity**:
  1. **Node affinity**: Defines which pod should be scheduled on which node based on node labels.
     - `requiredDuringSchedulingIgnoredDuringExecution`: The scheduler must place the pod on a node that meets the criteria.
     - `preferredDuringSchedulingIgnoredDuringExecution`: The scheduler tries but does not fail if it can't.
  2. **Pod affinity**: Used when multiple replicas of a service should run on the same node for reduced latency.

→ **Anti-affinity**: Ensures certain pods are not scheduled on the same node.
  - **Node anti-affinity**: Prevents pods from being scheduled on nodes that meet certain criteria.
  - **Pod anti-affinity**: Prevents pods from being scheduled on the same node as other specified pods — e.g., spreading replicas of a stateful app across nodes to avoid a single point of failure.

</details>

<details>
<summary>23. What is a Network Policy in Kubernetes?</summary>

→ A Network Policy is a specification defining how groups of pods are allowed to communicate with each other and with other network endpoints.

→ It controls traffic flow at the IP/port level, securing applications through rules you define.

**Key concepts:**

→ **Selectors**: Policies use label selectors to identify which pods they apply to.

→ **Ingress rules**: Define which incoming traffic is allowed to reach the selected pods.

→ **Egress rules**: Define which outgoing traffic is allowed from the selected pods.

→ **Default deny**: If a pod has a Network Policy applied, all traffic is denied by default unless explicitly allowed.

→ **Namespace scoped**: Policies only apply to pods within the namespace where they are defined.

</details>

<details>
<summary>24. What are taints and tolerations?</summary>

→ **Taint**: Applied to a node to repel pods that don't tolerate it.
<pre>
kubectl taint nodes &lt;node&gt; key=value:NoSchedule
</pre>

→ **Toleration**: Applied to a pod to allow it onto a tainted node.
<pre>
tolerations:
- key: "key"
  operator: "Equal"
  value: "value"
  effect: "NoSchedule"
</pre>

→ Use cases: dedicated nodes for specific workloads, GPU nodes, master/control-plane node isolation.

</details>

<details>
<summary>25. What are init containers?</summary>

→ Init containers are specialised containers that run before the main application containers in a pod.

→ They perform initialization tasks that must complete before the app containers start.

→ They run one at a time, in order, and must succeed before the next one runs.

<pre>
apiVersion: v1
kind: Pod
metadata:
  name: example-pod
spec:
  initContainers:
  - name: init-myservice
    image: busybox
    command: ['sh', '-c', 'echo Initializing; sleep 5;']
  containers:
  - name: myapp-container
    image: myapp:latest
</pre>

</details>

<details>
<summary>26. If I have multiple containers inside a pod and want to wait for a specific container to start before starting another, how do I do that?</summary>

→ Use an **init container**: init containers always complete before the main containers start.

→ If you need ordering between long-running main containers that must run together, add a readiness check in the dependent container's startup command (e.g., wait for a port or health endpoint to be ready).

→ Use a **sidecar with a startup probe** on the dependency, and a wait loop in the dependent container.

</details>

<details>
<summary>27. What is a sidecar container? Can you run multiple containers in a single pod?</summary>

→ Yes, a pod can run multiple containers. They share the same network namespace (same IP and port space) and can share volumes.

→ A **sidecar** is a helper container that runs alongside the main container — e.g., a logging agent, a proxy (Istio Envoy), or a config reloader.

→ Common patterns: sidecar, ambassador, adapter.

</details>

<details>
<summary>28. What is a PodDisruptionBudget (PDB) and its use?</summary>

→ A PDB is a policy ensuring a certain number or percentage of replicas of a pod are always running, even during voluntary disruptions.

→ **Voluntary disruptions**: admin-initiated, e.g., draining a node for maintenance.

→ **Involuntary disruptions**: hardware failure, node crash.

<pre>
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: my-app-pdb
spec:
  minAvailable: 1
  selector:
    matchLabels:
      app: my-app
</pre>

</details>

<details>
<summary>29. What kinds of errors can be seen in Kubernetes pods?</summary>

→ **Container Errors**
  - `CrashLoopBackOff`: Container repeatedly crashes after starting.
  - `ImagePullBackOff`: Unable to pull the image (wrong name, missing, auth issues).
  - `ErrImagePull`: Similar to above but not retrying.

→ **Initialization Errors**
  - `InitContainer Failures`: Init container exited non-zero, main containers won't start.
  - `Pending State`: Insufficient resources or unsatisfiable selectors.

→ **Resource Limit Errors**
  - `OOMKilled`: Container exceeded its memory limit.
  - `CPU throttling`: Container exceeded its CPU limit.

→ **Networking Issues**
  - Network Policy misconfiguration blocks traffic.
  - DNS resolution failures.

→ **Volume Mount Errors**
  - Failed to mount volume.
  - `ReadOnlyFilesystem` errors.

→ **Permission Errors**
  - SecurityContext violations.
  - ServiceAccount misconfiguration.

→ **Scheduling Issues**
  - Unschedulable pods.
  - Taints and tolerations mismatch.

→ **Application / Configuration Errors**
  - Runtime exceptions and timeouts.
  - Missing or incorrect environment variables, ConfigMaps, or Secrets.

</details>

<details>
<summary>30. How do you test a manifest without actually executing it?</summary>

→ Use `kubectl apply --dry-run=client`. It validates the manifest and simulates the apply without creating resources.

<pre>
kubectl apply --dry-run=client -f job.yaml
</pre>

→ Use `--dry-run=server` to have the API server fully validate (admission, defaults) without persisting.

</details>

<details>
<summary>31. How do you initiate a rollback for an application?</summary>

→ **Step 1**: Check the revision history.
<pre>
kubectl rollout history deployment/&lt;deployment-name&gt;
</pre>

→ **Step 2**: Roll back to the previous revision, or a specific one.
<pre>
kubectl rollout undo deployment/&lt;deployment-name&gt;
kubectl rollout undo deployment/&lt;deployment-name&gt; --to-revision=&lt;revision-number&gt;
</pre>

→ **Step 3**: Verify the rollback.
<pre>
kubectl rollout status deployment/&lt;deployment-name&gt;
</pre>

</details>

<details>
<summary>32. How do you package Kubernetes applications?</summary>

→ **Helm** is the most popular choice — charts package manifests into reusable, parameterised bundles.

→ Other approaches: **Kustomize** (overlay-based, built into kubectl), **Jsonnet**, or plain manifests in a Git repo (GitOps with Argo CD / Flux).

</details>

<details>
<summary>33. How do you implement a readiness probe in a deployment manifest?</summary>

→ Add a `readinessProbe` to the container spec.

<pre>
readinessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 10
  timeoutSeconds: 2
  failureThreshold: 3
</pre>

→ If the probe fails, the pod is removed from the Service endpoints and receives no traffic.

</details>

<details>
<summary>34. What are the different load balancers in AKS?</summary>

→ **Azure Load Balancer (L4)**: Used by default for `type: LoadBalancer` services — TCP/UDP.

→ **Application Gateway (L7)**: HTTP/HTTPS with WAF, path-based routing — used via the **AGIC** (Application Gateway Ingress Controller).

→ **NGINX Ingress Controller (L7)**: In-cluster ingress controller using an Azure Load Balancer as its front end.

→ **Internal Load Balancer**: Private ILB inside the VNet for internal-only services.

</details>

---

## Advanced Level

<details>
<summary>1. You want HPA with a Kubernetes cluster and, based on load, it has to scale up or down — how will you achieve it?</summary>

→ Manually scale AKS nodes:
<pre>
az aks scale --resource-group myResourceGroup --name myAKSCluster --node-count 3
</pre>

→ Manually scale a deployment:
<pre>
kubectl scale --replicas=3 deployment/ratings-api
</pre>

→ Create an HPA from the CLI:
<pre>
kubectl autoscale deployment ratings-web --cpu-percentage=50 --min=2 --max=10
</pre>

→ Or use a manifest:
<pre>
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: ratings-api
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: ratings-api
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50
</pre>

→ Check the HPA:
<pre>
kubectl get hpa -A
</pre>

</details>

<details>
<summary>2. How is scaling on worker nodes done?</summary>

→ Autoscaling can be **vertical** or **horizontal**.

→ **Vertical scaling**: Adds more resources such as RAM or CPU to existing nodes.

→ **Horizontal scaling**: The more popular option — adds or removes pods or servers.

→ **Cluster Autoscaler**: Watches for pods that cannot be scheduled due to resource constraints (CPU/memory) and automatically scales the cluster (adds/removes nodes).

→ Reference: https://www.techtarget.com/searchcloudcomputing/tutorial/How-to-set-up-autoscaling-in-Azure-Kubernetes-Service

</details>

<details>
<summary>3. In Ingress, how does a request flow from URL to pod? How are targets/destinations assigned?</summary>

→ Ingress is an advanced version of a load balancer that works at OSI Layer 7.

→ Install an ingress controller (e.g., NGINX) using Helm:
<pre>
helm repo add stable https://kubernetes-charts.storage.googleapis.com/
helm install nginx-ingress stable/nginx-ingress \
  --set controller.replicaCount=2 \
  --set controller.nodeSelector."beta\.kubernetes\.io/os"=linux
</pre>

→ Deploy an Ingress resource YAML for path-based or host-based routing:
<pre>
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ratings-web-ingress
  annotations:
    kubernetes.io/ingress.class: nginx
    cert-manager.io/cluster-issuer: letsencrypt
spec:
  tls:
  - hosts:
    - frontend.example.com
    secretName: ratings-web-cert
  rules:
  - host: frontend.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: ratings-web
            port:
              number: 80
</pre>

→ Flow: Client → DNS → Ingress Controller (LB IP) → matches host/path rule → Service → Endpoints → Pod.

</details>

<details>
<summary>4. What is a Pod Security Policy (PSP)?</summary>

→ PSPs allowed administrators to define a set of conditions a pod must meet to be accepted by the kubelet, enforcing security standards across the cluster.

→ **Privileged**: For workloads that need full access to the host.

→ **Baseline**: A minimum level of security for most workloads.

→ **Restricted**: For workloads with the highest security needs.

→ A common use was preventing containers from running in privileged mode, limiting host access.

→ Note: PSP is deprecated in favor of **Pod Security Admission (PSA)** with `baseline`/`restricted` namespace labels.

</details>

<details>
<summary>5. Your pods are running but users can't access the application. How will you troubleshoot?</summary>

**Step 1 — Check the Service, not the Pod.**
→ Traffic flows Client → Service → Pod. If the Service is broken, healthy pods don't matter.

**Step 2 — Verify Service selectors.**
→ If labels don't match, the Service has nothing to route traffic to.

**Step 3 — Check service endpoints.**
<pre>
kubectl get endpoints &lt;service-name&gt;
</pre>
→ No endpoints means the Service cannot see any pods — traffic will fail even if pods are running.

**Step 4 — Validate ports.**
→ Verify the Service `port`, the Service `targetPort`, and the container port inside the pod all align. If they don't, traffic silently fails.

**Step 5 — Check the readiness probe.**
→ If readiness is failing, pods will not receive traffic even though they show as `Running`.

**Step 6 — Check Ingress or Load Balancer.**
→ Review Ingress rules, load balancer health checks, DNS, and routing.

**Why interviewers like this answer:**
→ It shows you understand traffic flow, debug systematically, and don't panic or blindly redeploy.

</details>

<details>
<summary>6. How would you approach debugging a pod that's in a CrashLoopBackOff state?</summary>

→ A pod in CrashLoopBackOff means the container starts, crashes, then restarts repeatedly with an increasing delay.

→ **Step 1**: Check the pod logs.
<pre>
kubectl logs &lt;pod_name&gt; --previous
</pre>

→ **Step 2**: Describe the pod for events and status.
<pre>
kubectl describe pod &lt;pod_name&gt;
</pre>

→ **Step 3**: Verify resource limits and requests — the pod may be OOMKilled.

→ **Step 4**: Check config — missing env vars, bad ConfigMap/Secret references, wrong image tag, failing liveness/startup probes.

</details>

<details>
<summary>7. Can you explain the difference between a DaemonSet and a Deployment?</summary>

→ **DaemonSet**: Ensures a copy of a pod runs on all (or some) nodes. Useful for node-level operations like log collection or monitoring.

→ **Deployment**: Manages a set of identical pods, allowing easy scaling and updates. Ideal for stateless applications that can run on any node.

</details>

<details>
<summary>8. How do you handle persistent storage in Kubernetes?</summary>

→ **PersistentVolume (PV)**: A cluster resource representing physical storage.

→ **PersistentVolumeClaim (PVC)**: A user's request for storage.

→ **StorageClass**: Enables dynamic provisioning of PVs.

→ Pods mount the PVC as a volume.

<pre>
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 10Gi
</pre>

</details>

<details>
<summary>9. What strategies would you use to optimize resource utilization in a Kubernetes cluster?</summary>

→ Use resource **requests and limits** to ensure efficient pod scheduling.

→ Implement **Horizontal Pod Autoscaling** to adjust replica count based on metrics.

→ Use **Vertical Pod Autoscaling** to automatically adjust CPU and memory requests.

→ Use **Cluster Autoscaler** (or node auto-provisioning) to add/remove nodes based on demand.

→ Right-size workloads with monitoring data; use `kubectl top` and Azure Monitor Container Insights.

</details>

<details>
<summary>10. How would you implement a blue-green deployment in Kubernetes?</summary>

→ Blue-green runs two identical environments (blue and green) and switches traffic between them.

→ **Step 1**: Create two identical deployments with different labels (e.g., `version: blue` and `version: green`).

→ **Step 2**: Point the Service selector at the blue deployment's labels.

→ **Step 3**: Deploy the new version to green.

→ **Step 4**: Test green via a separate test Service or direct pod access.

→ **Step 5**: Update the main Service selector to green to switch traffic.

→ **Step 6**: Keep blue around for instant rollback, then scale it down once confidence is high.

</details>

<details>
<summary>11. How would you implement auto-scaling in Kubernetes based on custom metrics?</summary>

→ **Step 1**: Set up a metrics pipeline — Prometheus Adapter (or Azure Monitor Adapter) exposing custom metrics via the custom metrics API.

→ **Step 2**: Create an HPA that references the custom metric.

<pre>
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: my-app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: my-app
  minReplicas: 2
  maxReplicas: 20
  metrics:
  - type: Pods
    pods:
      metric:
        name: http_requests_per_second
      target:
        type: AverageValue
        averageValue: "100"
</pre>

→ **Step 3**: Verify metrics are flowing.
<pre>
kubectl get --raw /apis/custom.metrics.k8s.io/v1beta1
</pre>

</details>

<details>
<summary>12. Let's say a Kubernetes job should finish in 40 seconds, but on rare occasions it takes 5 minutes. How do you make sure the application is stopped if it exceeds 40 seconds?</summary>

→ Use `activeDeadlineSeconds` in the Job spec. The job is terminated if it runs longer than the specified duration.

<pre>
apiVersion: batch/v1
kind: Job
metadata:
  name: example-job
spec:
  activeDeadlineSeconds: 40
  template:
    spec:
      containers:
      - name: example-container
        image: your-container-image
        command: ["your-command"]
      restartPolicy: Never
</pre>

→ `activeDeadlineSeconds: 40` terminates the job if it runs longer than 40 seconds.

→ `restartPolicy: Never` ensures the container is not restarted if it fails or is terminated.

</details>

<details>
<summary>13. In Kubernetes, if you find a deployment slow, how do you debug and optimize?</summary>

→ **Debug**:
  - `kubectl describe deployment` and `kubectl rollout status` to check progress.
  - Check events with `kubectl get events --sort-by=.lastTimestamp`.
  - Inspect pod events: image pull time, readiness probe failures, scheduling delays.
  - Check node pressure and resource availability.

→ **Optimize**:
  - Pre-pull images on nodes, or use a node image cache.
  - Set realistic `readinessProbe`/`startupProbe` values to avoid false failures.
  - Use `maxSurge`/`maxUnavailable` to speed rolling updates.
  - Set resource requests correctly to avoid slow scheduling.
  - Use smaller images and multi-stage builds.

</details>

<details>
<summary>14. There are 3 worker nodes A, B, C, all with multiple pods. For some reason I don't want to deploy a new pod to node C. How do you achieve that?</summary>

→ **Option 1 — Taint node C and tolerate only what should run there:**
<pre>
kubectl taint nodes node-c dedicated=no-new-pods:NoSchedule
</pre>

→ **Option 2 — Use node affinity to exclude node C:**
<pre>
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: NotIn
          values:
          - node-c
</pre>

→ **Option 3 — Label nodes A and B and use nodeSelector on the workload.**

</details>

<details>
<summary>15. The kube-scheduler is down for some reason, and you want to deploy a pod to any node without using the scheduler. How is it achieved?</summary>

→ Set `nodeName` directly in the pod spec. This bypasses the scheduler entirely — the kubelet on that node picks up the pod.

<pre>
apiVersion: v1
kind: Pod
metadata:
  name: bypass-scheduler
spec:
  nodeName: worker-node-1
  containers:
  - name: app
    image: nginx
</pre>

→ Note: the pod won't benefit from scheduler decisions (affinity, taints, resource fit) — use only for recovery/troubleshooting.

</details>

<details>
<summary>16. What is the impact of upgrading the kubelet if we leave the pods on the worker node — will it break running pods? Why?</summary>

→ Upgrading the kubelet restarts the kubelet process on the node.

→ **Existing running pods are not killed** by a kubelet restart — containers continue running because containerd keeps them alive, and the kubelet re-attaches to them.

→ However, pods may temporarily become unmanageable (status/exec/logs may fail briefly).

→ Best practice is still to **cordon and drain** the node before upgrading so workloads move cleanly, since a kubelet/version skew with the control plane can cause issues.

→ Running pods can break if the upgrade also includes container runtime changes, or if there is a significant version skew against the API server.

</details>

<details>
<summary>17. How can a service that selects apps based on a label also have an externalIP?</summary>

→ A Service can use a label selector *and* an `externalIPs` field. The selector picks the pods; `externalIPs` adds extra routable IPs on the nodes that forward to the Service.

<pre>
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app: my-app
  ports:
  - port: 80
    targetPort: 8080
  externalIPs:
  - 203.0.113.10
</pre>

→ Traffic to `203.0.113.10:80` on any node is forwarded to the selected pods.

</details>

<details>
<summary>18. I have two tables, emp and empsal. If their schema changes, how does that deployment happen into containers/pods automatically?</summary>

→ Use a **database migration tool or init job**, not a manual step.

→ Common approaches:
  - **Init container** in the app pod that runs migrations before the app starts.
  - **Kubernetes Job** that applies the migration and must complete before the rollout (`helm` pre-install/pre-upgrade hook).
  - **Helm hooks** for ordered migration jobs.
  - Tools: Flyway, Liquibase, Alembic, or raw SQL scripts run by a job.

<pre>
apiVersion: batch/v1
kind: Job
metadata:
  name: db-migrate
spec:
  template:
    spec:
      containers:
      - name: migrate
        image: my-migrator:latest
      restartPolicy: Never
</pre>

→ Make migrations idempotent and versioned so re-runs are safe.

</details>

<details>
<summary>19. How to deploy Prometheus in Kubernetes?</summary>

→ The standard approach is the **kube-prometheus-stack** Helm chart (Prometheus + Alertmanager + Grafana + node exporters).

<pre>
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install monitoring prometheus-community/kube-prometheus-stack \
  --namespace monitoring --create-namespace
</pre>

→ Verify pods are running.
<pre>
kubectl get pods -n monitoring
</pre>

→ Access Grafana via port-forward or the configured ingress.
<pre>
kubectl port-forward -n monitoring svc/monitoring-grafana 3000:80
</pre>

</details>

<details>
<summary>20. How to map DNS to applications running in the Kubernetes cluster?</summary>

→ **External DNS operator**: watches Services/Ingresses and creates DNS records in Azure DNS automatically.
<pre>
helm repo add external-dns https://kubernetes-sigs.github.io/external-dns/
helm install external-dns external-dns/external-dns \
  --set provider=azure --set azure.resourceGroup=&lt;dns-rg&gt;
</pre>

→ **ALSO:** manually create an A record in Azure DNS pointing to the LoadBalancer/Ingress public IP.

→ Inside the cluster, CoreDNS handles service-name resolution automatically.

</details>

<details>
<summary>21. If I tell you to create a new AKS cluster, what commands do you use?</summary>

→ Using Azure CLI:
<pre>
az group create --name myRG --location eastus

az aks create \
  --resource-group myRG \
  --name myAKSCluster \
  --node-count 3 \
  --node-vm-size Standard_DS2_v2 \
  --enable-managed-identity \
  --generate-ssh-keys

az aks get-credentials --resource-group myRG --name myAKSCluster
kubectl get nodes
</pre>

→ In practice, we do this via **Terraform** (`azurerm_kubernetes_cluster` resource) through a CI/CD pipeline, not manually.

</details>

<details>
<summary>22. How do you define a VNet in your project, and what things do you add during VNet setup and connection setup?</summary>

→ Each environment has a dedicated VNet with non-overlapping CIDR ranges (e.g., `10.1.0.0/16` for dev, `10.2.0.0/16` for prod) to allow peering.

→ **Subnets**:
  - AKS node subnet (distinct, sized for future node growth).
  - AKS pod subnet (with Azure CNI) or use kubenet/overlay.
  - Application Gateway subnet.
  - Bastion subnet (`AzureBastionSubnet`).
  - Private endpoint subnet for PaaS services.

→ **On connection setup**:
  - VNet peering between hub/spoke or to on-prem.
  - NSG rules on each subnet.
  - Route tables / UDRs.
  - Private DNS zones linked to the VNet.
  - Service endpoints or Private Link for Storage, Key Vault, SQL.
  - Azure Firewall/NVA for egress control.

</details>

<details>
<summary>23. Where do you deploy your bastion host?</summary>

→ In a dedicated subnet named `AzureBastionSubnet` (required name) inside the VNet.

→ Azure Bastion gives RDP/SSH over TLS through the portal — no public IP on the VMs.

→ For a classic jumpbox, place a hardened VM with a public IP (or behind a firewall) in an isolated management subnet, restricted by NSG.

</details>

<details>
<summary>24. One pod has 2 containers running — Nginx and WordPress. How can I access these 2 containers from the browser with an IP address?</summary>

→ Containers in the same pod share the network namespace, so they share one IP. You expose each container on a different port and route with a Service.

→ Expose the pod via a Service:
<pre>
apiVersion: v1
kind: Service
metadata:
  name: wp-svc
spec:
  type: LoadBalancer
  selector:
    app: wordpress
  ports:
  - name: http-nginx
    port: 80
    targetPort: 80
  - name: http-wordpress
    port: 8080
    targetPort: 8080
</pre>

→ Wordpress connects to Nginx (or MySQL) over `localhost:&lt;port&gt;` because they share the pod IP.

→ Best practice is actually to run Nginx and WordPress in **separate pods** and connect via Services.

</details>

<details>
<summary>25. If there is an application to be exposed to the market, which service will you use?</summary>

→ A **LoadBalancer** Service (L4), typically fronted by an **Ingress Controller** (L7) for HTTP/HTTPS routing, TLS termination, and path/host-based rules.

→ In AKS, that's often NGINX Ingress or Application Gateway (AGIC) behind a public Azure Load Balancer.

</details>

<details>
<summary>26. What is an Autoscaling Group?</summary>

→ Automatically adjusts the number of instances in a group based on predefined conditions.

→ Helps ensure your application handles varying loads efficiently while maintaining availability and minimising costs.

→ Features dynamic scaling and health checks, and lets you define scaling policies.

→ **Example**: A web app with high traffic in the day and low at night. The autoscaling group adds instances during the day and scales back at night to save cost.

</details>

<details>
<summary>27. What type of Azure services have you used?</summary>

→ Azure Entra ID (AAD), Storage, Networking, VMs, VMSS, Azure DevOps, AKS, ACR, and Web Apps.

</details>

<details>
<summary>28. If a client wants to open a website "example.com", what resources will you use in Azure and AKS, and what will be the flow?</summary>

**Resources:**
→ **Resource Group**: Container for all related resources.

→ **AKS**: Managed Kubernetes cluster to run the containerized application.

→ **ACR**: To store Docker images.

→ **Azure DNS**: To manage the domain and DNS resolution.

→ **Load Balancer / Application Gateway**: To distribute incoming web traffic to AKS nodes.

→ **Azure Key Vault**: For secrets, keys, and certificates.

→ **Azure Storage**: For static assets like images, CSS, and JavaScript.

**Flow:**
1. **Build & Push Image**: Build the Docker image → push to ACR.
2. **Create & Configure AKS**: Create the cluster → attach ACR.
3. **Deploy Application**: Apply deployment and service manifests.
<pre>
kubectl apply -f &lt;manifest-file&gt;
</pre>
4. **Configure Ingress Controller**: Deploy NGINX ingress → create an Ingress resource routing `example.com`.
<pre>
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - host: example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: my-service
            port:
              number: 80
</pre>
5. **DNS Configuration**: Create an A record in Azure DNS pointing to the LB/App Gateway public IP.
6. **SSL/TLS**: Obtain the certificate → store in Key Vault → configure ingress TLS.
<pre>
spec:
  tls:
  - hosts:
    - example.com
    secretName: example-tls
</pre>

</details>

---

# Git

<details>
<summary>1. How would you create a new branch in Git based on the latest commit on the master branch?</summary>

<pre>
git checkout master
git pull
git checkout -b &lt;new_branch_name&gt;
</pre>

</details>

<details>
<summary>2. How would you squash multiple commits into one?</summary>

<pre>
git rebase -i HEAD~&lt;number_of_commits&gt;
</pre>

→ Change `pick` to `squash` (or `s`) for the commits you want to combine, save, and edit the message.

</details>

<details>
<summary>3. How would you discard changes in Git (revert a file to its previous state)?</summary>

<pre>
git checkout -- &lt;filename&gt;
# or in newer Git
git restore &lt;filename&gt;
</pre>

</details>

<details>
<summary>4. How would you view the differences between current changes in your working directory and the last commit?</summary>

<pre>
git diff
</pre>

</details>

<details>
<summary>5. How would you stage only some of the changes for commit?</summary>

<pre>
git add -p
</pre>

→ Git will prompt you to accept, skip, or split each hunk (hunk-by-hunk staging).

</details>

<details>
<summary>6. How would you remove a file from Git without deleting it from your local file system?</summary>

<pre>
git rm --cached &lt;filename&gt;
</pre>

</details>

<details>
<summary>7. How would you rename a file in Git while preserving its history?</summary>

<pre>
git mv &lt;old_filename&gt; &lt;new_filename&gt;
git commit -m "rename file"
</pre>

</details>

<details>
<summary>8. How would you discard all local changes and make your working directory match the last commit?</summary>

<pre>
git reset --hard HEAD
</pre>

</details>

<details>
<summary>9. How would you undo the last commit and make changes to the previous commit?</summary>

<pre>
git commit --amend
</pre>

→ This replaces the last commit with the amended one.

</details>

<details>
<summary>10. How would you check out a specific commit in Git to review its contents?</summary>

<pre>
git checkout &lt;commit_hash&gt;
</pre>

→ Return to your branch with `git checkout &lt;branch_name&gt;`.

</details>

<details>
<summary>11. How would you create a new branch in Git based on a specific commit?</summary>

<pre>
git checkout -b &lt;new_branch_name&gt; &lt;commit_hash&gt;
</pre>

</details>

<details>
<summary>12. How would you cherry-pick a specific commit from one branch to another?</summary>

<pre>
git checkout &lt;target_branch&gt;
git cherry-pick &lt;commit_hash&gt;
</pre>

</details>

<details>
<summary>13. How would you view the commit history of a specific file?</summary>

<pre>
git log -- &lt;filename&gt;
</pre>

</details>

<details>
<summary>14. How would you revert a specific commit and remove its changes from the repository?</summary>

<pre>
git revert &lt;commit_hash&gt;
</pre>

→ Creates a new commit that undoes the changes — safe for shared branches.

</details>

<details>
<summary>15. How would you merge changes from one branch into another?</summary>

<pre>
git checkout &lt;target_branch&gt;
git merge &lt;source_branch&gt;
</pre>

</details>

<details>
<summary>16. How would you discard all local changes, including untracked files, and revert to the last commit?</summary>

<pre>
git reset --hard HEAD
git clean -fd
</pre>

</details>

<details>
<summary>17. How would you see the changes introduced by a specific commit?</summary>

<pre>
git show &lt;commit_hash&gt;
</pre>

</details>

<details>
<summary>18. How would you delete a branch from your local repository?</summary>

<pre>
git branch -d &lt;branch_name&gt;
# force delete
git branch -D &lt;branch_name&gt;
</pre>

</details>

<details>
<summary>19. How would you view the differences between two branches?</summary>

<pre>
git diff &lt;branch1&gt; &lt;branch2&gt;
</pre>

</details>

<details>
<summary>20. How would you create a tag in Git to mark a specific commit?</summary>

<pre>
git tag -a &lt;tag_name&gt; -m "release notes"
git push origin &lt;tag_name&gt;
</pre>

</details>

<details>
<summary>21. How would you stash your current changes temporarily and come back later?</summary>

<pre>
git stash
git stash pop
</pre>

→ `git stash list` shows all stashes.

</details>

<details>
<summary>22. How would you revert a specific file to its state in a previous commit?</summary>

<pre>
git checkout &lt;commit_hash&gt; -- &lt;filename&gt;
</pre>

</details>

<details>
<summary>23. How would you change the message of your last commit?</summary>

<pre>
git commit --amend -m "new message"
</pre>

</details>

<details>
<summary>24. How would you see the differences between two commits?</summary>

<pre>
git diff &lt;commit1&gt; &lt;commit2&gt;
</pre>

</details>

<details>
<summary>25. How would you see the list of files changed in a specific commit?</summary>

<pre>
git show --name-only &lt;commit_hash&gt;
</pre>

</details>

<details>
<summary>26. How would you create a new branch and switch to it, all in one command?</summary>

<pre>
git checkout -b &lt;new_branch_name&gt;
# or
git switch -c &lt;new_branch_name&gt;
</pre>

</details>

<details>
<summary>27. How would you apply a specific commit from one branch to another without creating a new commit?</summary>

<pre>
git cherry-pick -n &lt;commit_hash&gt;
</pre>

→ `-n` (no commit) stages the changes without committing.

</details>

<details>
<summary>28. How would you set up Git to ignore specific files or directories?</summary>

→ Create or edit a `.gitignore` file in the repo root.

<pre>
*.log
node_modules/
.env
</pre>

</details>

<details>
<summary>29. How would you configure Git to automatically pull changes from a remote repository whenever you switch to a branch?</summary>

→ There's no fully automatic "pull on branch switch", but you can enable auto-fetch and auto-rebase-on-pull:

<pre>
git config --global fetch.prune true
git config --global pull.rebase true
</pre>

→ Best practice is to pull manually or via a script after checkout.
<pre>
git checkout &lt;branch&gt; && git pull
</pre>

</details>

<details>
<summary>30. How would you configure Git to automatically push changes to a remote repository whenever you make a commit?</summary>

→ Git has no built-in "auto-push on commit". The closest is a `post-commit` hook.

<pre>
# .git/hooks/post-commit
#!/bin/sh
git push origin HEAD
</pre>

→ Make the script executable:
<pre>
chmod +x .git/hooks/post-commit
</pre>

→ Note: auto-pushing on every commit is discouraged; it interferes with rebases and history rewrites.

</details>

<details>
<summary>31. What is your Git branching strategy?</summary>

→ **Trunk-based** for small teams — short-lived feature branches merged to `main` frequently.

→ **Git Flow** for larger release-driven teams — `main`, `develop`, `feature/*`, `release/*`, `hotfix/*`.

→ **GitHub Flow** — a single `main` with feature branches and PR-based merges.

→ We choose based on team size, release cadence, and how strict environment isolation needs to be.

</details>

<details>
<summary>32. How will verticals be included in the Git branch?</summary>

→ Each vertical (business unit/product line) can have its own repository or a top-level folder/branch namespace.

→ Main approaches:
  - **Poly-repo per vertical** — cleanest isolation, separate pipelines.
  - **Mono-repo with folder per vertical** — shared tooling, path-filtered pipelines.
  - **Branch namespacing** — prefix branches with the vertical (`vertical-a/feature-x`).

→ Ownership is enforced through CODEOWNERS files and protected branches.

</details>

<details>
<summary>33. How are naming and tagging conventions managed?</summary>

→ **Branch names**: `feature/&lt;ticket&gt;-&lt;short-desc&gt;`, `bugfix/...`, `hotfix/...`, `release/&lt;version&gt;`.

→ **Tags**: Semantic versioning — `v1.2.3`.

→ **Commits**: Conventional commits (`feat:`, `fix:`, `chore:`) to enable changelog automation.

→ Enforced via branch protection rules, commit hooks, and PR templates.

</details>

---

# Jenkins

<details>
<summary>1. You have a Jenkins job that fails intermittently. How would you troubleshoot and determine the root cause?</summary>

→ Review the full console log of a failed build, including the timestamp of the failure point.

→ Check if the failure correlates with a time, agent, or specific stage — look for flaky tests, network timeouts, or resource contention.

→ Re-run the build a few times to see if the failure reproduces.

→ Check agent health (disk, memory, running processes) and plugin versions.

→ Use `buildDiscarder` and archived logs for post-mortem. Consider isolating the flaky step and adding retries with backoff.

</details>

<details>
<summary>2. How would you automate the deployment of a Java web application using Jenkins?</summary>

→ Create a **Pipeline job** with stages: checkout → build (Maven/Gradle) → test → package (WAR/JAR) → deploy.

<pre>
pipeline {
  agent any
  stages {
    stage('Checkout') { steps { git 'https://repo.git' } }
    stage('Build')    { steps { sh 'mvn clean package' } }
    stage('Test')     { steps { sh 'mvn test' } }
    stage('Deploy')   { steps { sh 'deploy-script.sh target/app.war' } }
  }
}
</pre>

→ Store credentials in Jenkins Credentials and reuse via `withCredentials`.

</details>

<details>
<summary>3. You have multiple Jenkins agents and want to distribute the build workload evenly. How would you configure agent load balancing?</summary>

→ Give each agent labels and let the scheduler assign jobs based on load.

→ Enable **"Throttle Concurrent Builds"** per agent to limit concurrency.

→ Enable agent-specific executors so high-resource jobs are routed to appropriate agents.

→ Use Jenkins' default round-robin scheduler; or use the **Slave Utilization** plugin.

→ For multi-node scaling, use **Kubernetes plugin** to spawn dynamically sized agents per job.

</details>

<details>
<summary>4. How would you trigger a Jenkins job automatically whenever changes are pushed to a Git repository?</summary>

→ **Option 1 — Webhook**: Configure a webhook in GitHub/GitLab to hit Jenkins' `/github-webhook/` endpoint, and enable "GitHub hook trigger for GITScm polling" in the job.

→ **Option 2 — Poll SCM**: Set a cron-like interval (e.g., `H/5 * * * *`) so Jenkins polls for changes.

→ **Option 3 — Multibranch Pipeline**: Auto-detects new branches and PRs to build.

</details>

<details>
<summary>5. You have a Jenkins pipeline with multiple stages and want to pass data between them. How would you share data between pipeline stages?</summary>

→ **Option 1 — `script` block with global variables**:
<pre>
def version
pipeline {
  stages {
    stage('Build')  { steps { script { version = '1.2.3' } } }
    stage('Deploy') { steps { echo "Deploying ${version}" } }
  }
}
</pre>

→ **Option 2 — Write to a file** (`writeFile`/`readFile`).

→ **Option 3 — `env` variables** for simple strings.

→ **Option 4 — `archiveArtifacts` and `stash`/`unstash`** for build outputs.

</details>

<details>
<summary>6. How would you create a Jenkins job that runs every night at 2 AM?</summary>

→ Use a cron-style trigger in the pipeline or job configuration.

<pre>
triggers {
  cron('H 2 * * *')
}
</pre>

→ `H` means "hash" — Jenkins spreads load at a deterministic minute so many jobs don't all fire at exactly 2:00.

</details>

<details>
<summary>7. You have a Jenkins job that requires environment variables at runtime. How would you pass them?</summary>

→ In a Declarative Pipeline, use the `environment` block.
<pre>
environment {
  APP_ENV = 'production'
  VERSION = '1.4.2'
}
</pre>

→ For secrets, use `withCredentials`:
<pre>
withCredentials([string(credentialsId: 'api-token', variable: 'API_TOKEN')]) {
  sh 'deploy.sh'
}
</pre>

</details>

<details>
<summary>8. How would you deploy a Docker container using Jenkins?</summary>

→ Build → tag → push → deploy workflow in a pipeline:
<pre>
stage('Build Image') {
  steps {
    sh 'docker build -t myapp:${BUILD_NUMBER} .'
    sh 'docker tag myapp:${BUILD_NUMBER} &lt;registry&gt;/myapp:${BUILD_NUMBER}'
  }
}
stage('Push Image') {
  steps {
    withCredentials([usernamePassword(credentialsId: 'docker-creds',
        usernameVariable: 'USER', passwordVariable: 'PASS')]) {
      sh 'echo $PASS | docker login -u $USER --password-stdin &lt;registry&gt;'
      sh 'docker push &lt;registry&gt;/myapp:${BUILD_NUMBER}'
    }
  }
}
stage('Deploy') {
  steps { sh 'kubectl set image deployment/myapp myapp=&lt;registry&gt;/myapp:${BUILD_NUMBER}' }
}
</pre>

</details>

<details>
<summary>9. You have a long and resource-intensive build. How would you run it on a specific Jenkins agent with higher resources?</summary>

→ Label the agent (e.g., `high-memory`) and target it in the pipeline.

<pre>
pipeline {
  agent { label 'high-memory' }
  ...
}
</pre>

→ Or use `node('high-memory') { ... }` in Scripted pipelines.

</details>

<details>
<summary>10. How would you automate testing of a web application using Jenkins (Selenium or JUnit)?</summary>

→ Add a test stage that runs the test framework.

<pre>
stage('Unit Tests') {
  steps { sh 'mvn test' }
  post { always { junit 'target/surefire-reports/*.xml' } }
}
stage('UI Tests') {
  steps { sh 'mvn verify -Pselenium' }
}
</pre>

→ Publish reports with `junit` and `htmlpublisher` plugins for trend charts.

</details>

<details>
<summary>11. A Jenkins job requires credentials to access external systems. How do you securely store and use them?</summary>

→ Store credentials under **Manage Jenkins → Credentials** (username/password, secret text, SSH key, etc.).

→ Reference from a pipeline using `withCredentials`:
<pre>
withCredentials([usernamePassword(credentialsId: 'my-creds',
    usernameVariable: 'U', passwordVariable: 'P')]) {
  sh 'curl -u $U:$P https://api.example.com'
}
</pre>

→ Never hardcode secrets in the Jenkinsfile or console logs.

</details>

<details>
<summary>12. How would you configure Jenkins to send email notifications whenever a build fails?</summary>

→ Use the `post` section with the `mail` step.

<pre>
post {
  failure {
    mail to: 'team@example.com',
         subject: "Build Failed: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
         body: "Check ${env.BUILD_URL}console"
  }
}
</pre>

→ Configure SMTP in **Manage Jenkins → System → E-mail Notification**.

</details>

<details>
<summary>13. How would you handle inter-service communication and deployment coordination for a microservices app?</summary>

→ Use a **pipeline per service** with a shared library of standard stages.

→ Deploy services independently with compatible API contracts.

→ Add integration tests to catch contract breaks.

→ Use deployment ordering: deploy backing services first, then dependent services.

→ Use GitOps (Argo CD/Flux) so reconciliation handles ordering and drift automatically.

</details>

<details>
<summary>14. How would you configure Jenkins to automatically clean up old build artifacts and workspaces?</summary>

→ Set `buildDiscarder` in the pipeline:
<pre>
options {
  buildDiscarder(logRotator(numToKeepStr: '20', artifactNumToKeepStr: '5'))
}
</pre>

→ Also configure **Workspace Cleanup** plugin with `cleanWs()` at the end of the build.

→ Schedule disk-usage monitoring and node cleanup jobs.

</details>

<details>
<summary>15. A Jenkins job needs to access files from a remote server during the build. How would you configure it?</summary>

→ Use `scp`, `rsync`, or a plugin like **Publish over SSH** / **SSH Agent**.

<pre>
withCredentials([sshUserPrivateKey(credentialsId: 'ssh-key', keyFileVariable: 'KEY')]) {
  sh 'scp -i $KEY user@server:/path/file .'
}
</pre>

→ For repeated access, configure a shared credential and an SSH agent.

</details>

<details>
<summary>16. How would you configure post-build actions in Jenkins?</summary>

→ Use the `post` block in Declarative pipelines.

<pre>
post {
  success { echo 'Success!' }
  failure { echo 'Failed!' }
  always  { junit '**/target/surefire-reports/*.xml' }
}
</pre>

→ Supported conditions: `always`, `success`, `failure`, `unstable`, `changed`, `fixed`, `regression`.

</details>

<details>
<summary>17. A Jenkins job needs to access a database during the build. How do you configure that?</summary>

→ Store DB credentials as Jenkins Credentials, use `withCredentials` to inject them, and run a script that connects.

<pre>
withCredentials([usernamePassword(credentialsId: 'db-creds',
    usernameVariable: 'DB_USER', passwordVariable: 'DB_PASS')]) {
  sh 'psql -h dbserver -U $DB_USER -f schema.sql'
}
</pre>

→ For network-restricted DBs, use a Jenkins agent in the same VNet.

</details>

<details>
<summary>18. How would you configure Jenkins to run multiple builds in parallel?</summary>

→ Use `parallel` blocks in a stage:
<pre>
stage('Tests') {
  parallel {
    stage('Unit')  { steps { sh 'mvn test' } }
    stage('Integ') { steps { sh 'mvn verify' } }
  }
}
</pre>

→ Or use `matrix` for parameterised parallel builds.

</details>

<details>
<summary>19. How would you configure Jenkins for multi-environment deployments (dev, staging, prod)?</summary>

→ Use pipeline **parameters** to pick the environment, and **input** approval steps for prod.

<pre>
parameters {
  choice(name: 'ENV', choices: ['dev', 'staging', 'prod'])
}
stages {
  stage('Deploy') {
    steps { sh "deploy.sh ${params.ENV}" }
  }
}
</pre>

→ Gate prod with an `input` step and store env-specific config in Key Vault.

</details>

<details>
<summary>20. How would you configure Jenkins to generate and publish code documentation after each build?</summary>

→ Add a docs stage that runs the doc generator (e.g., `mvn javadoc:javadoc`, `jsdoc`, `sphinx`).

→ Publish with the **HTML Publisher** plugin:
<pre>
publishHTML(target: [
  reportDir: 'target/site/apidocs',
  reportFiles: 'index.html',
  reportName: 'API Docs'
])
</pre>

</details>

<details>
<summary>21. How would you configure Jenkins to run a job on a specific OS (Windows or Linux)?</summary>

→ Label agents by OS (`windows`, `linux`) and target in the pipeline.

<pre>
agent { label 'windows' }
</pre>

→ Use `bat` on Windows and `sh` on Linux steps.

</details>

<details>
<summary>22. How would you configure build triggers in Jenkins based on conditions (files present, previous build success)?</summary>

→ **Polling/webhook triggers** for SCM changes.

→ **Upstream/downstream triggers** for build chains.

→ For conditionals inside a pipeline, use the `when` block:
<pre>
stage('Deploy') {
  when {
    expression { fileExists('build/artifact.jar') }
  }
  steps { sh 'deploy.sh' }
}
</pre>

→ Use the **Conditional BuildStep** plugin for freestyle.

</details>

<details>
<summary>23. How would you configure Jenkins to distribute a pipeline across multiple Jenkins masters?</summary>

→ Connect masters via the **master-slave** model: each remote master registers as a node or agent.

→ Use the **Jenkins Remote Access API** or **Publish over SSH** to trigger remote masters.

→ Or converge all into a single master with agents labelled by purpose.

→ For true multi-cluster, use **Jenkins Kubernetes plugin** or a self-hosted CloudBees/Jenkins Enterprise setup.

</details>

<details>
<summary>24. How would you configure Jenkins for source code checkout from Git/SVN?</summary>

→ In Freestyle: use the SCM section and set the repo URL, credentials, and branch.

→ In Pipeline:
<pre>
stage('Checkout') {
  steps {
    git branch: 'main',
        credentialsId: 'git-creds',
        url: 'https://github.com/org/repo.git'
  }
}
</pre>

→ Enable webhooks for auto-trigger.

</details>

<details>
<summary>25. How would you configure Jenkins to install required dependencies for a build?</summary>

→ Install tool versions via **Global Tool Configuration** (JDK, Maven, Node) and select them per job with `tools {}`.

→ Or use agents whose images already have dependencies baked in.

→ For ad-hoc setup:
<pre>
sh 'apt-get install -y jq'
</pre>

→ For repeatable environments, build a Docker agent image.

</details>

<details>
<summary>26. How would you configure Jenkins to run tests in parallel across multiple agents and collect results?</summary>

→ Use `parallel` with distinct `agent` labels per branch:
<pre>
stage('Tests') {
  parallel {
    stage('Linux')   { agent { label 'linux' }   steps { sh 'mvn test' } }
    stage('Windows') { agent { label 'windows' } steps { bat 'mvn test' } }
  }
}
</pre>

→ Collect with `junit '**/surefire-reports/*.xml'` to get a unified report.

</details>

<details>
<summary>27. How would you pass Jenkins agent environment variables to a job?</summary>

→ Agent env vars are inherited by `sh`/`bat` steps.

<pre>
sh 'echo $PATH'
sh 'echo $HOME'
</pre>

→ To make them available in Groovy, use `env.PATH` etc. Or define custom ones under `environment {}`.

</details>

<details>
<summary>28. How would you configure Jenkins to run a build only if changes are made to specific files or directories?</summary>

→ Use `changeset` in the `when` block:
<pre>
stage('Build API') {
  when {
    changeset 'api/**'
  }
  steps { sh 'build-api.sh' }
}
</pre>

→ Or use path-filtered webhooks or the GitHub Pull Request plugin's path filtering.

</details>

<details>
<summary>29. How would you configure Jenkins to make API calls to an external service during a build?</summary>

→ Use `httpRequest` from the HTTP Request plugin:
<pre>
httpRequest url: 'https://api.example.com/deploy',
            httpMode: 'POST',
            contentType: 'APPLICATION_JSON',
            requestBody: '{"env":"prod"}'
</pre>

→ Or use `curl`/`Invoke-RestMethod` inside a shell/batch step.

</details>

<details>
<summary>30. How would you configure Jenkins to archive and store build artifacts?</summary>

→ Use `archiveArtifacts` in the post block:
<pre>
post {
  success {
    archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
  }
}
</pre>

→ Artifacts are stored and can be downloaded from the build page.

</details>

<details>
<summary>31. How would you configure Jenkins to assign a job to an agent with specific labels/tags?</summary>

<pre>
agent { label 'linux && docker && high-mem' }
</pre>

→ The scheduler matches the expression against available agents.

</details>

<details>
<summary>32. How would you configure Jenkins to generate and display test coverage reports?</summary>

→ Run the coverage tool (JaCoCo, Cobertura, Istanbul) in the test stage, then publish using the corresponding plugin.

<pre>
stage('Coverage') {
  steps { junit '**/surefire-reports/*.xml' }
  post { always { jacoco execPattern: '**/*.exec', classPattern: '**/classes', sourcePattern: '**/src/main' } }
}
</pre>

</details>

<details>
<summary>33. How would you configure Jenkins to authenticate and access a secure remote Maven repository?</summary>

→ Store the repo credentials as Jenkins Credentials and reference them in `settings.xml` via the **Config File Provider** plugin.

→ Or use `withCredentials` to inject the creds into the `mvn` command.

</details>

<details>
<summary>34. How would you configure Jenkins to run a build only when manually triggered by a user?</summary>

→ Do not enable SCM triggers. Optionally add an `input` step for approval.

<pre>
stage('Deploy') {
  input { message "Deploy to production?" }
  steps { sh 'deploy.sh' }
}
</pre>

</details>

<details>
<summary>35. How would you configure Jenkins to accept and use build parameters (e.g., version number, environment)?</summary>

<pre>
parameters {
  string(name: 'VERSION', defaultValue: '1.0.0', description: 'App version')
  choice(name: 'ENV', choices: ['dev', 'prod'])
}
steps {
  sh "deploy.sh --version ${params.VERSION} --env ${params.ENV}"
}
</pre>

</details>

<details>
<summary>36. How would you configure Jenkins for webhook triggering from an external system?</summary>

→ Expose a generic webhook endpoint via the **Generic Webhook Trigger** plugin.

→ Configure the build to trigger on a JSON payload from the external system.

→ Alternatively, use the JWT-based API with `curl -X POST $JENKINS_URL/job/JOB/build`.

</details>

<details>
<summary>37. How would you configure Jenkins to execute a shell script or batch file as part of the build?</summary>

<pre>
sh './scripts/deploy.sh'
bat 'scripts\\deploy.bat'
</pre>

→ Use `returnStatus` or `returnStdout` for programmatic control.

</details>

<details>
<summary>38. How would you configure Jenkins to run a build only when triggered by a specific user or group?</summary>

→ Use the **Matrix Authorization** plugin for user-based permissions.

→ In pipelines, check `currentBuild.getBuildCauses()` and fail if the trigger isn't the expected user.

→ Or use the **Authorize Project** plugin.

</details>

<details>
<summary>39. How would you configure Jenkins to handle mobile app signing and distribution?</summary>

→ Store signing keystores as Jenkins Credentials.

→ In the build step, inject the keystore, sign with `jarsigner`/`apksigner`/Xcode tools.

→ Distribute to Play Store (via `gradle-play-publisher`), App Store (via `xcrun altool`), or Firebase App Distribution.

</details>

<details>
<summary>40. How would you configure Jenkins for branch or tag-based build triggering?</summary>

→ Use **Multibranch Pipeline** with branch sources — auto-discovers branches and tags.

→ Or use `when { buildingTag() }` and `when { branch 'main' }` in stages.

<pre>
stage('Release') {
  when { buildingTag() }
  steps { sh 'release.sh' }
}
</pre>

</details>

<details>
<summary>41. How would you configure Jenkins to establish an SSH connection to a remote server during a build?</summary>

→ Use the **SSH Agent** plugin with an SSH key credential.

<pre>
sshagent(credentials: ['ssh-key-id']) {
  sh 'ssh user@remote-host "ls /var/www"'
}
</pre>

→ For file transfer, use `scp`/`rsync` inside the same block.

</details>

<details>
<summary>42. How would you configure time-based build triggering?</summary>

→ Use cron triggers directly.

<pre>
triggers { cron('H 22 * * 1-5') }  # weekday nights at 22:00
</pre>

→ Use `H` to spread load. Combine with `when { ... }` to add extra conditions.

</details>

<details>
<summary>43. How would you configure Jenkins to install and use a specific JDK or runtime environment?</summary>

→ Configure the JDK in **Global Tool Configuration**, then select it in the job.

<pre>
tools {
  jdk 'jdk17'
  maven 'maven3'
}
steps {
  sh 'java -version'
  sh 'mvn -version'
}
</pre>

→ Or run inside a Docker agent with the correct image.

</details>

<details>
<summary>44. How would you configure build conditions in Jenkins such as "success of previous build" or "absence of certain files"?</summary>

<pre>
stage('Deploy') {
  when {
    expression { currentBuild.result == null || currentBuild.result == 'SUCCESS' }
    expression { !fileExists('DO_NOT_DEPLOY') }
  }
  steps { sh 'deploy.sh' }
}
</pre>

→ Combine with `allOf`, `anyOf`, `not`.

</details>

<details>
<summary>45. How would you configure Jenkins to authenticate and access a secure Docker registry?</summary>

→ Store the registry creds as Jenkins Credentials and log in via `withCredentials`.

<pre>
withCredentials([usernamePassword(credentialsId: 'docker-registry',
    usernameVariable: 'U', passwordVariable: 'P')]) {
  sh 'echo $P | docker login myregistry.azurecr.io -u $U --password-stdin'
  sh 'docker push myregistry.azurecr.io/myapp:latest'
}
</pre>

</details>

<details>
<summary>46. How would you configure event-based build triggering?</summary>

→ Use webhooks from the event source (GitHub, GitLab, Bitbucket, or generic via Generic Webhook Trigger).

→ For system events (e.g., S3 uploads, message queue), use a Lambda/Function that hits the Jenkins API.

→ For build chains, use **upstream triggers** or the Build Trigger plugin.

</details>

<details>
<summary>47. How would you configure some stages to run in parallel while others run sequentially?</summary>

<pre>
stages {
  stage('Build & Test') {
    parallel {
      stage('Build') { steps { sh 'mvn package' } }
      stage('Lint')  { steps { sh 'mvn checkstyle' } }
    }
  }
  stage('Deploy') {          # runs after parallel
    steps { sh 'deploy.sh' }
  }
}
</pre>

</details>

<details>
<summary>48. How would you configure Jenkins for distributed build execution across multiple agents simultaneously?</summary>

→ Use `parallel` with different `agent` labels so each branch runs on its own agent.

→ Or use `matrix` with agent labels for scaling shard tests.

→ Use the **Throttle Concurrent Builds** plugin to control total concurrency.

</details>

<details>
<summary>49. How would you configure Jenkins to install and use custom tools on the agent?</summary>

→ Install on the agent image (Dockerfile) or via `Global Tool Configuration` with custom tool installers.

→ For ad-hoc, use `tool` or shell steps:
<pre>
sh 'apt-get update && apt-get install -y terraform'
</pre>

→ For repeatable, bake tools into a Docker agent image and run with `agent { docker { image 'my-tools:latest' } }`.

</details>

<details>
<summary>50. How would you configure Jenkins to track and display build and test trends over time using graphs?</summary>

→ Use the **JUnit**, **JaCoCo**, **Coverage**, **Performance**, and **Warnings Next Generation** plugins.

→ Publish these in `post` stages, and Jenkins will build trend charts automatically.

→ The **Blue Ocean** UI also visualises pipeline trends.

</details>

---

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

# Terraform

<details>
<summary>1. How is your infrastructure currently managed, and what are the existing Terraform configurations?</summary>

→ Infrastructure is managed as code in a dedicated repo with a modular structure (`modules/`, `env/dev/`, `env/prod/`).

→ State is stored in Azure Storage with locking.

→ CI/CD pipeline runs `plan` on PR, `apply` on merge, with a manual gate for production.

</details>

<details>
<summary>2. What are the key components of your infrastructure that need to be defined in Terraform?</summary>

→ Resource groups, VNets, subnets, NSGs, and route tables.

→ AKS clusters, node pools, and associated identities.

→ ACR, Key Vaults, Storage accounts (with private endpoints), and Application Gateway.

→ Monitoring (Log Analytics, App Insights), DNS zones, and RBAC assignments.

</details>

<details>
<summary>3. How do you handle state management and locking with Terraform?</summary>

→ Use a remote backend (Azure Storage or S3).

→ State locking is provided by blob leases (Azure) or DynamoDB (AWS).

<pre>
terraform {
  backend "azurerm" {
    resource_group_name  = "tfstate-rg"
    storage_account_name = "tfstatestorage"
    container_name       = "tfstate"
    key                  = "prod.terraform.tfstate"
  }
}
</pre>

→ State files are versioned; recover from prior versions if needed.

</details>

---

# Ansible

<details>
<summary>1. What configuration management practices are currently in place?</summary>

→ Ansible playbooks stored in Git, grouped into roles.

→ Inventory files per environment.

→ Idempotent, tagged tasks for selective execution.

→ Vault-encrypted secrets (`ansible-vault`).

</details>

<details>
<summary>2. Are there specific Ansible playbooks or roles that need to be developed or modified?</summary>

→ Common roles: `common` (users, ssh, base packages), `docker`, `kubernetes-node`, `monitoring-agent`, `security-hardening`.

→ Playbooks assembled per environment from roles.

→ New roles needed for any new technology (e.g., CNI network plugin deployment).

</details>

<details>
<summary>3. How do you manage configuration drift and ensure consistency across environments?</summary>

→ Run playbooks regularly (idempotency means no change = compliant).

→ Use `--check` (dry run) for drift detection.

→ Compare host state with a baseline; alert on diff.

→ Supplement with infrastructure-level tools (Terraform plan, Azure Policy).

</details>

---

# HashiCorp Packer

<details>
<summary>1. What are the current image-building processes, and what platforms do you need to support?</summary>

→ Images built with Packer, source = base image (e.g., Ubuntu), provisioners = Ansible roles.

→ Output → Azure Shared Image Gallery or ACR.

→ Platforms: Azure VMs, AKS node images, Windows Server images.

</details>

<details>
<summary>2. How often are new images built, and what triggers the builds?</summary>

→ Weekly scheduled builds for security patches.

→ On CVE disclosures for critical fixes.

→ On base image / provisioner updates.

→ Manual trigger for one-off builds.

</details>

<details>
<summary>3. Are there specific compliance or security requirements for the images?</summary>

→ CIS benchmark hardening.

→ No root login, SSH key-only, auditd, patched packages.

→ Signed image attestation and scans (defender for cloud) before release.

</details>

---

# Security & Compliance

<details>
<summary>1. What are the current security policies and compliance requirements for your application and infrastructure?</summary>

→ CIS, SOC 2, GDPR, and internal security baselines.

→ Azure Policy enforces in-tenant requirements (no public IPs, disk encryption, etc.).

→ Regular vulnerability scans and patch SLAs.

</details>

<details>
<summary>2. How are security vulnerabilities currently identified and addressed?</summary>

→ Continuous scanning: image (Trivy, Defender), dependencies (Snyk), code (SonarQube).

→ Triage → patch → rescan; track in the ticketing system.

→ Monthly security review with stakeholders.

</details>

<details>
<summary>3. How is access to the infrastructure and application managed?</summary>

→ Identity: Azure Entra ID with MFA.

→ Azure RBAC at subscription/RG/resource levels; least privilege.

→ Just-In-Time (PIM/PAM) for privileged roles.

→ Kubernetes RBAC + Azure AD integration for cluster access.

</details>

<details>
<summary>4. Are there any specific roles and permissions that need to be defined?</summary>

→ DevOps engineers: Contributor on dev/staging, restricted on prod (PEB — privileged elevation).

→ Developers: Reader + deploy-on-approval.

→ Security team: Reader + policy assignment + audit.

→ Break-glass accounts: emergency only, fully audited.

</details>

<details>
<summary>5. What are the data protection requirements for this project?</summary>

→ Data classified by sensitivity (public/internal/confidential/restricted).

→ Encryption at rest (Azure Storage/DB disk encryption, Key Vault CMK).

→ Encryption in transit (TLS 1.2+, HTTPS-only).

→ Backup with retention; geo-redundancy based on tier.

</details>

<details>
<summary>6. How is sensitive data currently encrypted and stored?</summary>

→ Secrets/keys stored in Azure Key Vault, accessed via managed identity.

→ Data at rest: SSE with CMK for storage, TDE for SQL.

→ Data in transit: TLS.

→ Data masking for PII in logs and non-prod environments.

</details>

<details>
<summary>7. How will you secure Azure resources?</summary>

→ Network: private endpoints, NSGs, Azure Firewall, private AKS clusters.

→ Identity: managed identity, RBAC, Azure AD Conditional Access.

→ Data: encryption at rest (CMK), TLS in transit, Key Vault for secrets/certs.

→ Monitoring: Defender for Cloud, Azure Monitor alerts, Microsoft Sentinel.

→ Governance: Azure Policy, resource locks, tagging.

</details>

<details>
<summary>8. In SonarQube, how do you make the gates focus only on new code?</summary>

→ Configure **New Code definition** in the project — options are Previous Version, Number of Days, or Reference Branch (`main`).

→ Set the **Quality Gate** so it evaluates only new code (SonarQube "Clean as You Code" approach).

→ Attach the Quality Gate as a mandatory condition on PRs so only new issues block merges — not legacy debt.

</details>

---

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

