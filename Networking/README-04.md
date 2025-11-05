📘 Scenario #176: Pod DNS Resolution Failure Due to CoreDNS Configuration Issue
Category: Networking
Environment: K8s v1.18, On-premise
Scenario Summary: DNS resolution failures occurred within pods due to a misconfiguration in the CoreDNS config map.
What Happened: CoreDNS was misconfigured to not forward DNS queries to external DNS servers, causing pods to fail when resolving services outside the cluster.
Diagnosis Steps:
	• Ran kubectl exec in the affected pods and verified DNS resolution failure.
	• Inspected the CoreDNS ConfigMap and found that the forward section was missing the external DNS servers.
Root Cause: CoreDNS was not configured to forward external queries, leading to DNS resolution failure for non-cluster services.
Fix/Workaround:
	• Updated the CoreDNS ConfigMap to add the missing external DNS server configuration.
	• Restarted the CoreDNS pods to apply the changes.
Lessons Learned: Always review and test DNS configurations in CoreDNS, especially for hybrid clusters.
How to Avoid:
	• Use automated validation tools to check CoreDNS configuration.
	• Set up tests for DNS resolution to catch errors before they impact production.

📘 Scenario #177: DNS Latency Due to Overloaded CoreDNS Pods
Category: Networking
Environment: K8s v1.19, GKE
Scenario Summary: CoreDNS pods experienced high latency and timeouts due to resource overutilization, causing slow DNS resolution for applications.
What Happened: CoreDNS pods were handling a high volume of DNS requests without sufficient resources, leading to increased latency and timeouts.
Diagnosis Steps:
	• Used kubectl top pod to observe high CPU and memory usage on CoreDNS pods.
	• Checked the DNS query logs and saw long response times.
Root Cause: CoreDNS was under-resourced, and the high DNS traffic caused resource contention.
Fix/Workaround:
	• Increased CPU and memory limits for CoreDNS pods.
	• Enabled horizontal pod autoscaling to dynamically scale CoreDNS based on traffic.
Lessons Learned: Proper resource allocation and autoscaling are critical for maintaining DNS performance under load.
How to Avoid:
	• Set up resource limits and autoscaling for CoreDNS pods.
	• Monitor DNS traffic and resource usage regularly to prevent overloads.

📘 Scenario #178: Pod Network Degradation Due to Overlapping CIDR Blocks
Category: Networking
Environment: K8s v1.21, AWS EKS
Scenario Summary: Network degradation occurred due to overlapping CIDR blocks between VPCs in a hybrid cloud setup, causing routing issues.
What Happened: In a hybrid cloud setup, the CIDR blocks of the Kubernetes cluster VPC and the on-premise VPC overlapped, causing routing issues that led to network degradation and service disruptions.
Diagnosis Steps:
	• Investigated network routes using kubectl describe node and confirmed overlapping CIDR blocks.
	• Verified routing tables and identified conflicts causing packets to be misrouted.
Root Cause: Overlapping CIDR blocks between the cluster VPC and the on-premise VPC caused routing conflicts.
Fix/Workaround:
	• Reconfigured the CIDR blocks of one VPC to avoid overlap.
	• Adjusted the network routing tables to ensure traffic was correctly routed.
Lessons Learned: Ensure that CIDR blocks are carefully planned to avoid conflicts in hybrid cloud environments.
How to Avoid:
	• Plan CIDR blocks in advance to ensure they do not overlap.
	• Review and validate network configurations during the planning phase of hybrid cloud setups.

📘 Scenario #179: Service Discovery Failures Due to Network Policy Blocking DNS Traffic
Category: Networking
Environment: K8s v1.22, Azure AKS
Scenario Summary: Service discovery failed when a network policy was mistakenly applied to block DNS traffic, preventing pods from resolving services within the cluster.
What Happened: A network policy was applied to restrict traffic between namespaces but unintentionally blocked DNS traffic on UDP port 53, causing service discovery to fail.
Diagnosis Steps:
	• Ran kubectl get networkpolicy and found an ingress rule that blocked UDP traffic.
	• Used kubectl exec to test DNS resolution inside the affected pods, which confirmed that DNS queries were being blocked.
Root Cause: The network policy unintentionally blocked DNS traffic due to a misconfigured ingress rule.
Fix/Workaround:
	• Updated the network policy to allow DNS traffic on UDP port 53.
	• Restarted the affected pods to restore service discovery functionality.
Lessons Learned: Always carefully test network policies to ensure they don't inadvertently block critical traffic like DNS.
How to Avoid:
	• Review and test network policies thoroughly before applying them in production.
	• Implement automated tests to verify that critical services like DNS are not affected by policy changes.

📘 Scenario #180: Intermittent Network Connectivity Due to Overloaded Overlay Network
Category: Networking
Environment: K8s v1.19, OpenStack
Scenario Summary: Pods experienced intermittent network connectivity issues due to an overloaded overlay network that could not handle the traffic.
What Happened: An overlay network (Flannel) used to connect pods was overwhelmed due to high traffic volume, resulting in intermittent packet drops and network congestion.
Diagnosis Steps:
	• Used kubectl exec to trace packet loss between pods and detected intermittent connectivity.
	• Monitored network interfaces and observed high traffic volume and congestion on the overlay network.
Root Cause: The overlay network (Flannel) could not handle the traffic load due to insufficient resources allocated to the network component.
Fix/Workaround:
	• Reconfigured the overlay network to use a more scalable network plugin.
	• Increased resource allocation for the network components and scaled the infrastructure to handle the load.
Lessons Learned: Ensure that network plugins are properly configured and scaled to handle the expected traffic volume.
How to Avoid:
	• Monitor network traffic patterns and adjust resource allocation as needed.
	• Consider using more scalable network plugins for high-traffic workloads.

📘 Scenario #181: Pod-to-Pod Communication Failure Due to CNI Plugin Configuration Issue
Category: Networking
Environment: K8s v1.22, AWS EKS
Scenario Summary: Pods were unable to communicate with each other due to a misconfiguration in the CNI plugin.
What Happened: The Calico CNI plugin configuration was missing the necessary IP pool definitions, which caused pods to fail to obtain IPs from the defined pool, resulting in communication failure between pods.
Diagnosis Steps:
	• Ran kubectl describe pod to identify that the pods had no assigned IP addresses.
	• Inspected the CNI plugin logs and identified missing IP pool configurations.
Root Cause: The IP pool was not defined in the Calico CNI plugin configuration, causing pods to be unable to get network addresses.
Fix/Workaround:
	• Updated the Calico configuration to include the correct IP pool definitions.
	• Restarted the affected pods to obtain new IPs.
Lessons Learned: Always verify CNI plugin configuration, especially IP pool settings, before deploying a cluster.
How to Avoid:
	• Automate the verification of CNI configurations during cluster setup.
	• Test network functionality before scaling applications.

📘 Scenario #182: Sporadic DNS Failures Due to Resource Contention in CoreDNS Pods
Category: Networking
Environment: K8s v1.19, GKE
Scenario Summary: Sporadic DNS resolution failures occurred due to resource contention in CoreDNS pods, which were not allocated enough CPU resources.
What Happened: CoreDNS pods were experiencing sporadic failures due to high CPU utilization. DNS resolution intermittently failed during peak load times.
Diagnosis Steps:
	• Used kubectl top pod to monitor resource usage and found that CoreDNS pods were CPU-bound.
	• Monitored DNS query logs and found a correlation between high CPU usage and DNS resolution failures.
Root Cause: CoreDNS pods were not allocated sufficient CPU resources to handle the DNS query load during peak times.
Fix/Workaround:
	• Increased CPU resource requests and limits for CoreDNS pods.
	• Enabled horizontal pod autoscaling for CoreDNS to scale during high demand.
Lessons Learned: CoreDNS should be adequately resourced, and autoscaling should be enabled to handle varying DNS query loads.
How to Avoid:
	• Set proper resource requests and limits for CoreDNS.
	• Implement autoscaling for DNS services based on real-time load.

📘 Scenario #183: High Latency in Pod-to-Node Communication Due to Overlay Network
Category: Networking
Environment: K8s v1.21, OpenShift
Scenario Summary: High latency was observed in pod-to-node communication due to network overhead introduced by the overlay network.
What Happened: The cluster was using Flannel as the CNI plugin, and network latency increased as the overlay network was unable to efficiently handle the traffic between pods and nodes.
Diagnosis Steps:
	• Used kubectl exec to measure network latency between pods and nodes.
	• Analyzed the network traffic and identified high latency due to the overlay network's encapsulation.
Root Cause: The Flannel overlay network introduced additional overhead, which caused latency in pod-to-node communication.
Fix/Workaround:
	• Switched to a different CNI plugin (Calico) that offered better performance for the network topology.
	• Retested pod-to-node communication after switching CNI plugins.
Lessons Learned: Choose the right CNI plugin based on network performance needs, especially in high-throughput environments.
How to Avoid:
	• Perform a performance evaluation of different CNI plugins during cluster planning.
	• Monitor network performance regularly and switch plugins if necessary.

📘 Scenario #184: Service Discovery Issues Due to DNS Cache Staleness
Category: Networking
Environment: K8s v1.20, On-premise
Scenario Summary: Service discovery failed due to stale DNS cache entries that were not updated when services changed IPs.
What Happened: The DNS resolver cached the old IP addresses for services, causing service discovery failures when the IPs of the services changed.
Diagnosis Steps:
	• Used kubectl exec to verify DNS cache entries.
	• Observed that the cached IPs were outdated and did not reflect the current service IPs.
Root Cause: The DNS cache was not being properly refreshed, causing stale DNS entries.
Fix/Workaround:
	• Cleared the DNS cache manually and implemented shorter TTL (Time-To-Live) values for DNS records.
	• Restarted CoreDNS pods to apply changes.
Lessons Learned: Ensure that DNS TTL values are appropriately set to avoid stale cache issues.
How to Avoid:
	• Regularly monitor DNS cache and refresh TTL values to ensure up-to-date resolution.
	• Implement a caching strategy that works well with Kubernetes service discovery.

📘 Scenario #185: Network Partition Between Node Pools in Multi-Zone Cluster
Category: Networking
Environment: K8s v1.18, GKE
Scenario Summary: Pods in different node pools located in different zones experienced network partitioning due to a misconfigured regional load balancer.
What Happened: The regional load balancer was not properly configured to handle traffic between node pools located in different zones, causing network partitioning between pods in different zones.
Diagnosis Steps:
	• Used kubectl exec to verify pod-to-pod communication between node pools and found packet loss.
	• Inspected the load balancer configuration and found that cross-zone traffic was not properly routed.
Root Cause: The regional load balancer was misconfigured, blocking traffic between nodes in different zones.
Fix/Workaround:
	• Updated the regional load balancer configuration to properly route cross-zone traffic.
	• Re-deployed the affected pods to restore connectivity.
Lessons Learned: Ensure proper configuration of load balancers to support multi-zone communication in cloud environments.
How to Avoid:
	• Test multi-zone communication setups thoroughly before going into production.
	• Automate the validation of load balancer configurations.

📘 Scenario #186: Pod Network Isolation Failure Due to Missing NetworkPolicy
Category: Networking
Environment: K8s v1.21, AKS
Scenario Summary: Pods that were intended to be isolated from each other could communicate freely due to a missing NetworkPolicy.
What Happened: The project had requirements for strict pod isolation, but the necessary NetworkPolicy was not created, resulting in unexpected communication between pods that should not have had network access to each other.
Diagnosis Steps:
	• Inspected kubectl get networkpolicy and found no policies defined for pod isolation.
	• Verified pod-to-pod communication and observed that pods in different namespaces could communicate without restriction.
Root Cause: Absence of a NetworkPolicy meant that all pods had default access to one another.
Fix/Workaround:
	• Created appropriate NetworkPolicy to restrict pod communication based on the namespace and labels.
	• Applied the NetworkPolicy and tested communication to ensure isolation was working.
Lessons Learned: Always implement and test network policies when security and isolation are a concern.
How to Avoid:
	• Implement strict NetworkPolicy from the outset when dealing with sensitive workloads.
	• Automate the validation of network policies during CI/CD pipeline deployment.

📘 Scenario #187: Flapping Node Network Connectivity Due to MTU Mismatch
Category: Networking
Environment: K8s v1.20, On-Premise
Scenario Summary: Nodes in the cluster were flapping due to mismatched MTU settings between Kubernetes and the underlying physical network, causing intermittent network connectivity issues.
What Happened: The physical network’s MTU was configured differently from the MTU settings in the Kubernetes CNI plugin, causing packet fragmentation. As a result, node-to-node communication was sporadic.
Diagnosis Steps:
	• Used kubectl describe node and checked the node’s network configuration.
	• Verified the MTU settings in the physical network and compared them to the Kubernetes settings, which were mismatched.
Root Cause: The mismatch in MTU settings caused fragmentation, resulting in unreliable connectivity between nodes.
Fix/Workaround:
	• Updated the Kubernetes network plugin's MTU setting to match the physical network MTU.
	• Restarted the affected nodes and validated the network stability.
Lessons Learned: Ensure that the MTU setting in the CNI plugin matches the physical network's MTU to avoid connectivity issues.
How to Avoid:
	• Always verify the MTU settings in both the physical network and the CNI plugin during cluster setup.
	• Include network performance testing in your cluster validation procedures.

📘 Scenario #188: DNS Query Timeout Due to Unoptimized CoreDNS Config
Category: Networking
Environment: K8s v1.18, GKE
Scenario Summary: DNS queries were timing out in the cluster, causing delays in service discovery, due to unoptimized CoreDNS configuration.
What Happened: The CoreDNS configuration was not optimized for the cluster size, resulting in DNS query timeouts under high load.
Diagnosis Steps:
	• Checked CoreDNS logs and saw frequent query timeouts.
	• Used kubectl describe pod on CoreDNS pods and found that they were under-resourced, leading to DNS query delays.
Root Cause: CoreDNS was misconfigured and lacked adequate CPU and memory resources to handle the query load.
Fix/Workaround:
	• Increased CPU and memory requests/limits for CoreDNS.
	• Optimized the CoreDNS configuration to use a more efficient query handling strategy.
Lessons Learned: CoreDNS needs to be properly resourced and optimized for performance, especially in large clusters.
How to Avoid:
	• Regularly monitor DNS performance and adjust CoreDNS resource allocations.
	• Fine-tune the CoreDNS configuration to improve query handling efficiency.

📘 Scenario #189: Traffic Splitting Failure Due to Incorrect Service LoadBalancer Configuration
Category: Networking
Environment: K8s v1.22, AWS EKS
Scenario Summary: Traffic splitting between two microservices failed due to a misconfiguration in the Service LoadBalancer.
What Happened: The load balancing rules were incorrectly set up for the service, which caused requests to only route to one instance of a microservice, despite the intention to split traffic between two.
Diagnosis Steps:
	• Used kubectl describe svc to inspect the Service configuration and discovered incorrect annotations for traffic splitting.
	• Analyzed AWS load balancer logs and saw that traffic was directed to only one pod.
Root Cause: Misconfigured traffic splitting annotations in the Service definition prevented the load balancer from distributing traffic correctly.
Fix/Workaround:
	• Corrected the annotations in the Service definition to enable proper traffic splitting.
	• Redeployed the Service and tested that traffic was split as expected.
Lessons Learned: Always double-check load balancer and service annotations when implementing traffic splitting in a microservices environment.
How to Avoid:
	• Test traffic splitting configurations in a staging environment before applying them in production.
	• Automate the verification of load balancer and service configurations.

📘 Scenario #190: Network Latency Between Pods in Different Regions
Category: Networking
Environment: K8s v1.19, Azure AKS
Scenario Summary: Pods in different Azure regions experienced high network latency, affecting application performance.
What Happened: The Kubernetes cluster spanned multiple Azure regions, but the inter-region networking was not optimized, resulting in significant network latency between pods in different regions.
Diagnosis Steps:
	• Used kubectl exec to measure ping times between pods in different regions and observed high latency.
	• Inspected Azure network settings and found that there were no specific optimizations in place for inter-region traffic.
Root Cause: Lack of inter-region network optimization and reliance on default settings led to high latency between regions.
Fix/Workaround:
	• Configured Azure Virtual Network peering with appropriate bandwidth settings.
	• Enabled specific network optimizations for inter-region communication.
Lessons Learned: When deploying clusters across multiple regions, network latency should be carefully managed and optimized.
How to Avoid:
	• Use region-specific optimizations and peering when deploying multi-region clusters.
	• Test the network performance before and after cross-region deployments to ensure acceptable latency.


📘 Scenario #191: Port Collision Between Services Due to Missing Port Ranges
Category: Networking
Environment: K8s v1.21, AKS
Scenario Summary: Two services attempted to bind to the same port, causing a port collision and service failures.
What Happened: The services were configured without specifying unique port ranges, and both attempted to use the same port on the same node, leading to port binding issues.
Diagnosis Steps:
	• Used kubectl get svc to check the services' port configurations and found that both services were trying to bind to the same port.
	• Verified node logs and observed port binding errors.
Root Cause: Missing port range configurations in the service definitions led to port collision.
Fix/Workaround:
	• Updated the service definitions to specify unique ports or port ranges.
	• Redeployed the services to resolve the conflict.
Lessons Learned: Always ensure that services use unique port configurations to avoid conflicts.
How to Avoid:
	• Define port ranges explicitly in service configurations.
	• Use tools like kubectl to validate port allocations before deploying services.

📘 Scenario #192: Pod-to-External Service Connectivity Failures Due to Egress Network Policy
Category: Networking
Environment: K8s v1.20, AWS EKS
Scenario Summary: Pods failed to connect to an external service due to an overly restrictive egress network policy.
What Happened: An egress network policy was too restrictive and blocked traffic from the pods to external services, leading to connectivity issues.
Diagnosis Steps:
	• Used kubectl describe networkpolicy to inspect egress rules and found that the policy was blocking all outbound traffic.
	• Verified connectivity to the external service and confirmed the network policy was the cause.
Root Cause: An overly restrictive egress network policy prevented pods from accessing external services.
Fix/Workaround:
	• Modified the egress network policy to allow traffic to the required external service.
	• Applied the updated policy and tested connectivity.
Lessons Learned: Be mindful when applying network policies, especially egress rules that affect external connectivity.
How to Avoid:
	• Test network policies in a staging environment before applying them in production.
	• Implement gradual rollouts for network policies to avoid wide-scale disruptions.

📘 Scenario #193: Pod Connectivity Loss After Network Plugin Upgrade
Category: Networking
Environment: K8s v1.18, GKE
Scenario Summary: Pods lost connectivity after an upgrade of the Calico network plugin due to misconfigured IP pool settings.
What Happened: After upgrading the Calico CNI plugin, the IP pool configuration was not correctly migrated, which caused pods to lose connectivity to other pods and services.
Diagnosis Steps:
	• Checked kubectl describe pod and found that the pods were not assigned IPs.
	• Inspected Calico configuration and discovered that the IP pool settings were not properly carried over during the upgrade.
Root Cause: The upgrade process failed to migrate the IP pool configuration, leading to network connectivity issues for the pods.
Fix/Workaround:
	• Manually updated the Calico configuration to restore the correct IP pool settings.
	• Restarted the Calico pods and verified pod connectivity.
Lessons Learned: Ensure network plugin upgrades are carefully tested and configurations are validated after upgrades.
How to Avoid:
	• Perform network plugin upgrades in a staging environment before applying to production.
	• Use configuration management tools to keep track of network plugin settings.

📘 Scenario #194: External DNS Not Resolving After Cluster Network Changes
Category: Networking
Environment: K8s v1.19, DigitalOcean
Scenario Summary: External DNS resolution stopped working after changes were made to the cluster network configuration.
What Happened: After modifying the CNI configuration and reconfiguring IP ranges, external DNS resolution failed for services outside the cluster.
Diagnosis Steps:
	• Checked DNS resolution inside the cluster using kubectl exec and found that internal DNS queries were working, but external queries were failing.
	• Verified DNS resolver configuration and noticed that the external DNS forwarders were misconfigured after network changes.
Root Cause: The external DNS forwarder settings were not correctly updated after network changes.
Fix/Workaround:
	• Updated CoreDNS configuration to correctly forward DNS queries to external DNS servers.
	• Restarted CoreDNS pods to apply changes.
Lessons Learned: Network configuration changes can impact DNS settings, and these should be verified post-change.
How to Avoid:
	• Implement automated DNS validation tests to ensure external DNS resolution works after network changes.
	• Document and verify DNS configurations before and after network changes.

📘 Scenario #195: Slow Pod Communication Due to Misconfigured MTU in Network Plugin
Category: Networking
Environment: K8s v1.22, On-premise
Scenario Summary: Pod-to-pod communication was slow due to an incorrect MTU setting in the network plugin.
What Happened: The network plugin was configured with an MTU that did not match the underlying network's MTU, leading to packet fragmentation and slower communication between pods.
Diagnosis Steps:
	• Used ping to check latency between pods and observed unusually high latency.
	• Inspected the network plugin’s MTU configuration and compared it with the host’s MTU, discovering a mismatch.
Root Cause: The MTU setting in the network plugin was too high, causing packet fragmentation and slow communication.
Fix/Workaround:
	• Corrected the MTU setting in the network plugin to match the host’s MTU.
	• Restarted the affected pods to apply the changes.
Lessons Learned: Ensure that MTU settings are aligned between the network plugin and the underlying network infrastructure.
How to Avoid:
	• Review and validate MTU settings when configuring network plugins.
	• Use monitoring tools to detect network performance issues like fragmentation.

📘 Scenario #196: High CPU Usage in Nodes Due to Overloaded Network Plugin
Category: Networking
Environment: K8s v1.22, AWS EKS
Scenario Summary: Nodes experienced high CPU usage due to an overloaded network plugin that couldn’t handle traffic spikes effectively.
What Happened: The network plugin was designed to handle a certain volume of traffic, but when the pod-to-pod communication increased, the plugin was unable to scale efficiently, leading to high CPU consumption.
Diagnosis Steps:
	• Monitored node metrics with kubectl top nodes and noticed unusually high CPU usage on affected nodes.
	• Checked logs for the network plugin and found evidence of resource exhaustion under high traffic conditions.
Root Cause: The network plugin was not adequately resourced to handle high traffic spikes, leading to resource exhaustion.
Fix/Workaround:
	• Increased resource allocation (CPU/memory) for the network plugin.
	• Configured scaling policies for the network plugin to dynamically adjust resources.
Lessons Learned: Network plugins need to be able to scale in response to increased traffic to prevent performance degradation.
How to Avoid:
	• Regularly monitor network plugin performance and resources.
	• Configure auto-scaling and adjust resource allocation based on traffic patterns.

📘 Scenario #197: Cross-Namespace Network Isolation Not Enforced
Category: Networking
Environment: K8s v1.19, OpenShift
Scenario Summary: Network isolation between namespaces failed due to an incorrectly applied NetworkPolicy.
What Happened: The NetworkPolicy intended to isolate communication between namespaces was not enforced because it was misconfigured.
Diagnosis Steps:
	• Checked the NetworkPolicy with kubectl describe networkpolicy and found that the selector was too broad, allowing communication across namespaces.
	• Verified namespace communication and found that pods in different namespaces could still communicate freely.
Root Cause: The NetworkPolicy selectors were too broad, and isolation was not enforced between namespaces.
Fix/Workaround:
	• Refined the NetworkPolicy to more specifically target pods within certain namespaces.
	• Re-applied the updated NetworkPolicy and validated the isolation.
Lessons Learned: Ensure that NetworkPolicy selectors are specific to prevent unintended communication.
How to Avoid:
	• Always validate network policies before deploying to production.
	• Use namespace-specific selectors to enforce isolation when necessary.

📘 Scenario #198: Inconsistent Service Discovery Due to CoreDNS Misconfiguration
Category: Networking
Environment: K8s v1.20, GKE
Scenario Summary: Service discovery was inconsistent due to misconfigured CoreDNS settings.
What Happened: The CoreDNS configuration was updated to use an external resolver, but the external resolver had intermittent issues, leading to service discovery failures.
Diagnosis Steps:
	• Checked CoreDNS logs with kubectl logs -n kube-system <coredns-pod> and noticed errors with the external resolver.
	• Used kubectl get svc to check service names and found that some services could not be resolved reliably.
Root Cause: Misconfigured external DNS resolver in CoreDNS caused service discovery failures.
Fix/Workaround:
	• Reverted CoreDNS configuration to use the internal DNS resolver instead of the external one.
	• Restarted CoreDNS pods to apply the changes.
Lessons Learned: External DNS resolvers can introduce reliability issues; test these changes carefully.
How to Avoid:
	• Use internal DNS resolvers for core service discovery within the cluster.
	• Implement monitoring for DNS resolution health.

📘 Scenario #199: Network Segmentation Issues Due to Misconfigured CNI
Category: Networking
Environment: K8s v1.18, IBM Cloud
Scenario Summary: Network segmentation between clusters failed due to incorrect CNI (Container Network Interface) plugin configuration.
What Happened: The CNI plugin was incorrectly configured, allowing pods from different network segments to communicate, violating security requirements.
Diagnosis Steps:
	• Inspected kubectl describe node and found that nodes were assigned to multiple network segments.
	• Used network monitoring tools to verify that pods in different segments were able to communicate.
Root Cause: The CNI plugin was not correctly segmented between networks, allowing unauthorized communication.
Fix/Workaround:
	• Reconfigured the CNI plugin to enforce correct network segmentation.
	• Applied the changes and tested communication between pods from different segments.
Lessons Learned: Network segmentation configurations should be thoroughly reviewed to prevent unauthorized communication.
How to Avoid:
	• Implement strong isolation policies in the network plugin.
	• Regularly audit network configurations and validate segmentation between clusters.

📘 Scenario #200: DNS Cache Poisoning in CoreDNS
Category: Networking
Environment: K8s v1.23, DigitalOcean
Scenario Summary: DNS cache poisoning occurred in CoreDNS, leading to incorrect IP resolution for services.
What Happened: A malicious actor compromised a DNS record by injecting a false IP address into the CoreDNS cache, causing services to resolve to an incorrect IP.
Diagnosis Steps:
	• Monitored CoreDNS logs and identified suspicious query patterns.
	• Used kubectl exec to inspect the DNS cache and found that some services had incorrect IP addresses cached.
Root Cause: CoreDNS cache was not sufficiently secured, allowing for DNS cache poisoning.
Fix/Workaround:
	• Implemented DNS query validation and hardened CoreDNS security by limiting cache lifetime and introducing DNSSEC.
	• Cleared the DNS cache and restarted CoreDNS to remove the poisoned entries.
Lessons Learned: Securing DNS caching is critical to prevent cache poisoning attacks.
How to Avoid:
	• Use DNSSEC or other DNS security mechanisms to validate responses.
	• Regularly monitor and audit CoreDNS logs for anomalies.
