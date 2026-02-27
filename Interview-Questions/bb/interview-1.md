# Systems Engineer Interview Questions & Answers (5+ Years Experience)

## Infrastructure as Code (Terraform/OpenTofu)

### Q1: Explain the difference between Terraform and OpenTofu. Why might an organization choose one over the other?

**Answer:**
OpenTofu is an open-source fork of Terraform that was created after HashiCorp changed Terraform's license to BSL (Business Source License). Key differences:

- **Licensing**: OpenTofu is MPL 2.0 (fully open source), while Terraform uses BSL
- **Governance**: OpenTofu is governed by the Linux Foundation, ensuring community-driven development
- **Compatibility**: OpenTofu maintains compatibility with Terraform configurations and state files
- **Features**: Both support similar features, but OpenTofu focuses on community-driven enhancements

Organizations might choose OpenTofu to:
- Avoid vendor lock-in
- Ensure long-term open-source availability
- Participate in community-driven development
- Avoid potential licensing restrictions

### Q2: How do you manage Terraform state in a team environment with multiple environments?

**Answer:**
Best practices for team Terraform state management:

1. **Remote Backend**: Use Azure Storage Account with state locking
```hcl
terraform {
  backend "azurerm" {
    resource_group_name  = "rg-tfstate"
    storage_account_name = "stgcsharedtfstate"
    container_name       = "tfstate"
    key                  = "terraform-shared.tfstate"
    use_azuread_auth     = true
  }
}
```

2. **Environment Separation**: Separate state files per environment
3. **State Locking**: Use Azure blob leases for concurrent access protection
4. **Access Control**: Use Azure AD authentication and RBAC
5. **Workspace Isolation**: Use Terraform workspaces for environment separation

### Q3: Explain Terraform modules and their benefits. How do you structure them?

**Answer:**
Terraform modules are reusable configurations that encapsulate resources. Benefits:

- **Reusability**: Write once, use multiple times
- **Consistency**: Standardized deployments across environments
- **Maintainability**: Centralized updates
- **Abstraction**: Hide complexity from users

Structure example:
```
modules/
├── azure_app_spn/
│   ├── main.tf
│   ├── variables.tf
│   ├── outputs.tf
│   └── versions.tf
└── aks/
    ├── cluster.tf
    ├── node_pools.tf
    ├── variables.tf
    └── outputs.tf
```

### Q4: How do you handle secrets in Terraform configurations?

**Answer:**
Secret management strategies:

1. **Azure Key Vault**: Store secrets externally
```hcl
data "azurerm_key_vault_secret" "api_key" {
  name         = "api-key"
  key_vault_id = azurerm_key_vault.main.id
}
```

2. **Variables**: Use sensitive variables
```hcl
variable "password" {
  description = "Database password"
  type        = string
  sensitive   = true
}
```

3. **Environment Variables**: Use TF_VAR_ prefix
4. **External Data Sources**: Fetch from external systems
5. **Never**: Store secrets in code or state files

## Azure Cloud Platform

### Q5: Explain Azure Resource Groups and their relationship with Subscriptions.

**Answer:**
- **Subscription**: Billing boundary and management unit
- **Resource Group**: Logical container for related resources
- **Relationship**: One subscription can have multiple resource groups

Best practices:
- Group resources by lifecycle (dev, staging, prod)
- Apply consistent naming conventions
- Use resource groups for RBAC and policy application
- Consider geographical proximity for performance

### Q6: What is Azure AD (Entra ID) and how does it integrate with Azure resources?

**Answer:**
Azure AD is Microsoft's cloud-based identity service providing:

- **Authentication**: User and service principal authentication
- **Authorization**: RBAC for resource access
- **Directory Services**: User/group management
- **Single Sign-On**: Across applications

Integration with Azure:
```hcl
resource "azuread_group" "dns_contributer" {
  display_name     = "shared-dns-contributer"
  security_enabled = true
}

resource "azurerm_role_assignment" "dns_contributer" {
  scope              = azurerm_dns_zone.root_domain.id
  role_definition_id = data.azurerm_role_definition.dns_zone_contributor.id
  principal_id       = azuread_group.dns_contributer.object_id
}
```

### Q7: Explain Azure Key Vault and its security features.

**Answer:**
Azure Key Vault is a secure storage service for:
- **Secrets**: API keys, passwords, connection strings
- **Keys**: Encryption keys with HSM backing
- **Certificates**: SSL/TLS certificates

Security features:
- **Access Policies**: Fine-grained permissions
- **Soft Delete**: Recovery from accidental deletion
- **Purge Protection**: Prevents permanent deletion
- **Network Access**: Private endpoints and firewall rules
- **Audit Logs**: Complete access logging
- **RBAC Integration**: Azure AD-based access control

### Q8: What are Azure Private Endpoints and when would you use them?

**Answer:**
Private Endpoints provide secure connectivity to Azure services over a private network connection.

Use cases:
- Database connections from applications
- Storage account access from specific VNets
- Key Vault access for applications
- Compliance requirements for private connectivity

Benefits:
- **Security**: Traffic stays on Microsoft backbone
- **Performance**: Reduced latency
- **Compliance**: Meet regulatory requirements
- **Network Segmentation**: Control access paths

## Kubernetes & Container Orchestration

### Q9: Explain the architecture of an Azure Kubernetes Service (AKS) cluster.

**Answer:**
AKS architecture components:

**Control Plane** (Managed by Azure):
- API Server
- etcd
- Controller Manager
- Scheduler

**Node Pools**:
- System Node Pool: Critical system services
- User Node Pools: Application workloads
- Spot Node Pools: Cost-effective for batch workloads

**Networking**:
- Azure CNI: Advanced networking
- Service mesh integration (Istio)
- Load balancers for ingress

### Q10: What is Istio and how does it benefit microservices architectures?

**Answer:**
Istio is a service mesh that provides:

**Traffic Management**:
```yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: reviews
spec:
  http:
  - route:
    - destination:
        host: reviews
        subset: v1
      weight: 90
    - destination:
        host: reviews
        subset: v2
      weight: 10
```

**Security**:
- mTLS between services
- Authentication and authorization policies
- Network policies

**Observability**:
- Distributed tracing
- Service metrics
- Access logs

**Benefits**:
- Zero-code service discovery
- Load balancing
- Circuit breaking
- Security policies

### Q11: What is Kyverno and how does it differ from OPA Gatekeeper?

**Answer:**
Kyverno is a Kubernetes-native policy management tool.

**Differences from OPA Gatekeeper**:

| Kyverno | OPA Gatekeeper |
|---------|----------------|
| YAML-based policies | Rego language |
| Kubernetes-native | General-purpose |
| Built-in functions | Custom functions |
| Generate/mutate resources | Primarily validation |

**Example Kyverno policy**:
```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: disallow-latest-tag
spec:
  validationFailureAction: enforce
  background: false
  rules:
  - name: check-image-tag
    match:
      any:
      - resources:
          kinds:
          - Pod
    validate:
      message: "Using latest tag is not allowed"
      pattern:
        spec:
          containers:
          - image: "!*:latest"
```

### Q12: Explain Kubernetes RBAC and how to implement it properly.

**Answer:**
RBAC (Role-Based Access Control) controls access to Kubernetes resources.

**Components**:
- **Role/ClusterRole**: Define permissions
- **RoleBinding/ClusterRoleBinding**: Assign roles to subjects
- **Subjects**: Users, groups, service accounts

**Implementation**:
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: production
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "watch", "list"]

---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods
  namespace: production
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
subjects:
- kind: User
  name: jane
  apiGroup: rbac.authorization.k8s.io
```

**Best Practices**:
- Principle of least privilege
- Use groups instead of individual users
- Regular access reviews
- Separate service accounts per application

## CI/CD and DevOps

### Q13: Design a CI/CD pipeline for a containerized application using GitHub Actions.

**Answer:**
Complete GitHub Actions workflow:

```yaml
name: CI/CD Pipeline
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v4
    - name: Run tests
      run: |
        make test
        make lint

  build:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
    - uses: actions/checkout@v4
    - name: Login to ACR
      uses: azure/docker-login@v1
      with:
        login-server: myacr.azurecr.io
        username: ${{ secrets.ACR_USERNAME }}
        password: ${{ secrets.ACR_PASSWORD }}
    
    - name: Build and push
      run: |
        docker build -t myacr.azurecr.io/app:${{ github.sha }} .
        docker push myacr.azurecr.io/app:${{ github.sha }}

  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment: production
    steps:
    - name: Deploy to AKS
      uses: azure/k8s-deploy@v1
      with:
        manifests: k8s/
        images: myacr.azurecr.io/app:${{ github.sha }}
```

### Q14: How do you implement GitOps for Kubernetes deployments?

**Answer:**
GitOps principles and implementation:

**Principles**:
1. Declarative configuration
2. Git as single source of truth
3. Automated deployment
4. Continuous monitoring

**Implementation with ArgoCD**:
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: my-app
spec:
  project: default
  source:
    repoURL: https://github.com/company/k8s-configs
    targetRevision: HEAD
    path: apps/production
  destination:
    server: https://kubernetes.default.svc
    namespace: production
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

**Repository Structure**:
```
k8s-configs/
├── apps/
│   ├── development/
│   ├── staging/
│   └── production/
└── infrastructure/
    ├── ingress/
    └── monitoring/
```

### Q15: Explain multi-environment deployment strategies.

**Answer:**
Common strategies:

**1. Environment Promotion**:
- Dev → Staging → Production
- Automated testing at each stage
- Manual approval gates

**2. Branch-based Environments**:
- Feature branches → dev environment
- Main branch → staging
- Tags → production

**3. Terraform Workspaces**:
```hcl
terraform {
  backend "azurerm" {
    # Environment-specific state files
    key = "terraform-${terraform.workspace}.tfstate"
  }
}

locals {
  environment_config = {
    dev = {
      vm_size = "Standard_B2s"
      replicas = 1
    }
    prod = {
      vm_size = "Standard_D4s_v3"
      replicas = 3
    }
  }
  config = local.environment_config[terraform.workspace]
}
```

## Monitoring and Observability

### Q16: Design a comprehensive monitoring strategy for a cloud application.

**Answer:**
Multi-layered monitoring approach:

**Infrastructure Monitoring**:
- Azure Monitor for resource metrics
- VM insights for system performance
- Network monitoring for connectivity

**Application Monitoring**:
- Application Insights for APM
- Custom metrics and dashboards
- Distributed tracing

**Log Management**:
- Centralized logging with Azure Log Analytics
- Log correlation and analysis
- Alerting on error patterns

**Example Datadog Monitor**:
```hcl
resource "datadog_monitor" "high_cpu" {
  name    = "High CPU Usage"
  type    = "metric alert"
  message = "CPU usage is above threshold"
  
  query = "avg(last_5m):avg:system.cpu.user{*} by {host} > 80"
  
  monitor_thresholds {
    warning  = 70
    critical = 80
  }
  
  notify_no_data    = true
  renotify_interval = 60
}
```

### Q17: How do you implement effective alerting without alert fatigue?

**Answer:**
Smart alerting strategies:

**1. Alert Hierarchy**:
- P1: Business-critical, immediate response
- P2: Important but not urgent
- P3: Informational

**2. Escalation Policies**:
```yaml
escalation_policy:
  - level: 1
    targets: [primary_oncall]
    timeout: 5_minutes
  - level: 2  
    targets: [secondary_oncall, manager]
    timeout: 15_minutes
```

**3. Alert Correlation**:
- Group related alerts
- Parent-child relationships
- Maintenance windows

**4. SLI/SLO-based Alerting**:
- Alert on SLO burn rate
- Multi-window approach
- Error budgets

### Q18: Explain the concept of SRE and how it differs from traditional operations.

**Answer:**
Site Reliability Engineering principles:

**Traditional Ops vs SRE**:

| Traditional Ops | SRE |
|-----------------|-----|
| Reactive | Proactive |
| Manual processes | Automation-first |
| Uptime focus | Reliability engineering |
| Separate from dev | Embedded with dev |

**SRE Practices**:
- **SLIs/SLOs**: Quantify reliability
- **Error Budgets**: Balance velocity vs stability
- **Toil Reduction**: Automate repetitive tasks
- **Blameless Post-mortems**: Learn from failures

**Example SLOs**:
- API Availability: 99.9% (43.2 minutes downtime/month)
- Request Latency: 95th percentile < 200ms
- Error Rate: < 0.1% of requests

## Security and Compliance

### Q19: Explain implementation of Privileged Identity Management (PIM) in Azure.

**Answer:**
PIM provides just-in-time access to privileged roles:

**Configuration**:
```hcl
resource "azurerm_pim_active_role_assignment" "example" {
  scope              = azurerm_resource_group.example.id
  role_definition_id = data.azurerm_role_definition.contributor.id
  principal_id       = azuread_user.example.object_id
  
  schedule {
    expiration {
      duration_hours = 8
    }
  }
  
  ticket {
    number = "INC123456"
    system = "ServiceNow"
  }
}
```

**Benefits**:
- **Just-in-time access**: Reduce attack surface
- **Approval workflows**: Require justification
- **Access reviews**: Regular validation
- **Audit trails**: Complete access logging

### Q20: How do you implement network security in Azure and Kubernetes?

**Answer:**
Multi-layer network security:

**Azure Network Security**:
- Network Security Groups (NSGs)
- Azure Firewall
- Private Endpoints
- VNet peering with hub-spoke topology

**Kubernetes Network Security**:
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all-ingress
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
  
  # Allow specific ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: frontend
    ports:
    - protocol: TCP
      port: 8080
```

**Service Mesh Security**:
- mTLS between services
- Authorization policies
- Security scanning

### Q21: Describe secret management best practices in Kubernetes.

**Answer:**
Comprehensive secret management:

**1. External Secret Management**:
```yaml
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: vault-secret
spec:
  secretStoreRef:
    name: vault-backend
    kind: SecretStore
  target:
    name: mysecret
    creationPolicy: Owner
  data:
  - secretKey: password
    remoteRef:
      key: secret/data/database
      property: password
```

**2. Secret Encryption**:
- Enable encryption at rest
- Use Azure Key Vault for key management
- Regular key rotation

**3. Access Control**:
- RBAC for secret access
- Service account isolation
- Pod security policies

**4. Secret Injection**:
- Volume mounts (preferred)
- Environment variables (avoid if possible)
- Init containers for secret fetching

## Networking and DNS

### Q22: Explain DNS management in a multi-cloud/hybrid environment.

**Answer:**
DNS management strategy:

**Zone Management**:
```hcl
resource "azurerm_dns_zone" "root_domain" {
  name                = "company.com"
  resource_group_name = azurerm_resource_group.dns.name
}

resource "azurerm_dns_a_record" "app" {
  name                = "app"
  zone_name           = azurerm_dns_zone.root_domain.name
  resource_group_name = azurerm_resource_group.dns.name
  ttl                 = 300
  records             = ["10.0.1.10"]
}
```

**Multi-environment DNS**:
- Environment-specific subdomains
- Geographic load balancing
- Failover configurations
- Private DNS zones for internal services

**Best Practices**:
- Short TTLs for dynamic records
- Health check integration
- DNS monitoring and alerting
- Documentation of DNS changes

### Q23: How do you design network architecture for a cloud-native application?

**Answer:**
Cloud-native network design:

**High-level Architecture**:
- Hub-spoke topology
- East-west traffic inspection
- Zero-trust networking

**Components**:
```
Internet Gateway
    ↓
Azure Firewall/Application Gateway
    ↓
Hub VNet (Shared Services)
    ↓
Spoke VNets (Applications)
    ↓
Private Endpoints/Service Endpoints
```

**Kubernetes Networking**:
- CNI implementation (Azure CNI)
- Service mesh (Istio)
- Ingress controllers
- Network policies

## Testing and Quality Assurance

### Q24: Design a testing strategy for infrastructure as code.

**Answer:**
Multi-level testing approach:

**1. Unit Testing**:
```python
import pytest
from unittest.mock import Mock

def test_resource_group_creation():
    # Test Terraform module logic
    result = terraform_plan("dev")
    assert "azurerm_resource_group" in result.planned_values
    assert result.planned_values["azurerm_resource_group"]["location"] == "East US"
```

**2. Integration Testing**:
```yaml
# Test actual deployment
apiVersion: v1
kind: ConfigMap
metadata:
  name: test-config
data:
  test-script: |
    #!/bin/bash
    kubectl get pods -n production
    curl -f http://app.example.com/health
```

**3. Policy Testing**:
```python
def test_kyverno_policy_enforcement():
    """Test Kyverno policies are properly enforced"""
    # Try to create pod with latest tag
    pod_manifest = create_pod_with_latest_tag()
    
    with pytest.raises(ValidationError):
        kubectl_apply(pod_manifest)
```

**4. End-to-End Testing**:
- Synthetic monitoring
- User journey testing
- Performance testing

### Q25: How do you implement automated compliance checking?

**Answer:**
Automated compliance framework:

**Infrastructure Scanning**:
```yaml
# Checkov configuration
checkov:
  framework: terraform
  checks:
    - CKV_AZURE_50  # Storage account secure transfer
    - CKV_AZURE_109 # Key vault firewall enabled
  
  skip-checks:
    - CKV_AZURE_35  # Skip if not applicable
```

**Runtime Compliance**:
```yaml
# Kyverno compliance policy
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: require-security-context
spec:
  validationFailureAction: enforce
  rules:
  - name: check-security-context
    match:
      any:
      - resources:
          kinds:
          - Pod
    validate:
      message: "Security context is required"
      pattern:
        spec:
          securityContext:
            runAsNonRoot: true
            runAsUser: ">0"
```

**Continuous Monitoring**:
- Azure Policy compliance
- CIS benchmark checks
- Custom compliance rules

## Performance Optimization

### Q26: How do you optimize costs in Azure cloud deployments?

**Answer:**
Cost optimization strategies:

**1. Right-sizing Resources**:
```hcl
# Use appropriate VM sizes
resource "azurerm_kubernetes_cluster" "main" {
  default_node_pool {
    vm_size = var.environment == "prod" ? "Standard_D4s_v3" : "Standard_B2s"
    
    auto_scaling_enabled = true
    min_count            = var.min_nodes[var.environment]
    max_count            = var.max_nodes[var.environment]
  }
}
```

**2. Spot Instances**:
```hcl
resource "azurerm_kubernetes_cluster_node_pool" "spot" {
  priority        = "Spot"
  eviction_policy = "Delete"
  spot_max_price  = 0.1  # Max price per hour
  
  node_taints = ["kubernetes.azure.com/scalesetpriority=spot:NoSchedule"]
}
```

**3. Resource Scheduling**:
- Auto-shutdown for non-production
- Reserved instances for predictable workloads
- Azure Hybrid Benefit for Windows licenses

**4. Monitoring and Alerting**:
- Cost alerts and budgets
- Resource utilization monitoring
- Unused resource identification

### Q27: Explain performance tuning for Kubernetes workloads.

**Answer:**
Performance optimization techniques:

**Resource Management**:
```yaml
apiVersion: v1
kind: Pod
spec:
  containers:
  - name: app
    resources:
      requests:
        memory: "128Mi"
        cpu: "250m"
      limits:
        memory: "256Mi"
        cpu: "500m"
```

**Horizontal/Vertical Scaling**:
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: app-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: app
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
```

**Node Optimization**:
- Node affinity rules
- Pod disruption budgets
- Quality of Service classes

## Disaster Recovery and Business Continuity

### Q28: Design a disaster recovery strategy for a cloud application.

**Answer:**
Comprehensive DR strategy:

**1. RTO/RPO Requirements**:
- Recovery Time Objective: < 4 hours
- Recovery Point Objective: < 15 minutes

**2. Multi-Region Architecture**:
```hcl
# Primary region
resource "azurerm_kubernetes_cluster" "primary" {
  name     = "aks-primary"
  location = "East US"
  # Configuration
}

# DR region
resource "azurerm_kubernetes_cluster" "dr" {
  name     = "aks-dr"
  location = "West US 2"
  # Configuration
}
```

**3. Data Replication**:
- Database geo-replication
- Storage account geo-redundancy
- Regular backup testing

**4. Traffic Management**:
```hcl
resource "azurerm_traffic_manager_profile" "main" {
  traffic_routing_method = "Priority"
  
  monitor_config {
    protocol = "HTTPS"
    port     = 443
    path     = "/health"
  }
}
```

### Q29: How do you implement backup and restore procedures?

**Answer:**
Comprehensive backup strategy:

**Infrastructure Backup**:
- Terraform state backup
- Configuration as code in Git
- Infrastructure snapshots

**Application Data Backup**:
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: database-backup
spec:
  schedule: "0 2 * * *"  # Daily at 2 AM
  jobTemplate:
    spec:
      template:
        spec:
          containers:
          - name: backup
            image: postgres:13
            command:
            - /bin/bash
            - -c
            - |
              pg_dump $DATABASE_URL | \
              gzip | \
              azure storage blob upload --file - --name "backup-$(date +%Y%m%d).sql.gz"
```

**Recovery Testing**:
- Regular restore drills
- Automated recovery validation
- Documentation updates

## Advanced Troubleshooting

### Q30: Describe your approach to troubleshooting a complex production incident.

**Answer:**
Systematic incident response:

**1. Immediate Response**:
- Assess impact and severity
- Implement temporary mitigation
- Communicate with stakeholders

**2. Investigation Process**:
```bash
# Check system health
kubectl get nodes
kubectl get pods --all-namespaces
kubectl top nodes

# Check logs
kubectl logs -f deployment/app --previous
kubectl describe pod failing-pod

# Check metrics
curl -s "http://prometheus:9090/api/v1/query?query=up"
```

**3. Root Cause Analysis**:
- Timeline reconstruction
- Change correlation
- System dependency mapping

**4. Resolution and Follow-up**:
- Implement permanent fix
- Update runbooks
- Conduct blameless post-mortem

---

## Conclusion

This interview guide covers the essential technologies and practices for a senior systems engineer working with modern cloud infrastructure. The questions progress from foundational knowledge to complex real-world scenarios, testing both theoretical understanding and practical experience.

Key areas emphasized:
- Infrastructure as Code best practices
- Cloud-native technologies
- Security and compliance
- Monitoring and observability  
- Automation and CI/CD
- Performance optimization
- Disaster recovery planning

Successful candidates should demonstrate not only technical knowledge but also experience with enterprise-scale implementations, troubleshooting skills, and understanding of operational best practices.
