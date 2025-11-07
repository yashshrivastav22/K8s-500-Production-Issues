```
📘 Scenario #201: Unauthorized Access to Secrets Due to Incorrect RBAC Permissions
Category: Security
Environment: K8s v1.22, GKE
Scenario Summary: Unauthorized users were able to access Kubernetes secrets due to overly permissive RBAC roles.
What Happened: A service account was granted cluster-admin permissions, which allowed users to access sensitive secrets via kubectl. This led to a security breach when one of the users exploited the permissions.
Diagnosis Steps:
	• Inspected RBAC roles with kubectl get roles and kubectl get clusterroles to identify misconfigured roles.
	• Checked logs and found that sensitive secrets were accessed using a service account that shouldn't have had access.
Root Cause: The service account was granted excessive permissions via RBAC roles.
Fix/Workaround:
	• Reconfigured RBAC roles to adhere to the principle of least privilege.
	• Limited the permissions of the service account and tested access controls.
Lessons Learned: Always follow the principle of least privilege when configuring RBAC for service accounts and users.
How to Avoid:
	• Regularly audit RBAC roles and service account permissions.
	• Implement role-based access control (RBAC) with tight restrictions on who can access secrets.

📘 Scenario #202: Insecure Network Policies Leading to Pod Exposure
Category: Security
Environment: K8s v1.19, AWS EKS
Scenario Summary: Pods intended to be isolated were exposed to unauthorized traffic due to misconfigured network policies.
What Happened: A network policy was meant to block communication between pods in different namespaces, but it was misconfigured, allowing unauthorized access between pods.
Diagnosis Steps:
	• Used kubectl get networkpolicy to check existing network policies.
	• Observed that the network policy’s podSelector was incorrectly configured, allowing access between pods from different namespaces.
Root Cause: Misconfigured NetworkPolicy selectors allowed unwanted access between pods.
Fix/Workaround:
	• Corrected the NetworkPolicy by refining podSelector and applying stricter isolation.
	• Tested the updated policy to confirm proper isolation between namespaces.
Lessons Learned: Network policies must be carefully crafted to prevent unauthorized access between pods.
How to Avoid:
	• Implement and test network policies in a staging environment before applying to production.
	• Regularly audit network policies to ensure they align with security requirements.

📘 Scenario #203: Privileged Container Vulnerability Due to Incorrect Security Context
Category: Security
Environment: K8s v1.21, Azure AKS
Scenario Summary: A container running with elevated privileges due to an incorrect security context exposed the cluster to potential privilege escalation attacks.
What Happened: A container was configured with privileged: true in its security context, which allowed it to gain elevated permissions and access sensitive parts of the node.
Diagnosis Steps:
	• Inspected the pod security context with kubectl describe pod and found that the container was running as a privileged container.
	• Cross-referenced the container's security settings with the deployment YAML and identified the privileged: true setting.
Root Cause: Misconfigured security context allowed the container to run with elevated privileges, leading to security risks.
Fix/Workaround:
	• Removed privileged: true from the container's security context.
	• Applied the updated deployment and monitored the pod for any security incidents.
Lessons Learned: Always avoid using privileged: true unless absolutely necessary for certain workloads.
How to Avoid:
	• Review security contexts in deployment configurations to ensure containers are not running with excessive privileges.
	• Implement automated checks to flag insecure container configurations.

📘 Scenario #204: Exposed Kubernetes Dashboard Due to Misconfigured Ingress
Category: Security
Environment: K8s v1.20, GKE
Scenario Summary: The Kubernetes dashboard was exposed to the public internet due to a misconfigured Ingress resource.
What Happened: The Ingress resource for the Kubernetes dashboard was incorrectly set up to allow external traffic from all IPs, making the dashboard accessible without authentication.
Diagnosis Steps:
	• Used kubectl describe ingress to inspect the Ingress resource configuration.
	• Found that the Ingress had no restrictions on IP addresses, allowing anyone with the URL to access the dashboard.
Root Cause: Misconfigured Ingress resource with open access to the Kubernetes dashboard.
Fix/Workaround:
	• Updated the Ingress resource to restrict access to specific IP addresses or require authentication for access.
	• Re-applied the updated configuration and tested access controls.
Lessons Learned: Always secure the Kubernetes dashboard by restricting access to trusted IPs or requiring strong authentication.
How to Avoid:
	• Apply strict network policies or use ingress controllers with authentication for access to the Kubernetes dashboard.
	• Regularly review Ingress resources for security misconfigurations.

📘 Scenario #205: Unencrypted Communication Between Pods Due to Missing TLS Configuration
Category: Security
Environment: K8s v1.18, On-Premise
Scenario Summary: Communication between microservices in the cluster was not encrypted due to missing TLS configuration, exposing data to potential interception.
What Happened: The microservices were communicating over HTTP instead of HTTPS, and there was no mutual TLS (mTLS) configured for secure communication, making data vulnerable to interception.
Diagnosis Steps:
	• Reviewed service-to-service communication with network monitoring tools and found that HTTP was being used instead of HTTPS.
	• Inspected the Ingress and service definitions and found that no TLS secrets or certificates were configured.
Root Cause: Lack of TLS configuration for service communication led to unencrypted communication.
Fix/Workaround:
	• Configured mTLS between services to ensure encrypted communication.
	• Deployed certificates and updated services to use HTTPS for communication.
Lessons Learned: Secure communication between microservices is crucial to prevent data leakage or interception.
How to Avoid:
	• Always configure TLS for service-to-service communication, especially for sensitive workloads.
	• Automate the generation and renewal of certificates.

📘 Scenario #206: Sensitive Data in Logs Due to Improper Log Sanitization
Category: Security
Environment: K8s v1.23, Azure AKS
Scenario Summary: Sensitive data, such as API keys and passwords, was logged due to improper sanitization in application logs.
What Happened: A vulnerability in the application caused API keys and secrets to be included in logs, which were not sanitized before being stored in the central logging system.
Diagnosis Steps:
	• Examined the application logs using kubectl logs and found that sensitive data was included in plain text.
	• Inspected the logging configuration and found that there were no filters in place to scrub sensitive data.
Root Cause: Lack of proper sanitization in the logging process allowed sensitive data to be exposed.
Fix/Workaround:
	• Updated the application to sanitize sensitive data before it was logged.
	• Configured the logging system to filter out sensitive information from logs.
Lessons Learned: Sensitive data should never be included in logs in an unencrypted or unsanitized format.
How to Avoid:
	• Implement log sanitization techniques to ensure that sensitive information is never exposed in logs.
	• Regularly audit logging configurations to ensure that they are secure.

📘 Scenario #207: Insufficient Pod Security Policies Leading to Privilege Escalation
Category: Security
Environment: K8s v1.21, GKE
Scenario Summary: Privilege escalation was possible due to insufficiently restrictive PodSecurityPolicies (PSPs).
What Happened: The PodSecurityPolicy (PSP) was not configured to prevent privilege escalation, allowing containers to run with excessive privileges and exploit vulnerabilities within the cluster.
Diagnosis Steps:
	• Inspected the PSPs using kubectl get psp and noticed that the allowPrivilegeEscalation flag was set to true.
	• Cross-referenced the pod configurations and found that containers were running with root privileges and escalated privileges.
Root Cause: Insufficiently restrictive PodSecurityPolicies allowed privilege escalation.
Fix/Workaround:
	• Updated the PSPs to restrict privilege escalation by setting allowPrivilegeEscalation: false.
	• Applied the updated policies and tested pod deployments to confirm proper restrictions.
Lessons Learned: Always configure restrictive PodSecurityPolicies to prevent privilege escalation within containers.
How to Avoid:
	• Regularly review and apply restrictive PSPs to enforce security best practices in the cluster.
	• Use automated tools to enforce security policies on all pods and containers.

📘 Scenario #208: Service Account Token Compromise
Category: Security
Environment: K8s v1.22, DigitalOcean
Scenario Summary: A compromised service account token was used to gain unauthorized access to the cluster's API server.
What Happened: A service account token was leaked through an insecure deployment configuration, allowing attackers to gain unauthorized access to the Kubernetes API server.
Diagnosis Steps:
	• Analyzed the audit logs and identified that the compromised service account token was being used to make API calls.
	• Inspected the deployment YAML and found that the service account token was exposed as an environment variable.
Root Cause: Exposing the service account token in environment variables allowed it to be compromised.
Fix/Workaround:
	• Rotated the service account token and updated the deployment to prevent exposure.
	• Used Kubernetes secrets management to securely store sensitive tokens.
Lessons Learned: Never expose sensitive tokens or secrets through environment variables or unsecured channels.
How to Avoid:
	• Use Kubernetes Secrets to store sensitive information securely.
	• Regularly rotate service account tokens and audit access logs for suspicious activity.

📘 Scenario #209: Lack of Regular Vulnerability Scanning in Container Images
Category: Security
Environment: K8s v1.19, On-Premise
Scenario Summary: The container images used in the cluster were not regularly scanned for vulnerabilities, leading to deployment of vulnerable images.
What Happened: A critical vulnerability in one of the base images was discovered after deployment, as no vulnerability scanning tools were used to validate the images before use.
Diagnosis Steps:
	• Checked the container image build pipeline and confirmed that no vulnerability scanning tools were integrated.
	• Analyzed the CVE database and identified that a vulnerability in the image was already known.
Root Cause: Lack of regular vulnerability scanning in the container image pipeline.
Fix/Workaround:
	• Integrated a vulnerability scanning tool like Clair or Trivy into the CI/CD pipeline.
	• Rebuilt the container images with a fixed version and redeployed them.
Lessons Learned: Regular vulnerability scanning of container images is essential to ensure secure deployments.
How to Avoid:
	• Integrate automated vulnerability scanning tools into the container build process.
	• Perform regular image audits and keep base images updated.

📘 Scenario #210: Insufficient Container Image Signing Leading to Unverified Deployments
Category: Security
Environment: K8s v1.20, Google Cloud
Scenario Summary: Unverified container images were deployed due to the lack of image signing, exposing the cluster to potential malicious code.
What Happened: Malicious code was deployed when a container image was pulled from a public registry without being properly signed or verified.
Diagnosis Steps:
	• Checked the image pull policies and found that image signing was not enabled for the container registry.
	• Inspected the container image and found that it had not been signed.
Root Cause: Lack of image signing led to the deployment of unverified images.
Fix/Workaround:
	• Enabled image signing in the container registry and integrated it with Kubernetes for secure image verification.
	• Re-pulled and deployed only signed images to the cluster.
Lessons Learned: Always use signed images to ensure the integrity and authenticity of containers being deployed.
How to Avoid:
	• Implement image signing as part of the container build and deployment pipeline.
	• Regularly audit deployed container images to verify their integrity.

📘 Scenario #211: Insecure Default Namespace Leading to Unauthorized Access
Category: Security
Environment: K8s v1.22, AWS EKS
Scenario Summary: Unauthorized users gained access to resources in the default namespace due to lack of namespace isolation.
What Happened: Users without explicit permissions accessed and modified resources in the default namespace because the default namespace was not protected by network policies or RBAC rules.
Diagnosis Steps:
	• Checked RBAC policies and confirmed that users had access to resources in the default namespace.
	• Inspected network policies and found no restrictions on traffic to/from the default namespace.
Root Cause: Insufficient access control to the default namespace allowed unauthorized access.
Fix/Workaround:
	• Restricted access to the default namespace using RBAC and network policies.
	• Created separate namespaces for different workloads and applied appropriate isolation policies.
Lessons Learned: Avoid using the default namespace for critical resources and ensure that proper access control and isolation are in place.
How to Avoid:
	• Use dedicated namespaces for different workloads with appropriate RBAC and network policies.
	• Regularly audit namespace access and policies.

📘 Scenario #212: Vulnerable OpenSSL Version in Container Images
Category: Security
Environment: K8s v1.21, DigitalOcean
Scenario Summary: A container image was using an outdated and vulnerable version of OpenSSL, exposing the cluster to known security vulnerabilities.
What Happened: A critical vulnerability in OpenSSL was discovered after deploying a container that had not been updated to use a secure version of the library.
Diagnosis Steps:
	• Analyzed the Dockerfile and confirmed the container image was based on an outdated version of OpenSSL.
	• Cross-referenced the CVE database and identified that the version used in the container had known vulnerabilities.
Root Cause: The container image was built with an outdated version of OpenSSL that contained unpatched vulnerabilities.
Fix/Workaround:
	• Rebuilt the container image using a newer, secure version of OpenSSL.
	• Deployed the updated image and monitored for any further issues.
Lessons Learned: Always ensure that containers are built using updated and patched versions of libraries to mitigate known vulnerabilities.
How to Avoid:
	• Integrate automated vulnerability scanning tools into the CI/CD pipeline to identify outdated or vulnerable dependencies.
	• Regularly update container base images to the latest secure versions.

📘 Scenario #213: Misconfigured API Server Authentication Allowing External Access
Category: Security
Environment: K8s v1.20, GKE
Scenario Summary: API server authentication was misconfigured, allowing external unauthenticated users to access the Kubernetes API.
What Happened: The Kubernetes API server was mistakenly exposed without authentication, allowing external users to query resources without any credentials.
Diagnosis Steps:
	• Examined the API server configuration and found that the authentication was set to allow unauthenticated access (--insecure-allow-any-token was enabled).
	• Reviewed ingress controllers and firewall rules and confirmed that the API server was publicly accessible.
Root Cause: The API server was misconfigured to allow unauthenticated access, exposing the cluster to unauthorized requests.
Fix/Workaround:
	• Disabled unauthenticated access by removing --insecure-allow-any-token from the API server configuration.
	• Configured proper authentication methods, such as client certificates or OAuth2.
Lessons Learned: Always secure the Kubernetes API server and ensure proper authentication is in place to prevent unauthorized access.
How to Avoid:
	• Regularly audit the API server configuration to ensure proper authentication mechanisms are enabled.
	• Use firewalls and access controls to limit access to the API server.

📘 Scenario #214: Insufficient Node Security Due to Lack of OS Hardening
Category: Security
Environment: K8s v1.22, Azure AKS
Scenario Summary: Nodes in the cluster were insecure due to a lack of proper OS hardening, making them vulnerable to attacks.
What Happened: The nodes in the cluster were not properly hardened according to security best practices, leaving them vulnerable to potential exploitation.
Diagnosis Steps:
	• Conducted a security audit of the nodes and identified unpatched vulnerabilities in the operating system.
	• Verified that security settings like SSH root login and password authentication were not properly disabled.
Root Cause: Insufficient OS hardening on the nodes exposed them to security risks.
Fix/Workaround:
	• Applied OS hardening guidelines, such as disabling root SSH access and ensuring only key-based authentication.
	• Updated the operating system with the latest security patches.
Lessons Learned: Proper OS hardening is essential for securing Kubernetes nodes and reducing the attack surface.
How to Avoid:
	• Implement automated checks to enforce OS hardening settings across all nodes.
	• Regularly update nodes with the latest security patches.

📘 Scenario #215: Unrestricted Ingress Access to Sensitive Resources
Category: Security
Environment: K8s v1.21, GKE
Scenario Summary: Sensitive services were exposed to the public internet due to unrestricted ingress rules.
What Happened: An ingress resource was misconfigured, exposing sensitive internal services such as the Kubernetes dashboard and internal APIs to the public.
Diagnosis Steps:
	• Inspected the ingress rules and found that they allowed traffic from all IPs (host: *).
	• Confirmed that the services were critical and should not have been exposed to external traffic.
Root Cause: Misconfigured ingress resource allowed unrestricted access to sensitive services.
Fix/Workaround:
	• Restrict ingress traffic by specifying allowed IP ranges or adding authentication for access to sensitive resources.
	• Used a more restrictive ingress controller and verified that access was limited to trusted sources.
Lessons Learned: Always secure ingress access to critical resources by applying proper access controls.
How to Avoid:
	• Regularly review and audit ingress configurations to prevent exposing sensitive services.
	• Implement access control lists (ACLs) and authentication for sensitive endpoints.

📘 Scenario #216: Exposure of Sensitive Data in Container Environment Variables
Category: Security
Environment: K8s v1.19, AWS EKS
Scenario Summary: Sensitive data, such as database credentials, was exposed through environment variables in container configurations.
What Happened: Sensitive environment variables containing credentials were directly included in Kubernetes deployment YAML files, making them visible to anyone with access to the deployment.
Diagnosis Steps:
	• Examined the deployment manifests and discovered sensitive data in the environment variables section.
	• Used kubectl describe deployment and found that credentials were stored in plain text in the environment section of containers.
Root Cause: Storing sensitive data in plaintext environment variables exposed it to unauthorized users.
Fix/Workaround:
	• Moved sensitive data into Kubernetes Secrets instead of directly embedding them in environment variables.
	• Updated the deployment YAML to reference the Secrets and applied the changes.
Lessons Learned: Sensitive data should always be stored securely in Kubernetes Secrets or external secret management systems.
How to Avoid:
	• Use Kubernetes Secrets for storing sensitive data like passwords, API keys, and certificates.
	• Regularly audit configurations to ensure secrets are not exposed in plain text.

📘 Scenario #217: Inadequate Container Resource Limits Leading to DoS Attacks
Category: Security
Environment: K8s v1.20, On-Premise
Scenario Summary: A lack of resource limits on containers allowed a denial-of-service (DoS) attack to disrupt services by consuming excessive CPU and memory.
What Happened: A container without resource limits was able to consume all available CPU and memory on the node, causing other containers to become unresponsive and leading to a denial-of-service (DoS).
Diagnosis Steps:
	• Monitored resource usage with kubectl top pods and identified a container consuming excessive resources.
	• Inspected the deployment and found that resource limits were not set for the container.
Root Cause: Containers without resource limits allowed resource exhaustion, which led to a DoS situation.
Fix/Workaround:
	• Set appropriate resource requests and limits in the container specification to prevent resource exhaustion.
	• Applied resource quotas to limit the total resource usage for namespaces.
Lessons Learned: Always define resource requests and limits to ensure containers do not overconsume resources and cause instability.
How to Avoid:
	• Apply resource requests and limits to all containers.
	• Monitor resource usage and set appropriate quotas to prevent resource abuse.

📘 Scenario #218: Exposure of Container Logs Due to Insufficient Log Management
Category: Security
Environment: K8s v1.21, Google Cloud
Scenario Summary: Container logs were exposed to unauthorized users due to insufficient log management controls.
What Happened: Logs were stored in plain text and exposed to users who should not have had access, revealing sensitive data like error messages and stack traces.
Diagnosis Steps:
	• Reviewed log access permissions and found that they were too permissive, allowing unauthorized users to access logs.
	• Checked the log storage system and found logs were being stored unencrypted.
Root Cause: Insufficient log management controls led to unauthorized access to sensitive logs.
Fix/Workaround:
	• Implemented access controls to restrict log access to authorized users only.
	• Encrypted logs at rest and in transit to prevent exposure.
Lessons Learned: Logs should be securely stored and access should be restricted to authorized personnel only.
How to Avoid:
	• Implement access control and encryption for logs.
	• Regularly review log access policies to ensure security best practices are followed.

📘 Scenario #219: Using Insecure Docker Registry for Container Images
Category: Security
Environment: K8s v1.18, On-Premise
Scenario Summary: The cluster was pulling container images from an insecure, untrusted Docker registry, exposing the system to the risk of malicious images.
What Happened: The Kubernetes cluster was configured to pull images from an untrusted Docker registry, which lacked proper security measures such as image signing or vulnerability scanning.
Diagnosis Steps:
	• Inspected the image pull configuration and found that the registry URL pointed to an insecure registry.
	• Analyzed the images and found they lacked proper security scans or signing.
Root Cause: Using an insecure registry without proper image signing and scanning introduced the risk of malicious images.
Fix/Workaround:
	• Configured Kubernetes to pull images only from trusted and secure registries.
	• Implemented image signing and vulnerability scanning in the CI/CD pipeline.
Lessons Learned: Always use trusted and secure Docker registries and implement image security practices.
How to Avoid:
	• Use secure image registries with image signing and vulnerability scanning enabled.
	• Implement image whitelisting to control where container images can be pulled from.

📘 Scenario #220: Weak Pod Security Policies Leading to Privileged Containers
Category: Security
Environment: K8s v1.19, AWS EKS
Scenario Summary: Privileged containers were deployed due to weak or missing Pod Security Policies (PSPs), exposing the cluster to security risks.
What Happened: The absence of strict Pod Security Policies allowed containers to run with elevated privileges, leading to a potential security risk as malicious pods could gain unauthorized access to node resources.
Diagnosis Steps:
	• Inspected the cluster configuration and found that PSPs were either missing or improperly configured.
	• Verified that certain containers were running as privileged, which allowed them to access kernel-level resources.
Root Cause: Weak or missing Pod Security Policies allowed privileged containers to be deployed without restriction.
Fix/Workaround:
	• Created and applied strict Pod Security Policies to limit the permissions of containers.
	• Enforced the use of non-privileged containers for sensitive workloads.
Lessons Learned: Strict Pod Security Policies are essential for securing containers and limiting the attack surface.
How to Avoid:
	• Implement and enforce strong Pod Security Policies to limit the privileges of containers.
	• Regularly audit containers to ensure they do not run with unnecessary privileges.

📘 Scenario #221: Unsecured Kubernetes Dashboard
Category: Security
Environment: K8s v1.21, GKE
Scenario Summary: The Kubernetes Dashboard was exposed to the public internet without proper authentication or access controls, allowing unauthorized users to access sensitive cluster information.
What Happened: The Kubernetes Dashboard was deployed without proper access control or authentication mechanisms, leaving it open to the internet and allowing unauthorized users to access sensitive cluster data.
Diagnosis Steps:
	• Checked the Dashboard configuration and found that the kubectl proxy option was used without authentication enabled.
	• Verified that the Dashboard was accessible via the internet without any IP restrictions.
Root Cause: The Kubernetes Dashboard was exposed without proper authentication or network restrictions.
Fix/Workaround:
	• Enabled authentication and RBAC rules for the Kubernetes Dashboard.
	• Restricted access to the Dashboard by allowing connections only from trusted IP addresses.
Lessons Learned: Always secure the Kubernetes Dashboard with authentication and limit access using network policies.
How to Avoid:
	• Configure proper authentication for the Kubernetes Dashboard.
	• Use network policies to restrict access to sensitive resources like the Dashboard.

📘 Scenario #222: Using HTTP Instead of HTTPS for Ingress Resources
Category: Security
Environment: K8s v1.22, Google Cloud
Scenario Summary: Sensitive applications were exposed using HTTP instead of HTTPS, leaving communication vulnerable to eavesdropping and man-in-the-middle attacks.
What Happened: Sensitive application traffic was served over HTTP rather than HTTPS, allowing attackers to potentially intercept or manipulate traffic.
Diagnosis Steps:
	• Inspected ingress resource configurations and confirmed that TLS termination was not configured.
	• Verified that sensitive endpoints were exposed over HTTP without encryption.
Root Cause: Lack of TLS encryption in the ingress resources exposed sensitive traffic to security risks.
Fix/Workaround:
	• Configured ingress controllers to use HTTPS by setting up TLS termination with valid SSL certificates.
	• Redirected all HTTP traffic to HTTPS to ensure encrypted communication.
Lessons Learned: Always use HTTPS for secure communication between clients and Kubernetes applications, especially for sensitive data.
How to Avoid:
	• Configure TLS termination for all ingress resources to encrypt traffic.
	• Regularly audit ingress resources to ensure that sensitive applications are protected by HTTPS.

📘 Scenario #223: Insecure Network Policies Exposing Internal Services
Category: Security
Environment: K8s v1.20, On-Premise
Scenario Summary: Network policies were too permissive, exposing internal services to unnecessary access, increasing the risk of lateral movement within the cluster.
What Happened: Network policies were overly permissive, allowing services within the cluster to communicate with each other without restriction. This made it easier for attackers to move laterally if they compromised one service.
Diagnosis Steps:
	• Reviewed the network policy configurations and found that most services were allowed to communicate with any other service within the cluster.
	• Inspected the logs for unauthorized connections between services.
Root Cause: Permissive network policies allowed unnecessary communication between services, increasing the potential attack surface.
Fix/Workaround:
	• Restricted network policies to only allow communication between services that needed to interact.
	• Used namespace-based segmentation and ingress/egress rules to enforce tighter security.
Lessons Learned: Proper network segmentation and restrictive network policies are crucial for securing the internal traffic between services.
How to Avoid:
	• Apply the principle of least privilege when defining network policies, ensuring only necessary communication is allowed.
	• Regularly audit network policies to ensure they are as restrictive as needed.

📘 Scenario #224: Exposing Sensitive Secrets in Environment Variables
Category: Security
Environment: K8s v1.21, AWS EKS
Scenario Summary: Sensitive credentials were stored in environment variables within the pod specification, exposing them to potential attackers.
What Happened: Sensitive data such as database passwords and API keys were stored as environment variables in plain text within Kubernetes pod specifications, making them accessible to anyone who had access to the pod's configuration.
Diagnosis Steps:
	• Examined the pod specification files and found that sensitive credentials were stored as environment variables in plaintext.
	• Verified that no secrets management solution like Kubernetes Secrets was being used to handle sensitive data.
Root Cause: Sensitive data was stored insecurely in environment variables rather than using Kubernetes Secrets or an external secrets management solution.
Fix/Workaround:
	• Moved sensitive data to Kubernetes Secrets and updated the pod configurations to reference the secrets.
	• Ensured that secrets were encrypted and only accessible by the relevant services.
Lessons Learned: Always store sensitive data securely using Kubernetes Secrets or an external secrets management solution, and avoid embedding it in plain text.
How to Avoid:
	• Use Kubernetes Secrets to store sensitive data and reference them in your deployments.
	• Regularly audit your configuration files to ensure sensitive data is not exposed in plaintext.

📘 Scenario #225: Insufficient RBAC Permissions Leading to Unauthorized Access
Category: Security
Environment: K8s v1.20, On-Premise
Scenario Summary: Insufficient Role-Based Access Control (RBAC) configurations allowed unauthorized users to access and modify sensitive resources within the cluster.
What Happened: The RBAC configurations were not properly set up, granting more permissions than necessary. As a result, unauthorized users were able to access sensitive resources such as secrets, config maps, and deployments.
Diagnosis Steps:
	• Reviewed RBAC policies and roles and found that users had been granted broad permissions, including access to sensitive namespaces and resources.
	• Verified that the principle of least privilege was not followed.
Root Cause: RBAC roles were not properly configured, resulting in excessive permissions being granted to users.
Fix/Workaround:
	• Reconfigured RBAC roles to ensure that users only had the minimum necessary permissions.
	• Applied the principle of least privilege and limited access to sensitive resources.
Lessons Learned: RBAC should be configured according to the principle of least privilege to minimize security risks.
How to Avoid:
	• Regularly review and audit RBAC configurations to ensure they align with the principle of least privilege.
	• Implement strict role definitions and limit access to only the resources necessary for each user.
```
