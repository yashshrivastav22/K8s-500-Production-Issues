```
📘 Scenario #401: HPA Didn't Scale Due to Missing Metrics Server
Category: Scaling & Load
Environment: Kubernetes v1.22, Minikube
Summary: Horizontal Pod Autoscaler (HPA) didn’t scale pods as expected.
What Happened: HPA showed unknown metrics and pod count remained constant despite CPU stress.
Diagnosis Steps:
	• kubectl get hpa showed Metrics not available.
	• Confirmed metrics-server not installed.
Root Cause: Metrics server was missing, which is required by HPA for decision making.
Fix/Workaround:
	• Installed metrics-server using official manifests.
Lessons Learned: HPA silently fails without metrics-server.
How to Avoid:
	• Include metrics-server in base cluster setup.
	• Monitor HPA status regularly.

📘 Scenario #402: CPU Throttling Prevented Effective Autoscaling
Category: Scaling & Load
Environment: Kubernetes v1.24, EKS, Burstable QoS
Summary: Application CPU throttled even under low usage, leading to delayed scaling.
What Happened: HPA didn’t trigger scale-up due to misleading low CPU usage stats.
Diagnosis Steps:
	• Metrics showed low CPU, but app performance was poor.
	• kubectl top pod confirmed low utilization.
	• cgroups showed heavy throttling.
Root Cause: CPU limits were set too close to requests, causing throttling.
Fix/Workaround:
	• Increased CPU limits or removed them entirely for key services.
Lessons Learned: CPU throttling can suppress scaling metrics.
How to Avoid:
	• Monitor cgroup throttling stats.
	• Tune CPU requests/limits carefully.

📘 Scenario #403: Overprovisioned Pods Starved the Cluster
Category: Scaling & Load
Environment: Kubernetes v1.21, on-prem
Summary: Aggressively overprovisioned pod resources led to failed scheduling and throttling.
What Happened: Apps were deployed with excessive CPU/memory, blocking HPA and new workloads.
Diagnosis Steps:
	• kubectl describe node: Insufficient CPU errors.
	• Top nodes showed 50% actual usage, 100% requested.
Root Cause: Reserved resources were never used but blocked the scheduler.
Fix/Workaround:
	• Adjusted requests/limits based on real usage.
Lessons Learned: Resource requests ≠ real consumption.
How to Avoid:
	• Right-size pods using VPA recommendations or Prometheus usage data.

📘 Scenario #404: HPA and VPA Conflicted, Causing Flapping
Category: Scaling & Load
Environment: Kubernetes v1.25, GKE
Summary: HPA scaled replicas based on CPU while VPA changed pod resources dynamically, creating instability.
What Happened: HPA scaled up, VPA shrank resources → load spike → HPA scaled again.
Diagnosis Steps:
	• Logs showed frequent pod terminations and creations.
	• Pod count flapped repeatedly.
Root Cause: HPA and VPA were configured on the same deployment without proper coordination.
Fix/Workaround:
	• Disabled VPA on workloads using HPA.
Lessons Learned: HPA and VPA should be used carefully together.
How to Avoid:
	• Use HPA for scale-out and VPA for fixed-size workloads.
	• Avoid combining on the same object.

📘 Scenario #405: Cluster Autoscaler Didn't Scale Due to Pod Affinity Rules
Category: Scaling & Load
Environment: Kubernetes v1.23, AWS EKS
Summary: Workloads couldn't be scheduled and CA didn’t scale nodes because affinity rules restricted placement.
What Happened: Pods failed to schedule and were stuck in Pending, but no scale-out occurred.
Diagnosis Steps:
	• Events: FailedScheduling with affinity violations.
	• CA logs: “no matching node group”.
Root Cause: Pod anti-affinity restricted nodes that CA could provision.
Fix/Workaround:
	• Relaxed anti-affinity or labeled node groups appropriately.
Lessons Learned: Affinity rules affect autoscaler decisions.
How to Avoid:
	• Use soft affinity (preferredDuringScheduling) where possible.
	• Monitor unschedulable pods with alerting.

📘 Scenario #406: Load Test Crashed Cluster Due to Insufficient Node Quotas
Category: Scaling & Load
Environment: Kubernetes v1.24, AKS
Summary: Stress test resulted in API server crash due to unthrottled pod burst.
What Happened: Locust load test created hundreds of pods, exceeding node count limits.
Diagnosis Steps:
	• API server latency spiked, etcd logs flooded.
	• Cluster hit node quota limit on Azure.
Root Cause: No upper limit on replica count during load test; hit cloud provider limits.
Fix/Workaround:
	• Added maxReplicas to HPA.
	• Throttled CI tests.
Lessons Learned: CI/CD and load tests should obey cluster quotas.
How to Avoid:
	• Monitor node count vs quota in metrics.
	• Set maxReplicas in HPA and cap CI workloads.

📘 Scenario #407: Scale-To-Zero Caused Cold Starts and SLA Violations
Category: Scaling & Load
Environment: Kubernetes v1.25, KEDA + Knative
Summary: Pods scaled to zero, but requests during cold start breached SLA.
What Happened: First request after inactivity hit cold-start delay of ~15s.
Diagnosis Steps:
	• Prometheus response latency showed spikes after idle periods.
	• Knative logs: cold-start events.
Root Cause: Cold starts on scale-from-zero under high latency constraint.
Fix/Workaround:
	• Added minReplicaCount: 1 to high-SLA services.
Lessons Learned: Scale-to-zero saves cost, but not for latency-sensitive apps.
How to Avoid:
	• Use minReplicaCount and warmers for performance-critical services.

📘 Scenario #408: Misconfigured Readiness Probe Blocked HPA Scaling
Category: Scaling & Load
Environment: Kubernetes v1.24, DigitalOcean
Summary: HPA didn’t scale pods because readiness probes failed and metrics were not reported.
What Happened: Misconfigured probe returned 404, making pods invisible to HPA.
Diagnosis Steps:
	• kubectl describe pod: readiness failed.
	• kubectl get hpa: no metrics available.
Root Cause: Failed readiness probes excluded pods from metrics aggregation.
Fix/Workaround:
	• Corrected readiness endpoint in manifest.
Lessons Learned: HPA only sees "ready" pods.
How to Avoid:
	• Validate probe paths before production.
	• Monitor readiness failures via alerts.

📘 Scenario #409: Custom Metrics Adapter Crashed, Breaking Custom HPA
Category: Scaling & Load
Environment: Kubernetes v1.25, Prometheus Adapter
Summary: Custom HPA didn’t function after metrics adapter pod crashed silently.
What Happened: HPA relying on Prometheus metrics didn't scale for hours.
Diagnosis Steps:
	• kubectl get hpa: metric unavailable.
	• Checked prometheus-adapter logs: crashloop backoff.
Root Cause: Misconfigured rules in adapter config caused panic.
Fix/Workaround:
	• Fixed Prometheus query in adapter configmap.
Lessons Learned: Custom HPA is fragile to adapter errors.
How to Avoid:
	• Set alerts on prometheus-adapter health.
	• Validate custom queries before deploy.

📘 Scenario #410: Application Didn’t Handle Scale-In Gracefully
Category: Scaling & Load
Environment: Kubernetes v1.22, Azure AKS
Summary: App lost in-flight requests during scale-down, causing 5xx spikes.
What Happened: Pods were terminated abruptly during autoscaling down, mid-request.
Diagnosis Steps:
	• Observed 502/504 errors in logs during scale-in events.
	• No termination hooks present.
Root Cause: No preStop hooks or graceful shutdown handling in the app.
Fix/Workaround:
	• Implemented preStop hook with delay.
	• Added graceful shutdown in app logic.
Lessons Learned: Scale-in should be as graceful as scale-out.
How to Avoid:
	• Always include termination handling in apps.
	• Use terminationGracePeriodSeconds wisely.


📘 Scenario #411: Cluster Autoscaler Ignored Pod PriorityClasses
Category: Scaling & Load
Environment: Kubernetes v1.25, AWS EKS with PriorityClasses
Summary: Low-priority workloads blocked scaling of high-priority ones due to misconfigured Cluster Autoscaler.
What Happened: High-priority pods remained pending, even though Cluster Autoscaler was active.
Diagnosis Steps:
	• kubectl get pods --all-namespaces | grep Pending showed stuck critical workloads.
	• CA logs indicated scale-up denied due to resource reservation by lower-priority pods.
Root Cause: Default CA config didn't preempt lower-priority pods.
Fix/Workaround:
	• Enabled preemption.
	• Re-tuned PriorityClass definitions to align with business SLAs.
Lessons Learned: CA doesn’t preempt unless explicitly configured.
How to Avoid:
	• Validate PriorityClass behavior in test environments.
	• Use preemptionPolicy: PreemptLowerPriority for critical workloads.

📘 Scenario #412: ReplicaSet Misalignment Led to Excessive Scale-Out
Category: Scaling & Load
Environment: Kubernetes v1.23, GKE
Summary: A stale ReplicaSet with label mismatches caused duplicate pod scale-out.
What Happened: Deployment scaled twice the required pod count after an upgrade.
Diagnosis Steps:
	• kubectl get replicasets showed multiple active sets with overlapping match labels.
	• Pod count exceeded expected limits.
Root Cause: A new deployment overlapped labels with an old one; HPA acted on both.
Fix/Workaround:
	• Cleaned up old ReplicaSets.
	• Scoped matchLabels more tightly.
Lessons Learned: Label discipline is essential for reliable scaling.
How to Avoid:
	• Use distinct labels per version or release.
	• Automate cleanup of unused ReplicaSets.

📘 Scenario #413: StatefulSet Didn't Scale Due to PodDisruptionBudget
Category: Scaling & Load
Environment: Kubernetes v1.26, AKS
Summary: StatefulSet couldn’t scale-in during node pressure due to a restrictive PDB.
What Happened: Nodes under memory pressure tried to evict pods, but eviction was blocked.
Diagnosis Steps:
	• Checked kubectl describe pdb and kubectl get evictions.
	• Events showed "Cannot evict pod as it would violate PDB".
Root Cause: PDB defined minAvailable: 100%, preventing any disruption.
Fix/Workaround:
	• Adjusted PDB to tolerate one pod disruption.
Lessons Learned: Aggressive PDBs block both scaling and upgrades.
How to Avoid:
	• Use realistic minAvailable or maxUnavailable settings.
	• Review PDB behavior in test scaling operations.

📘 Scenario #414: Horizontal Pod Autoscaler Triggered by Wrong Metric
Category: Scaling & Load
Environment: Kubernetes v1.24, DigitalOcean
Summary: HPA used memory instead of CPU, causing unnecessary scale-ups.
What Happened: Application scaled even under light CPU usage due to memory caching behavior.
Diagnosis Steps:
	• HPA target: memory utilization.
	• kubectl top pods: memory always high due to in-memory cache.
Root Cause: Application design led to consistently high memory usage.
Fix/Workaround:
	• Switched HPA to CPU metric.
	• Tuned caching logic in application.
Lessons Learned: Choose scaling metrics that reflect true load.
How to Avoid:
	• Profile application behavior before configuring HPA.
	• Avoid memory-based autoscaling unless necessary.

📘 Scenario #415: Prometheus Scraper Bottlenecked Custom HPA Metrics
Category: Scaling & Load
Environment: Kubernetes v1.25, custom metrics + Prometheus Adapter
Summary: Delays in Prometheus scraping caused lag in HPA reactions.
What Happened: HPA lagged 1–2 minutes behind actual load spike.
Diagnosis Steps:
	• prometheus-adapter logs showed stale data timestamps.
	• HPA scale-up occurred after delay.
Root Cause: Scrape interval was 60s, making HPA respond too slowly.
Fix/Workaround:
	• Reduced scrape interval for critical metrics.
Lessons Learned: Scrape intervals affect autoscaler agility.
How to Avoid:
	• Match Prometheus scrape intervals with HPA polling needs.
	• Use rate() or avg_over_time() to smooth metrics.

📘 Scenario #416: Kubernetes Downscaled During Rolling Update
Category: Scaling & Load
Environment: Kubernetes v1.23, on-prem
Summary: Pods were prematurely scaled down during rolling deployment.
What Happened: Rolling update caused a drop in available replicas, triggering autoscaler.
Diagnosis Steps:
	• Observed spike in 5xx errors during update.
	• HPA decreased replica count despite live traffic.
Root Cause: Deployment strategy interfered with autoscaling logic.
Fix/Workaround:
	• Tuned maxUnavailable and minReadySeconds.
	• Added load-based HPA stabilization window.
Lessons Learned: HPA must be aligned with rolling deployment behavior.
How to Avoid:
	• Use behavior.scaleDown.stabilizationWindowSeconds.
	• Monitor scaling decisions during rollouts.

📘 Scenario #417: KEDA Failed to Scale on Kafka Lag Metric
Category: Scaling & Load
Environment: Kubernetes v1.26, KEDA + Kafka
Summary: Consumers didn’t scale out despite Kafka topic lag.
What Happened: High message lag persisted but consumer replicas remained at baseline.
Diagnosis Steps:
	• kubectl get scaledobject showed no trigger activation.
	• Logs: authentication to Kafka metrics endpoint failed.
Root Cause: Incorrect TLS cert in KEDA trigger config.
Fix/Workaround:
	• Updated Kafka trigger auth to use correct secret.
Lessons Learned: External metric sources require secure, stable access.
How to Avoid:
	• Validate all trigger auth and endpoints before production.
	• Alert on trigger activation failures.

📘 Scenario #418: Spike in Load Exceeded Pod Init Time
Category: Scaling & Load
Environment: Kubernetes v1.24, self-hosted
Summary: Sudden burst of traffic overwhelmed services due to slow pod boot time.
What Happened: HPA triggered scale-out, but pods took too long to start. Users got errors.
Diagnosis Steps:
	• Noticed gap between scale-out and readiness.
	• Startup probes took 30s+ per pod.
Root Cause: App container had heavy init routines.
Fix/Workaround:
	• Optimized Docker image layers and moved setup to init containers.
Lessons Learned: Scale-out isn’t instant; pod readiness matters.
How to Avoid:
	• Track ReadySeconds vs ReplicaScale delay.
	• Pre-pull images and optimize pod init time.

📘 Scenario #419: Overuse of Liveness Probes Disrupted Load Balance
Category: Scaling & Load
Environment: Kubernetes v1.21, bare metal
Summary: Misfiring liveness probes killed healthy pods during load test.
What Happened: Sudden scale-out introduced new pods, which were killed due to false negatives on liveness probes.
Diagnosis Steps:
	• Pod logs showed probe failures under high CPU.
	• Readiness was OK, liveness killed them anyway.
Root Cause: CPU starvation during load caused probe timeouts.
Fix/Workaround:
	• Increased probe timeoutSeconds and failureThreshold.
Lessons Learned: Under load, even health checks need headroom.
How to Avoid:
	• Separate readiness from liveness logic.
	• Gracefully handle CPU-heavy workloads.

📘 Scenario #420: Scale-In Happened Before Queue Was Drained
Category: Scaling & Load
Environment: Kubernetes v1.26, RabbitMQ + consumers
Summary: Consumers scaled in while queue still had unprocessed messages.
What Happened: Queue depth remained, but pods were terminated.
Diagnosis Steps:
	• Observed message backlog after autoscaler scale-in.
	• Consumers had no shutdown hook to drain queue.
Root Cause: Scale-in triggered without consumer workload cleanup.
Fix/Workaround:
	• Added preStop hook to finish queue processing.
Lessons Learned: Consumers must handle shutdown gracefully.
How to Avoid:
	• Track message queues with KEDA or custom metrics.
	• Add drain() logic on signal trap in consumer code.

📘 Scenario #421: Node Drain Race Condition During Scale Down
Category: Scaling & Load
Environment: Kubernetes v1.23, GKE
Summary: Node drain raced with pod termination, causing pod loss.
What Happened: Pods were terminated while the node was still draining, leading to data loss.
Diagnosis Steps:
	• kubectl describe node showed multiple eviction races.
	• Pod logs showed abrupt termination without graceful shutdown.
Root Cause: Scale-down process didn’t wait for node draining to complete fully.
Fix/Workaround:
	• Adjusted terminationGracePeriodSeconds for pods.
	• Introduced node draining delay in scaling policy.
Lessons Learned: Node draining should be synchronized with pod termination.
How to Avoid:
	• Use PodDisruptionBudget to ensure safe scaling.
	• Implement pod graceful shutdown hooks.

📘 Scenario #422: HPA Disabled Due to Missing Resource Requests
Category: Scaling & Load
Environment: Kubernetes v1.22, AWS EKS
Summary: Horizontal Pod Autoscaler (HPA) failed to trigger because resource requests weren’t set.
What Happened: HPA couldn’t scale pods up despite high traffic due to missing CPU/memory resource requests.
Diagnosis Steps:
	• kubectl describe deployment revealed missing resources.requests.
	• Logs indicated HPA couldn’t fetch metrics without resource requests.
Root Cause: Missing resource request fields prevented HPA from making scaling decisions.
Fix/Workaround:
	• Set proper resources.requests in the deployment YAML.
Lessons Learned: Always define resource requests to enable autoscaling.
How to Avoid:
	• Define resource requests/limits for every pod.
	• Enable autoscaling based on requests/limits.

📘 Scenario #423: Unexpected Overprovisioning of Pods
Category: Scaling & Load
Environment: Kubernetes v1.24, DigitalOcean
Summary: Unnecessary pod scaling due to misconfigured resource limits.
What Happened: Pods scaled up unnecessarily due to excessively high resource limits.
Diagnosis Steps:
	• HPA logs showed frequent scale-ups even during low load.
	• Resource limits were higher than actual usage.
Root Cause: Overestimated resource limits in pod configuration.
Fix/Workaround:
	• Reduced resource limits to more realistic values.
Lessons Learned: Proper resource allocation helps prevent scaling inefficiencies.
How to Avoid:
	• Monitor resource consumption patterns before setting limits.
	• Use Kubernetes resource usage metrics to adjust configurations.

📘 Scenario #424: Autoscaler Failed During StatefulSet Upgrade
Category: Scaling & Load
Environment: Kubernetes v1.25, AKS
Summary: Horizontal scaling issues occurred during rolling upgrade of StatefulSet.
What Happened: StatefulSet failed to scale out during a rolling upgrade, causing delayed availability of new pods.
Diagnosis Steps:
	• Observed kubectl get pods showing delayed stateful pod restarts.
	• HPA did not trigger due to stuck pod state.
Root Cause: Rolling upgrade conflicted with autoscaler logic due to StatefulSet constraints.
Fix/Workaround:
	• Adjusted StatefulSet rollingUpdate strategy.
	• Tuned autoscaler thresholds for more aggressive scaling.
Lessons Learned: Ensure compatibility between scaling and StatefulSet updates.
How to Avoid:
	• Test upgrade and scaling processes in staging environments.
	• Separate stateful workloads from stateless ones for scaling flexibility.

📘 Scenario #425: Inadequate Load Distribution in a Multi-AZ Setup
Category: Scaling & Load
Environment: Kubernetes v1.27, AWS EKS
Summary: Load balancing wasn’t even across availability zones, leading to inefficient scaling.
What Happened: More traffic hit one availability zone (AZ), causing scaling delays in the other AZs.
Diagnosis Steps:
	• Analyzed kubectl describe svc and found skewed traffic distribution.
	• Observed insufficient pod presence in multiple AZs.
Root Cause: The Kubernetes service didn’t properly distribute traffic across AZs.
Fix/Workaround:
	• Updated service to use topologySpreadConstraints for better AZ distribution.
Lessons Learned: Multi-AZ distribution requires proper spread constraints for effective scaling.
How to Avoid:
	• Use topologySpreadConstraints in services to ensure balanced load.
	• Review multi-AZ architecture for traffic efficiency.
```
