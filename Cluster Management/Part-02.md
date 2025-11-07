```
📘 Scenario #26: Taints and Tolerations Mismatch Prevented Workload Scheduling
Category: Cluster Management
Environment: K8s v1.22, managed AKS
Scenario Summary: Workloads failed to schedule on new nodes that had a taint the workloads didn’t tolerate.
What Happened: Platform team added a new node pool with node-role.kubernetes.io/gpu:NoSchedule, but forgot to add tolerations to GPU workloads.
Diagnosis Steps:
	• kubectl describe pod – showed reason: “0/3 nodes are available: node(s) had taints”.
	• Checked node taints via kubectl get nodes -o json.
Root Cause: Taints on new node pool weren’t matched by tolerations in pods.
Fix/Workaround:
	• Added proper tolerations to workloads:

yaml
CopyEdit
tolerations:
- key: "node-role.kubernetes.io/gpu"
  operator: "Exists"
  effect: "NoSchedule"
Lessons Learned: Node taints should be coordinated with scheduling policies.
How to Avoid:
	• Use preset toleration templates in CI/CD pipelines.
	• Test new node pools with dummy workloads.

📘 Scenario #27: Node Bootstrap Failure Due to Unavailable Container Registry
Category: Cluster Management
Environment: K8s v1.21, on-prem, private registry
Scenario Summary: New nodes failed to join the cluster due to container runtime timeout when pulling base images.
What Happened: The internal Docker registry was down during node provisioning, so containerd couldn't pull pauseand CNI images. Nodes stayed in NotReady state.
Diagnosis Steps:
	• journalctl -u containerd – repeated image pull failures.
	• Node conditions showed ContainerRuntimeNotReady.
Root Cause: Bootstrap process relies on image pulls from unavailable registry.
Fix/Workaround:
	• Brought internal registry back online.
	• Pre-pulled pause/CNI images to node image templates.
Lessons Learned: Registry availability is a bootstrap dependency.
How to Avoid:
	• Preload all essential images into AMI/base image.
	• Monitor registry uptime independently.

📘 Scenario #28: kubelet Fails to Start Due to Expired TLS Certs
Category: Cluster Management
Environment: K8s v1.19, kubeadm cluster
Scenario Summary: Several nodes went NotReady after reboot due to kubelet failing to start with expired client certs.
What Happened: Kubelet uses a client certificate for authentication with the API server. These are typically auto-rotated, but the nodes were offline when the rotation was due.
Diagnosis Steps:
	• journalctl -u kubelet – cert expired error.
	• /var/lib/kubelet/pki/kubelet-client-current.pem – expired date.
Root Cause: Kubelet cert rotation missed due to node downtime.
Fix/Workaround:
	• Regenerated kubelet certs using kubeadm.

bash
CopyEdit
kubeadm certs renew all
Lessons Learned: Cert rotation has a dependency on uptime.
How to Avoid:
	• Monitor cert expiry proactively.
	• Rotate certs manually before planned outages.

📘 Scenario #29: kube-scheduler Crash Due to Invalid Leader Election Config
Category: Cluster Management
Environment: K8s v1.24, custom scheduler deployment
Scenario Summary: kube-scheduler pod failed with panic due to misconfigured leader election flags.
What Happened: An override in the Helm chart introduced an invalid leader election namespace, causing the scheduler to panic and crash on startup.
Diagnosis Steps:
	• Pod logs showed panic: cannot create leader election record.
	• Checked Helm values – found wrong namespace name.
Root Cause: Namespace specified for leader election did not exist.
Fix/Workaround:
	• Created the missing namespace.
	• Restarted the scheduler pod.
Lessons Learned: Leader election is sensitive to namespace scoping.
How to Avoid:
	• Use default kube-system unless explicitly scoped.
	• Validate all scheduler configs with CI linting.

📘 Scenario #30: Cluster DNS Resolution Broken After Calico CNI Update
Category: Cluster Management
Environment: K8s v1.23, self-hosted Calico
Scenario Summary: DNS resolution broke after Calico CNI update due to iptables policy drop changes.
What Happened: New version of Calico enforced stricter iptables drop policies, blocking traffic from CoreDNS to pods.
Diagnosis Steps:
	• DNS requests timed out.
	• Packet capture showed ICMP unreachable from pods to CoreDNS.
	• Checked Calico policy and iptables rules.
Root Cause: Calico’s default deny policy applied to kube-dns traffic.
Fix/Workaround:
	• Added explicit Calico policy allowing kube-dns to pod traffic.

yaml:
egress:
- action: Allow
  destination:
    selector: "k8s-app == 'kube-dns'"

Lessons Learned: CNI policy changes can impact DNS without warning.
How to Avoid:
	• Review and test all network policy upgrades in staging.
	• Use canary upgrade strategy for CNI.

📘 Scenario #31: Node Clock Drift Causing Authentication Failures
Category: Cluster Management
Environment: K8s v1.22, on-prem, kubeadm
Scenario Summary: Authentication tokens failed across the cluster due to node clock skew.
What Happened: Token-based authentication failed for all workloads and kubectl access due to time drift between worker nodes and the API server.
Diagnosis Steps:
	• Ran kubectl logs and found expired token errors.
	• Checked node time using date on each node – found significant drift.
	• Verified NTP daemon status – not running.
Root Cause: NTP daemon disabled on worker nodes.
Fix/Workaround:
	• Re-enabled and restarted NTP on all nodes.
	• Synchronized system clocks manually.
Lessons Learned: Time synchronization is critical for certificate and token-based auth.
How to Avoid:
	• Ensure NTP or chrony is enabled via bootstrap configuration.
	• Monitor time drift via node-exporter.

📘 Scenario #32: Inconsistent Node Labels Causing Scheduling Bugs
Category: Cluster Management
Environment: K8s v1.24, multi-zone GKE
Scenario Summary: Zone-aware workloads failed to schedule due to missing zone labels on some nodes.
What Happened: Pods using topologySpreadConstraints for zone balancing failed to find valid nodes because some nodes lacked the topology.kubernetes.io/zone label.
Diagnosis Steps:
	• Pod events showed no matching topology key errors.
	• Compared node labels across zones – found inconsistency.
Root Cause: A few nodes were manually added without required zone labels.
Fix/Workaround:
	• Manually patched node labels to restore zone metadata.
Lessons Learned: Label uniformity is essential for topology constraints.
How to Avoid:
	• Automate label injection using cloud-init or DaemonSet.
	• Add CI checks for required labels on node join.

📘 Scenario #33: API Server Slowdowns from High Watch Connection Count
Category: Cluster Management
Environment: K8s v1.23, OpenShift
Scenario Summary: API latency rose sharply due to thousands of watch connections from misbehaving clients.
What Happened: Multiple pods opened persistent watch connections and never closed them, overloading the API server.
Diagnosis Steps:
	• Monitored API metrics /metrics for apiserver_registered_watchers.
	• Identified top offenders using connection source IPs.
Root Cause: Custom controller with poor watch logic never closed connections.
Fix/Workaround:
	• Restarted offending pods.
	• Updated controller to reuse watches.
Lessons Learned: Unbounded watches can exhaust server resources.
How to Avoid:
	• Use client-go with resync periods and connection limits.
	• Enable metrics to detect watch leaks early.

📘 Scenario #34: Etcd Disk Full Crashing the Cluster
Category: Cluster Management
Environment: K8s v1.21, self-managed with local etcd
Scenario Summary: Entire control plane crashed due to etcd disk running out of space.
What Happened: Continuous writes from custom resources filled the disk where etcd data was stored.
Diagnosis Steps:
	• Observed etcdserver: mvcc: database space exceeded errors.
	• Checked disk usage: df -h showed 100% full.
Root Cause: No compaction or defragmentation done on etcd for weeks.
Fix/Workaround:
	• Performed etcd compaction and defragmentation.
	• Added disk space temporarily.
Lessons Learned: Etcd needs regular maintenance.
How to Avoid:
	• Set up cron jobs or alerts for etcd health.
	• Monitor disk usage and trigger auto-compaction.

📘 Scenario #35: ClusterConfigMap Deleted by Accident Bringing Down Addons
Category: Cluster Management
Environment: K8s v1.24, Rancher
Scenario Summary: A user accidentally deleted the kube-root-ca.crt ConfigMap, which many workloads relied on.
What Happened: Pods mounting the kube-root-ca.crt ConfigMap failed to start after deletion. DNS, metrics-server, and other system components failed.
Diagnosis Steps:
	• Pod events showed missing ConfigMap errors.
	• Attempted to remount volumes manually.
Root Cause: System-critical ConfigMap was deleted without RBAC protections.
Fix/Workaround:
	• Recreated ConfigMap from backup.
	• Re-deployed affected system workloads.
Lessons Learned: Some ConfigMaps are essential and must be protected.
How to Avoid:
	• Add RBAC restrictions to system namespaces.
	• Use OPA/Gatekeeper to prevent deletions of protected resources.

📘 Scenario #36: Misconfigured NodeAffinity Excluding All Nodes
Category: Cluster Management
Environment: K8s v1.22, Azure AKS
Scenario Summary: A critical deployment was unschedulable due to strict nodeAffinity rules.
What Happened: nodeAffinity required a zone that did not exist in the cluster, making all nodes invalid.
Diagnosis Steps:
	• Pod events showed 0/10 nodes available errors.
	• Checked spec.affinity section in deployment YAML.
Root Cause: Invalid or overly strict requiredDuringScheduling nodeAffinity.
Fix/Workaround:
	• Updated deployment YAML to reflect actual zones.
	• Re-deployed workloads.
Lessons Learned: nodeAffinity is strict and should be used carefully.
How to Avoid:
	• Validate node labels before setting affinity.
	• Use preferredDuringScheduling for soft constraints.

📘 Scenario #37: Outdated Admission Webhook Blocking All Deployments
Category: Cluster Management
Environment: K8s v1.25, self-hosted
Scenario Summary: A stale mutating webhook caused all deployments to fail due to TLS certificate errors.
What Happened: The admission webhook had expired TLS certs, causing validation errors on all resource creation attempts.
Diagnosis Steps:
	• Created a dummy pod and observed webhook errors.
	• Checked logs of the webhook pod – found TLS handshake failures.
Root Cause: Webhook server was down due to expired TLS cert.
Fix/Workaround:
	• Renewed cert and redeployed webhook.
	• Disabled webhook temporarily for emergency deployments.
Lessons Learned: Webhooks are gatekeepers – they must be monitored.
How to Avoid:
	• Rotate webhook certs using cert-manager.
	• Alert on webhook downtime or errors.

📘 Scenario #38: API Server Certificate Expiry Blocking Cluster Access
Category: Cluster Management
Environment: K8s v1.19, kubeadm
Scenario Summary: After 1 year of uptime, API server certificate expired, blocking access to all components.
What Happened: Default kubeadm cert rotation didn’t occur, leading to expiry of API server and etcd peer certs.
Diagnosis Steps:
	• kubectl failed with x509: certificate has expired.
	• Checked /etc/kubernetes/pki/apiserver.crt expiry date.
Root Cause: kubeadm certificates were never rotated or renewed.
Fix/Workaround:
	• Used kubeadm certs renew all.
	• Restarted control plane components.
Lessons Learned: Certificates expire silently unless monitored.
How to Avoid:
	• Rotate certs before expiry.
	• Monitor /metrics for cert validity.

📘 Scenario #39: CRI Socket Mismatch Preventing kubelet Startup
Category: Cluster Management
Environment: K8s v1.22, containerd switch
Scenario Summary: kubelet failed to start after switching from Docker to containerd due to incorrect CRI socket path.
What Happened: The node image had containerd installed, but the kubelet still pointed to the Docker socket.
Diagnosis Steps:
	• Checked kubelet logs for failed to connect to CRI socket.
	• Verified config file at /var/lib/kubelet/kubeadm-flags.env.
Root Cause: Wrong --container-runtime-endpoint specified.
Fix/Workaround:
	• Updated kubelet flags to point to /run/containerd/containerd.sock.
	• Restarted kubelet.
Lessons Learned: CRI migration requires explicit config updates.
How to Avoid:
	• Use migration scripts or kubeadm migration guides.
	• Validate container runtime on node bootstrap.

📘 Scenario #40: Cluster-Wide Crash Due to Misconfigured Resource Quotas
Category: Cluster Management
Environment: K8s v1.24, multi-tenant namespace setup
Scenario Summary: Cluster workloads failed after applying overly strict resource quotas that denied new pod creation.
What Happened: A new quota was applied with very low CPU/memory limits. All new pods across namespaces failed scheduling.
Diagnosis Steps:
	• Pod events showed failed quota check errors.
	• Checked quota via kubectl describe quota in all namespaces.
Root Cause: Misconfigured CPU/memory limits set globally.
Fix/Workaround:
	• Rolled back the quota to previous values.
	• Unblocked critical namespaces manually.
Lessons Learned: Quota changes should be staged and validated.
How to Avoid:
	• Test new quotas in shadow or dry-run mode.
	• Use automated checks before applying quotas.

📘 Scenario #41: Cluster Upgrade Failing Due to CNI Compatibility
Category: Cluster Management
Environment: K8s v1.21 to v1.22, custom CNI plugin
Scenario Summary: Cluster upgrade failed due to an incompatible version of the CNI plugin.
What Happened: After upgrading the control plane, CNI plugins failed to work, resulting in no network connectivity between pods.
Diagnosis Steps:
	• Checked kubelet and container runtime logs – observed CNI errors.
	• Verified CNI plugin version – it was incompatible with K8s v1.22.
Root Cause: CNI plugin was not upgraded alongside the Kubernetes control plane.
Fix/Workaround:
	• Upgraded the CNI plugin to the version compatible with K8s v1.22.
	• Restarted affected pods and nodes.
Lessons Learned: Always ensure compatibility between the Kubernetes version and CNI plugin.
How to Avoid:
	• Follow Kubernetes upgrade documentation and ensure CNI plugins are upgraded.
	• Test in a staging environment before performing production upgrades.

📘 Scenario #42: Failed Pod Security Policy Enforcement Causing Privileged Container Launch
Category: Cluster Management
Environment: K8s v1.22, AWS EKS
Scenario Summary: Privileged containers were able to run despite Pod Security Policy enforcement.
What Happened: A container was able to run as privileged despite a restrictive PodSecurityPolicy being in place.
Diagnosis Steps:
	• Checked pod events and logs, found no violations of PodSecurityPolicy.
	• Verified PodSecurityPolicy settings and namespace annotations.
Root Cause: PodSecurityPolicy was not enforced due to missing podsecuritypolicy admission controller.
Fix/Workaround:
	• Enabled the podsecuritypolicy admission controller.
	• Updated the PodSecurityPolicy to restrict privileged containers.
Lessons Learned: Admission controllers must be properly configured for security policies to be enforced.
How to Avoid:
	• Double-check admission controller configurations during initial cluster setup.
	• Regularly audit security policies and admission controllers.

📘 Scenario #43: Node Pool Scaling Impacting StatefulSets
Category: Cluster Management
Environment: K8s v1.24, GKE
Scenario Summary: StatefulSet pods were rescheduled across different nodes, breaking persistent volume bindings.
What Happened: Node pool scaling in GKE triggered a rescheduling of StatefulSet pods, breaking persistent volume claims that were tied to specific nodes.
Diagnosis Steps:
	• Observed failed to bind volume errors.
	• Checked StatefulSet configuration for node affinity and volume binding policies.
Root Cause: Lack of proper node affinity or persistent volume binding policies in StatefulSet configuration.
Fix/Workaround:
	• Added proper node affinity rules and volume binding policies to StatefulSet.
	• Rescheduled the pods successfully.
Lessons Learned: StatefulSets require careful management of node affinity and persistent volume binding policies.
How to Avoid:
	• Use pod affinity rules for StatefulSets to ensure proper scheduling and volume binding.
	• Monitor volume binding status when scaling node pools.

📘 Scenario #44: Kubelet Crash Due to Out of Memory (OOM) Errors
Category: Cluster Management
Environment: K8s v1.20, bare metal
Scenario Summary: Kubelet crashed after running out of memory due to excessive pod resource usage.
What Happened: The kubelet on a node crashed after the available memory was exhausted due to pods consuming more memory than allocated.
Diagnosis Steps:
	• Checked kubelet logs for OOM errors.
	• Used kubectl describe node to check resource utilization.
Root Cause: Pod resource requests and limits were not set properly, leading to excessive memory consumption.
Fix/Workaround:
	• Set proper resource requests and limits on pods to prevent memory over-consumption.
	• Restarted the kubelet on the affected node.
Lessons Learned: Pod resource limits and requests are essential for proper node resource utilization.
How to Avoid:
	• Set reasonable resource requests and limits for all pods.
	• Monitor node resource usage to catch resource overuse before it causes crashes.

📘 Scenario #45: DNS Resolution Failure in Multi-Cluster Setup
Category: Cluster Management
Environment: K8s v1.23, multi-cluster federation
Scenario Summary: DNS resolution failed between two federated clusters due to missing DNS records.
What Happened: DNS queries failed between two federated clusters, preventing services from accessing each other across clusters.
Diagnosis Steps:
	• Used kubectl get svc to check DNS records.
	• Identified missing service entries in the DNS server configuration.
Root Cause: DNS configuration was incomplete, missing records for federated services.
Fix/Workaround:
	• Added missing DNS records manually.
	• Updated DNS configurations to include service records for all federated clusters.
Lessons Learned: In multi-cluster setups, DNS configuration is critical to service discovery.
How to Avoid:
	• Automate DNS record creation during multi-cluster federation setup.
	• Regularly audit DNS configurations in multi-cluster environments.

📘 Scenario #46: Insufficient Resource Limits in Autoscaling Setup
Category: Cluster Management
Environment: K8s v1.21, GKE with Horizontal Pod Autoscaler (HPA)
Scenario Summary: Horizontal Pod Autoscaler did not scale pods up as expected due to insufficient resource limits.
What Happened: The Horizontal Pod Autoscaler failed to scale the application pods up, even under load, due to insufficient resource limits set on the pods.
Diagnosis Steps:
	• Observed HPA metrics showing no scaling action.
	• Checked pod resource requests and limits.
Root Cause: Resource limits were too low for HPA to trigger scaling actions.
Fix/Workaround:
	• Increased resource requests and limits for the affected pods.
	• Manually scaled the pods and monitored the autoscaling behavior.
Lessons Learned: Proper resource limits are essential for autoscaling to function correctly.
How to Avoid:
	• Set adequate resource requests and limits for workloads managed by HPA.
	• Monitor autoscaling events to identify under-scaling issues.

📘 Scenario #47: Control Plane Overload Due to High Audit Log Volume
Category: Cluster Management
Environment: K8s v1.22, Azure AKS
Scenario Summary: The control plane became overloaded and slow due to excessive audit log volume.
What Happened: A misconfigured audit policy led to high volumes of audit logs being generated, overwhelming the control plane.
Diagnosis Steps:
	• Monitored control plane metrics and found high CPU usage due to audit logs.
	• Reviewed audit policy and found it was logging excessive data.
Root Cause: Overly broad audit log configuration captured too many events.
Fix/Workaround:
	• Refined audit policy to log only critical events.
	• Restarted the API server.
Lessons Learned: Audit logging needs to be fine-tuned to prevent overload.
How to Avoid:
	• Regularly review and refine audit logging policies.
	• Set alerts for high audit log volumes.

📘 Scenario #48: Resource Fragmentation Causing Cluster Instability
Category: Cluster Management
Environment: K8s v1.23, bare metal
Scenario Summary: Resource fragmentation due to unbalanced pod distribution led to cluster instability.
What Happened: Over time, pod distribution became uneven, with some nodes over-committed while others remained underutilized. This caused resource fragmentation, leading to cluster instability.
Diagnosis Steps:
	• Checked node resource utilization and found over-committed nodes with high pod density.
	• Examined pod distribution and noticed skewed placement.
Root Cause: Lack of proper pod scheduling and resource management.
Fix/Workaround:
	• Applied pod affinity and anti-affinity rules to achieve balanced scheduling.
	• Rescheduled pods manually to redistribute workload.
Lessons Learned: Resource management and scheduling rules are crucial for maintaining cluster stability.
How to Avoid:
	• Use affinity and anti-affinity rules to control pod placement.
	• Regularly monitor resource utilization and adjust pod placement strategies.

📘 Scenario #49: Failed Cluster Backup Due to Misconfigured Volume Snapshots
Category: Cluster Management
Environment: K8s v1.21, AWS EBS
Scenario Summary: Cluster backup failed due to a misconfigured volume snapshot driver.
What Happened: The backup process failed because the EBS volume snapshot driver was misconfigured, resulting in incomplete backups.
Diagnosis Steps:
	• Checked backup logs for error messages related to volume snapshot failures.
	• Verified snapshot driver configuration in storage class.
Root Cause: Misconfigured volume snapshot driver prevented proper backups.
Fix/Workaround:
	• Corrected snapshot driver configuration in storage class.
	• Ran the backup process again, which completed successfully.
Lessons Learned: Backup configuration must be thoroughly checked and tested.
How to Avoid:
	• Automate backup testing and validation in staging environments.
	• Regularly verify backup configurations.

📘 Scenario #50: Failed Deployment Due to Image Pulling Issues
Category: Cluster Management
Environment: K8s v1.22, custom Docker registry
Scenario Summary: Deployment failed due to image pulling issues from a custom Docker registry.
What Happened: A deployment failed because Kubernetes could not pull images from a custom Docker registry due to misconfigured image pull secrets.
Diagnosis Steps:
	• Observed ImagePullBackOff errors for the failing pods.
	• Checked image pull secrets and registry configuration.
Root Cause: Incorrect or missing image pull secrets for accessing the custom registry.
Fix/Workaround:
	• Corrected the image pull secrets in the deployment YAML.
	• Re-deployed the application.
Lessons Learned: Image pull secrets must be configured properly for private registries.
How to Avoid:
	• Always verify image pull secrets for private registries.
	• Use Kubernetes secrets management tools for image pull credentials.
```
