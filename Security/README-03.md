📘 Scenario #251: Insufficient RBAC Permissions for Cluster Admin
Category: Security
Environment: K8s v1.22, GKE
Scenario Summary: A cluster administrator was mistakenly granted insufficient RBAC permissions, preventing them from performing essential management tasks.
What Happened: A new RBAC policy was applied, which inadvertently restricted the cluster admin’s ability to manage critical components such as deployments, services, and namespaces. This caused operational issues and hindered the ability to scale or fix issues in the cluster.
Diagnosis Steps:
	• Audited the RBAC policy and identified restrictive permissions applied to the admin role.
	• Attempted various management tasks and encountered "forbidden" errors when accessing critical cluster resources.
Root Cause: Misconfiguration in the RBAC policy prevented the cluster admin from accessing necessary resources.
Fix/Workaround:
	• Updated the RBAC policy to ensure that the cluster admin role had the correct permissions to manage all resources.
	• Implemented a more granular RBAC policy review process to avoid future issues.
Lessons Learned: Always test RBAC configurations in a staging environment to avoid accidental misconfigurations.
How to Avoid:
	• Implement automated RBAC policy checks and enforce least privilege principles.
	• Regularly review and update RBAC roles to ensure they align with operational needs.

📘 Scenario #252: Insufficient Pod Security Policies Leading to Privilege Escalation
Category: Security
Environment: K8s v1.21, AWS EKS
Scenario Summary: Insufficiently restrictive PodSecurityPolicies (PSPs) allowed the deployment of privileged pods, which were later exploited by attackers.
What Happened: A cluster had PodSecurityPolicies enabled, but the policies were too permissive, allowing containers with root privileges and host network access. Attackers exploited these permissions to escalate privileges within the cluster.
Diagnosis Steps:
	• Checked the PodSecurityPolicy settings and found that they allowed privileged pods and host network access.
	• Identified compromised pods that had root access and were able to communicate freely with other sensitive resources in the cluster.
Root Cause: Misconfigured PodSecurityPolicy allowed unsafe pods to be deployed with excessive privileges.
Fix/Workaround:
	• Updated PodSecurityPolicies to enforce stricter controls, such as disallowing privileged containers and restricting host network access.
	• Applied RBAC restrictions to limit who could deploy privileged pods.
Lessons Learned: It is crucial to configure PodSecurityPolicies with the least privilege principle to prevent privilege escalation.
How to Avoid:
	• Use strict PodSecurityPolicies to enforce safe configurations for all pod deployments.
	• Regularly audit pod configurations and PodSecurityPolicy settings to ensure compliance with security standards.

📘 Scenario #253: Exposed Service Account Token in Pod
Category: Security
Environment: K8s v1.20, On-Premise
Scenario Summary: A service account token was mistakenly exposed in a pod, allowing attackers to gain unauthorized access to the Kubernetes API.
What Happened: A developer mistakenly included the service account token in a pod environment variable, making it accessible to anyone with access to the pod. The token was then exploited by attackers to gain unauthorized access to the Kubernetes API.
Diagnosis Steps:
	• Inspected the pod configuration and identified that the service account token was stored in an environment variable.
	• Monitored the API server logs and detected unauthorized API calls using the exposed token.
Root Cause: Service account token was inadvertently exposed in the pod's environment variables, allowing attackers to use it for unauthorized access.
Fix/Workaround:
	• Removed the service account token from the environment variable and stored it in a more secure location (e.g., as a Kubernetes Secret).
	• Reissued the service account token and rotated the credentials to mitigate potential risks.
Lessons Learned: Never expose sensitive credentials like service account tokens in environment variables or in pod specs.
How to Avoid:
	• Store sensitive data, such as service account tokens, in secure locations (Secrets).
	• Regularly audit pod configurations to ensure no sensitive information is exposed.

📘 Scenario #254: Rogue Container Executing Malicious Code
Category: Security
Environment: K8s v1.22, Azure AKS
Scenario Summary: A compromised container running a known exploit executed malicious code that allowed the attacker to gain access to the underlying node.
What Happened: A container running an outdated image with known vulnerabilities was exploited. The attacker used this vulnerability to gain access to the underlying node and execute malicious commands.
Diagnosis Steps:
	• Conducted a forensic investigation and found that a container was running an outdated image with an unpatched exploit.
	• Detected that the attacker used this vulnerability to escape the container and execute commands on the node.
Root Cause: Running containers with outdated or unpatched images introduced security vulnerabilities.
Fix/Workaround:
	• Updated the container images to the latest versions with security patches.
	• Implemented automatic image scanning and vulnerability scanning as part of the CI/CD pipeline to catch outdated images before deployment.
Lessons Learned: Regularly update container images and scan for vulnerabilities to reduce the attack surface.
How to Avoid:
	• Implement automated image scanning tools to identify vulnerabilities before deploying containers.
	• Enforce policies to only allow trusted and updated images to be used in production.

📘 Scenario #255: Overly Permissive Network Policies Allowing Lateral Movement
Category: Security
Environment: K8s v1.19, Google Cloud
Scenario Summary: Network policies were not restrictive enough, allowing compromised pods to move laterally across the cluster and access other services.
What Happened: The lack of restrictive network policies allowed any pod to communicate with any other pod in the cluster, even sensitive ones. After a pod was compromised, the attacker moved laterally to other pods and services, leading to further compromise.
Diagnosis Steps:
	• Reviewed the network policy configurations and found that no network isolation was enforced between pods.
	• Conducted a post-compromise analysis and found that the attacker moved across multiple services without restriction.
Root Cause: Insufficient network policies allowed unrestricted traffic between pods, increasing the potential for lateral movement.
Fix/Workaround:
	• Implemented restrictive network policies to segment the cluster and restrict traffic between pods based on specific labels and namespaces.
	• Ensured that sensitive services were isolated with network policies that only allowed access from trusted sources.
Lessons Learned: Strong network segmentation is essential to contain breaches and limit the potential for lateral movement within the cluster.
How to Avoid:
	• Implement and enforce network policies that restrict pod-to-pod communication, especially for sensitive services.
	• Regularly audit network policies and adjust them to ensure proper segmentation of workloads.

📘 Scenario #256: Insufficient Encryption for In-Transit Data
Category: Security
Environment: K8s v1.23, AWS EKS
Scenario Summary: Sensitive data was transmitted in plaintext between services, exposing it to potential eavesdropping and data breaches.
What Happened: Some internal communications between services in the cluster were not encrypted, which exposed sensitive information during transit. This could have been exploited by attackers using tools to intercept traffic.
Diagnosis Steps:
	• Analyzed service-to-service communication and discovered that some APIs were being called over HTTP rather than HTTPS.
	• Monitored network traffic and observed unencrypted data in transit.
Root Cause: Lack of encryption in communication between internal services, resulting in unprotected data being transmitted over the network.
Fix/Workaround:
	• Configured all services to communicate over HTTPS using TLS encryption.
	• Implemented mutual TLS authentication for all pod-to-pod communications within the cluster.
Lessons Learned: Never allow sensitive data to be transmitted in plaintext across the network. Always enforce encryption.
How to Avoid:
	• Use Kubernetes network policies to enforce HTTPS communication.
	• Implement and enforce mutual TLS authentication between services.

📘 Scenario #257: Exposing Cluster Services via LoadBalancer with Public IP
Category: Security
Environment: K8s v1.21, Google Cloud
Scenario Summary: A service was exposed to the public internet via a LoadBalancer without proper access control, making it vulnerable to attacks.
What Happened: A service was inadvertently exposed to the internet via an external LoadBalancer, which was not secured. Attackers were able to send requests directly to the service, attempting to exploit vulnerabilities.
Diagnosis Steps:
	• Inspected the service configuration and found that the type: LoadBalancer was used without any access restrictions.
	• Detected unauthorized attempts to interact with the service from external IPs.
Root Cause: Misconfiguration allowed the service to be exposed to the public internet without access control.
Fix/Workaround:
	• Updated the service configuration to use type: ClusterIP or added an appropriate ingress controller with restricted access.
	• Added IP whitelisting or authentication to the exposed services.
Lessons Learned: Always secure services exposed via LoadBalancer by restricting public access or using proper authentication mechanisms.
How to Avoid:
	• Use ingress controllers with proper access control lists (ACLs) to control inbound traffic.
	• Avoid exposing services unnecessarily; restrict access to only trusted IP ranges.

📘 Scenario #258: Privileged Containers Running Without Seccomp or AppArmor Profiles
Category: Security
Environment: K8s v1.20, On-Premise
Scenario Summary: Privileged containers were running without seccomp or AppArmor profiles, leaving the host vulnerable to attacks.
What Happened: Several containers were deployed with the privileged: true flag, but no seccomp or AppArmor profiles were applied. These containers had unrestricted access to the host kernel, which could lead to security breaches if exploited.
Diagnosis Steps:
	• Reviewed container configurations and identified containers running with the privileged: true flag.
	• Checked if seccomp or AppArmor profiles were applied and found that none were in place.
Root Cause: Running privileged containers without applying restrictive security profiles (e.g., seccomp, AppArmor) exposes the host to potential exploitation.
Fix/Workaround:
	• Disabled the privileged: true flag unless absolutely necessary and applied restrictive seccomp and AppArmor profiles to all privileged containers.
	• Used Kubernetes security policies to prevent the deployment of privileged containers without appropriate security profiles.
Lessons Learned: Avoid running containers with excessive privileges. Always apply security profiles to limit the scope of potential attacks.
How to Avoid:
	• Use Kubernetes PodSecurityPolicies (PSPs) or admission controllers to restrict privileged container deployments.
	• Enforce the use of seccomp and AppArmor profiles for all containers.

📘 Scenario #259: Malicious Container Image from Untrusted Source
Category: Security
Environment: K8s v1.19, Azure AKS
Scenario Summary: A malicious container image from an untrusted source was deployed, leading to a security breach in the cluster.
What Happened: A container image from an untrusted registry was pulled and deployed. The image contained malicious code, which was executed once the container started. The attacker used this to gain unauthorized access to the cluster.
Diagnosis Steps:
	• Analyzed the container image and identified malicious scripts that were executed during the container startup.
	• Detected abnormal activity in the cluster, including unauthorized API calls and data exfiltration.
Root Cause: The use of an untrusted container registry allowed the deployment of a malicious container image, which compromised the cluster.
Fix/Workaround:
	• Removed the malicious container image from the cluster and quarantined the affected pods.
	• Scanned all images for known vulnerabilities before redeploying containers.
	• Configured image admission controllers to only allow images from trusted registries.
Lessons Learned: Only use container images from trusted sources, and always scan images for vulnerabilities before deployment.
How to Avoid:
	• Use image signing and validation tools to ensure only trusted images are deployed.
	• Implement an image scanning process in the CI/CD pipeline to detect vulnerabilities and malware before deployment.

📘 Scenario #260: Unrestricted Ingress Controller Allowing External Attacks
Category: Security
Environment: K8s v1.24, GKE
Scenario Summary: The ingress controller was misconfigured, allowing external attackers to bypass network security controls and exploit internal services.
What Happened: The ingress controller was configured without proper access controls, allowing external users to directly access internal services. Attackers were able to target unprotected services within the cluster.
Diagnosis Steps:
	• Inspected the ingress configuration and found that it was accessible from any IP without authentication.
	• Observed attack attempts to access internal services that were supposed to be restricted.
Root Cause: Ingress controller misconfiguration allowed external access to internal services without proper authentication or authorization.
Fix/Workaround:
	• Reconfigured the ingress controller to restrict access to trusted IPs or users via IP whitelisting or authentication.
	• Enabled role-based access control (RBAC) to limit access to sensitive services.
Lessons Learned: Always configure ingress controllers with proper access control mechanisms to prevent unauthorized access to internal services.
How to Avoid:
	• Use authentication and authorization mechanisms with ingress controllers to protect internal services.
	• Regularly audit and update ingress configurations to ensure they align with security policies.

📘 Scenario #261: Misconfigured Ingress Controller Exposing Internal Services
Category: Security
Environment: Kubernetes v1.24, GKE
Summary: An Ingress controller was misconfigured, inadvertently exposing internal services to the public internet.
What Happened: The default configuration of the Ingress controller allowed all incoming traffic without proper authentication or IP restrictions. This oversight exposed internal services, making them accessible to unauthorized users.
Diagnosis Steps:
	• Reviewed Ingress controller configurations.
	• Identified lack of authentication mechanisms and IP whitelisting.
	• Detected unauthorized access attempts in logs.
Root Cause: Default Ingress controller settings lacked necessary security configurations.
Fix/Workaround:
	• Implemented IP whitelisting to restrict access.
	• Enabled authentication mechanisms for sensitive services.
	• Regularly audited Ingress configurations for security compliance.
Lessons Learned: Always review and harden default configurations of Ingress controllers to prevent unintended exposure.
How to Avoid:
	• Utilize security best practices when configuring Ingress controllers.
	• Regularly audit and update configurations to align with security standards.

📘 Scenario #262: Privileged Containers Without Security Context
Category: Security
Environment: Kubernetes v1.22, EKS
Summary: Containers were running with elevated privileges without defined security contexts, increasing the risk of host compromise.
What Happened: Several pods were deployed with the privileged: true flag but lacked defined security contexts. This configuration allowed containers to perform operations that could compromise the host system.
Diagnosis Steps:
	• Inspected pod specifications for security context configurations.
	• Identified containers running with elevated privileges.
	• Assessed potential risks associated with these configurations.
Root Cause: Absence of defined security contexts for privileged containers.
Fix/Workaround:
	• Defined appropriate security contexts for all containers.
	• Removed unnecessary privileged access where possible.
	• Implemented Pod Security Policies to enforce security standards.
Lessons Learned: Clearly define security contexts for all containers, especially those requiring elevated privileges.
How to Avoid:
	• Implement and enforce Pod Security Policies.
	• Regularly review and update security contexts for all deployments.

📘 Scenario #263: Unrestricted Network Policies Allowing Lateral Movement
Category: Security
Environment: Kubernetes v1.21, Azure AKS
Summary: Lack of restrictive network policies permitted lateral movement within the cluster after a pod compromise.
What Happened: An attacker compromised a pod and, due to unrestricted network policies, was able to move laterally within the cluster, accessing other pods and services.
Diagnosis Steps:
	• Reviewed network policy configurations.
	• Identified absence of restrictions between pods.
	• Traced unauthorized access patterns in network logs.
Root Cause: Inadequate network segmentation due to missing or misconfigured network policies.
Fix/Workaround:
	• Implemented network policies to restrict inter-pod communication.
	• Segmented the network based on namespaces and labels.
	• Monitored network traffic for unusual patterns.
Lessons Learned: Proper network segmentation is crucial to contain breaches and prevent lateral movement.
How to Avoid:
	• Define and enforce strict network policies.
	• Regularly audit network configurations and traffic patterns.

📘 Scenario #264: Exposed Kubernetes Dashboard Without Authentication
Category: Security
Environment: Kubernetes v1.20, On-Premise
Summary: The Kubernetes Dashboard was exposed without authentication, allowing unauthorized access to cluster resources.
What Happened: The Kubernetes Dashboard was deployed with default settings, lacking authentication mechanisms. This oversight allowed anyone with network access to interact with the dashboard and manage cluster resources.
Diagnosis Steps:
	• Accessed the dashboard without credentials.
	• Identified the ability to perform administrative actions.
	• Checked deployment configurations for authentication settings.
Root Cause: Deployment of the Kubernetes Dashboard without enabling authentication.
Fix/Workaround:
	• Enabled authentication mechanisms for the dashboard.
	• Restricted access to the dashboard using network policies.
	• Monitored dashboard access logs for unauthorized attempts.
Lessons Learned: Always secure administrative interfaces with proper authentication and access controls.
How to Avoid:
	• Implement authentication and authorization for all administrative tools.
	• Limit access to management interfaces through network restrictions.

📘 Scenario #265: Use of Vulnerable Container Images
Category: Security
Environment: Kubernetes v1.23, AWS EKS
Summary: Deployment of container images with known vulnerabilities led to potential exploitation risks.
What Happened: Applications were deployed using outdated container images that contained known vulnerabilities. These vulnerabilities could be exploited by attackers to compromise the application and potentially the cluster.
Diagnosis Steps:
	• Scanned container images for known vulnerabilities.
	• Identified outdated packages and unpatched security issues.
	• Assessed the potential impact of the identified vulnerabilities.
Root Cause: Use of outdated and vulnerable container images in deployments.
Fix/Workaround:
	• Updated container images to the latest versions with security patches.
	• Implemented automated image scanning in the CI/CD pipeline.
	• Established a policy to use only trusted and regularly updated images.
Lessons Learned: Regularly update and scan container images to mitigate security risks.
How to Avoid:
	• Integrate image scanning tools into the development workflow.
	• Maintain an inventory of approved and secure container images.

📘 Scenario #266: Misconfigured Role-Based Access Control (RBAC)
Category: Security
Environment: Kubernetes v1.22, GKE
Summary: Overly permissive RBAC configurations granted users more access than necessary, posing security risks.
What Happened: Users were assigned roles with broad permissions, allowing them to perform actions beyond their responsibilities. This misconfiguration increased the risk of accidental or malicious changes to the cluster.
Diagnosis Steps:
	• Reviewed RBAC role and role binding configurations.
	• Identified users with excessive permissions.
	• Assessed the potential impact of the granted permissions.
Root Cause: Lack of adherence to the principle of least privilege in RBAC configurations.
Fix/Workaround:
	• Revised RBAC roles to align with user responsibilities.
	• Implemented the principle of least privilege across all roles.
	• Regularly audited RBAC configurations for compliance.
Lessons Learned: Properly configured RBAC is essential to limit access and reduce security risks.
How to Avoid:
	• Define clear access requirements for each role.
	• Regularly review and update RBAC configurations.

📘 Scenario #267: Insecure Secrets Management
Category: Security
Environment: Kubernetes v1.21, On-Premise
Summary: Secrets were stored in plaintext within configuration files, leading to potential exposure.
What Happened: Sensitive information, such as API keys and passwords, was stored directly in configuration files without encryption. This practice risked exposure if the files were accessed by unauthorized individuals.
Diagnosis Steps:
	• Inspected configuration files for embedded secrets.
	• Identified plaintext storage of sensitive information.
	• Evaluated access controls on configuration files.
Root Cause: Inadequate handling and storage of sensitive information.
Fix/Workaround:
	• Migrated secrets to Kubernetes Secrets objects.
	• Implemented encryption for secrets at rest and in transit.
	• Restricted access to secrets using RBAC.
Lessons Learned: Proper secrets management is vital to protect sensitive information.
How to Avoid:
	• Use Kubernetes Secrets for managing sensitive data.
	• Implement encryption and access controls for secrets.

📘 Scenario #268: Lack of Audit Logging
Category: Security
Environment: Kubernetes v1.24, Azure AKS
Summary: Absence of audit logging hindered the ability to detect and investigate security incidents.
What Happened: A security incident occurred, but due to the lack of audit logs, it was challenging to trace the actions leading up to the incident and identify the responsible parties.
Diagnosis Steps:
	• Attempted to review audit logs for the incident timeframe.
	• Discovered that audit logging was not enabled.
	• Assessed the impact of missing audit data on the investigation.
Root Cause: Audit logging was not configured in the Kubernetes cluster.
Fix/Workaround:
	• Enabled audit logging in the cluster.
	• Configured log retention and monitoring policies.
	• Integrated audit logs with a centralized logging system for analysis.
Lessons Learned: Audit logs are essential for monitoring and investigating security events.
How to Avoid:
	• Enable and configure audit logging in all clusters.
	• Regularly review and analyze audit logs for anomalies.

📘 Scenario #269: Unrestricted Access to etcd
Category: Security
Environment: Kubernetes v1.20, On-Premise
Summary: The etcd datastore was accessible without authentication, risking exposure of sensitive cluster data.
What Happened: The etcd service was configured without authentication or encryption, allowing unauthorized users to access and modify cluster state data.
Diagnosis Steps:
	• Attempted to connect to etcd without credentials.
	• Successfully accessed sensitive cluster information.
	• Evaluated the potential impact of unauthorized access.
Root Cause: Misconfiguration of etcd lacking proper security controls.
Fix/Workaround:
	• Enabled authentication and encryption for etcd.
	• Restricted network access to etcd endpoints.
	• Regularly audited etcd configurations for security compliance.
Lessons Learned: Securing etcd is critical to protect the integrity and confidentiality of cluster data.
How to Avoid:
	• Implement authentication and encryption for etcd.
	• Limit access to etcd to authorized personnel and services.

📘 Scenario #270: Absence of Pod Security Policies
Category: Security
Environment: Kubernetes v1.23, AWS EKS
Summary: Without Pod Security Policies, pods were deployed with insecure configurations, increasing the attack surface.
What Happened: Pods were deployed without restrictions, allowing configurations such as running as root, using host networking, and mounting sensitive host paths, which posed security risks.
Diagnosis Steps:
	• Reviewed pod specifications for security configurations.
	• Identified insecure settings in multiple deployments.
	• Assessed the potential impact of these configurations.
Root Cause: Lack of enforced Pod Security Policies to govern pod configurations.
Fix/Workaround:
	• Implemented Pod Security Policies to enforce security standards.
	• Restricted the use of privileged containers and host resources.
	• Educated development teams on secure pod configurations.
Lessons Learned: Enforcing Pod Security Policies helps maintain a secure and compliant cluster environment.
How to Avoid:
	• Define and enforce Pod Security Policies.
	• Regularly review pod configurations for adherence to security standards.

📘 Scenario #271: Service Account Token Mounted in All Pods
Category: Security
Environment: Kubernetes v1.23, AKS
Summary: All pods had default service account tokens mounted, increasing the risk of credential leakage.
What Happened: Developers were unaware that service account tokens were being auto-mounted into every pod, even when not required. If any pod was compromised, its token could be misused to access the Kubernetes API.
Diagnosis Steps:
	• Inspected pod specs for automountServiceAccountToken.
	• Found all pods had tokens mounted by default.
	• Reviewed logs and discovered unnecessary API calls using those tokens.
Root Cause: The default behavior of auto-mounting tokens was not overridden.
Fix/Workaround:
	• Set automountServiceAccountToken: false in non-privileged pods.
	• Reviewed RBAC permissions to ensure tokens were scoped correctly.
Lessons Learned: Don’t give more access than necessary; disable token mounts where not needed.
How to Avoid:
	• Disable token mounting unless required.
	• Enforce security-aware pod templates across teams.

📘 Scenario #272: Sensitive Logs Exposed via Centralized Logging
Category: Security
Environment: Kubernetes v1.22, EKS with Fluentd
Summary: Secrets and passwords were accidentally logged and shipped to a centralized logging service accessible to many teams.
What Happened: Application code logged sensitive values like passwords and access keys, which were picked up by Fluentd and visible in Kibana.
Diagnosis Steps:
	• Reviewed logs after a security audit.
	• Discovered multiple log lines with secrets embedded.
	• Traced the logs back to specific applications.
Root Cause: Insecure logging practices combined with centralized aggregation.
Fix/Workaround:
	• Removed sensitive logging in app code.
	• Configured Fluentd filters to redact secrets.
	• Restricted access to sensitive log indices in Kibana.
Lessons Learned: Be mindful of what gets logged; logs can become a liability.
How to Avoid:
	• Implement logging best practices.
	• Scrub sensitive content before logs leave the app.

📘 Scenario #273: Broken Container Escape Detection
Category: Security
Environment: Kubernetes v1.24, GKE
Summary: A malicious container escaped to host level due to an unpatched kernel, but went undetected due to insufficient monitoring.
What Happened: A CVE affecting cgroups allowed container breakout. The attacker executed host-level commands and pivoted laterally across nodes.
Diagnosis Steps:
	• Investigated suspicious node-level activity.
	• Detected unexpected binaries and processes running as root.
	• Correlated with pod logs that had access to /proc.
Root Cause: Outdated host kernel + lack of runtime monitoring.
Fix/Workaround:
	• Patched all nodes to a secure kernel version.
	• Implemented Falco to monitor syscall anomalies.
Lessons Learned: Container escape is rare but possible—plan for it.
How to Avoid:
	• Patch host OS regularly.
	• Deploy tools like Falco or Sysdig for anomaly detection.

📘 Scenario #274: Unauthorized Cloud Metadata API Access
Category: Security
Environment: Kubernetes v1.22, AWS
Summary: A pod was able to access the EC2 metadata API and retrieve IAM credentials due to open network access.
What Happened: A compromised pod accessed the instance metadata service via the default route and used the credentials to access S3 and RDS.
Diagnosis Steps:
	• Analyzed cloudtrail logs for unauthorized S3 access.
	• Found requests coming from node metadata credentials.
	• Matched with pod’s activity timeline.
Root Cause: Lack of egress restrictions from pods to 169.254.169.254.
Fix/Workaround:
	• Restricted pod egress using network policies.
	• Enabled IMDSv2 with hop limit = 1 to block pod access.
Lessons Learned: Default cloud behaviors can become vulnerabilities in shared nodes.
How to Avoid:
	• Secure instance metadata access.
	• Use IRSA (IAM Roles for Service Accounts) instead of node-level credentials.

📘 Scenario #275: Admin Kubeconfig Checked into Git
Category: Security
Environment: Kubernetes v1.23, On-Prem
Summary: A developer accidentally committed a kubeconfig file with full admin access into a public Git repository.
What Happened: During a code review, a sensitive kubeconfig file was found in a GitHub repo. The credentials allowed full control over the production cluster.
Diagnosis Steps:
	• Used GitHub search to identify exposed secrets.
	• Retrieved the commit and verified credentials.
	• Checked audit logs for any misuse.
Root Cause: Lack of .gitignore and secret scanning.
Fix/Workaround:
	• Rotated the admin credentials immediately.
	• Added secret scanning to CI/CD.
	• Configured .gitignore templates across repos.
Lessons Learned: Accidental leaks happen—monitor and respond quickly.
How to Avoid:
	• Never store secrets in source code.
	• Use automated secret scanning (e.g., GitHub Advanced Security, TruffleHog).
