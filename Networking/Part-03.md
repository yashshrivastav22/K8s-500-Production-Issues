```
📘 Scenario #151: NodePort Service Not Accessible Due to Firewall Rules
Category: Networking
Environment: K8s v1.23, Google GKE
Scenario Summary: A NodePort service became inaccessible due to restrictive firewall rules on the cloud provider.
What Happened: External access to a service using a NodePort was blocked because the cloud provider's firewall rules were too restrictive.
Diagnosis Steps:
	• Checked service configuration and confirmed that it was correctly exposed as a NodePort.
	• Used kubectl describe svc to verify the NodePort assigned.
	• Verified the firewall rules for the cloud provider and found that ingress was blocked on the NodePort range.
Root Cause: Firewall rules on the cloud provider were not configured to allow traffic on the NodePort range.
Fix/Workaround:
	• Updated the firewall rules to allow inbound traffic to the NodePort range.
	• Ensured that the required port was open on all nodes.
Lessons Learned: Always check cloud firewall rules when exposing services using NodePort.
How to Avoid:
	• Automate the validation of firewall rules after deploying NodePort services.
	• Document and standardize firewall configurations for all exposed services.

📘 Scenario #152: DNS Latency Due to Overloaded CoreDNS Pods
Category: Networking
Environment: K8s v1.19, AWS EKS
Scenario Summary: CoreDNS latency increased due to resource constraints on the CoreDNS pods.
What Happened: CoreDNS started experiencing high response times due to CPU and memory resource constraints, leading to DNS resolution delays.
Diagnosis Steps:
	• Checked CoreDNS pod resource usage and found high CPU usage.
	• Verified that DNS resolution was slowing down for multiple namespaces and services.
	• Increased logging verbosity for CoreDNS and identified high query volume.
Root Cause: CoreDNS pods did not have sufficient resources allocated to handle the query load.
Fix/Workaround:
	• Increased CPU and memory resource limits for CoreDNS pods.
	• Restarted CoreDNS pods to apply the new resource limits.
Lessons Learned: CoreDNS should be allocated appropriate resources based on expected load, especially in large clusters.
How to Avoid:
	• Set resource requests and limits for CoreDNS based on historical query volume.
	• Monitor CoreDNS performance and scale resources dynamically.

📘 Scenario #153: Network Performance Degradation Due to Misconfigured MTU
Category: Networking
Environment: K8s v1.20, on-premise
Scenario Summary: Network performance degraded due to an incorrect Maximum Transmission Unit (MTU) setting.
What Happened: Network performance between pods degraded after a change in the MTU settings in the CNI plugin.
Diagnosis Steps:
	• Used ping tests to diagnose high latency and packet drops between nodes.
	• Verified MTU settings on the nodes and CNI plugin, and found that the MTU was mismatched between the nodes and the CNI.
Root Cause: MTU mismatch between Kubernetes nodes and the CNI plugin.
Fix/Workaround:
	• Aligned the MTU settings between the CNI plugin and the Kubernetes nodes.
	• Rebooted affected nodes to apply the configuration changes.
Lessons Learned: Ensure that MTU settings are consistent across the network stack to avoid performance degradation.
How to Avoid:
	• Implement monitoring and alerting for MTU mismatches.
	• Validate network configurations before applying changes to the CNI plugin.

📘 Scenario #154: Application Traffic Routing Issue Due to Incorrect Ingress Resource
Category: Networking
Environment: K8s v1.22, Azure AKS
Scenario Summary: Application traffic was routed incorrectly due to an error in the ingress resource definition.
What Happened: Traffic intended for a specific application was routed to the wrong backend service because the ingress resource had a misconfigured path.
Diagnosis Steps:
	• Reviewed the ingress resource and found that the path definition did not match the expected URL.
	• Validated that the backend service was correctly exposed and running.
Root Cause: Incorrect path specification in the ingress resource, causing traffic to be routed incorrectly.
Fix/Workaround:
	• Corrected the path definition in the ingress resource.
	• Redeployed the ingress configuration to ensure correct traffic routing.
Lessons Learned: Always carefully review and test ingress path definitions before applying them in production.
How to Avoid:
	• Implement a staging environment to test ingress resources before production deployment.
	• Use automated tests to verify ingress configuration correctness.

📘 Scenario #155: Intermittent Service Disruptions Due to DNS Caching Issue
Category: Networking
Environment: K8s v1.21, GCP GKE
Scenario Summary: Intermittent service disruptions occurred due to stale DNS cache in CoreDNS.
What Happened: Services failed intermittently because CoreDNS had cached stale DNS records, causing them to resolve incorrectly.
Diagnosis Steps:
	• Verified DNS resolution using nslookup and found incorrect IP addresses being returned.
	• Cleared the DNS cache in CoreDNS and noticed that the issue was temporarily resolved.
Root Cause: CoreDNS was caching stale DNS records due to incorrect TTL settings.
Fix/Workaround:
	• Reduced the TTL value in CoreDNS configuration.
	• Restarted CoreDNS pods to apply the new TTL setting.
Lessons Learned: Be cautious of DNS TTL settings, especially in dynamic environments where IP addresses change frequently.
How to Avoid:
	• Monitor DNS records and TTL values actively.
	• Implement cache invalidation or reduce TTL for critical services.

📘 Scenario #156: Flannel Overlay Network Interruption Due to Node Failure
Category: Networking
Environment: K8s v1.18, on-premise
Scenario Summary: Flannel overlay network was interrupted after a node failure, causing pod-to-pod communication issues.
What Happened: A node failure caused the Flannel CNI plugin to lose its network routes, disrupting communication between pods on different nodes.
Diagnosis Steps:
	• Used kubectl get pods -o wide to identify affected pods.
	• Checked the Flannel daemon logs and found errors related to missing network routes.
Root Cause: Flannel CNI plugin was not re-establishing network routes after the node failure.
Fix/Workaround:
	• Restarted the Flannel pods on the affected nodes to re-establish network routes.
	• Verified that communication between pods was restored.
Lessons Learned: Ensure that CNI plugins can gracefully handle node failures and re-establish connectivity.
How to Avoid:
	• Implement automatic recovery or self-healing mechanisms for CNI plugins.
	• Monitor CNI plugin logs to detect issues early.

📘 Scenario #157: Network Traffic Loss Due to Port Collision in Network Policy
Category: Networking
Environment: K8s v1.19, GKE
Scenario Summary: Network traffic was lost due to a port collision in the network policy, affecting application availability.
What Happened: Network traffic was dropped because a network policy inadvertently blocked traffic to a port that was required by another application.
Diagnosis Steps:
	• Inspected the network policy using kubectl describe netpol and identified the port conflict.
	• Verified traffic flow using kubectl logs to identify blocked traffic.
Root Cause: Misconfigured network policy that blocked traffic to a necessary port due to port collision.
Fix/Workaround:
	• Updated the network policy to allow the necessary port.
	• Applied the updated network policy and tested the traffic flow.
Lessons Learned: Thoroughly test network policies to ensure that they do not block critical application traffic.
How to Avoid:
	• Review network policies in detail before applying them in production.
	• Use automated tools to validate network policies.

📘 Scenario #158: CoreDNS Service Failures Due to Resource Exhaustion
Category: Networking
Environment: K8s v1.20, Azure AKS
Scenario Summary: CoreDNS service failed due to resource exhaustion, causing DNS resolution failures.
What Happened: CoreDNS pods exhausted available CPU and memory, leading to service failures and DNS resolution issues.
Diagnosis Steps:
	• Checked CoreDNS logs and found out-of-memory errors.
	• Verified that the CPU usage was consistently high for the CoreDNS pods.
Root Cause: Insufficient resources allocated to CoreDNS pods, causing service crashes.
Fix/Workaround:
	• Increased the resource requests and limits for CoreDNS pods.
	• Restarted the CoreDNS pods to apply the updated resource allocation.
Lessons Learned: Ensure that critical components like CoreDNS have sufficient resources allocated for normal operation.
How to Avoid:
	• Set appropriate resource requests and limits based on usage patterns.
	• Monitor resource consumption of CoreDNS and other critical components.

📘 Scenario #159: Pod Network Partition Due to Misconfigured IPAM
Category: Networking
Environment: K8s v1.22, VMware Tanzu
Scenario Summary: Pod network partition occurred due to an incorrectly configured IP Address Management (IPAM) in the CNI plugin.
What Happened: Pods were unable to communicate across nodes because the IPAM configuration was improperly set, causing an address space overlap.
Diagnosis Steps:
	• Inspected the CNI configuration and discovered overlapping IP address ranges.
	• Verified network policies and found no conflicts, but the IP address allocation was incorrect.
Root Cause: Misconfiguration of IPAM settings in the CNI plugin.
Fix/Workaround:
	• Corrected the IPAM configuration to use non-overlapping IP address ranges.
	• Redeployed the CNI plugin and restarted affected pods.
Lessons Learned: Carefully configure IPAM in CNI plugins to prevent network address conflicts.
How to Avoid:
	• Validate network configurations before deploying.
	• Use automated checks to detect IP address conflicts in multi-node environments.

📘 Scenario #160: Network Performance Degradation Due to Overloaded CNI Plugin
Category: Networking
Environment: K8s v1.21, AWS EKS
Scenario Summary: Network performance degraded due to the CNI plugin being overwhelmed by high traffic volume.
What Happened: A sudden spike in traffic caused the CNI plugin to become overloaded, resulting in significant packet loss and network latency between pods.
Diagnosis Steps:
	• Monitored network traffic using kubectl top pods and observed unusually high traffic to and from a few specific pods.
	• Inspected CNI plugin logs and found errors related to resource exhaustion.
Root Cause: The CNI plugin lacked sufficient resources to handle the spike in traffic, leading to packet loss and network degradation.
Fix/Workaround:
	• Increased resource limits for the CNI plugin pods.
	• Used network policies to limit the traffic spikes to specific services.
Lessons Learned: Ensure that the CNI plugin is properly sized to handle peak traffic loads, and monitor its health regularly.
How to Avoid:
	• Set up traffic rate limiting to prevent sudden spikes from overwhelming the network.
	• Use resource limits and horizontal pod autoscaling for critical CNI components.


📘 Scenario #161: Network Performance Degradation Due to Overloaded CNI Plugin
Category: Networking
Environment: K8s v1.21, AWS EKS
Scenario Summary: Network performance degraded due to the CNI plugin being overwhelmed by high traffic volume.
What Happened: A sudden spike in traffic caused the CNI plugin to become overloaded, resulting in significant packet loss and network latency between pods.
Diagnosis Steps:
	• Monitored network traffic using kubectl top pods and observed unusually high traffic to and from a few specific pods.
	• Inspected CNI plugin logs and found errors related to resource exhaustion.
Root Cause: The CNI plugin lacked sufficient resources to handle the spike in traffic, leading to packet loss and network degradation.
Fix/Workaround:
	• Increased resource limits for the CNI plugin pods.
	• Used network policies to limit the traffic spikes to specific services.
Lessons Learned: Ensure that the CNI plugin is properly sized to handle peak traffic loads, and monitor its health regularly.
How to Avoid:
	• Set up traffic rate limiting to prevent sudden spikes from overwhelming the network.
	• Use resource limits and horizontal pod autoscaling for critical CNI components.

📘 Scenario #162: DNS Resolution Failures Due to Misconfigured CoreDNS
Category: Networking
Environment: K8s v1.19, Google GKE
Scenario Summary: DNS resolution failures due to misconfigured CoreDNS, leading to application errors.
What Happened: CoreDNS was misconfigured with the wrong upstream DNS resolver, causing DNS lookups to fail and leading to application connectivity issues.
Diagnosis Steps:
	• Ran kubectl logs -l k8s-app=coredns to view the CoreDNS logs and identified errors related to upstream DNS resolution.
	• Used kubectl get configmap coredns -n kube-system -o yaml to inspect the CoreDNS configuration.
Root Cause: CoreDNS was configured with an invalid upstream DNS server that was unreachable.
Fix/Workaround:
	• Updated CoreDNS ConfigMap to point to a valid upstream DNS server.
	• Restarted CoreDNS pods to apply the new configuration.
Lessons Learned: Double-check DNS configurations during deployment and monitor CoreDNS health regularly.
How to Avoid:
	• Automate the validation of DNS configurations and use reliable upstream DNS servers.
	• Set up monitoring for DNS resolution latency and errors.

📘 Scenario #163: Network Partition Due to Incorrect Calico Configuration
Category: Networking
Environment: K8s v1.20, Azure AKS
Scenario Summary: Network partitioning due to incorrect Calico CNI configuration, resulting in pods being unable to communicate with each other.
What Happened: Calico was misconfigured with an incorrect CIDR range, leading to network partitioning where some pods could not reach other pods in the same cluster.
Diagnosis Steps:
	• Verified pod connectivity using kubectl exec and confirmed network isolation between pods.
	• Inspected Calico configuration and discovered the incorrect CIDR range in the calicoctl configuration.
Root Cause: Incorrect CIDR range in the Calico configuration caused pod networking issues.
Fix/Workaround:
	• Updated the Calico CIDR range configuration to match the cluster's networking plan.
	• Restarted Calico pods to apply the new configuration and restore network connectivity.
Lessons Learned: Ensure that network configurations, especially for CNI plugins, are thoroughly tested before deployment.
How to Avoid:
	• Use automated network validation tools to check for partitioning and misconfigurations.
	• Regularly review and update CNI configuration as the cluster grows.

📘 Scenario #164: IP Overlap Leading to Communication Failure Between Pods
Category: Networking
Environment: K8s v1.19, On-premise
Scenario Summary: Pods failed to communicate due to IP address overlap caused by an incorrect subnet configuration.
What Happened: The pod network subnet overlapped with another network on the host machine, causing IP address conflicts and preventing communication between pods.
Diagnosis Steps:
	• Verified pod IPs using kubectl get pods -o wide and identified overlapping IPs with host network IPs.
	• Checked network configuration on the host and discovered the overlapping subnet.
Root Cause: Incorrect subnet configuration that caused overlapping IP ranges between the Kubernetes pod network and the host network.
Fix/Workaround:
	• Updated the pod network CIDR range to avoid overlapping with host network IPs.
	• Restarted the Kubernetes networking components to apply the new configuration.
Lessons Learned: Pay careful attention to subnet planning when setting up networking for Kubernetes clusters to avoid conflicts.
How to Avoid:
	• Use a tool to validate network subnets during cluster setup.
	• Avoid using overlapping IP ranges when planning pod and host network subnets.

📘 Scenario #165: Pod Network Latency Due to Overloaded Kubernetes Network Interface
Category: Networking
Environment: K8s v1.21, AWS EKS
Scenario Summary: Pod network latency increased due to an overloaded network interface on the Kubernetes nodes.
What Happened: A sudden increase in traffic caused the network interface on the nodes to become overloaded, leading to high network latency between pods and degraded application performance.
Diagnosis Steps:
	• Used kubectl top node to observe network interface metrics and saw high network throughput and packet drops.
	• Checked AWS CloudWatch metrics and confirmed that the network interface was approaching its maximum throughput.
Root Cause: The network interface on the nodes was unable to handle the high network traffic due to insufficient capacity.
Fix/Workaround:
	• Increased the network bandwidth for the AWS EC2 instances hosting the Kubernetes nodes.
	• Used network policies to limit traffic to critical pods and avoid overwhelming the network interface.
Lessons Learned: Ensure that Kubernetes nodes are provisioned with adequate network capacity for expected traffic loads.
How to Avoid:
	• Monitor network traffic and resource utilization at the node level.
	• Scale nodes appropriately or use higher-bandwidth instances for high-traffic workloads.

📘 Scenario #166: Intermittent Connectivity Failures Due to Pod DNS Cache Expiry
Category: Networking
Environment: K8s v1.22, Google GKE
Scenario Summary: Intermittent connectivity failures due to pod DNS cache expiry, leading to failed DNS lookups for external services.
What Happened: Pods experienced intermittent connectivity failures because the DNS cache expired too quickly, causing DNS lookups to fail for external services.
Diagnosis Steps:
	• Checked pod logs and observed errors related to DNS lookup failures.
	• Inspected the CoreDNS configuration and identified a low TTL (time-to-live) value for DNS cache.
Root Cause: The DNS TTL was set too low, causing DNS entries to expire before they could be reused.
Fix/Workaround:
	• Increased the DNS TTL value in the CoreDNS configuration.
	• Restarted CoreDNS pods to apply the new configuration.
Lessons Learned: Proper DNS caching settings are critical for maintaining stable connectivity to external services.
How to Avoid:
	• Set appropriate DNS TTL values based on the requirements of your services.
	• Regularly monitor DNS performance and adjust TTL settings as needed.

📘 Scenario #167: Flapping Network Connections Due to Misconfigured Network Policies
Category: Networking
Environment: K8s v1.20, Azure AKS
Scenario Summary: Network connections between pods were intermittently dropping due to misconfigured network policies, causing application instability.
What Happened: Network policies were incorrectly configured, leading to intermittent drops in network connectivity between pods, especially under load.
Diagnosis Steps:
	• Used kubectl describe networkpolicy to inspect network policies and found overly restrictive ingress rules.
	• Verified pod-to-pod communication using kubectl exec and confirmed that traffic was being blocked intermittently.
Root Cause: Misconfigured network policies that were too restrictive, blocking legitimate traffic between pods.
Fix/Workaround:
	• Updated the network policies to allow necessary pod-to-pod communication.
	• Tested connectivity to ensure stability after the update.
Lessons Learned: Ensure that network policies are tested thoroughly before being enforced, especially in production.
How to Avoid:
	• Use a staged approach for deploying network policies, first applying them to non-critical pods.
	• Implement automated tests to validate network policy configurations.

📘 Scenario #168: Cluster Network Downtime Due to CNI Plugin Upgrade
Category: Networking
Environment: K8s v1.22, On-premise
Scenario Summary: Cluster network downtime occurred during a CNI plugin upgrade, affecting pod-to-pod communication.
What Happened: During an upgrade to the CNI plugin, the network was temporarily disrupted due to incorrect version compatibility and missing network configurations.
Diagnosis Steps:
	• Inspected pod logs and noticed failed network interfaces after the upgrade.
	• Checked CNI plugin version compatibility and identified missing configurations for the new version.
Root Cause: The new version of the CNI plugin required additional configuration settings that were not applied during the upgrade.
Fix/Workaround:
	• Applied the required configuration changes for the new CNI plugin version.
	• Restarted affected pods and network components to restore connectivity.
Lessons Learned: Always verify compatibility and required configurations before upgrading the CNI plugin.
How to Avoid:
	• Test plugin upgrades in a staging environment to catch compatibility issues.
	• Follow a defined upgrade process that includes validation of configurations.


📘 Scenario #169: Inconsistent Pod Network Connectivity in Multi-Region Cluster
Category: Networking
Environment: K8s v1.21, GCP
Scenario Summary: Pods in a multi-region cluster experienced inconsistent network connectivity between regions due to misconfigured VPC peering.
What Happened: The VPC peering between two regions was misconfigured, leading to intermittent network connectivity issues between pods in different regions.
Diagnosis Steps:
	• Used kubectl exec to check network latency and packet loss between pods in different regions.
	• Inspected VPC peering settings and found that the correct routes were not configured to allow cross-region traffic.
Root Cause: Misconfigured VPC peering between the regions prevented proper routing of network traffic.
Fix/Workaround:
	• Updated VPC peering routes and ensured proper configuration between the regions.
	• Tested connectivity after the change to confirm resolution.
Lessons Learned: Ensure that all network routing and peering configurations are validated before deploying cross-region clusters.
How to Avoid:
	• Regularly review VPC and peering configurations.
	• Use automated network tests to confirm inter-region connectivity.

📘 Scenario #170: Pod Network Partition Due to Network Policy Blocking DNS Requests
Category: Networking
Environment: K8s v1.19, Azure AKS
Scenario Summary: Pods were unable to resolve DNS due to a network policy blocking DNS traffic, causing service failures.
What Happened: A network policy was accidentally configured to block DNS (UDP port 53) traffic between pods, preventing DNS resolution and causing services to fail.
Diagnosis Steps:
	• Observed that pods were unable to reach external services, and kubectl exec into the pods showed DNS resolution failures.
	• Used kubectl describe networkpolicy and found the DNS traffic was blocked in the policy.
Root Cause: The network policy accidentally blocked DNS traffic due to misconfigured ingress and egress rules.
Fix/Workaround:
	• Updated the network policy to allow DNS traffic.
	• Restarted affected pods to ensure they could access DNS again.
Lessons Learned: Always verify that network policies allow necessary traffic, especially for DNS.
How to Avoid:
	• Regularly test and validate network policies in non-production environments.
	• Set up monitoring for blocked network traffic.

📘 Scenario #171: Network Bottleneck Due to Overutilized Network Interface
Category: Networking
Environment: K8s v1.22, AWS EKS
Scenario Summary: Network bottleneck occurred due to overutilization of a single network interface on the worker nodes.
What Happened: The worker nodes were using a single network interface to handle both pod traffic and node communication. The high volume of pod traffic caused the network interface to become overutilized, resulting in slow communication.
Diagnosis Steps:
	• Checked the network interface metrics using AWS CloudWatch and found that the interface was nearing its throughput limit.
	• Used kubectl top node and observed high network usage on the affected nodes.
Root Cause: The network interface on the worker nodes was not properly partitioned to handle separate types of traffic, leading to resource contention.
Fix/Workaround:
	• Added a second network interface to the worker nodes for pod traffic and node-to-node communication.
	• Reconfigured the nodes to distribute traffic across the two interfaces.
Lessons Learned: Proper network interface design is crucial for handling high traffic loads and preventing bottlenecks.
How to Avoid:
	• Design network topologies that segregate different types of traffic (e.g., pod traffic, node communication).
	• Regularly monitor network utilization and scale resources as needed.

📘 Scenario #172: Network Latency Caused by Overloaded VPN Tunnel
Category: Networking
Environment: K8s v1.20, On-premise
Scenario Summary: Network latency increased due to an overloaded VPN tunnel between the Kubernetes cluster and an on-premise data center.
What Happened: The VPN tunnel between the Kubernetes cluster in the cloud and an on-premise data center became overloaded, causing increased latency for communication between services located in the two environments.
Diagnosis Steps:
	• Used kubectl exec to measure response times between pods and services in the on-premise data center.
	• Monitored VPN tunnel usage and found it was reaching its throughput limits during peak hours.
Root Cause: The VPN tunnel was not sized correctly to handle the required traffic between the cloud and on-premise environments.
Fix/Workaround:
	• Upgraded the VPN tunnel to a higher bandwidth option.
	• Optimized the data flow by reducing unnecessary traffic over the tunnel.
Lessons Learned: Ensure that hybrid network connections like VPNs are appropriately sized and optimized for traffic.
How to Avoid:
	• Test VPN tunnels with real traffic before moving to production.
	• Monitor tunnel utilization and upgrade bandwidth as needed.

📘 Scenario #173: Dropped Network Packets Due to MTU Mismatch
Category: Networking
Environment: K8s v1.21, GKE
Scenario Summary: Network packets were dropped due to a mismatch in Maximum Transmission Unit (MTU) settings across different network components.
What Happened: Pods experienced connectivity issues and packet loss because the MTU settings on the nodes and CNI plugin were inconsistent, causing packets to be fragmented and dropped.
Diagnosis Steps:
	• Used ping and tracepath tools to identify dropped packets and packet fragmentation.
	• Inspected the CNI plugin and node MTU configurations and found a mismatch.
Root Cause: Inconsistent MTU settings between the CNI plugin and the Kubernetes nodes caused packet fragmentation and loss.
Fix/Workaround:
	• Unified MTU settings across all nodes and the CNI plugin configuration.
	• Restarted the network components to apply the changes.
Lessons Learned: Ensure consistent MTU settings across the entire networking stack in Kubernetes clusters.
How to Avoid:
	• Automate MTU validation checks during cluster setup and upgrades.
	• Monitor network packet loss and fragmentation regularly.

📘 Scenario #174: Pod Network Isolation Due to Misconfigured Network Policy
Category: Networking
Environment: K8s v1.20, Azure AKS
Scenario Summary: Pods in a specific namespace were unable to communicate due to an incorrectly applied network policy blocking traffic between namespaces.
What Happened: A network policy was incorrectly configured to block communication between namespaces, leading to service failures and inability to reach certain pods.
Diagnosis Steps:
	• Used kubectl describe networkpolicy to inspect the policy and confirmed it was overly restrictive.
	• Tested pod-to-pod communication using kubectl exec and verified the isolation.
Root Cause: The network policy was too restrictive and blocked cross-namespace communication.
Fix/Workaround:
	• Updated the network policy to allow traffic between namespaces.
	• Restarted affected pods to re-establish communication.
Lessons Learned: Always test network policies in a staging environment to avoid unintentional isolation.
How to Avoid:
	• Use a staged approach to apply network policies and validate them before enforcing them in production.
	• Implement automated tests for network policy validation.

📘 Scenario #175: Service Discovery Failures Due to CoreDNS Pod Crash
Category: Networking
Environment: K8s v1.19, AWS EKS
Scenario Summary: Service discovery failures occurred when CoreDNS pods crashed due to resource exhaustion, causing DNS resolution issues.
What Happened: CoreDNS pods crashed due to high CPU utilization caused by excessive DNS queries, which prevented service discovery and caused communication failures.
Diagnosis Steps:
	• Checked pod logs and observed frequent crashes related to out-of-memory (OOM) errors.
	• Monitored CoreDNS resource utilization and confirmed CPU spikes from DNS queries.
Root Cause: Resource exhaustion in CoreDNS due to an overload of DNS queries.
Fix/Workaround:
	• Increased CPU and memory resources for CoreDNS pods.
	• Optimized the DNS query patterns from applications to reduce the load.
Lessons Learned: Ensure that DNS services like CoreDNS are properly resourced and monitored.
How to Avoid:
	• Set up monitoring for DNS query rates and resource utilization.
	• Scale CoreDNS horizontally to distribute the load.
```
