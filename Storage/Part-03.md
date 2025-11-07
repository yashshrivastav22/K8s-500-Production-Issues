```
📘 Scenario #351: VolumeMount Conflict Between Init and Main Containers
Category: Storage
Environment: Kubernetes v1.25, containerized database restore job
Summary: Init container and main container used the same volume path but with different modes, causing the main container to crash.
What Happened: An init container wrote a backup file to a shared volume. The main container expected a clean mount, found conflicting content, and failed on startup.
Diagnosis Steps:
	• Pod logs showed file already exists error.
	• Examined pod manifest: both containers used the same volumeMount.path.
Root Cause: Shared volume path caused file conflicts between lifecycle stages.
Fix/Workaround:
	• Used a subPath for the init container to isolate file writes.
	• Moved backup logic to an external init job.
Lessons Learned: Volume sharing across containers must be carefully scoped.
How to Avoid:
	• Always use subPath if write behavior differs.
	• Isolate volume use per container stage when possible.

📘 Scenario #352: PVCs Stuck in “Terminating” Due to Finalizers
Category: Storage
Environment: Kubernetes v1.24, CSI driver with finalizer
Summary: After deleting PVCs, they remained in Terminating state indefinitely due to stuck finalizers.
What Happened: The CSI driver responsible for finalizer cleanup was crash-looping, preventing PVC finalizer execution.
Diagnosis Steps:
	• PVCs had finalizer external-attacher.csi.driver.io.
	• CSI pod logs showed repeated panics due to malformed config.
Root Cause: Driver bug prevented cleanup logic, blocking PVC deletion.
Fix/Workaround:
	• Patched the driver deployment.
	• Manually removed finalizers using kubectl patch.
Lessons Learned: CSI finalizer bugs can block resource lifecycle.
How to Avoid:
	• Regularly update CSI drivers.
	• Monitor PVC lifecycle duration metrics.

📘 Scenario #353: Misconfigured ReadOnlyMany Mount Blocks Write Operations
Category: Storage
Environment: Kubernetes v1.23, NFS volume
Summary: Volume mounted as ReadOnlyMany blocked necessary write operations, despite NFS server allowing writes.
What Happened: VolumeMount was incorrectly marked as readOnly: true. Application failed on write attempts.
Diagnosis Steps:
	• Application logs: read-only filesystem.
	• Pod manifest showed readOnly: true.
Root Cause: Misconfiguration in the volumeMounts spec.
Fix/Workaround:
	• Updated the manifest to readOnly: false.
Lessons Learned: Read-only flags silently break expected behavior.
How to Avoid:
	• Validate volume mount flags in CI.
	• Use initContainer to test mount behavior.

📘 Scenario #354: In-Tree Plugin PVs Lost After Driver Migration
Category: Storage
Environment: Kubernetes v1.26, in-tree to CSI migration
Summary: Existing in-tree volumes became unrecognized after enabling CSI migration.
What Happened: Migrated GCE volumes to CSI plugin. Old PVs had legacy annotations and didn’t bind correctly.
Diagnosis Steps:
	• PVs showed Unavailable state.
	• Migration feature gates enabled but missing annotations.
Root Cause: Backward incompatibility in migration logic for pre-existing PVs.
Fix/Workaround:
	• Manually edited PV annotations to match CSI requirements.
Lessons Learned: Migration feature gates must be tested in staging.
How to Avoid:
	• Run migration with shadow mode first.
	• Migrate PVs gradually using tools like pv-migrate.

📘 Scenario #355: Pod Deleted but Volume Still Mounted on Node
Category: Storage
Environment: Kubernetes v1.24, CSI
Summary: Pod was force-deleted, but its volume wasn’t unmounted from the node, blocking future pod scheduling.
What Happened: Force deletion bypassed CSI driver cleanup. Mount lingered and failed future pod volume attach.
Diagnosis Steps:
	• kubectl describe node showed volume still attached.
	• lsblk confirmed mount on node.
	• Logs showed attach errors.
Root Cause: Orphaned mount due to force deletion.
Fix/Workaround:
	• Manually unmounted the volume on node.
	• Drained and rebooted the node.
Lessons Learned: Forced pod deletions should be last resort.
How to Avoid:
	• Set up automated orphaned mount detection scripts.
	• Use graceful deletion with finalizer handling.

📘 Scenario #356: Ceph RBD Volume Crashes Pods Under IOPS Saturation
Category: Storage
Environment: Kubernetes v1.23, Ceph CSI
Summary: Under heavy I/O, Ceph volumes became unresponsive, leading to kernel-level I/O errors in pods.
What Happened: Application workload created sustained random writes. Ceph cluster’s IOPS limit was reached.
Diagnosis Steps:
	• dmesg logs: blk_update_request: I/O error.
	• Pod logs: database fsync errors.
	• Ceph health: HEALTH_WARN: slow ops.
Root Cause: Ceph RBD pool under-provisioned for the workload.
Fix/Workaround:
	• Migrated to SSD-backed Ceph pools.
	• Throttled application concurrency.
Lessons Learned: Distributed storage systems fail silently under stress.
How to Avoid:
	• Benchmark storage before rollout.
	• Alert on high RBD latency.

📘 Scenario #357: ReplicaSet Using PVCs Fails Due to VolumeClaimTemplate Misuse
Category: Storage
Environment: Kubernetes v1.25
Summary: Developer tried using volumeClaimTemplates in a ReplicaSet manifest, which isn’t supported.
What Happened: Deployment applied, but pods failed to create PVCs.
Diagnosis Steps:
	• Controller logs: volumeClaimTemplates is not supported in ReplicaSet.
	• No PVCs appeared in kubectl get pvc.
Root Cause: volumeClaimTemplates is only supported in StatefulSet.
Fix/Workaround:
	• Refactored ReplicaSet to StatefulSet.
Lessons Learned: Not all workload types support dynamic PVCs.
How to Avoid:
	• Use workload reference charts during manifest authoring.
	• Validate manifests with policy engines like OPA.

📘 Scenario #358: Filesystem Type Mismatch During Volume Attach
Category: Storage
Environment: Kubernetes v1.24, ext4 vs xfs
Summary: A pod failed to start because the PV expected ext4 but the node formatted it as xfs.
What Happened: Pre-provisioned disk had xfs, but StorageClass defaulted to ext4.
Diagnosis Steps:
	• Attach logs: mount failed: wrong fs type.
	• blkid on node showed xfs.
Root Cause: Filesystem mismatch between PV and node assumptions.
Fix/Workaround:
	• Reformatted disk to ext4.
	• Aligned StorageClass with PV fsType.
Lessons Learned: Filesystem types must match across the stack.
How to Avoid:
	• Explicitly set fsType in StorageClass.
	• Document provisioner formatting logic.

📘 Scenario #359: iSCSI Volumes Fail After Node Kernel Upgrade
Category: Storage
Environment: Kubernetes v1.26, CSI iSCSI plugin
Summary: Post-upgrade, all pods using iSCSI volumes failed to mount due to kernel module incompatibility.
What Happened: Kernel upgrade removed or broke iscsi_tcp module needed by CSI driver.
Diagnosis Steps:
	• CSI logs: no such device iscsi_tcp.
	• modprobe iscsi_tcp failed.
	• Pod events: mount timeout.
Root Cause: Node image didn’t include required kernel modules post-upgrade.
Fix/Workaround:
	• Installed open-iscsi and related modules.
	• Rebooted node.
Lessons Learned: OS updates can break CSI compatibility.
How to Avoid:
	• Pin node kernel versions.
	• Run upgrade simulations in canary clusters.

📘 Scenario #360: PVs Not Deleted After PVC Cleanup Due to Retain Policy
Category: Storage
Environment: Kubernetes v1.23, AWS EBS
Summary: After PVCs were deleted, underlying PVs and disks remained, leading to cloud resource sprawl.
What Happened: Retain policy on the PV preserved the disk after PVC was deleted.
Diagnosis Steps:
	• kubectl get pv showed status Released.
	• Disk still visible in AWS console.
Root Cause: PV reclaimPolicy was Retain, not Delete.
Fix/Workaround:
	• Manually deleted PVs and EBS volumes.
Lessons Learned: Retain policy needs operational follow-up.
How to Avoid:
	• Use Delete policy unless manual cleanup is required.
	• Audit dangling PVs regularly.

📘 Scenario #361: Concurrent Pod Scheduling on the Same PVC Causes Mount Conflict
Category: Storage
Environment: Kubernetes v1.24, AWS EBS, ReadWriteOnce PVC
Summary: Two pods attempted to use the same PVC simultaneously, causing one pod to be stuck in ContainerCreating.
What Happened: A deployment scale-up triggered duplicate pods trying to mount the same EBS volume on different nodes.
Diagnosis Steps:
	• One pod was running, the other stuck in ContainerCreating.
	• Events showed Volume is already attached to another node.
Root Cause: EBS supports ReadWriteOnce, not multi-node attach.
Fix/Workaround:
	• Added anti-affinity to restrict pod scheduling to a single node.
	• Used EFS (ReadWriteMany) for workloads needing shared storage.
Lessons Learned: Not all storage supports multi-node access.
How to Avoid:
	• Understand volume access modes.
	• Use StatefulSets or anti-affinity for PVC sharing.

📘 Scenario #362: StatefulSet Pod Replacement Fails Due to PVC Retention
Category: Storage
Environment: Kubernetes v1.23, StatefulSet with volumeClaimTemplates
Summary: Deleted a StatefulSet pod manually, but new pod failed due to existing PVC conflict.
What Happened: PVC persisted after pod deletion due to StatefulSet retention policy.
Diagnosis Steps:
	• kubectl get pvc showed PVC still bound.
	• New pod stuck in Pending.
Root Cause: StatefulSet retains PVCs unless explicitly deleted.
Fix/Workaround:
	• Deleted old PVC manually to let StatefulSet recreate it.
Lessons Learned: Stateful PVCs are tightly coupled to pod identity.
How to Avoid:
	• Use persistentVolumeReclaimPolicy: Delete only when data can be lost.
	• Automate cleanup for failed StatefulSet replacements.

📘 Scenario #363: HostPath Volume Access Leaks Host Data into Container
Category: Storage
Environment: Kubernetes v1.22, single-node dev cluster
Summary: HostPath volume mounted the wrong directory, exposing sensitive host data to the container.
What Happened: Misconfigured path / instead of /data allowed container full read access to host.
Diagnosis Steps:
	• Container listed host files under /mnt/host.
	• Pod manifest showed path: /.
Root Cause: Typo in the volume path.
Fix/Workaround:
	• Corrected volume path in manifest.
	• Revoked pod access.
Lessons Learned: HostPath has minimal safety nets.
How to Avoid:
	• Avoid using HostPath unless absolutely necessary.
	• Validate mount paths through automated policies.

📘 Scenario #364: CSI Driver Crashes When Node Resource Is Deleted Prematurely
Category: Storage
Environment: Kubernetes v1.25, custom CSI driver
Summary: Deleting a node object before the CSI driver detached volumes caused crash loops.
What Happened: Admin manually deleted a node before volume detach completed.
Diagnosis Steps:
	• CSI logs showed panic due to missing node metadata.
	• Pods remained in Terminating.
Root Cause: Driver attempted to clean up mounts from a non-existent node resource.
Fix/Workaround:
	• Waited for CSI driver to timeout and self-recover.
	• Rebooted node to forcibly detach volumes.
Lessons Learned: Node deletion should follow strict lifecycle policies.
How to Avoid:
	• Use node cordon + drain before deletion.
	• Monitor CSI cleanup completion before proceeding.

📘 Scenario #365: Retained PV Blocks New Claim Binding with Identical Name
Category: Storage
Environment: Kubernetes v1.21, NFS
Summary: A PV stuck in Released state with Retain policy blocked new PVCs from binding with the same name.
What Happened: Deleted old PVC and recreated a new one with the same name, but it stayed Pending.
Diagnosis Steps:
	• PV was in Released, PVC was Pending.
	• Events: PVC is not bound.
Root Cause: Retained PV still owned the identity, blocking rebinding.
Fix/Workaround:
	• Manually deleted the old PV to allow dynamic provisioning.
Lessons Learned: Retain policies require admin cleanup.
How to Avoid:
	• Use Delete policy for short-lived PVCs.
	• Automate orphan PV audits.

📘 Scenario #366: CSI Plugin Panic on Missing Mount Option
Category: Storage
Environment: Kubernetes v1.26, custom CSI plugin
Summary: Missing mountOptions in StorageClass led to runtime nil pointer exception in CSI driver.
What Happened: StorageClass defined mountOptions: null, causing driver to crash during attach.
Diagnosis Steps:
	• CSI logs showed panic: nil pointer dereference.
	• StorageClass YAML had an empty mountOptions: field.
Root Cause: Plugin didn't check for nil before reading options.
Fix/Workaround:
	• Removed mountOptions: from manifest.
	• Patched CSI driver to add nil checks.
Lessons Learned: CSI drivers must gracefully handle incomplete specs.
How to Avoid:
	• Validate StorageClass manifests.
	• Write defensive CSI plugin code.

📘 Scenario #367: Pod Fails to Mount Volume Due to SELinux Context Mismatch
Category: Storage
Environment: Kubernetes v1.24, RHEL with SELinux enforcing
Summary: Pod failed to mount volume due to denied SELinux permissions.
What Happened: Volume was created with an incorrect SELinux context, preventing pod access.
Diagnosis Steps:
	• Pod logs: permission denied.
	• dmesg showed SELinux AVC denial.
Root Cause: Volume not labeled with container_file_t.
Fix/Workaround:
	• Relabeled volume with chcon -Rt container_file_t /data.
Lessons Learned: SELinux can silently block mounts.
How to Avoid:
	• Use CSI drivers that support SELinux integration.
	• Validate volume contexts post-provisioning.

📘 Scenario #368: VolumeExpansion on Bound PVC Fails Due to Pod Running
Category: Storage
Environment: Kubernetes v1.25, GCP PD
Summary: PVC resize operation failed because the pod using it was still running.
What Happened: Tried to resize a PVC while its pod was active.
Diagnosis Steps:
	• PVC showed Resizing then back to Bound.
	• Events: PVC resize failed while volume in use.
Root Cause: Filesystem resize required pod to restart.
Fix/Workaround:
	• Deleted pod to trigger offline volume resize.
	• PVC then showed FileSystemResizePending → Bound.
Lessons Learned: Some resizes need pod restart.
How to Avoid:
	• Plan PVC expansion during maintenance.
	• Use fsResizePolicy: "OnRestart" if supported.

📘 Scenario #369: CSI Driver Memory Leak on Volume Detach Loop
Category: Storage
Environment: Kubernetes v1.24, external CSI
Summary: CSI plugin leaked memory due to improper garbage collection on detach failure loop.
What Happened: Detach failed repeatedly due to stale metadata, causing plugin to grow in memory use.
Diagnosis Steps:
	• Plugin memory exceeded 1GB.
	• Logs showed repeated detach failed with no backoff.
Root Cause: Driver retry loop without cleanup or GC.
Fix/Workaround:
	• Restarted CSI plugin.
	• Patched driver to implement exponential backoff.
Lessons Learned: CSI error paths need memory safety.
How to Avoid:
	• Stress-test CSI paths for failure.
	• Add Prometheus memory alerts for plugins.

📘 Scenario #370: Volume Mount Timeout Due to Slow Cloud API
Category: Storage
Environment: Kubernetes v1.23, Azure Disk CSI
Summary: During a cloud outage, Azure Disk operations timed out, blocking pod mounts.
What Happened: Pods remained in ContainerCreating due to delayed volume attachment.
Diagnosis Steps:
	• Event logs: timed out waiting for attach.
	• Azure portal showed degraded disk API service.
Root Cause: Cloud provider API latency blocked CSI attach.
Fix/Workaround:
	• Waited for Azure API to stabilize.
	• Used local PVs for critical workloads moving forward.
Lessons Learned: Cloud API reliability is a hidden dependency.
How to Avoid:
	• Use local volumes or ephemeral storage for high-availability needs.
	• Monitor CSI attach/detach durations.

📘 Scenario #371: Volume Snapshot Restore Misses Application Consistency
Category: Storage
Environment: Kubernetes v1.26, Velero with CSI VolumeSnapshot
Summary: Snapshot restore completed successfully, but restored app data was corrupt.
What Happened: A volume snapshot was taken while the database was mid-write. Restore completed, but database wouldn't start due to file inconsistencies.
Diagnosis Steps:
	• Restored volume had missing WAL files.
	• Database logs showed corruption errors.
	• Snapshot logs showed no pre-freeze hook execution.
Root Cause: No coordination between snapshot and application quiescence.
Fix/Workaround:
	• Integrated pre-freeze and post-thaw hooks via Velero Restic.
	• Enabled application-aware backups.
Lessons Learned: Volume snapshot ≠ app-consistent backup.
How to Avoid:
	• Use app-specific backup tools or hooks.
	• Never snapshot during heavy write activity.

📘 Scenario #372: File Locking Issue Between Multiple Pods on NFS
Category: Storage
Environment: Kubernetes v1.22, NFS with ReadWriteMany
Summary: Two pods wrote to the same file concurrently, causing lock conflicts and data loss.
What Happened: Lack of advisory file locking on the NFS server led to race conditions between pods.
Diagnosis Steps:
	• Log files had overlapping, corrupted data.
	• File locks were not honored.
Root Cause: POSIX locks not enforced reliably over NFS.
Fix/Workaround:
	• Introduced flock-based locking in application code.
	• Used local persistent volume instead for critical data.
Lessons Learned: NFS doesn’t guarantee strong file locking semantics.
How to Avoid:
	• Architect apps to handle distributed file access carefully.
	• Avoid shared writable files unless absolutely needed.

📘 Scenario #373: Pod Reboots Erase Data on EmptyDir Volume
Category: Storage
Environment: Kubernetes v1.24, default EmptyDir
Summary: Pod restarts caused in-memory volume to be wiped, resulting in lost logs.
What Happened: Logging container used EmptyDir with memory medium. Node rebooted, and logs were lost.
Diagnosis Steps:
	• Post-reboot, EmptyDir was reinitialized.
	• Logs had disappeared from the container volume.
Root Cause: EmptyDir with medium: Memory is ephemeral and tied to node lifecycle.
Fix/Workaround:
	• Switched to hostPath for logs or persisted to object storage.
Lessons Learned: Understand EmptyDir behavior before using for critical data.
How to Avoid:
	• Use PVs or centralized logging for durability.
	• Avoid medium: Memory unless necessary.

📘 Scenario #374: PVC Resize Fails on In-Use Block Device
Category: Storage
Environment: Kubernetes v1.25, CSI with block mode
Summary: PVC expansion failed for a block device while pod was still running.
What Happened: Attempted to resize a raw block volume without terminating the consuming pod.
Diagnosis Steps:
	• PVC stuck in Resizing.
	• Logs: device busy.
Root Cause: Some storage providers require offline resizing for block devices.
Fix/Workaround:
	• Stopped the pod and retried resize.
Lessons Learned: Raw block volumes behave differently than filesystem PVCs.
How to Avoid:
	• Schedule maintenance windows for volume changes.
	• Know volume mode differences.

📘 Scenario #375: Default StorageClass Prevents PVC Binding to Custom Class
Category: Storage
Environment: Kubernetes v1.23, GKE
Summary: A PVC remained in Pending because the default StorageClass kept getting assigned instead of a custom one.
What Happened: PVC YAML didn’t specify storageClassName, so the default one was used.
Diagnosis Steps:
	• PVC described with wrong StorageClass.
	• Events: no matching PV.
Root Cause: Default StorageClass mismatch with intended PV type.
Fix/Workaround:
	• Explicitly set storageClassName in the PVC.
Lessons Learned: Implicit defaults can cause hidden behavior.
How to Avoid:
	• Always specify StorageClass explicitly in manifests.
	• Audit your cluster’s default classes.
```
