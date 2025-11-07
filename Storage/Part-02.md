```
📘 Scenario #326: Mount Volume Exceeded Timeout
Category: Storage
Environment: Kubernetes v1.26, Azure Disk CSI
Summary: Pod remained stuck in ContainerCreating state because volume mount operations timed out.
What Happened: CSI node plugin had stale cache and attempted mount on incorrect device path. Retry logic delayed pod start by ~15 minutes.
Diagnosis Steps:
	• Described pod: stuck with Unable to mount volume error.
	• Node CSI logs: device not found.
	• Saw old mount references in plugin cache.
Root Cause: Plugin did not invalidate mount state properly after a failed mount.
Fix/Workaround:
	• Cleared plugin cache manually.
	• Upgraded CSI driver to fixed version.
Lessons Learned: CSI drivers can introduce delays through stale state.
How to Avoid:
	• Keep CSI drivers up-to-date.
	• Use pre-mount checks to validate device paths.

📘 Scenario #327: Static PV Bound to Wrong PVC
Category: Storage
Environment: Kubernetes v1.21, Manually created PVs
Summary: A misconfigured static PV got bound to the wrong PVC, exposing sensitive data.
What Happened: Two PVCs had overlapping selectors. The PV intended for app-A was bound to app-B, which accessed restricted files.
Diagnosis Steps:
	• Checked PV annotations: saw wrong PVC UID.
	• File system showed app-A data.
	• Both PVCs used identical storageClassName and no selector.
Root Cause: Ambiguous PV selection caused unintended binding.
Fix/Workaround:
	• Used volumeName field in PVCs for direct binding.
	• Set explicit labels/selectors to isolate.
Lessons Learned: Manual PVs require strict binding rules.
How to Avoid:
	• Use volumeName for static PV binding.
	• Avoid reusing storageClassName across different apps.

📘 Scenario #328: Pod Eviction Due to DiskPressure Despite PVC
Category: Storage
Environment: Kubernetes v1.22, Local PVs
Summary: Node evicted pods due to DiskPressure, even though app used dedicated PVC backed by a separate disk.
What Happened: Node root disk filled up with log data, triggering eviction manager. The PVC itself was healthy and not full.
Diagnosis Steps:
	• Node describe: showed DiskPressure condition true.
	• Application pod evicted due to node pressure, not volume pressure.
	• Root disk had full /var/log.
Root Cause: Kubelet doesn’t distinguish between root disk and attached volumes for eviction triggers.
Fix/Workaround:
	• Cleaned logs from root disk.
	• Moved logging to PVC-backed location.
Lessons Learned: PVCs don’t protect from node-level disk pressure.
How to Avoid:
	• Monitor node root disks in addition to volume usage.
	• Redirect logs and temp files to PVCs.

📘 Scenario #329: Pod Gets Stuck Due to Ghost Mount Point
Category: Storage
Environment: Kubernetes v1.20, iSCSI volumes
Summary: Pod failed to start because the mount point was partially deleted, leaving the system confused.
What Happened: After node crash, the iSCSI mount folder remained but device wasn’t attached. New pod couldn’t proceed due to leftover mount artifacts.
Diagnosis Steps:
	• CSI logs: mount path exists but not a mount point.
	• mount | grep iscsi — returned nothing.
	• ls /mnt/... — folder existed with empty contents.
Root Cause: Stale mount folder confused CSI plugin logic.
Fix/Workaround:
	• Manually deleted stale mount folders.
	• Restarted kubelet on affected node.
Lessons Learned: Mount lifecycle must be cleanly managed.
How to Avoid:
	• Use pre-start hooks to validate mount point integrity.
	• Include cleanup logic in custom CSI deployments.

📘 Scenario #330: PVC Resize Broke StatefulSet Ordering
Category: Storage
Environment: Kubernetes v1.24, StatefulSets + RWO PVCs
Summary: When resizing PVCs, StatefulSet pods restarted in parallel, violating ordinal guarantees.
What Happened: PVC expansion triggered pod restarts, but multiple pods came up simultaneously, causing database quorum failures.
Diagnosis Steps:
	• Checked StatefulSet controller behavior — PVC resize didn’t preserve pod startup order.
	• App logs: quorum could not be established.
Root Cause: StatefulSet controller didn’t serialize PVC resizes.
Fix/Workaround:
	• Manually controlled pod restarts during PVC resize.
	• Added readiness gates to enforce sequential boot.
Lessons Learned: StatefulSets don't coordinate PVC changes well.
How to Avoid:
	• Use podManagementPolicy: OrderedReady.
	• Handle resizes during maintenance windows.

📘 Scenario #331: ReadAfterWrite Inconsistency on Object Store-Backed CSI
Category: Storage
Environment: Kubernetes v1.26, MinIO CSI driver, Ceph RGW backend
Summary: Applications experienced stale reads immediately after writing to the same file via CSI mount backed by an S3-like object store.
What Happened: A distributed app wrote metadata and then read it back to validate—however, the file content was outdated due to eventual consistency in object backend.
Diagnosis Steps:
	• Logged file hashes before and after write — mismatch seen.
	• Found underlying storage was S3-compatible with eventual consistency.
	• CSI driver buffered writes asynchronously.
Root Cause: Object store semantics (eventual consistency) not suitable for synchronous read-after-write patterns.
Fix/Workaround:
	• Introduced write barriers and retry logic in app.
	• Switched to CephFS for strong consistency.
Lessons Learned: Object store-backed volumes need strong consistency guards.
How to Avoid:
	• Avoid using S3-style backends for workloads expecting POSIX semantics.
	• Use CephFS, NFS, or block storage for transactional I/O.

📘 Scenario #332: PV Resize Fails After Node Reboot
Category: Storage
Environment: Kubernetes v1.24, AWS EBS
Summary: After a node reboot, a PVC resize request remained pending, blocking pod start.
What Happened: VolumeExpansion was triggered via PVC patch. But after a node reboot, controller couldn't find the in-use mount point to complete fsResize.
Diagnosis Steps:
	• PVC status.conditions showed FileSystemResizePending.
	• CSI node plugin logs showed missing device.
	• Node reboot removed mount references prematurely.
Root Cause: Resize operation depends on volume being mounted at the time of filesystem expansion.
Fix/Workaround:
	• Reattached volume by starting pod temporarily on the node.
	• Resize completed automatically.
Lessons Learned: Filesystem resize requires node readiness and volume mount.
How to Avoid:
	• Schedule resizes during stable node windows.
	• Use pvc-resize readiness gates in automation.

📘 Scenario #333: CSI Driver Crash Loops on VolumeAttach
Category: Storage
Environment: Kubernetes v1.22, OpenEBS Jiva CSI
Summary: CSI node plugin entered CrashLoopBackOff due to panic during volume attach, halting all storage provisioning.
What Happened: VolumeAttachment object triggered a plugin bug—CSI crashed during RPC call, making storage class unusable.
Diagnosis Steps:
	• Checked CSI node logs — Go panic in attach handler.
	• Pods using Jiva SC failed with AttachVolume.Attach failed error.
	• CSI pod restarted every few seconds.
Root Cause: Volume metadata had an unexpected field due to version mismatch.
Fix/Workaround:
	• Rolled back CSI driver to stable version.
	• Purged corrupted volume metadata.
Lessons Learned: CSI versioning must be tightly managed.
How to Avoid:
	• Use upgrade staging before deploying new CSI versions.
	• Enable CSI health monitoring via liveness probes.

📘 Scenario #334: PVC Binding Fails Due to Multiple Default StorageClasses
Category: Storage
Environment: Kubernetes v1.23
Summary: PVC creation failed intermittently because the cluster had two storage classes marked as default.
What Happened: Two different teams installed their storage plugins (EBS and Rook), both marked default. PVC binding randomly chose one.
Diagnosis Steps:
	• Ran kubectl get storageclass — two entries with is-default-class=true.
	• PVCs had no storageClassName, leading to random binding.
	• One SC used unsupported reclaimPolicy.
Root Cause: Multiple default StorageClasses confuse the scheduler.
Fix/Workaround:
	• Patched one SC to remove the default annotation.
	• Explicitly specified SC in Helm charts.
Lessons Learned: Default SC conflicts silently break provisioning.
How to Avoid:
	• Enforce single default SC via cluster policy.
	• Always specify storageClassName explicitly in critical apps.

📘 Scenario #335: Zombie VolumeAttachment Blocks New PVC
Category: Storage
Environment: Kubernetes v1.21, Longhorn
Summary: After a node crash, a VolumeAttachment object was not garbage collected, blocking new PVCs from attaching.
What Happened: Application tried to use the volume, but Longhorn saw the old attachment from a dead node and refused reattachment.
Diagnosis Steps:
	• Listed VolumeAttachment resources — found one pointing to a non-existent node.
	• Longhorn logs: volume already attached to another node.
	• Node was removed forcefully.
Root Cause: VolumeAttachment controller did not clean up orphaned entries on node deletion.
Fix/Workaround:
	• Manually deleted VolumeAttachment.
	• Restarted CSI pods to refresh state.
Lessons Learned: Controller garbage collection is fragile post-node failure.
How to Avoid:
	• Use node lifecycle hooks to detach volumes gracefully.
	• Alert on dangling VolumeAttachments.

📘 Scenario #336: Persistent Volume Bound But Not Mounted
Category: Storage
Environment: Kubernetes v1.25, NFS
Summary: Pod entered Running state, but data was missing because PV was bound but not properly mounted.
What Happened: NFS server was unreachable during pod start. Pod started, but mount failed silently due to default retry behavior.
Diagnosis Steps:
	• mount output lacked NFS entry.
	• Pod logs: No such file or directory errors.
	• CSI logs showed silent NFS timeout.
Root Cause: CSI driver didn’t fail pod start when mount failed.
Fix/Workaround:
	• Added mountOptions: [hard,intr] to NFS SC.
	• Set pod readiness probe to check file existence.
Lessons Learned: Mount failures don’t always stop pod startup.
How to Avoid:
	• Validate mounts via init containers or probes.
	• Monitor CSI logs on pod lifecycle events.

📘 Scenario #337: CSI Snapshot Restore Overwrites Active Data
Category: Storage
Environment: Kubernetes v1.26, CSI snapshots (v1beta1)
Summary: User triggered a snapshot restore to an existing PVC, unintentionally overwriting live data.
What Happened: Snapshot restore process recreated PVC from source but didn't prevent overwriting an already-mounted volume.
Diagnosis Steps:
	• Traced VolumeSnapshotContent and PVC references.
	• PVC had reclaimPolicy: Retain, but was reused.
	• SnapshotClass used Delete policy.
Root Cause: No validation existed between snapshot restore and in-use PVCs.
Fix/Workaround:
	• Restored snapshot to a new PVC and used manual copy/move.
	• Added lifecycle checks before invoking restores.
Lessons Learned: Restoring snapshots can be destructive.
How to Avoid:
	• Never restore to in-use PVCs without backup.
	• Build snapshot workflows that validate PVC state.

📘 Scenario #338: Incomplete Volume Detach Breaks Node Scheduling
Category: Storage
Environment: Kubernetes v1.22, iSCSI
Summary: Scheduler skipped a healthy node due to a ghost VolumeAttachment that was never cleaned up.
What Happened: Node marked as ready, but volume controller skipped scheduling new pods due to “in-use” flag on volumes from a deleted pod.
Diagnosis Steps:
	• Described unscheduled pod — failed to bind due to volume already attached.
	• VolumeAttachment still referenced old pod.
	• CSI logs showed no detach command received.
Root Cause: CSI controller restart dropped detach request queue.
Fix/Workaround:
	• Recreated CSI controller pod.
	• Requeued detach operation via manual deletion.
Lessons Learned: CSI recovery from mid-state crash is critical.
How to Avoid:
	• Persist attach/detach queues.
	• Use cloud-level health checks for cleanup.

📘 Scenario #339: App Breaks Due to Missing SubPath After Volume Expansion
Category: Storage
Environment: Kubernetes v1.24, PVC with subPath
Summary: After PVC expansion, the mount inside pod pointed to root of volume, not the expected subPath.
What Happened: Application was configured to mount /data/subdir. After resizing, pod restarted, and subPath was ignored, mounting full volume at /data.
Diagnosis Steps:
	• Pod logs showed missing directory structure.
	• Inspected pod spec: subPath was correct.
	• CSI logs: subPath expansion failed due to permissions.
Root Cause: CSI driver did not remap subPath after resize correctly.
Fix/Workaround:
	• Changed pod to recreate the subPath explicitly.
	• Waited for bugfix release from CSI provider.
Lessons Learned: PVC expansion may break subPath unless handled explicitly.
How to Avoid:
	• Avoid complex subPath usage unless tested under all lifecycle events.
	• Watch CSI release notes carefully.

📘 Scenario #340: Backup Restore Process Created Orphaned PVCs
Category: Storage
Environment: Kubernetes v1.23, Velero
Summary: A namespace restore from backup recreated PVCs that had no matching PVs, blocking further deployment.
What Happened: Velero restored PVCs without matching spec.volumeName. Since PVs weren’t backed up, they remained Pending.
Diagnosis Steps:
	• PVC status showed Pending, with no bound PV.
	• Described PVC: no volumeName, no SC.
	• Velero logs: skipped PV restore due to config.
Root Cause: Restore policy did not include PVs.
Fix/Workaround:
	• Recreated PVCs manually with correct storage class.
	• Re-enabled PV backup in Velero settings.
Lessons Learned: Partial restores break PVC-PV binding logic.
How to Avoid:
	• Always back up PVs with PVCs in stateful applications.
	• Validate restore completeness before deployment.

📘 Scenario #341: Cross-Zone Volume Binding Fails with StatefulSet
Category: Storage
Environment: Kubernetes v1.25, AWS EBS, StatefulSet with anti-affinity
Summary: Pods in a StatefulSet failed to start due to volume binding constraints when spread across zones.
What Happened: Each pod had a PVC, but volumes couldn’t be bound because the preferred zones didn't match pod scheduling constraints.
Diagnosis Steps:
	• Pod events: failed to provision volume with StorageClass "gp2" due to zone mismatch.
	• kubectl describe pvc showed Pending.
	• StorageClass had allowedTopologies defined, conflicting with affinity rules.
Root Cause: StatefulSet pods with zone anti-affinity clashed with single-zone EBS volume provisioning.
Fix/Workaround:
	• Updated StorageClass to allow all zones.
	• Aligned affinity rules with allowed topologies.
Lessons Learned: StatefulSets and volume topology must be explicitly aligned.
How to Avoid:
	• Use multi-zone-aware volume plugins like EFS or FSx when spreading pods.

📘 Scenario #342: Volume Snapshot Controller Race Condition
Category: Storage
Environment: Kubernetes v1.23, CSI Snapshot Controller
Summary: Rapid creation/deletion of snapshots caused the controller to panic due to race conditions in snapshot finalizers.
What Happened: Automation created/deleted hundreds of snapshots per minute. The controller panicked due to concurrent finalizer modifications.
Diagnosis Steps:
	• Observed controller crash loop in logs.
	• Snapshot objects stuck in Terminating state.
	• Controller logs: resourceVersion conflict.
Root Cause: Finalizer updates not serialized under high load.
Fix/Workaround:
	• Throttled snapshot requests.
	• Patched controller deployment to limit concurrency.
Lessons Learned: High snapshot churn breaks stability.
How to Avoid:
	• Monitor snapshot queue metrics.
	• Apply rate limits in CI/CD snapshot tests.

📘 Scenario #343: Failed Volume Resize Blocks Rollout
Category: Storage
Environment: Kubernetes v1.24, CSI VolumeExpansion enabled
Summary: Deployment rollout got stuck because one of the pods couldn’t start due to a failed volume expansion.
What Happened: Admin updated PVC to request more storage. Resize failed due to volume driver limitation. New pods remained in Pending.
Diagnosis Steps:
	• PVC events: resize not supported for current volume type.
	• Pod events: volume resize pending.
Root Cause: Underlying CSI driver didn't support in-use resize.
Fix/Workaround:
	• Deleted affected pods, allowed volume to unmount.
	• Resize succeeded offline.
Lessons Learned: Not all CSI drivers handle online expansion.
How to Avoid:
	• Check CSI driver support for in-use expansion.
	• Add pre-checks before resizing PVCs.

📘 Scenario #344: Application Data Lost After Node Eviction
Category: Storage
Environment: Kubernetes v1.23, hostPath volumes
Summary: Node drained for maintenance led to permanent data loss for apps using hostPath volumes.
What Happened: Stateful workloads were evicted. When pods rescheduled on new nodes, the volume path was empty.
Diagnosis Steps:
	• Observed empty application directories post-scheduling.
	• Confirmed hostPath location was not shared across nodes.
Root Cause: hostPath volumes are node-specific and not portable.
Fix/Workaround:
	• Migrated to CSI-based dynamic provisioning.
	• Used NFS for shared storage.
Lessons Learned: hostPath is unsafe for stateful production apps.
How to Avoid:
	• Use portable CSI drivers for persistent data.
	• Restrict hostPath usage with admission controllers.

📘 Scenario #345: Read-Only PV Caused Write Failures After Restore
Category: Storage
Environment: Kubernetes v1.22, Velero, AWS EBS
Summary: After restoring from backup, the volume was attached as read-only, causing application crashes.
What Happened: Backup included PVCs and PVs, but not associated VolumeAttachment states. Restore marked volume read-only to avoid conflicts.
Diagnosis Steps:
	• Pod logs: permission denied on writes.
	• PVC events: attached in read-only mode.
	• AWS console showed volume attachment flag.
Root Cause: Velero restored volumes without resetting VolumeAttachment mode.
Fix/Workaround:
	• Detached and reattached the volume manually as read-write.
	• Updated Velero plugin to handle VolumeAttachment explicitly.
Lessons Learned: Restores need to preserve attachment metadata.
How to Avoid:
	• Validate post-restore PVC/PV attachment states.
	• Use snapshot/restore plugins that track attachment mode.

📘 Scenario #346: NFS Server Restart Crashes Pods
Category: Storage
Environment: Kubernetes v1.24, in-cluster NFS server
Summary: NFS server restarted for upgrade. All dependent pods crashed due to stale file handles and unmount errors.
What Happened: NFS mount became stale after server restart. Pods using volumes got stuck in crash loops.
Diagnosis Steps:
	• Pod logs: Stale file handle, I/O error.
	• Kernel logs showed NFS timeout.
Root Cause: NFS state is not stateless across server restarts unless configured.
Fix/Workaround:
	• Enabled NFSv4 stateless mode.
	• Recovered pods by restarting them post-reboot.
Lessons Learned: In-cluster storage servers need HA design.
How to Avoid:
	• Use managed NFS services or replicated storage.
	• Add pod liveness checks for filesystem readiness.

📘 Scenario #347: VolumeBindingBlocked Condition Causes Pod Scheduling Delay
Category: Storage
Environment: Kubernetes v1.25, dynamic provisioning
Summary: Scheduler skipped over pods with pending PVCs due to VolumeBindingBlocked status, even though volumes were eventually created.
What Happened: PVC triggered provisioning, but until PV was available, pod scheduling was deferred.
Diagnosis Steps:
	• Pod condition: PodScheduled: False, reason VolumeBindingBlocked.
	• StorageClass had delayed provisioning.
	• PVC was Pending for ~60s.
Root Cause: Volume provisioning time exceeded scheduling delay threshold.
Fix/Workaround:
	• Increased controller timeout thresholds.
	• Optimized provisioning backend latency.
Lessons Learned: Storage latency can delay workloads unexpectedly.
How to Avoid:
	• Monitor PVC creation latency in Prometheus.
	• Use pre-created PVCs for latency-sensitive apps.

📘 Scenario #348: Data Corruption from Overprovisioned Thin Volumes
Category: Storage
Environment: Kubernetes v1.22, LVM-CSI thin provisioning
Summary: Under heavy load, pods reported data corruption. Storage layer had thinly provisioned LVM volumes that overcommitted disk.
What Happened: Thin pool ran out of physical space during write bursts, leading to partial writes and corrupted files.
Diagnosis Steps:
	• Pod logs: checksum mismatches.
	• Node logs: thin pool out of space.
	• LVM command showed 100% usage.
Root Cause: Thin provisioning wasn't monitored and exceeded safe limits.
Fix/Workaround:
	• Increased physical volume backing the pool.
	• Set strict overcommit alerting.
Lessons Learned: Thin provisioning is risky under unpredictable loads.
How to Avoid:
	• Monitor usage with lvdisplay, dmsetup.
	• Avoid thin pools in production without full monitoring.

📘 Scenario #349: VolumeProvisioningFailure on GKE Due to IAM Misconfiguration
Category: Storage
Environment: GKE, Workload Identity enabled
Summary: CSI driver failed to provision new volumes due to missing IAM permissions, even though StorageClass was valid.
What Happened: GCP Persistent Disk CSI driver couldn't create disks because the service account lacked compute permissions.
Diagnosis Steps:
	• Event logs: failed to provision volume with StorageClass: permission denied.
	• IAM policy lacked compute.disks.create.
Root Cause: CSI driver operated under workload identity with incorrect bindings.
Fix/Workaround:
	• Granted missing IAM permissions to the bound service account.
	• Restarted CSI controller.
Lessons Learned: IAM and CSI need constant alignment in cloud environments.
How to Avoid:
	• Use pre-flight IAM checks during cluster provisioning.
	• Bind GKE Workload Identity properly.

📘 Scenario #350: Node Crash Triggers Volume Remount Loop
Category: Storage
Environment: Kubernetes v1.26, CSI, NVMes
Summary: After a node crash, volume remount loop occurred due to conflicting device paths.
What Happened: Volume had a static device path cached in CSI driver. Upon node recovery, OS assigned a new device path. CSI couldn't reconcile.
Diagnosis Steps:
	• CSI logs: device path not found.
	• Pod remained in ContainerCreating.
	• OS showed volume present under different path.
Root Cause: CSI assumed static device path, OS changed it post-reboot.
Fix/Workaround:
	• Added udev rules for consistent device naming.
	• Restarted CSI daemon to detect new device path.
Lessons Learned: Relying on device paths can break persistence.
How to Avoid:
	• Use device UUIDs or filesystem labels where supported.
	• Restart CSI pods post-reboot events.
```
