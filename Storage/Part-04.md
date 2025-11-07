📘 Scenario #376: Ceph RBD Volume Mount Failure Due to Kernel Mismatch
Category: Storage
Environment: Kubernetes v1.21, Rook-Ceph
Summary: Mounting Ceph RBD volume failed after a node kernel upgrade.
What Happened: The new kernel lacked required RBD modules.
Diagnosis Steps:
	• dmesg showed rbd: module not found.
	• CSI logs indicated mount failed.
Root Cause: Kernel modules not pre-installed after OS patching.
Fix/Workaround:
	• Reinstalled kernel modules and rebooted node.
Lessons Learned: Kernel upgrades can silently break storage drivers.
How to Avoid:
	• Validate CSI compatibility post-upgrade.
	• Use DaemonSet to check required modules.

📘 Scenario #377: CSI Volume Cleanup Delay Leaves Orphaned Devices
Category: Storage
Environment: Kubernetes v1.24, Azure Disk CSI
Summary: Volume deletion left orphaned devices on the node, consuming disk space.
What Happened: Node failed to clean up mount paths after volume detach due to a kubelet bug.
Diagnosis Steps:
	• Found stale device mounts in /var/lib/kubelet/plugins/kubernetes.io/csi.
Root Cause: Kubelet failed to unmount due to corrupted symlink.
Fix/Workaround:
	• Manually removed symlinks and restarted kubelet.
Lessons Learned: CSI volume cleanup isn’t always reliable.
How to Avoid:
	• Monitor stale mounts.
	• Automate cleanup scripts in node maintenance routines.

📘 Scenario #378: Immutable ConfigMap Used in CSI Sidecar Volume Mount
Category: Storage
Environment: Kubernetes v1.23, EKS
Summary: CSI sidecar depended on a ConfigMap that was updated, but volume behavior didn’t change.
What Happened: Sidecar didn’t restart, so old config was retained.
Diagnosis Steps:
	• Volume behavior didn't reflect updated parameters.
	• Verified sidecar was still running with old config.
Root Cause: ConfigMap change wasn’t detected because it was mounted as a volume.
Fix/Workaround:
	• Restarted CSI sidecar pods.
Lessons Learned: Mounting ConfigMaps doesn’t auto-reload them.
How to Avoid:
	• Use checksum/config annotations to force rollout.
	• Don’t rely on in-place ConfigMap mutation.

📘 Scenario #379: PodMount Denied Due to SecurityContext Constraints
Category: Storage
Environment: Kubernetes v1.25, OpenShift with SCCs
Summary: Pod failed to mount PVC due to restricted SELinux type in pod’s security context.
What Happened: OpenShift SCC prevented the pod from mounting a volume with a mismatched SELinux context.
Diagnosis Steps:
	• Events: permission denied during mount.
	• Reviewed SCC and found allowedSELinuxOptions was too strict.
Root Cause: Security policies blocked mount operation.
Fix/Workaround:
	• Modified SCC to allow required context or used correct volume labeling.
Lessons Learned: Storage + security integration is often overlooked.
How to Avoid:
	• In tightly controlled environments, align volume labels with pod policies.
	• Audit SCCs with volume access in mind.

📘 Scenario #380: VolumeProvisioner Race Condition Leads to Duplicated PVC
Category: Storage
Environment: Kubernetes v1.24, CSI with dynamic provisioning
Summary: Simultaneous provisioning requests created duplicate PVs for a single PVC.
What Happened: PVC provisioning logic retried rapidly, and CSI provisioner created two volumes.
Diagnosis Steps:
	• Observed two PVs with same claimRef.
	• Events showed duplicate provision succeeded entries.
Root Cause: CSI controller did not lock claim state.
Fix/Workaround:
	• Patched CSI controller to implement idempotent provisioning.
Lessons Learned: CSI must be fault-tolerant to API retries.
How to Avoid:
	• Ensure CSI drivers enforce claim uniqueness.
	• Use exponential backoff and idempotent logic.

📘 Scenario #381: PVC Bound to Deleted PV After Restore
Category: Storage
Environment: Kubernetes v1.25, Velero restore with CSI driver
Summary: Restored PVC bound to a PV that no longer existed, causing stuck pods.
What Happened: During a cluster restore, PVC definitions were restored before their associated PVs. The missing PV names were still referenced.
Diagnosis Steps:
	• PVCs stuck in Pending state.
	• Events: PV does not exist.
	• Velero logs showed PVCs restored first.
Root Cause: Restore ordering issue in backup tool.
Fix/Workaround:
	• Deleted and re-created PVCs manually or re-triggered restore in correct order.
Lessons Learned: PVC-PV binding is tightly coupled.
How to Avoid:
	• Use volume snapshot restores or ensure PVs are restored before PVCs.
	• Validate backup tool restore ordering.

📘 Scenario #382: Unexpected Volume Type Defaults to HDD Instead of SSD
Category: Storage
Environment: Kubernetes v1.24, GKE with dynamic provisioning
Summary: Volumes defaulted to HDD even though workloads needed SSD.
What Happened: StorageClass used default pd-standard instead of pd-ssd.
Diagnosis Steps:
	• IOPS metrics showed high latency.
	• Checked StorageClass: wrong type.
Root Cause: Implicit default used in dynamic provisioning.
Fix/Workaround:
	• Updated manifests to explicitly reference pd-ssd.
Lessons Learned: Defaults may not match workload expectations.
How to Avoid:
	• Always define storage class with performance explicitly.
	• Audit default class across environments.

📘 Scenario #383: ReclaimPolicy Retain Caused Resource Leaks
Category: Storage
Environment: Kubernetes v1.22, bare-metal CSI
Summary: Deleting PVCs left behind unused PVs and disks.
What Happened: PVs had ReclaimPolicy: Retain, so disks weren’t deleted.
Diagnosis Steps:
	• PVs stuck in Released state.
	• Disk usage on nodes kept increasing.
Root Cause: Misconfigured reclaim policy.
Fix/Workaround:
	• Manually cleaned up PVs and external disk artifacts.
Lessons Learned: Retain policy requires manual lifecycle management.
How to Avoid:
	• Use Delete for ephemeral workloads.
	• Periodically audit released PVs.

📘 Scenario #384: ReadWriteOnce PVC Mounted by Multiple Pods
Category: Storage
Environment: Kubernetes v1.23, AWS EBS
Summary: Attempt to mount a ReadWriteOnce PVC on two pods in different AZs failed silently.
What Happened: Pods scheduled across AZs; EBS volume couldn't attach to multiple nodes.
Diagnosis Steps:
	• Pods stuck in ContainerCreating.
	• Events showed volume not attachable.
Root Cause: ReadWriteOnce restriction and AZ mismatch.
Fix/Workaround:
	• Updated deployment to use ReadWriteMany (EFS) for shared access.
Lessons Learned: RWX vs RWO behavior varies by volume type.
How to Avoid:
	• Use appropriate access modes per workload.
	• Restrict scheduling to compatible zones.

📘 Scenario #385: VolumeAttach Race on StatefulSet Rolling Update
Category: Storage
Environment: Kubernetes v1.26, StatefulSet with CSI driver
Summary: Volume attach operations failed during parallel pod updates.
What Happened: Two pods in a StatefulSet update attempted to use the same PVC briefly due to quick scale down/up.
Diagnosis Steps:
	• Events: Multi-Attach error for volume.
	• CSI logs showed repeated attach/detach.
Root Cause: StatefulSet update policy did not wait for volume detachment.
Fix/Workaround:
	• Set podManagementPolicy: OrderedReady.
Lessons Learned: StatefulSet updates need to be serialized with volume awareness.
How to Avoid:
	• Tune StatefulSet rollout policies.
	• Monitor CSI attach/detach metrics.

📘 Scenario #386: CSI Driver CrashLoop Due to Missing Node Labels
Category: Storage
Environment: Kubernetes v1.24, OpenEBS CSI
Summary: CSI sidecars failed to initialize due to missing node topology labels.
What Happened: A node upgrade wiped custom labels needed for topology-aware provisioning.
Diagnosis Steps:
	• Logs: missing topology key node label.
	• CSI pods in CrashLoopBackOff.
Root Cause: Topology-based provisioning misconfigured.
Fix/Workaround:
	• Reapplied node labels and restarted sidecars.
Lessons Learned: Custom node labels are critical for CSI topology hints.
How to Avoid:
	• Enforce node label consistency using DaemonSets or node admission webhooks.

📘 Scenario #387: PVC Deleted While Volume Still Mounted
Category: Storage
Environment: Kubernetes v1.22, on-prem CSI
Summary: PVC deletion didn’t unmount volume due to finalizer stuck on pod.
What Happened: Pod was terminating but stuck, so volume detach never happened.
Diagnosis Steps:
	• PVC deleted, but disk remained attached.
	• Pod in Terminating state for hours.
Root Cause: Finalizer logic bug in kubelet.
Fix/Workaround:
	• Force deleted pod, manually detached volume.
Lessons Learned: Volume lifecycle is tied to pod finalization.
How to Avoid:
	• Monitor long-running Terminating pods.
	• Use proper finalizer cleanup logic.

📘 Scenario #388: In-Tree Volume Plugin Migration Caused Downtime
Category: Storage
Environment: Kubernetes v1.25, GKE
Summary: GCE PD plugin migration to CSI caused volume mount errors.
What Happened: After upgrade, in-tree plugin was disabled but CSI driver wasn’t fully configured.
Diagnosis Steps:
	• Events: failed to provision volume.
	• CSI driver not installed.
Root Cause: Incomplete migration preparation.
Fix/Workaround:
	• Re-enabled legacy plugin until CSI was functional.
Lessons Learned: Plugin migration is not automatic.
How to Avoid:
	• Review CSI migration readiness for your storage before upgrades.

📘 Scenario #389: Overprovisioned Thin Volumes Hit Underlying Limit
Category: Storage
Environment: Kubernetes v1.24, LVM-based CSI
Summary: Thin-provisioned volumes ran out of physical space, affecting all pods.
What Happened: Overcommitted volumes filled up the disk pool.
Diagnosis Steps:
	• df on host showed 100% disk.
	• LVM pool full, volumes became read-only.
Root Cause: No enforcement of provisioning limits.
Fix/Workaround:
	• Resized physical disk and added monitoring.
Lessons Learned: Thin provisioning must be paired with storage usage enforcement.
How to Avoid:
	• Monitor volume pool usage.
	• Set quotas or alerts for overcommit.

📘 Scenario #390: Dynamic Provisioning Failure Due to Quota Exhaustion
Category: Storage
Environment: Kubernetes v1.26, vSphere CSI
Summary: PVCs failed to provision silently due to exhausted storage quota.
What Happened: Storage backend rejected volume create requests.
Diagnosis Steps:
	• PVC stuck in Pending.
	• CSI logs: quota exceeded.
Root Cause: Backend quota exceeded without Kubernetes alerting.
Fix/Workaround:
	• Increased quota or deleted old volumes.
Lessons Learned: Kubernetes doesn’t surface backend quota status clearly.
How to Avoid:
	• Integrate storage backend alerts into cluster monitoring.
	• Tag and age out unused PVCs periodically.

📘 Scenario #391: PVC Resizing Didn’t Expand Filesystem Automatically
Category: Storage
Environment: Kubernetes v1.24, AWS EBS, ext4 filesystem
Summary: PVC was resized but the pod’s filesystem didn’t reflect the new size.
What Happened: The PersistentVolume was expanded, but the pod using it didn’t see the increased size until restarted.
Diagnosis Steps:
	• df -h inside the pod showed old capacity.
	• PVC showed updated size in Kubernetes.
Root Cause: Filesystem expansion requires a pod restart unless using CSI drivers with ExpandInUse support.
Fix/Workaround:
	• Restarted the pod to trigger filesystem expansion.
Lessons Learned: Volume expansion is two-step: PV resize and filesystem resize.
How to Avoid:
	• Use CSI drivers that support in-use expansion.
	• Add automation to restart pods after volume resize.

📘 Scenario #392: StatefulSet Pods Lost Volume Data After Node Reboot
Category: Storage
Environment: Kubernetes v1.22, local-path-provisioner
Summary: Node reboots caused StatefulSet volumes to disappear due to ephemeral local storage.
What Happened: After node maintenance, pods were rescheduled and couldn’t find their PVC data.
Diagnosis Steps:
	• ls inside pod showed empty volumes.
	• PVCs bound to node-specific paths that no longer existed.
Root Cause: Using local-path provisioner without persistence guarantees.
Fix/Workaround:
	• Migrated to network-attached persistent storage (NFS/CSI).
Lessons Learned: Local storage is node-specific and non-resilient.
How to Avoid:
	• Use proper CSI drivers with data replication for StatefulSets.

📘 Scenario #393: VolumeSnapshots Failed to Restore with Immutable Fields
Category: Storage
Environment: Kubernetes v1.25, VolumeSnapshot API
Summary: Restore operation failed due to immutable PVC spec fields like access mode.
What Happened: Attempted to restore snapshot into a PVC with modified parameters.
Diagnosis Steps:
	• Error: cannot change accessMode after creation.
Root Cause: Snapshot restore tried to override immutable PVC fields.
Fix/Workaround:
	• Created a new PVC with correct parameters and attached manually.
Lessons Learned: PVC fields are not override-safe during snapshot restores.
How to Avoid:
	• Restore into newly created PVCs.
	• Match snapshot PVC spec exactly.

📘 Scenario #394: GKE Autopilot PVCs Stuck Due to Resource Class Conflict
Category: Storage
Environment: GKE Autopilot, dynamic PVC provisioning
Summary: PVCs remained in Pending state due to missing resource class binding.
What Happened: GKE Autopilot required both PVC and pod to define compatible resourceClassName.
Diagnosis Steps:
	• Events: No matching ResourceClass.
	• Pod log: PVC resource class mismatch.
Root Cause: Autopilot restrictions on dynamic provisioning.
Fix/Workaround:
	• Updated PVCs and workload definitions to specify supported resource classes.
Lessons Learned: GKE Autopilot enforces stricter policies on storage.
How to Avoid:
	• Follow GKE Autopilot documentation carefully.
	• Avoid implicit defaults in manifests.

📘 Scenario #395: Cross-Zone Volume Scheduling Failed in Regional Cluster
Category: Storage
Environment: Kubernetes v1.24, GKE regional cluster
Summary: Pods failed to schedule because volumes were provisioned in a different zone than the node.
What Happened: Regional cluster scheduling pods to one zone while PVCs were created in another.
Diagnosis Steps:
	• Events: FailedScheduling: volume not attachable.
Root Cause: Storage class used zonal disks instead of regional.
Fix/Workaround:
	• Updated storage class to use regional persistent disks.
Lessons Learned: Volume zone affinity must match cluster layout.
How to Avoid:
	• Use regional disks in regional clusters.
	• Always define zone spreading policy explicitly.

📘 Scenario #396: Stuck Finalizers on Deleted PVCs Blocking Namespace Deletion
Category: Storage
Environment: Kubernetes v1.22, CSI driver
Summary: Finalizers on PVCs blocked namespace deletion for hours.
What Happened: Namespace was stuck in Terminating due to PVCs with finalizers not being properly removed.
Diagnosis Steps:
	• Checked PVC YAML: finalizers section present.
	• Logs: CSI controller error during cleanup.
Root Cause: CSI cleanup failed due to stale volume handles.
Fix/Workaround:
	• Patched PVCs to remove finalizers manually.
Lessons Learned: Finalizers can hang namespace deletion.
How to Avoid:
	• Monitor PVCs with stuck finalizers.
	• Regularly validate volume plugin cleanup.

📘 Scenario #397: CSI Driver Upgrade Corrupted Volume Attachments
Category: Storage
Environment: Kubernetes v1.23, OpenEBS
Summary: CSI driver upgrade introduced a regression causing volume mounts to fail.
What Happened: After a helm-based CSI upgrade, pods couldn’t mount volumes.
Diagnosis Steps:
	• Logs: mount timeout errors.
	• CSI logs showed broken symlinks.
Root Cause: Helm upgrade deleted old CSI socket paths before new one started.
Fix/Workaround:
	• Rolled back to previous CSI driver version.
Lessons Learned: Upgrades should always be tested in staging clusters.
How to Avoid:
	• Perform canary upgrades.
	• Backup CSI configurations and verify volume health post-upgrade.

📘 Scenario #398: Stale Volume Handles After Disaster Recovery Cutover
Category: Storage
Environment: Kubernetes v1.25, Velero restore to DR cluster
Summary: Stale volume handles caused new PVCs to fail provisioning.
What Happened: Restored PVs referenced non-existent volume handles in new cloud region.
Diagnosis Steps:
	• CSI logs: volume handle not found.
	• kubectl describe pvc: stuck in Pending.
Root Cause: Velero restore didn’t remap volume handles for the DR environment.
Fix/Workaround:
	• Manually edited PV specs or recreated PVCs from scratch.
Lessons Learned: Volume handles are environment-specific.
How to Avoid:
	• Customize Velero restore templates.
	• Use snapshots or backups that are region-agnostic.

📘 Scenario #399: Application Wrote Outside Mounted Path and Lost Data
Category: Storage
Environment: Kubernetes v1.24, default mountPath
Summary: Application wrote logs to /tmp, not mounted volume, causing data loss on pod eviction.
What Happened: Application configuration didn’t match the PVC mount path.
Diagnosis Steps:
	• Pod deleted → logs disappeared.
	• PVC had no data.
Root Cause: Application not configured to use the mounted volume path.
Fix/Workaround:
	• Updated application config to write into the mount path.
Lessons Learned: Mounted volumes don’t capture all file writes by default.
How to Avoid:
	• Review app config during volume integration.
	• Validate mount paths with a test write-read cycle.

📘 Scenario #400: Cluster Autoscaler Deleted Nodes with Mounted Volumes
Category: Storage
Environment: Kubernetes v1.23, AWS EKS with CA
Summary: Cluster Autoscaler aggressively removed nodes with attached volumes, causing workload restarts.
What Happened: Nodes were deemed underutilized and deleted while volumes were still mounted.
Diagnosis Steps:
	• Volumes detached mid-write, causing file corruption.
	• Events showed node scale-down triggered by CA.
Root Cause: No volume-aware protection in CA.
Fix/Workaround:
	• Enabled --balance-similar-node-groups and --skip-nodes-with-local-storage.
Lessons Learned: Cluster Autoscaler must be volume-aware.
How to Avoid:
	• Configure CA to respect mounted volumes.
	• Tag volume-critical nodes as unschedulable before scale-down.