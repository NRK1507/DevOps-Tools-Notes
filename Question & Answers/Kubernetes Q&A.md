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
