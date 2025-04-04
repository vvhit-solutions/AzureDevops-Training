# Azure DevOps Engineer - Cloud & DevOps Scenarios

## **Azure Cloud**

### **1. Designing Landing Zones in Azure**
- Scenario-based questions on designing Azure Landing Zones.
- Best practices for governance, security, and networking.
- Implementation using Azure Blueprints and Policies.

### **2. Implementing Hub and Spoke Model**
- Architecture overview.
- Implementing VNet Peering, Route Tables, and Private Link.
- Securing hub-to-spoke communication.

### **3. On-Premises to Azure Connectivity**
- VPN Gateway vs. ExpressRoute.
- Configuring Site-to-Site and Point-to-Site VPNs.
- Private Link vs. Service Endpoints.

### **4. Security & Traffic Management**
- **Azure Front Door, Application Gateway, WAF Policies, and Traffic Manager.**
- Load balancing strategies and DDoS protection.

### **5. Azure RBAC & PIM**
- Role-Based Access Control (RBAC) scenarios.
- Configuring Privileged Identity Management (PIM) for Just-in-Time (JIT) access.

### **6. Storage Accounts & Key Vaults**
- Implementing secure storage access using Private Endpoints.
- Role of Managed Identities in securing access to storage and secrets.

## **Terraform**

### **1. Resource Creation in Azure**
- Using Terraform to create Azure resources.

### **2. Terraform Modules**
- Structuring Terraform code using modules.
- Example of reusable modules for resource provisioning.

### **3. Terraform Output Commands**
- Capturing and utilizing Terraform outputs in automation workflows.

### **4. State Commands & Lookup Function**
- Managing state files (init, plan, apply, refresh, state list, show, rm, mv, import).
- Using `lookup` function to fetch values dynamically.

### **5. Foreach & Lookup Functions**
- Use cases of `foreach` and `lookup` for resource provisioning.

## **AKS/Kubernetes**

### **1. Rolling Upgrade Strategy**
- Implementing rolling updates and rollback in AKS.

### **2. AKS Networking**
- CNI vs. Kubenet networking.
- Network policies and ingress controllers.

### **3. Helm & Configuration Management**
- Writing Helm charts and managing application configurations.

### **4. Taint Toleration & Node Affinity**
- Scheduling workloads using taints, tolerations, and node affinity rules.

### **5. Security & Access**
- Implementing RBAC and Azure AD integration.
- Pod security policies and network security controls.

### **6. Probes in AKS**
- Readiness, Liveness, and Startup probes use cases.

## **Scripting (Python, Bash, PowerShell)**

### **1. Sample Scripts**
- Counting occurrences of a string in a file.
- Printing logs from a file.
- Generating prime numbers.

## **Azure DevOps / GitLab / GitHub Actions**

### **1. YAML Pipeline**
- Writing a YAML pipeline with `dependsOn` and `conditions`.

### **2. Agent Pools & Runners**
- Selecting self-hosted vs. Microsoft-hosted agents.

### **3. Branching & Deployment Strategies**
- GitFlow, Trunk-based development, Feature branching.

### **4. Security & Code Quality Tools**
- SonarQube, Veracode, Fortify integration.

### **5. End-to-End Pipeline Setup**
- CI/CD setup for a tech application in Azure DevOps.

### **6. Variables & Deployment Groups**
- Managing variables using Variable Groups and Library.
- Deployment strategies using Deployment Groups.

### **7. Writing Pipelines in GitLab & GitHub Actions**
- Example scenarios and best practices.

---

### **Contributions**
Feel free to raise PRs to enhance this repository with additional use cases and examples.

### **License**
This project is licensed under the MIT License.

