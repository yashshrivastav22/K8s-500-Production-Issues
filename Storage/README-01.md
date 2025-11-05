📘 Scenario #301: PVC Stuck in Terminating State After Node Crash
Category: Storage
Environment: Kubernetes v1.22, EBS CSI Driver on EKS
Summary: A node crash caused a PersistentVolumeClaim (PVC) to be stuck in Terminating, blocking pod deletion.
What Happened: The node hosting the pod with the PVC crashed and never returned. The volume was still attached, and Kubernetes couldn’t cleanly unmount or delete it.
Diagnosis Steps:
	• Described the PVC: status was Terminating.
	• Checked finalizers on the PVC object.
	• Verified the volume was still attached to the crashed node via AWS Console.
Root Cause: The volume attachment record wasn’t cleaned up due to the ungraceful node failure.
Fix/Workaround:
	• Manually removed the PVC finalizers.
	• Used aws ec2 detach-volume to forcibly detach.
Lessons Learned: Finalizers can block PVC deletion in edge cases.
How to Avoid:
	• Use the external-attacher CSI sidecar with leader election.
	• Implement automation to detect and clean up stuck attachments.

📘 Scenario #302: Data Corruption on HostPath Volumes
Category: Storage
Environment: Kubernetes v1.20, Bare Metal
Summary: Multiple pods sharing a HostPath volume led to inconsistent file states and eventual corruption.
What Happened: Two pods were writing to the same HostPath volume concurrently, which wasn’t designed for concurrent write access. Files became corrupted due to race conditions.
Diagnosis Steps:
	• Identified common HostPath mount across pods.
	• Checked application logs — showed file write conflicts.
	• Inspected corrupted data on disk.
Root Cause: Lack of coordination and access control on shared HostPath.
Fix/Workaround:
	• Moved workloads to CSI-backed volumes with ReadWriteOnce enforcement.
	• Ensured only one pod accessed a volume at a time.
Lessons Learned: HostPath volumes offer no isolation or locking guarantees.
How to Avoid:
	• Use CSI volumes with enforced access modes.
	• Avoid HostPath unless absolutely necessary.

📘 Scenario #303: Volume Mount Fails Due to Node Affinity Mismatch
Category: Storage
Environment: Kubernetes v1.23, GCE PD on GKE
Summary: A pod was scheduled on a node that couldn’t access the persistent disk due to zone mismatch.
What Happened: A StatefulSet PVC was bound to a disk in us-central1-a, but the pod got scheduled in us-central1-b, causing volume mount failure.
Diagnosis Steps:
	• Described pod: showed MountVolume.MountDevice failed.
	• Described PVC and PV: zone mismatch confirmed.
	• Looked at scheduler decisions — no awareness of volume zone.
Root Cause: Scheduler was unaware of zone constraints on the PV.
Fix/Workaround:
	• Added topology.kubernetes.io/zone node affinity to match PV.
	• Ensured StatefulSets used storage classes with volume binding mode WaitForFirstConsumer.
Lessons Learned: Without delayed binding, PVs can bind in zones that don’t match future pods.
How to Avoid:
	• Use WaitForFirstConsumer for dynamic provisioning.
	• Always define zone-aware topology constraints.

📘 Scenario #304: PVC Not Rescheduled After Node Deletion
Category: Storage
Environment: Kubernetes v1.21, Azure Disk CSI
Summary: A StatefulSet pod failed to reschedule after its node was deleted, due to Azure disk still being attached.
What Happened: A pod using Azure Disk was on a node that was manually deleted. Azure did not automatically detach the disk, so rescheduling failed.
Diagnosis Steps:
	• Pod stuck in ContainerCreating.
	• CSI logs showed "Volume is still attached to another node".
	• Azure Portal confirmed volume was attached.
Root Cause: Manual node deletion bypassed volume detachment logic.
Fix/Workaround:
	• Detached the disk from the Azure console.
	• Recreated pod successfully on another node.
Lessons Learned: Manual infrastructure changes can break Kubernetes assumptions.
How to Avoid:
	• Use automation/scripts for safe node draining and deletion.
	• Monitor CSI detachment status on node removal.

📘 Scenario #305: Long PVC Rebinding Time on StatefulSet Restart
Category: Storage
Environment: Kubernetes v1.24, Rook Ceph
Summary: Restarting a StatefulSet with many PVCs caused long downtime due to slow rebinding.
What Happened: A 20-replica StatefulSet was restarted, and each pod waited for its PVC to rebind and attach. Ceph mount operations were sequential and slow.
Diagnosis Steps:
	• Pods stuck at Init stage for 15–20 minutes.
	• Ceph logs showed delayed attachment per volume.
	• Described PVCs: bound but not mounted.
Root Cause: Sequential volume mount throttling and inefficient CSI attach policies.
Fix/Workaround:
	• Tuned CSI attach concurrency.
	• Split the StatefulSet into smaller chunks.
Lessons Learned: Large-scale StatefulSets need volume attach tuning.
How to Avoid:
	• Parallelize pod restarts using partitioned rollouts.
	• Monitor CSI mount throughput.

📘 Scenario #306: CSI Volume Plugin Crash Loops Due to Secret Rotation
Category: Storage
Environment: Kubernetes v1.25, Vault CSI Provider
Summary: Volume plugin entered crash loop after secret provider’s token was rotated unexpectedly.
What Happened: A service account used by the Vault CSI plugin had its token rotated mid-operation. The plugin couldn’t fetch new credentials and crashed.
Diagnosis Steps:
	• CrashLoopBackOff on csi-vault-provider pods.
	• Logs showed "401 Unauthorized" from Vault.
	• Verified service account token changed recently.
Root Cause: No logic in plugin to handle token change or re-auth.
Fix/Workaround:
	• Restarted the CSI plugin pods.
	• Upgraded plugin to a version with token refresh logic.
Lessons Learned: CSI providers must gracefully handle credential rotations.
How to Avoid:
	• Use projected service account tokens with auto-refresh.
	• Monitor plugin health on secret rotations.

📘 Scenario #307: ReadWriteMany PVCs Cause IO Bottlenecks
Category: Storage
Environment: Kubernetes v1.23, NFS-backed PVCs
Summary: Heavy read/write on a shared PVC caused file IO contention and throttling across pods.
What Happened: Multiple pods used a shared ReadWriteMany PVC for scratch space. Concurrent writes led to massive IO wait times and high pod latency.
Diagnosis Steps:
	• High pod latency and CPU idle time.
	• Checked NFS server: high disk and network usage.
	• Application logs showed timeouts.
Root Cause: No coordination or locking on shared writable volume.
Fix/Workaround:
	• Partitioned workloads to use isolated volumes.
	• Added cache layer for reads.
Lessons Learned: RWX volumes are not always suitable for concurrent writes.
How to Avoid:
	• Use RWX volumes for read-shared data only.
	• Avoid writes unless using clustered filesystems (e.g., CephFS).

📘 Scenario #308: PVC Mount Timeout Due to PodSecurityPolicy
Category: Storage
Environment: Kubernetes v1.21, PSP Enabled Cluster
Summary: A pod couldn’t mount a volume because PodSecurityPolicy (PSP) rejected required fsGroup.
What Happened: A storage class required fsGroup for volume mount permissions. The pod didn’t set it, and PSP disallowed dynamic group assignment.
Diagnosis Steps:
	• Pod stuck in CreateContainerConfigError.
	• Events showed “pod rejected by PSP”.
	• Storage class required fsGroup.
Root Cause: Incompatible PSP with volume mount security requirements.
Fix/Workaround:
	• Modified PSP to allow required fsGroup range.
	• Updated pod security context.
Lessons Learned: Storage plugins often need security context alignment.
How to Avoid:
	• Review storage class requirements.
	• Align security policies with volume specs.

📘 Scenario #309: Orphaned PVs After Namespace Deletion
Category: Storage
Environment: Kubernetes v1.20, Self-Hosted
Summary: Deleting a namespace did not clean up PersistentVolumes, leading to leaked storage.
What Happened: A team deleted a namespace with PVCs, but the associated PVs (with Retain policy) remained and weren’t cleaned up.
Diagnosis Steps:
	• Listed all PVs: found orphaned volumes in Released state.
	• Checked reclaim policy: Retain.
Root Cause: Manual cleanup required for Retain policy.
Fix/Workaround:
	• Deleted old PVs and disks manually.
	• Changed reclaim policy to Delete for dynamic volumes.
Lessons Learned: Reclaim policy should match cleanup expectations.
How to Avoid:
	• Use Delete unless you need manual volume recovery.
	• Monitor Released PVs for leaks.

📘 Scenario #310: StorageClass Misconfiguration Blocks Dynamic Provisioning
Category: Storage
Environment: Kubernetes v1.25, GKE
Summary: New PVCs failed to bind due to a broken default StorageClass with incorrect parameters.
What Happened: A recent update modified the default StorageClass to use a non-existent disk type. All PVCs created with default settings failed provisioning.
Diagnosis Steps:
	• PVCs in Pending state.
	• Checked events: “failed to provision volume with StorageClass”.
	• Described StorageClass: invalid parameter type: ssd2.
Root Cause: Mistyped disk type in StorageClass definition.
Fix/Workaround:
	• Corrected StorageClass parameters.
	• Manually bound PVCs with valid classes.
Lessons Learned: Default StorageClass affects many workloads.
How to Avoid:
	• Validate StorageClass on cluster upgrades.
	• Use automated tests for provisioning paths.

📘 Scenario #311: StatefulSet Volume Cloning Results in Data Leakage
Category: Storage
Environment: Kubernetes v1.24, CSI Volume Cloning enabled
Summary: Cloning PVCs between StatefulSet pods led to shared data unexpectedly appearing in new replicas.
What Happened: Engineers used volume cloning to duplicate data for new pods. They assumed data would be copied and isolated. However, clones preserved file locks and session metadata, which caused apps to behave erratically.
Diagnosis Steps:
	• New pods accessed old session data unexpectedly.
	• lsblk and md5sum on cloned volumes showed identical data.
	• Verified cloning was done via StorageClass that didn't support true snapshot isolation.
Root Cause: Misunderstanding of cloning behavior — logical clone ≠ deep copy.
Fix/Workaround:
	• Stopped cloning and switched to backup/restore-based provisioning.
	• Used rsync with integrity checks instead.
Lessons Learned: Not all clones are deep copies; understand your CSI plugin's clone semantics.
How to Avoid:
	• Use cloning only for stateless data unless supported thoroughly.
	• Validate cloned volume content before production use.

📘 Scenario #312: Volume Resize Not Reflected in Mounted Filesystem
Category: Storage
Environment: Kubernetes v1.22, OpenEBS
Summary: Volume expansion was successful on the PV, but pods didn’t see the increased space.
What Happened: After increasing PVC size, the PV reflected the new size, but df -h inside the pod still showed the old size.
Diagnosis Steps:
	• Checked PVC and PV: showed expanded size.
	• Pod logs indicated no disk space.
	• mount inside pod showed volume was mounted but not resized.
Root Cause: Filesystem resize not triggered automatically.
Fix/Workaround:
	• Restarted pod to remount the volume and trigger resize.
	• Verified resize2fs logs in CSI driver.
Lessons Learned: Volume resizing may require pod restarts depending on CSI driver.
How to Avoid:
	• Schedule a rolling restart after volume resize operations.
	• Check if your CSI driver supports online filesystem resizing.

📘 Scenario #313: CSI Controller Pod Crash Due to Log Overflow
Category: Storage
Environment: Kubernetes v1.23, Longhorn
Summary: The CSI controller crashed repeatedly due to unbounded logging filling up ephemeral storage.
What Happened: A looped RPC error generated thousands of log lines per second. Node /var/log/containers hit 100% disk usage.
Diagnosis Steps:
	• kubectl describe pod: showed OOMKilled and failed to write logs.
	• Checked node disk: /var was full.
	• Logs rotated too slowly.
Root Cause: Verbose logging + missing log throttling + small disk.
Fix/Workaround:
	• Added log rate limits via CSI plugin config.
	• Increased node ephemeral storage.
Lessons Learned: Logging misconfigurations can become outages.
How to Avoid:
	• Monitor log volume and disk usage.
	• Use log rotation and retention policies.

📘 Scenario #314: PVs Stuck in Released Due to Missing Finalizer Removal
Category: Storage
Environment: Kubernetes v1.21, NFS
Summary: PVCs were deleted, but PVs remained stuck in Released, preventing reuse.
What Happened: PVC deletion left behind PVs marked as Released, and the NFS driver didn’t remove finalizers, blocking clean-up.
Diagnosis Steps:
	• Listed PVs: showed Released, with kubernetes.io/pv-protection finalizer still present.
	• Couldn’t bind new PVCs due to status: Released.
Root Cause: Driver didn’t implement Delete reclaim logic properly.
Fix/Workaround:
	• Patched PVs to remove finalizers.
	• Recycled or deleted volumes manually.
Lessons Learned: Some drivers require manual cleanup unless fully CSI-compliant.
How to Avoid:
	• Use CSI drivers with full lifecycle support.
	• Monitor PV statuses regularly.

📘 Scenario #315: CSI Driver DaemonSet Deployment Missing Tolerations for Taints
Category: Storage
Environment: Kubernetes v1.25, Bare Metal
Summary: CSI Node plugin DaemonSet didn’t deploy on all nodes due to missing taint tolerations.
What Happened: Storage nodes were tainted (node-role.kubernetes.io/storage:NoSchedule), and the CSI DaemonSet didn’t tolerate it, so pods failed to mount volumes.
Diagnosis Steps:
	• CSI node pods not scheduled on certain nodes.
	• Checked node taints vs DaemonSet tolerations.
	• Pods stuck in Pending.
Root Cause: Taint/toleration mismatch in CSI node plugin manifest.
Fix/Workaround:
	• Added required tolerations to DaemonSet.
Lessons Learned: Storage plugins must tolerate relevant node taints to function correctly.
How to Avoid:
	• Review node taints and CSI tolerations during setup.
	• Use node affinity and tolerations for critical system components.

📘 Scenario #316: Mount Propagation Issues with Sidecar Containers
Category: Storage
Environment: Kubernetes v1.22, GKE
Summary: Sidecar containers didn’t see mounted volumes due to incorrect mountPropagation settings.
What Happened: An app container wrote to a mounted path, but sidecar container couldn’t read the changes.
Diagnosis Steps:
	• Logs in sidecar showed empty directory.
	• Checked volumeMounts: missing mountPropagation: Bidirectional.
Root Cause: Default mount propagation is None, blocking volume visibility between containers.
Fix/Workaround:
	• Added mountPropagation: Bidirectional to shared volumeMounts.
Lessons Learned: Without correct propagation, shared volumes don’t work across containers.
How to Avoid:
	• Understand container mount namespaces.
	• Always define propagation when using shared mounts.

📘 Scenario #317: File Permissions Reset on Pod Restart
Category: Storage
Environment: Kubernetes v1.20, CephFS
Summary: Pod volume permissions reset after each restart, breaking application logic.
What Happened: App wrote files with specific UID/GID. After restart, files were inaccessible due to CephFS resetting ownership.
Diagnosis Steps:
	• Compared ls -l before/after restart.
	• Storage class used fsGroup: 9999 by default.
Root Cause: PodSecurityContext didn't override fsGroup, so default applied every time.
Fix/Workaround:
	• Set explicit securityContext.fsGroup in pod spec.
Lessons Learned: CSI plugins may enforce ownership unless overridden.
How to Avoid:
	• Always declare expected ownership with securityContext.

📘 Scenario #318: Volume Mount Succeeds but Application Can't Write
Category: Storage
Environment: Kubernetes v1.23, EBS
Summary: Volume mounted correctly, but application failed to write due to filesystem mismatch.
What Happened: App expected xfs but volume formatted as ext4. Some operations silently failed or corrupted.
Diagnosis Steps:
	• Application logs showed invalid argument on file ops.
	• CSI driver defaulted to ext4.
	• Verified with df -T.
Root Cause: Application compatibility issue with default filesystem.
Fix/Workaround:
	• Used storage class parameter to specify xfs.
Lessons Learned: Filesystem types matter for certain workloads.
How to Avoid:
	• Align volume formatting with application expectations.

📘 Scenario #319: Volume Snapshot Restore Includes Corrupt Data
Category: Storage
Environment: Kubernetes v1.24, Velero + CSI Snapshots
Summary: Snapshot-based restore brought back corrupted state due to hot snapshot timing.
What Happened: Velero snapshot was taken during active write burst. Filesystem was inconsistent at time of snapshot.
Diagnosis Steps:
	• App logs showed corrupted files after restore.
	• Snapshot logs showed no quiescing.
	• Restore replayed same state.
Root Cause: No pre-freeze or app-level quiescing before snapshot.
Fix/Workaround:
	• Paused writes before snapshot.
	• Enabled filesystem freeze hook in Velero plugin.
Lessons Learned: Snapshots must be coordinated with app state.
How to Avoid:
	• Use pre/post hooks for consistent snapshotting.

📘 Scenario #320: Zombie Volumes Occupying Cloud Quota
Category: Storage
Environment: Kubernetes v1.25, AWS EBS
Summary: Deleted PVCs didn’t release volumes due to failed detach steps, leading to quota exhaustion.
What Happened: PVCs were deleted, but EBS volumes stayed in-use, blocking provisioning of new ones due to quota limits.
Diagnosis Steps:
	• Checked AWS Console: volumes remained.
	• Described events: detach errors during node crash.
Root Cause: CSI driver missed final detach due to abrupt node termination.
Fix/Workaround:
	• Manually detached and deleted volumes.
	• Adjusted controller retry limits.
Lessons Learned: Cloud volumes may silently linger even after PVC/PV deletion.
How to Avoid:
	• Use cloud resource monitoring.
	• Add alerts for orphaned volumes.

📘 Scenario #321: Volume Snapshot Garbage Collection Fails
Category: Storage
Environment: Kubernetes v1.25, CSI Snapshotter with Velero
Summary: Volume snapshots piled up because snapshot objects were not getting garbage collected after use.
What Happened: Snapshots triggered via Velero remained in the cluster even after restore, eventually exhausting cloud snapshot limits and storage quota.
Diagnosis Steps:
	• Listed all VolumeSnapshots and VolumeSnapshotContents — saw hundreds still in ReadyToUse: true state.
	• Checked finalizers on snapshot objects — found snapshot.storage.kubernetes.io/volumesnapshot not removed.
	• Velero logs showed successful restore but no cleanup action.
Root Cause: Snapshot GC controller didn’t remove finalizers due to missing permissions in Velero's service account.
Fix/Workaround:
	• Added required RBAC rules to Velero.
	• Manually deleted stale snapshot objects.
Lessons Learned: Improperly configured snapshot permissions can stall GC.
How to Avoid:
	• Always test snapshot and restore flows end-to-end.
	• Enable automated cleanup in your backup tooling.

📘 Scenario #322: Volume Mount Delays Due to Node Drain Stale Attachment
Category: Storage
Environment: Kubernetes v1.23, AWS EBS CSI
Summary: Volumes took too long to attach on new nodes after pod rescheduling due to stale attachment metadata.
What Happened: After draining a node for maintenance, workloads failed over, but volume attachments still pointed to old node, causing delays in remount.
Diagnosis Steps:
	• Described PV: still had attachedNode as drained one.
	• Cloud logs showed volume in-use errors.
	• CSI controller didn’t retry detach fast enough.
Root Cause: Controller had exponential backoff on detach retries.
Fix/Workaround:
	• Reduced backoff limit in CSI controller config.
	• Used manual detach via cloud CLI in emergencies.
Lessons Learned: Volume operations can get stuck in edge-node cases.
How to Avoid:
	• Use health checks to ensure detach success before draining.
	• Monitor VolumeAttachment objects during node ops.

📘 Scenario #323: Application Writes Lost After Node Reboot
Category: Storage
Environment: Kubernetes v1.21, Local Persistent Volumes
Summary: After a node reboot, pod restarted, but wrote to a different volume path, resulting in apparent data loss.
What Happened: Application data wasn’t persisted after a power cycle because the mount point dynamically changed.
Diagnosis Steps:
	• Compared volume paths before and after reboot.
	• Found PV had hostPath mount with no stable binding.
	• Volume wasn’t pinned to specific disk partition.
Root Cause: Local PV was defined with generic hostPath, not using local volume plugin with device references.
Fix/Workaround:
	• Refactored PV to use local with nodeAffinity.
	• Explicitly mounted disk partitions.
Lessons Learned: hostPath should not be used for production data.
How to Avoid:
	• Always use local storage plugin for node-local disks.
	• Avoid loosely defined persistent paths.

📘 Scenario #324: Pod CrashLoop Due to Read-Only Volume Remount
Category: Storage
Environment: Kubernetes v1.22, GCP Filestore
Summary: Pod volume was remounted as read-only after a transient network disconnect, breaking app write logic.
What Happened: During a brief NFS outage, volume was remounted in read-only mode by the NFS client. Application kept crashing due to inability to write logs.
Diagnosis Steps:
	• Checked mount logs: showed NFS remounted as read-only.
	• kubectl describe pod: showed volume still mounted.
	• Pod logs: permission denied on write.
Root Cause: NFS client behavior defaults to remount as read-only after timeout.
Fix/Workaround:
	• Restarted pod to trigger clean remount.
	• Tuned NFS mount options (soft, timeo, retry).
Lessons Learned: NFS remount behavior can silently switch access mode.
How to Avoid:
	• Monitor for dmesg or NFS client remounts.
	• Add alerts for unexpected read-only volume transitions.

📘 Scenario #325: Data Corruption on Shared Volume With Two Pods
Category: Storage
Environment: Kubernetes v1.23, NFS PVC shared by 2 pods
Summary: Two pods writing to the same volume caused inconsistent files and data loss.
What Happened: Both pods ran jobs writing to the same output files. Without file locking, one pod overwrote data from the other.
Diagnosis Steps:
	• Logs showed incomplete file writes.
	• File hashes changed mid-run.
	• No mutual exclusion mechanism implemented.
Root Cause: Shared volume used without locking or coordination between pods.
Fix/Workaround:
	• Refactored app logic to coordinate file writes via leader election.
	• Used a queue-based processing system.
Lessons Learned: Shared volume access must be controlled explicitly.
How to Avoid:
	• Never assume coordination when using shared volumes.
	• Use per-pod PVCs or job-level locking.
