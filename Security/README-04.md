📘 Scenario #276: JWT Token Replay Attack in Webhook Auth
Category: Security
Environment: Kubernetes v1.21, AKS
Summary: Reused JWT tokens from intercepted API requests were used to impersonate authorized users.
What Happened: A webhook-based authentication system accepted JWTs without checking their freshness. Tokens were reused in replay attacks.
Diagnosis Steps:
	• Inspected API server logs for duplicate token use.
	• Found repeated requests with same JWT from different IPs.
	• Correlated with the webhook server not validating expiry/nonce.
Root Cause: Webhook did not validate tokens properly.
Fix/Workaround:
	• Updated webhook to validate expiry and nonce in tokens.
	• Rotated keys and invalidated sessions.
Lessons Learned: Token reuse must be considered in authentication systems.
How to Avoid:
	• Use time-limited tokens.
	• Implement replay protection with nonces or one-time tokens.

📘 Scenario #277: Container With Hardcoded SSH Keys
Category: Security
Environment: Kubernetes v1.20, On-Prem
Summary: A base image included hardcoded SSH keys which allowed attackers lateral access between environments.
What Happened: A developer reused a base image with an embedded SSH private key. This key was used across environments and eventually leaked.
Diagnosis Steps:
	• Analyzed image layers with Trivy.
	• Found hardcoded private key in /root/.ssh/id_rsa.
	• Tested and confirmed it allowed access to multiple systems.
Root Cause: Insecure base image with sensitive files included.
Fix/Workaround:
	• Rebuilt images without sensitive content.
	• Rotated all affected SSH keys.
Lessons Learned: Never embed sensitive credentials in container images.
How to Avoid:
	• Scan images before use.
	• Use multistage builds to exclude dev artifacts.

📘 Scenario #278: Insecure Helm Chart Defaults
Category: Security
Environment: Kubernetes v1.24, GKE
Summary: A popular Helm chart had insecure defaults, like exposing dashboards or running as root.
What Happened: A team installed a chart from a public Helm repo and unknowingly exposed a dashboard on the internet.
Diagnosis Steps:
	• Discovered open dashboards in a routine scan.
	• Reviewed Helm chart’s default values.
	• Found insecure values.yaml configurations.
Root Cause: Use of Helm chart without overriding insecure defaults.
Fix/Workaround:
	• Overrode defaults in values.yaml.
	• Audited Helm charts for misconfigurations.
Lessons Learned: Don’t trust defaults—validate every Helm deployment.
How to Avoid:
	• Read charts carefully before applying.
	• Maintain internal forks of public charts with hardened defaults.

📘 Scenario #279: Shared Cluster with Overlapping Namespaces
Category: Security
Environment: Kubernetes v1.22, Shared Dev Cluster
Summary: Multiple teams used the same namespace naming conventions, causing RBAC overlaps and security concerns.
What Happened: Two teams created namespaces with the same name across dev environments. RBAC rules overlapped and one team accessed another’s workloads.
Diagnosis Steps:
	• Reviewed RBAC bindings across namespaces.
	• Found conflicting roles due to reused namespace names.
	• Inspected access logs and verified misuse.
Root Cause: Lack of namespace naming policies in a shared cluster.
Fix/Workaround:
	• Introduced prefix-based namespace naming (e.g., team1-dev).
	• Scoped RBAC permissions tightly.
Lessons Learned: Namespace naming is security-sensitive in shared clusters.
How to Avoid:
	• Enforce naming policies.
	• Use automated namespace creation with templates.

📘 Scenario #280: CVE Ignored in Base Image for Months
Category: Security
Environment: Kubernetes v1.23, AWS
Summary: A known CVE affecting the base image used by multiple services remained unpatched due to no alerting.
What Happened: A vulnerability in glibc went unnoticed for months because there was no automated CVE scan or alerting. Security only discovered it during a quarterly audit.
Diagnosis Steps:
	• Scanned container image layers manually.
	• Confirmed multiple CVEs, including critical ones.
	• Traced image origin to a legacy Dockerfile.
Root Cause: No vulnerability scanning in CI/CD.
Fix/Workaround:
	• Integrated Clair + Trivy scans into CI/CD pipelines.
	• Setup Slack alerts for critical CVEs.
Lessons Learned: Continuous scanning is vital to security hygiene.
How to Avoid:
	• Integrate image scanning into build pipelines.
	• Monitor CVE databases for base images regularly.

📘 Scenario #281: Misconfigured PodSecurityPolicy Allowed Privileged Containers
Category: Security
Environment: Kubernetes v1.21, On-Prem Cluster
Summary: Pods were running with privileged: true due to a permissive PodSecurityPolicy (PSP) left enabled during testing.
What Happened: Developers accidentally left a wide-open PSP in place that allowed privileged containers, host networking, and host path mounts. This allowed a compromised container to access host files.
Diagnosis Steps:
	• Audited active PSPs.
	• Identified a PSP with overly permissive rules.
	• Found pods using privileged: true.
Root Cause: Lack of PSP review before production deployment.
Fix/Workaround:
	• Removed the insecure PSP.
	• Implemented a restrictive default PSP.
	• Migrated to PodSecurityAdmission after PSP deprecation.
Lessons Learned: Security defaults should be restrictive, not permissive.
How to Avoid:
	• Review PSP or PodSecurity configurations regularly.
	• Implement strict admission control policies.

📘 Scenario #282: GitLab Runners Spawning Privileged Containers
Category: Security
Environment: Kubernetes v1.23, GitLab CI on EKS
Summary: GitLab runners were configured to run privileged containers to support Docker-in-Docker (DinD), leading to a high-risk setup.
What Happened: A developer pipeline was hijacked and used to build malicious images, which had access to the underlying node due to privileged mode.
Diagnosis Steps:
	• Detected unusual image pushes to private registry.
	• Reviewed runner configuration – found privileged: true enabled.
	• Audited node access logs.
Root Cause: Runners configured with elevated privileges for convenience.
Fix/Workaround:
	• Disabled DinD and used Kaniko for builds.
	• Set runner securityContext to avoid privilege escalation.
Lessons Learned: Privileged mode should be a last resort.
How to Avoid:
	• Avoid using DinD where possible.
	• Use rootless build tools like Kaniko or Buildah.

📘 Scenario #283: Kubernetes Secrets Mounted in World-Readable Volumes
Category: Security
Environment: Kubernetes v1.24, GKE
Summary: Secret volumes were mounted with 0644 permissions, allowing any user process inside the container to read them.
What Happened: A poorly configured application image had other processes running that could access mounted secrets (e.g., service credentials).
Diagnosis Steps:
	• Reviewed mounted secret volumes and permissions.
	• Identified 0644 file mode on mounted files.
	• Verified multiple processes in the pod could access the secrets.
Root Cause: Secret volume default mode wasn't overridden.
Fix/Workaround:
	• Set defaultMode: 0400 on all secret volumes.
	• Isolated processes via containers.
Lessons Learned: Least privilege applies to file access too.
How to Avoid:
	• Set correct permissions on secret mounts.
	• Use multi-container pods to isolate secrets access.

📘 Scenario #284: Kubelet Port Exposed on Public Interface
Category: Security
Environment: Kubernetes v1.20, Bare Metal
Summary: Kubelet was accidentally exposed on port 10250 to the public internet, allowing unauthenticated metrics and logs access.
What Happened: Network misconfiguration led to open Kubelet ports without authentication. Attackers scraped pod logs and exploited the /exec endpoint.
Diagnosis Steps:
	• Scanned node ports using nmap.
	• Discovered open port 10250 without TLS.
	• Verified logs and metrics access externally.
Root Cause: Kubelet served insecure API without proper firewall rules.
Fix/Workaround:
	• Enabled Kubelet authentication and authorization.
	• Restricted access via firewall and node security groups.
Lessons Learned: Never expose internal components publicly.
How to Avoid:
	• Audit node ports regularly.
	• Harden Kubelet with authN/authZ and TLS.

📘 Scenario #285: Cluster Admin Bound to All Authenticated Users
Category: Security
Environment: Kubernetes v1.21, AKS
Summary: A ClusterRoleBinding accidentally granted cluster-admin to all authenticated users due to system:authenticated group.
What Happened: A misconfigured YAML granted admin access broadly, bypassing intended RBAC restrictions.
Diagnosis Steps:
	• Audited ClusterRoleBindings.
	• Found binding: subjects: kind: Group, name: system:authenticated.
	• Verified users could create/delete resources cluster-wide.
Root Cause: RBAC misconfiguration during onboarding automation.
Fix/Workaround:
	• Deleted the binding immediately.
	• Implemented an RBAC policy validation webhook.
Lessons Learned: Misuse of built-in groups can be catastrophic.
How to Avoid:
	• Avoid using broad group bindings.
	• Implement pre-commit checks for RBAC files.

📘 Scenario #286: Webhook Authentication Timing Out, Causing Denial of Service
Category: Security
Environment: Kubernetes v1.22, EKS
Summary: Authentication webhook for custom RBAC timed out under load, rejecting valid users and causing cluster-wide issues.
What Happened: Spike in API requests caused the external webhook server to time out. This led to mass access denials and degraded API server performance.
Diagnosis Steps:
	• Checked API server logs for webhook timeout messages.
	• Monitored external auth service – saw 5xx errors.
	• Replayed request load to replicate.
Root Cause: Auth webhook couldn't scale with API server traffic.
Fix/Workaround:
	• Increased webhook timeouts and horizontal scaling.
	• Added local caching for frequent identities.
Lessons Learned: External dependencies can introduce denial of service risks.
How to Avoid:
	• Stress-test webhooks.
	• Use token-based or in-cluster auth where possible.

📘 Scenario #287: CSI Driver Exposing Node Secrets
Category: Security
Environment: Kubernetes v1.24, CSI Plugin (AWS Secrets Store)
Summary: Misconfigured CSI driver exposed secrets on hostPath mount accessible to privileged pods.
What Happened: Secrets mounted via the CSI driver were not isolated properly, allowing another pod with hostPath access to read them.
Diagnosis Steps:
	• Reviewed CSI driver logs and configurations.
	• Found secrets mounted in shared path (/var/lib/...).
	• Identified privilege escalation path via hostPath.
Root Cause: CSI driver exposed secrets globally on node filesystem.
Fix/Workaround:
	• Scoped CSI mounts with per-pod directories.
	• Disabled hostPath access for workloads.
Lessons Learned: CSI drivers must be hardened like apps.
How to Avoid:
	• Test CSI secrets exposure under threat models.
	• Restrict node-level file access via policies.

📘 Scenario #288: EphemeralContainers Used for Reconnaissance
Category: Security
Environment: Kubernetes v1.25, GKE
Summary: A compromised user deployed ephemeral containers to inspect and copy secrets from running pods.
What Happened: A user with access to ephemeralcontainers feature spun up containers in critical pods and read mounted secrets and env vars.
Diagnosis Steps:
	• Audited API server calls to ephemeralcontainers API.
	• Found suspicious container launches.
	• Inspected shell history and accessed secrets.
Root Cause: Overprivileged user with ephemeralcontainers access.
Fix/Workaround:
	• Removed permissions to ephemeral containers for all roles.
	• Set audit policies for their use.
Lessons Learned: New features introduce new attack vectors.
How to Avoid:
	• Lock down access to new APIs.
	• Monitor audit logs for container injection attempts.

📘 Scenario #289: hostAliases Used for Spoofing Internal Services
Category: Security
Environment: Kubernetes v1.22, On-Prem
Summary: Malicious pod used hostAliases to spoof internal service hostnames and intercept requests.
What Happened: An insider attack modified /etc/hosts in a pod using hostAliases to redirect requests to attacker-controlled services.
Diagnosis Steps:
	• Reviewed pod manifests with hostAliases.
	• Captured outbound DNS traffic and traced redirections.
	• Detected communication with rogue internal services.
Root Cause: Abuse of hostAliases field in PodSpec.
Fix/Workaround:
	• Disabled use of hostAliases via OPA policies.
	• Logged all pod specs with custom host entries.
Lessons Learned: Host file spoofing can bypass DNS-based security.
How to Avoid:
	• Restrict or disallow use of hostAliases.
	• Rely on service discovery via DNS only.

📘 Scenario #290: Privilege Escalation via Unchecked securityContext in Helm Chart
Category: Security
Environment: Kubernetes v1.21, Helm v3.8
Summary: A third-party Helm chart allowed setting arbitrary securityContext, letting users run pods as root in production.
What Happened: A chart exposed securityContext overrides without constraints. A developer added runAsUser: 0during deployment, leading to root-level containers.
Diagnosis Steps:
	• Inspected Helm chart values and rendered manifests.
	• Detected containers with runAsUser: 0.
	• Reviewed change logs in GitOps pipeline.
Root Cause: Chart did not validate or restrict securityContext fields.
Fix/Workaround:
	• Forked chart and restricted overrides via schema.
	• Implemented OPA Gatekeeper to block root containers.
Lessons Learned: Helm charts can be as dangerous as code.
How to Avoid:
	• Validate all chart values.
	• Use policy engines to restrict risky configurations.

📘 Scenario #291: Service Account Token Leakage via Logs
Category: Security
Environment: Kubernetes v1.23, AKS
Summary: Application inadvertently logged its mounted service account token, exposing it to log aggregation systems.
What Happened: A misconfigured logging library dumped all environment variables and mounted file contents at startup, including the token from /var/run/secrets/kubernetes.io/serviceaccount/token.
Diagnosis Steps:
	• Searched central logs for token patterns.
	• Confirmed multiple logs contained valid JWTs.
	• Validated token usage in audit logs.
Root Cause: Poor logging hygiene in application code.
Fix/Workaround:
	• Rotated all impacted service account tokens.
	• Added environment and file sanitization to logging library.
Lessons Learned: Tokens are sensitive credentials and should never be logged.
How to Avoid:
	• Add a startup check to prevent token exposure.
	• Use static analysis or OPA to block risky mounts/logs.

📘 Scenario #292: Escalation via Editable Validating WebhookConfiguration
Category: Security
Environment: Kubernetes v1.24, EKS
Summary: User with edit rights on a validating webhook modified it to bypass critical security policies.
What Happened: An internal user reconfigured the webhook to always return allow, disabling cluster-wide security checks.
Diagnosis Steps:
	• Detected anomaly: privileged pods getting deployed.
	• Checked webhook configuration history in GitOps.
	• Verified that failurePolicy: Ignore and static allow logic were added.
Root Cause: Lack of control over webhook configuration permissions.
Fix/Workaround:
	• Restricted access to ValidatingWebhookConfiguration objects.
	• Added checksums to webhook definitions in GitOps.
Lessons Learned: Webhooks must be tightly controlled to preserve cluster security.
How to Avoid:
	• Lock down RBAC access to webhook configurations.
	• Monitor changes with alerts and diff checks.

📘 Scenario #293: Stale Node Certificates After Rejoining Cluster
Category: Security
Environment: Kubernetes v1.21, Kubeadm-based cluster
Summary: A node was rejoined to the cluster using a stale certificate, giving it access it shouldn't have.
What Happened: A node that was previously removed was added back using an old /var/lib/kubelet/pki/kubelet-client.crt, which was still valid.
Diagnosis Steps:
	• Compared certificate expiry and usage.
	• Found stale kubelet cert on rejoined node.
	• Verified node had been deleted previously.
Root Cause: Old credentials not purged before node rejoin.
Fix/Workaround:
	• Manually deleted old certificates from the node.
	• Set short TTLs for client certificates.
Lessons Learned: Node certs should be one-time-use and short-lived.
How to Avoid:
	• Rotate node credentials regularly.
	• Use automation to purge sensitive files before rejoining.

📘 Scenario #294: ArgoCD Exploit via Unverified Helm Charts
Category: Security
Environment: Kubernetes v1.24, ArgoCD
Summary: ArgoCD deployed a malicious Helm chart that added privileged pods and container escape backdoors.
What Happened: A team added a new Helm repo that wasn’t verified. The chart had post-install hooks that ran containers with host access.
Diagnosis Steps:
	• Found unusual pods using hostNetwork and hostPID.
	• Traced deployment to ArgoCD application with external chart.
	• Inspected chart source – found embedded malicious hooks.
Root Cause: Lack of chart verification or provenance checks.
Fix/Workaround:
	• Removed the chart and all related workloads.
	• Enabled Helm OCI signatures and repo allow-lists.
Lessons Learned: Supply chain security is critical, even with GitOps.
How to Avoid:
	• Only use verified or internal Helm repos.
	• Enable ArgoCD Helm signature verification.

📘 Scenario #295: Node Compromise via Insecure Container Runtime
Category: Security
Environment: Kubernetes v1.22, CRI-O on Bare Metal
Summary: A CVE in the container runtime allowed a container breakout, leading to full node compromise.
What Happened: An attacker exploited CRI-O vulnerability (CVE-2022-0811) that allowed containers to overwrite host paths via sysctl injection.
Diagnosis Steps:
	• Detected abnormal node CPU spike and external traffic.
	• Inspected containers – found sysctl modifications.
	• Cross-verified with known CVEs.
Root Cause: Unpatched CRI-O vulnerability and default seccomp profile disabled.
Fix/Workaround:
	• Upgraded CRI-O to patched version.
	• Enabled seccomp and AppArmor by default.
Lessons Learned: Container runtimes must be hardened and patched like any system component.
How to Avoid:
	• Automate CVE scanning for runtime components.
	• Harden runtimes with security profiles.

📘 Scenario #296: Workload with Wildcard RBAC Access to All Secrets
Category: Security
Environment: Kubernetes v1.23, Self-Hosted
Summary: A microservice was granted get and list access to all secrets cluster-wide using *.
What Happened: Developers gave overly broad access to a namespace-wide controller, leading to accidental exposure of unrelated team secrets.
Diagnosis Steps:
	• Audited RBAC for secrets access.
	• Found RoleBinding with resources: [“secrets”], verbs: [“get”, “list”], resourceNames: ["*"].
Root Cause: Overly broad RBAC permissions in service manifest.
Fix/Workaround:
	• Replaced wildcard permissions with explicit named secrets.
	• Enabled audit logging on all secrets API calls.
Lessons Learned: * in RBAC is often overkill and dangerous.
How to Avoid:
	• Use least privilege principle.
	• Validate RBAC via CI/CD linting tools.

📘 Scenario #297: Malicious Init Container Used for Reconnaissance
Category: Security
Environment: Kubernetes v1.25, GKE
Summary: A pod was launched with a benign main container and a malicious init container that copied node metadata.
What Happened: Init container wrote node files (e.g., /etc/resolv.conf, cloud instance metadata) to an external bucket before terminating.
Diagnosis Steps:
	• Enabled audit logs for object storage.
	• Traced writes back to a pod with suspicious init container.
	• Reviewed init container image – found embedded exfil logic.
Root Cause: Lack of validation on init container behavior.
Fix/Workaround:
	• Blocked unknown container registries via policy.
	• Implemented runtime security agents to inspect init behavior.
Lessons Learned: Init containers must be treated as full-fledged security risks.
How to Avoid:
	• Verify init container images and registries.
	• Use runtime tools (e.g., Falco) for behavior analysis.

📘 Scenario #298: Ingress Controller Exposed /metrics Without Auth
Category: Security
Environment: Kubernetes v1.24, NGINX Ingress
Summary: Prometheus scraping endpoint /metrics was exposed without authentication and revealed sensitive internal details.
What Happened: A misconfigured ingress rule allowed external users to access /metrics, which included upstream paths, response codes, and error logs.
Diagnosis Steps:
	• Scanned public URLs.
	• Found /metrics exposed to unauthenticated traffic.
	• Inspected NGINX ingress annotations.
Root Cause: Ingress annotations missing auth and whitelist rules.
Fix/Workaround:
	• Applied IP whitelist and basic auth for /metrics.
	• Added network policies to restrict access.
Lessons Learned: Even observability endpoints need protection.
How to Avoid:
	• Enforce auth for all public endpoints.
	• Separate internal vs. external monitoring targets.

📘 Scenario #299: Secret Stored in ConfigMap by Mistake
Category: Security
Environment: Kubernetes v1.23, AKS
Summary: A sensitive API key was accidentally stored in a ConfigMap instead of a Secret, making it visible in plain text.
What Happened: Developer used a ConfigMap for application config, and mistakenly included an apiKey in it. Anyone with view rights could read it.
Diagnosis Steps:
	• Reviewed config files for plaintext secrets.
	• Found hardcoded credentials in ConfigMap YAML.
Root Cause: Misunderstanding of Secret vs. ConfigMap usage.
Fix/Workaround:
	• Moved key to a Kubernetes Secret.
	• Rotated exposed credentials.
Lessons Learned: Educate developers on proper resource usage.
How to Avoid:
	• Lint manifests to block secrets in ConfigMaps.
	• Train developers in security best practices.

📘 Scenario #300: Token Reuse After Namespace Deletion and Recreation
Category: Security
Environment: Kubernetes v1.24, Self-Hosted
Summary: A previously deleted namespace was recreated, and old tokens (from backups) were still valid and worked.
What Happened: Developer restored a backup including secrets from a deleted namespace. The token was still valid and allowed access to cluster resources.
Diagnosis Steps:
	• Found access via old token in logs.
	• Verified namespace was deleted, then recreated with same name.
	• Checked secrets in restored backup.
Root Cause: Static tokens persisted after deletion and recreation.
Fix/Workaround:
	• Rotated all tokens after backup restore.
	• Implemented TTL-based token policies.
Lessons Learned: Tokens must be invalidated after deletion or restore.
How to Avoid:
	• Don’t restore old secrets blindly.
	• Rotate and re-issue credentials post-restore.
