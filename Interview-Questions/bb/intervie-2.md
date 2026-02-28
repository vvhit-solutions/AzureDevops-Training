#  System Engineer Interview Questions
*Based on Central Infrastructure and ECOS Applications*

## Table of Contents
1. [Azure Cloud and Terraform (Questions 1-20)](#azure-cloud-and-terraform)
2. [Kubernetes Fundamentals (Questions 21-35)](#kubernetes-fundamentals)
3. [Kubernetes Application Deployment (Questions 36-50)](#kubernetes-application-deployment)
4. [Kubernetes Operators (Questions 51-60)](#kubernetes-operators)
5. [GitHub and GitHub Actions (Questions 61-70)](#github-and-github-actions)
6. [GitOps - ArgoCD/FluxCD (Questions 71-80)](#gitops)
7. [Troubleshooting (Questions 81-90)](#troubleshooting)
8. [Networking (Questions 91-100)](#networking)

---

## Azure Cloud and Terraform

### Question 1: Infrastructure as Code Structure
**Scenario:** You're looking at the `gc-infra` repository structure. Explain the purpose of the `bootstrap`, `shared`, and `mh` directories in the Terraform organization.

**Answer:** 
- **Bootstrap**: Contains initial Azure infrastructure setup including Azure AD, management groups, subscriptions, and foundational resources needed before other infrastructure can be deployed
- **Shared**: Contains common infrastructure components that are shared across multiple environments like ACR, Key Vault, DNS, virtual networks, and observability tools
- **MH** (Multi-tenant Hub): Contains application-specific infrastructure organized by service types (application, baseline, compute, observability, runtime-classes, services, storage)

### Question 2: Azure AD Configuration
**Scenario:** Looking at `bootstrap/azureinit/azuread.tf`, you need to explain the security model for service principals and conditional access policies.

**Answer:**
The Azure AD configuration implements a zero-trust security model with:
- Service principals with minimal required permissions for automation
- Conditional access policies enforcing MFA and device compliance
- PIM (Privileged Identity Management) for just-in-time access
- GitHub OIDC integration for secure CI/CD without storing credentials
- Segregated permissions between different environments and workloads

### Question 3: Terraform State Management
**Scenario:** How would you ensure Terraform state is properly managed and secured in this multi-environment setup?

**Answer:**
- Use Azure Storage Account with blob storage for remote state
- Enable versioning and soft delete on storage account
- Implement state locking using Azure Storage Account lease mechanism
- Separate state files per environment and component
- Use service principals with minimal permissions for state access
- Encrypt state files at rest and in transit
- Implement backup and disaster recovery for state files

### Question 4: Resource Group Strategy
**Scenario:** Explain the resource group strategy used in `shared/resourcegroups.tf` and why it's organized this way.

**Answer:**
The resource group strategy follows Azure best practices:
- Logical grouping by function (shared, observability, networking)
- Environment-specific resource groups for isolation
- Consistent naming conventions for easy identification
- Resource lifecycle alignment within groups
- RBAC boundaries at resource group level
- Cost management and billing separation
- Disaster recovery and backup policies per group

### Question 5: Key Vault Implementation
**Scenario:** You need to add a new secret to Key Vault that will be used by multiple applications. Walk through the process using the current `keyvault.tf` configuration.

**Answer:**
1. Add the secret definition to `keyvault.tf`:
```hcl
resource "azurerm_key_vault_secret" "new_secret" {
  name         = "new-application-secret"
  value        = var.new_secret_value
  key_vault_id = azurerm_key_vault.shared.id
  
  tags = local.common_tags
}
```
2. Create corresponding variable in `variables.tf`
3. Add access policy if new service principal needs access
4. Update terraform plan and apply
5. Update application configuration to reference the new secret

### Question 6: Virtual Network Design
**Scenario:** Analyze the virtual network configuration in `shared/virtual_network.tf`. How would you add a new subnet for a database tier?

**Answer:**
1. Add subnet configuration:
```hcl
resource "azurerm_subnet" "database" {
  name                 = "${var.environment}-database-subnet"
  resource_group_name  = azurerm_resource_group.networking.name
  virtual_network_name = azurerm_virtual_network.main.name
  address_prefixes     = ["10.0.4.0/24"]
  
  delegation {
    name = "database-delegation"
    service_delegation {
      name    = "Microsoft.DBforPostgreSQL/flexibleServers"
      actions = ["Microsoft.Network/virtualNetworks/subnets/action"]
    }
  }
}
```
2. Create NSG rules for database access
3. Update route tables if necessary
4. Configure service endpoints or private endpoints
5. Update firewall rules and security groups

### Question 7: ACR (Azure Container Registry) Configuration
**Scenario:** Explain the ACR setup in the shared infrastructure and how the push/read policies are implemented.

**Answer:**
The ACR configuration implements:
- **Push access** (`acr_push.tf`): Service principals with `AcrPush` role for CI/CD pipelines
- **Read-only access** (`acr_read_only.tf`): Kubernetes service accounts with `AcrPull` role
- **Policies** (`acr-policies.tf`): Content trust, image quarantine, and retention policies
- **Multi-environment support**: Separate registries or repositories per environment
- **Security scanning**: Integration with Azure Security Center for vulnerability scanning
- **Geographic replication**: For disaster recovery and performance

### Question 8: Diagnostic Settings Implementation
**Scenario:** You need to ensure all Azure resources send logs to Log Analytics. How is this implemented in `diagnostic-settings.tf`?

**Answer:**
```hcl
resource "azurerm_monitor_diagnostic_setting" "example" {
  name                       = "diagnostic-setting"
  target_resource_id         = azurerm_resource.example.id
  log_analytics_workspace_id = azurerm_log_analytics_workspace.main.id
  
  log {
    category = "AuditEvent"
    enabled  = true
    retention_policy {
      enabled = true
      days    = 365
    }
  }
  
  metric {
    category = "AllMetrics"
    enabled  = true
    retention_policy {
      enabled = true
      days    = 365
    }
  }
}
```
Apply to all critical resources (Key Vault, ACR, AKS, NSGs, etc.)

### Question 9: Azure Defender Configuration
**Scenario:** Explain the security monitoring setup using Azure Defender as configured in `defender.tf`.

**Answer:**
Azure Defender provides:
- **Threat detection**: Real-time monitoring for suspicious activities
- **Vulnerability assessment**: Container and VM scanning
- **Security recommendations**: Azure Security Center integration
- **Compliance monitoring**: Regulatory compliance dashboards
- **Incident response**: Automated alerts and playbooks
- **Resource coverage**: Kubernetes, SQL, Storage, Key Vault, App Service

### Question 10: Terraform Modules Strategy
**Scenario:** How would you create a reusable Terraform module for the Azure App Service Principal pattern seen in `modules/azure_app_spn/`?

**Answer:**
Module structure:
```
modules/azure_app_spn/
├── main.tf          # Service principal and certificate resources
├── variables.tf     # Input variables
├── outputs.tf       # Output values
├── versions.tf      # Provider requirements
└── README.md        # Usage documentation
```

Key components:
- Service principal creation with federation
- Certificate generation and Key Vault storage
- Role assignments with different scopes
- Output sensitive data appropriately
- Version constraints for providers

### Question 11: Environment Promotion Strategy
**Scenario:** How would you promote infrastructure changes from dev → test → staging → production using this Terraform structure?

**Answer:**
1. **Branch Strategy**: Feature branches → dev → staging → main (production)
2. **Terraform Workspaces**: Separate workspace per environment
3. **Variable Files**: Environment-specific `.tfvars` files
4. **Pipeline Gates**: Approval processes between environments
5. **State Separation**: Independent state files per environment
6. **Testing**: Plan validation and policy checks before apply
7. **Rollback**: Maintain previous known-good state backups

### Question 12: Azure Policy Implementation
**Scenario:** How would you implement Azure Policy to enforce compliance across all resources in this infrastructure?

**Answer:**
Implement policies for:
- **Naming conventions**: Enforce consistent resource naming
- **Required tags**: Ensure cost center, environment, owner tags
- **Security**: Require encryption, disable public access
- **Compliance**: SOX, GDPR, SOC2 requirements
- **Cost management**: Prevent expensive SKUs in non-production
- **Networking**: Enforce NSG rules, disable public IPs

Example policy assignment in Terraform:
```hcl
resource "azurerm_policy_assignment" "require_tags" {
  name                 = "require-tags"
  scope               = azurerm_resource_group.main.id
  policy_definition_id = "/providers/Microsoft.Authorization/policyDefinitions/1e30110a-5ceb-460c-a204-c1c3969c6d62"
}
```

### Question 13: Data Protection and Backup
**Scenario:** Design a backup strategy for the infrastructure and application data in this Azure environment.

**Answer:**
**Infrastructure Backup:**
- Terraform state files: Automated backups to separate storage account
- Key Vault: Soft delete enabled, backup keys and certificates
- Configuration: Git-based storage with tags for versions

**Application Backup:**
- Database: Point-in-time restore, geo-redundant backups
- File storage: Azure Backup for persistent volumes
- Container images: Multi-region ACR replication
- Application data: Application-specific backup strategies

**Disaster Recovery:**
- Infrastructure: Terraform can recreate in alternate region
- Data: Cross-region replication and backup
- RTO/RPO: Define recovery time and point objectives

### Question 14: Cost Optimization
**Scenario:** What cost optimization strategies would you implement for this Azure infrastructure?

**Answer:**
1. **Resource Right-sizing**: Monitor utilization and adjust SKUs
2. **Reserved Instances**: Purchase reservations for predictable workloads
3. **Auto-scaling**: Implement HPA and VPA for dynamic scaling
4. **Development/Test pricing**: Use dev/test subscriptions and pricing
5. **Resource cleanup**: Automated deletion of temporary resources
6. **Storage optimization**: Choose appropriate storage tiers
7. **Network optimization**: Minimize cross-region traffic
8. **Monitoring**: Cost alerts and budgets

### Question 15: Terraform Best Practices
**Scenario:** Code review this Terraform configuration. What improvements would you suggest?

```hcl
resource "azurerm_resource_group" "rg" {
  name     = "my-rg"
  location = "East US"
}

resource "azurerm_storage_account" "storage" {
  name                     = "storage123"
  resource_group_name      = "my-rg"
  location                 = "East US"
  account_tier             = "Standard"
  account_replication_type = "LRS"
}
```

**Answer:**
Improvements needed:
```hcl
resource "azurerm_resource_group" "main" {
  name     = "${var.project_name}-${var.environment}-rg"
  location = var.azure_region
  
  tags = local.common_tags
}

resource "azurerm_storage_account" "main" {
  name                     = "${var.project_name}${var.environment}sta"
  resource_group_name      = azurerm_resource_group.main.name
  location                = azurerm_resource_group.main.location
  account_tier             = var.storage_account_tier
  account_replication_type = var.storage_replication_type
  
  network_rules {
    default_action = "Deny"
    ip_rules       = var.allowed_ip_ranges
  }
  
  tags = local.common_tags
}
```

Issues fixed:
- Hard-coded values → variables
- Missing tags
- Inconsistent naming
- No network security
- Resource references instead of hard-coding

### Question 16: Multi-Tenant Architecture
**Scenario:** Explain how the `mh` (multi-tenant hub) directory structure supports multiple tenants or applications.

**Answer:**
The MH structure provides:
- **Isolation**: Separate deployments per tenant/application
- **Shared infrastructure**: Common services (networking, monitoring)
- **Resource organization**: By service type and environment
- **Security boundaries**: RBAC and network policies per tenant
- **Scaling**: Independent scaling per tenant
- **Billing**: Separate cost tracking and chargeback
- **Compliance**: Tenant-specific compliance requirements

Tenant onboarding process:
1. Create tenant-specific resource groups
2. Deploy baseline infrastructure
3. Configure networking and security
4. Set up monitoring and observability
5. Deploy application infrastructure
6. Configure backup and disaster recovery

### Question 17: Infrastructure Validation
**Scenario:** How would you implement automated testing and validation for this Terraform infrastructure?

**Answer:**
**Pre-deployment validation:**
- `terraform validate`: Syntax validation
- `terraform plan`: Change preview
- `tflint`: Terraform linting
- `terraform-compliance`: Policy-as-code testing
- `conftest`: OPA policy checking

**Post-deployment testing:**
- `terratest`: Go-based infrastructure testing
- Azure Resource Graph queries for compliance
- Health checks for deployed resources
- Integration testing for service connectivity

**Continuous validation:**
- Daily drift detection runs
- Security compliance scanning
- Performance monitoring
- Cost variance alerts

Example terratest:
```go
func TestResourceGroupExists(t *testing.T) {
    resourceGroupName := "test-rg"
    subscriptionID := os.Getenv("AZURE_SUBSCRIPTION_ID")
    
    exists := azure.ResourceGroupExists(t, resourceGroupName, subscriptionID)
    assert.True(t, exists)
}
```

### Question 18: Secret Management Strategy
**Scenario:** Design a comprehensive secret management strategy using Azure Key Vault for this infrastructure.

**Answer:**
**Secret Types and Storage:**
- Database passwords: Azure Key Vault secrets
- API keys: Key Vault secrets with rotation
- Certificates: Key Vault certificates with auto-renewal
- SSH keys: Key Vault secrets or Azure Key Vault HSM

**Access Control:**
- Service principals with minimal permissions
- Azure AD integration for human access
- Kubernetes CSI driver for pod access
- Time-bound access using PIM

**Rotation Strategy:**
- Automated rotation using Azure Functions
- Deployment pipeline integration
- Zero-downtime rotation procedures
- Audit logging for all access

**Integration:**
```yaml
# Kubernetes integration
apiVersion: v1
kind: SecretProviderClass
metadata:
  name: app-secrets
spec:
  provider: azure
  parameters:
    keyvaultName: "shared-kv"
    objects: |
      array:
        - |
          objectName: database-password
          objectType: secret
```

### Question 19: Monitoring and Observability
**Scenario:** Design a comprehensive monitoring strategy for this Azure infrastructure.

**Answer:**
**Infrastructure Monitoring:**
- Azure Monitor for resource metrics
- Log Analytics for centralized logging
- Application Insights for application telemetry
- Azure Service Health for service status

**Alerting Strategy:**
- Resource health degradation
- Performance threshold breaches
- Security events and anomalies
- Cost budget exceedances

**Dashboards:**
- Infrastructure overview dashboard
- Application performance dashboard
- Security and compliance dashboard
- Cost management dashboard

**Integration with DataDog:**
```hcl
resource "azurerm_eventhub_namespace" "monitoring" {
  name                = "${var.project}-monitoring-eh"
  location           = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
  sku                = "Standard"
}

resource "azurerm_monitor_diagnostic_setting" "datadog" {
  name                       = "datadog-integration"
  target_resource_id         = azurerm_kubernetes_cluster.main.id
  eventhub_name             = azurerm_eventhub.datadog.name
  eventhub_authorization_rule_id = azurerm_eventhub_authorization_rule.datadog.id
}
```

### Question 20: Disaster Recovery Planning
**Scenario:** Design a disaster recovery plan for this multi-region Azure infrastructure.

**Answer:**
**RTO/RPO Requirements:**
- Critical applications: RTO 1 hour, RPO 15 minutes
- Standard applications: RTO 4 hours, RPO 1 hour
- Development: RTO 24 hours, RPO 24 hours

**Multi-Region Strategy:**
- Primary region: Production workloads
- Secondary region: DR site with minimal resources
- Tertiary region: Backup and long-term storage

**Infrastructure DR:**
- Terraform code in git for infrastructure recreation
- ARM templates for rapid deployment
- Azure Site Recovery for VM replication
- Database geo-replication

**Data Protection:**
- Automated daily backups
- Cross-region backup replication
- Point-in-time recovery capability
- Regular restore testing

**Failover Process:**
1. Detect primary region failure
2. Activate secondary region resources
3. Update DNS to point to secondary region
4. Validate application functionality
5. Communicate status to stakeholders

---

## Kubernetes Fundamentals

### Question 21: Kubernetes Architecture in ECOS
**Scenario:** Explain how the `gc-ecos-applications-live` repository organizes Kubernetes deployments across environments.

**Answer:**
The repository uses a GitOps structure with:
- **Environment separation**: `/runtimes/{dev,stg,test,uat}/`
- **Resource organization**: Separate directories for apps, configmaps, secrets, istio, apim, synchub
- **Values override pattern**: Environment-specific values in `/values/` subdirectories
- **Application definitions**: Declarative YAML manifests for each application
- **Platform components**: Shared platform services and configurations

This enables:
- Environment promotion through Git workflows
- Consistent application definitions across environments
- Separation of configuration from code
- Audit trail through Git history

### Question 22: Application Deployment Manifest Analysis
**Scenario:** Analyze this application manifest from `runtimes/dev/apps/gc-mambu.yaml`. What improvements would you suggest?

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: gc-mambu
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://charts.grandcentral.com
    chart: gc-connector
    targetRevision: "1.0.0"
    helm:
      valueFiles:
        - values.yaml
      values: |
        image:
          repository: acr.azurecr.io/gc-mambu
          tag: "latest"
  destination:
    namespace: mambu
    server: https://kubernetes.default.svc
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

**Answer:**
Improvements needed:

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: gc-mambu
  namespace: argocd
  labels:
    environment: dev
    application: mambu
    team: integration
  annotations:
    argocd.argoproj.io/sync-wave: "2"
spec:
  project: tenant-applications  # Dedicated project instead of default
  source:
    repoURL: https://charts.grandcentral.com
    chart: gc-connector
    targetRevision: "1.2.3"  # Specific version instead of "1.0.0"
    helm:
      valueFiles:
        - values/mambu/dev.yaml  # Environment-specific values
      parameters:
        - name: image.tag
          value: "sha256:abc123..."  # Specific digest instead of "latest"
  destination:
    namespace: mambu
    server: https://kubernetes.default.svc
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
      - PruneLast=true
    retry:
      limit: 5
      backoff:
        duration: 5s
        factor: 2
        maxDuration: 3m
  ignoreDifferences:
    - group: v1
      kind: Secret
      jsonPointers:
        - /data
```

Key improvements:
- Specific version tags instead of "latest"
- Environment-specific value files
- Proper labeling and annotations
- Sync wave ordering
- Retry policies
- Dedicated ArgoCD project

### Question 23: Kubernetes Service Mesh with Istio
**Scenario:** Based on the Istio configuration in `runtimes/dev/istio/values.yaml`, explain how service mesh is implemented in this environment.

**Answer:**
Istio provides:
- **Traffic Management**: Routing rules, load balancing, circuit breakers
- **Security**: mTLS encryption, authentication policies, authorization
- **Observability**: Distributed tracing, metrics collection, logging
- **Policy Enforcement**: Rate limiting, access control

Configuration approach:
```yaml
# Istio Gateway
apiVersion: networking.istio.io/v1beta1
kind: Gateway
metadata:
  name: mambu-gateway
spec:
  selector:
    istio: ingressgateway
  servers:
  - port:
      number: 443
      name: https
      protocol: HTTPS
    tls:
      mode: SIMPLE
      credentialName: mambu-tls-secret
    hosts:
    - mambu.grandcentral.com
```

### Question 24: ConfigMap Management Strategy
**Scenario:** How would you manage configuration changes across environments using the ConfigMap structure in `runtimes/*/configmaps/`?

**Answer:**
**Environment-specific configurations:**
- Base configuration in common ConfigMaps
- Environment overrides in environment directories
- Sensitive data moved to Secrets
- Version control through Git
- Validation through admission controllers

**Best practices:**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: mambu-config
  namespace: mambu
  labels:
    app: mambu
    environment: dev
data:
  application.properties: |
    mambu.api.url=https://dev.mambu.com/api
    mambu.api.timeout=30000
    logging.level.root=INFO
    spring.profiles.active=dev
```

**Change management:**
1. Update configuration in Git
2. ArgoCD detects changes
3. Automatic rollout with health checks
4. Rollback capability if issues detected

### Question 25: Secret Management in Kubernetes
**Scenario:** Design a secure secret management strategy for the applications deployed in this Kubernetes environment.

**Answer:**
**Secret Sources:**
- Azure Key Vault with CSI driver integration
- Kubernetes native secrets for non-sensitive config
- External secret operator for automated sync

**Implementation:**
```yaml
apiVersion: secrets-store.csi.x-k8s.io/v1
kind: SecretProviderClass
metadata:
  name: mambu-secrets
  namespace: mambu
spec:
  provider: azure
  parameters:
    keyvaultName: "shared-keyvault"
    cloudName: ""
    objects: |
      array:
        - |
          objectName: mambu-api-key
          objectType: secret
          objectVersion: ""
        - |
          objectName: mambu-db-password
          objectType: secret
          objectVersion: ""
    tenantId: "tenant-id"
```

**Security measures:**
- Secret rotation policies
- Audit logging for secret access
- Pod security contexts
- Network policies for secret access
- Encryption at rest and in transit

### Question 26: Namespace Strategy and Multi-Tenancy
**Scenario:** Design a namespace strategy for multi-tenant applications based on the current deployment structure.

**Answer:**
**Namespace Design:**
```yaml
# Per-application namespace
apiVersion: v1
kind: Namespace
metadata:
  name: mambu
  labels:
    tenant: mambu
    environment: dev
    team: integration
  annotations:
    opa.policy.bundle: "tenant-policies"
spec: {}
---
# Resource quotas per namespace
apiVersion: v1
kind: ResourceQuota
metadata:
  name: mambu-quota
  namespace: mambu
spec:
  hard:
    requests.cpu: "4"
    requests.memory: 8Gi
    limits.cpu: "8"
    limits.memory: 16Gi
    persistentvolumeclaims: "10"
```

**Network isolation:**
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: mambu-network-policy
  namespace: mambu
spec:
  podSelector:
    matchLabels:
      app: mambu
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          name: istio-system
    - namespaceSelector:
        matchLabels:
          name: platform
  egress:
  - to:
    - namespaceSelector:
        matchLabels:
          name: shared-services
```

### Question 27: Kubernetes Resource Management
**Scenario:** How would you implement proper resource requests and limits for the applications in this environment?

**Answer:**
**Resource specification strategy:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mambu-connector
spec:
  template:
    spec:
      containers:
      - name: mambu-connector
        image: acr.azurecr.io/gc-mambu:v1.2.3
        resources:
          requests:
            memory: "512Mi"
            cpu: "250m"
            ephemeral-storage: "1Gi"
          limits:
            memory: "1Gi"
            cpu: "500m"
            ephemeral-storage: "2Gi"
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 60
          periodSeconds: 30
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
```

**Horizontal Pod Autoscaler:**
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: mambu-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: mambu-connector
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
```

### Question 28: Platform Values Configuration
**Scenario:** Explain the platform values configuration in `runtimes/dev/values/platform.values.yaml` and how it supports multiple applications.

**Answer:**
Platform values provide:
- **Shared configurations**: Common settings across applications
- **Environment-specific overrides**: Different values per environment
- **Platform services**: Monitoring, logging, security configurations
- **Infrastructure settings**: Cluster-specific configurations

Example structure:
```yaml
# platform.values.yaml
global:
  environment: dev
  registry: acr.azurecr.io
  imagePullPolicy: Always
  
monitoring:
  enabled: true
  datadog:
    apiKey: ${DATADOG_API_KEY}
    clusterName: gc-dev-aks
  
security:
  psp:
    enabled: true
  networkPolicies:
    enabled: true
  
ingress:
  className: istio
  annotations:
    kubernetes.io/ingress.class: istio
  
storage:
    storageClass: managed-premium
    backup:
      enabled: true
      schedule: "0 2 * * *"
```

### Question 29: Health Checks and Probes
**Scenario:** Design comprehensive health checking strategy for the applications deployed in this environment.

**Answer:**
**Three types of probes:**

1. **Liveness Probe**: Restart container if unhealthy
```yaml
livenessProbe:
  httpGet:
    path: /actuator/health/liveness
    port: 8080
    scheme: HTTP
  initialDelaySeconds: 120
  periodSeconds: 30
  timeoutSeconds: 5
  failureThreshold: 3
  successThreshold: 1
```

2. **Readiness Probe**: Remove from load balancer if not ready
```yaml
readinessProbe:
  httpGet:
    path: /actuator/health/readiness
    port: 8080
    scheme: HTTP
  initialDelaySeconds: 30
  periodSeconds: 10
  timeoutSeconds: 5
  failureThreshold: 3
  successThreshold: 1
```

3. **Startup Probe**: Handle slow-starting containers
```yaml
startupProbe:
  httpGet:
    path: /actuator/health/startup
    port: 8080
  initialDelaySeconds: 30
  periodSeconds: 10
  timeoutSeconds: 5
  failureThreshold: 20
  successThreshold: 1
```

**Application health endpoint implementation (Spring Boot):**
```java
@Component
public class MambuHealthIndicator implements HealthIndicator {
    @Override
    public Health health() {
        // Check external dependencies
        if (mambuApiService.isHealthy()) {
            return Health.up()
                .withDetail("mambu-api", "Available")
                .build();
        }
        return Health.down()
            .withDetail("mambu-api", "Unavailable")
            .build();
    }
}
```

### Question 30: Kubernetes Storage Management
**Scenario:** Design a storage strategy for persistent data in the applications deployed in this environment.

**Answer:**
**Storage Classes:**
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: managed-premium-retain
provisioner: disk.csi.azure.com
parameters:
  storageaccounttype: Premium_LRS
  kind: Managed
reclaimPolicy: Retain
allowVolumeExpansion: true
volumeBindingMode: WaitForFirstConsumer
```

**Persistent Volume Claims:**
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mambu-data
  namespace: mambu
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: managed-premium-retain
  resources:
    requests:
      storage: 10Gi
```

**Backup Strategy:**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: backup-script
data:
  backup.sh: |
    #!/bin/bash
    kubectl exec -n mambu deployment/mambu-db -- \
      pg_dump -U postgres mambu_db | \
      gzip > /backup/mambu-$(date +%Y%m%d-%H%M%S).sql.gz
    
    # Upload to Azure Storage
    az storage blob upload \
      --account-name backupsa \
      --container-name mambu-backups \
      --file /backup/mambu-$(date +%Y%m%d-%H%M%S).sql.gz
```

### Question 31: Service Account and RBAC
**Scenario:** Design RBAC (Role-Based Access Control) for the applications in this Kubernetes environment.

**Answer:**
**Service Account per application:**
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: mambu-service-account
  namespace: mambu
  annotations:
    azure.workload.identity/client-id: "client-id-for-mambu"
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: mambu
  name: mambu-role
rules:
- apiGroups: [""]
  resources: ["configmaps", "secrets"]
  verbs: ["get", "list"]
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: mambu-role-binding
  namespace: mambu
subjects:
- kind: ServiceAccount
  name: mambu-service-account
  namespace: mambu
roleRef:
  kind: Role
  name: mambu-role
  apiGroup: rbac.authorization.k8s.io
```

**Pod Security Standards:**
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: mambu
  labels:
    pod-security.kubernetes.io/enforce: restricted
    pod-security.kubernetes.io/audit: restricted
    pod-security.kubernetes.io/warn: restricted
```

### Question 32: Kubernetes Networking and DNS
**Scenario:** Explain how DNS resolution works for services in this Kubernetes environment and how to troubleshoot DNS issues.

**Answer:**
**DNS Configuration:**
```yaml
# Service definition
apiVersion: v1
kind: Service
metadata:
  name: mambu-service
  namespace: mambu
spec:
  selector:
    app: mambu
  ports:
  - port: 80
    targetPort: 8080
    protocol: TCP
```

**DNS Resolution Patterns:**
- Within namespace: `mambu-service`
- Cross-namespace: `mambu-service.mambu.svc.cluster.local`
- External: `mambu.grandcentral.com`

**DNS Troubleshooting:**
1. **Check CoreDNS pods:**
```bash
kubectl get pods -n kube-system -l k8s-app=kube-dns
kubectl logs -n kube-system -l k8s-app=kube-dns
```

2. **Test DNS resolution:**
```bash
kubectl run dns-test --image=busybox --rm -it -- nslookup mambu-service.mambu.svc.cluster.local
```

3. **Check service endpoints:**
```bash
kubectl get endpoints mambu-service -n mambu
```

### Question 33: Kubernetes Deployment Strategies
**Scenario:** Implement a canary deployment strategy for one of the applications in this environment.

**Answer:**
**Canary Deployment with Istio:**

1. **Base deployment (stable):**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mambu-stable
spec:
  replicas: 3
  template:
    metadata:
      labels:
        app: mambu
        version: stable
    spec:
      containers:
      - name: mambu
        image: acr.azurecr.io/gc-mambu:v1.2.3
```

2. **Canary deployment:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mambu-canary
spec:
  replicas: 1
  template:
    metadata:
      labels:
        app: mambu
        version: canary
    spec:
      containers:
      - name: mambu
        image: acr.azurecr.io/gc-mambu:v1.3.0
```

3. **Traffic splitting with Istio:**
```yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: mambu-vs
spec:
  hosts:
  - mambu-service
  http:
  - match:
    - headers:
        canary:
          exact: "true"
    route:
    - destination:
        host: mambu-service
        subset: canary
  - route:
    - destination:
        host: mambu-service
        subset: stable
      weight: 90
    - destination:
        host: mambu-service
        subset: canary
      weight: 10
```

### Question 34: Pod Security and Compliance
**Scenario:** Implement security best practices for pods in this environment including Pod Security Standards and security contexts.

**Answer:**
**Secure Pod Configuration:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mambu-secure
spec:
  template:
    spec:
      serviceAccountName: mambu-service-account
      securityContext:
        runAsNonRoot: true
        runAsUser: 10001
        runAsGroup: 10001
        fsGroup: 10001
        seccompProfile:
          type: RuntimeDefault
      containers:
      - name: mambu
        image: acr.azurecr.io/gc-mambu:v1.2.3
        securityContext:
          allowPrivilegeEscalation: false
          readOnlyRootFilesystem: true
          runAsNonRoot: true
          runAsUser: 10001
          capabilities:
            drop:
            - ALL
        volumeMounts:
        - name: tmp-volume
          mountPath: /tmp
        - name: cache-volume
          mountPath: /app/cache
      volumes:
      - name: tmp-volume
        emptyDir: {}
      - name: cache-volume
        emptyDir: {}
```

**Network Security:**
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: mambu-netpol
  namespace: mambu
spec:
  podSelector:
    matchLabels:
      app: mambu
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: api-gateway
    ports:
    - protocol: TCP
      port: 8080
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: database
    ports:
    - protocol: TCP
      port: 5432
  - to: []
    ports:
    - protocol: TCP
      port: 443  # HTTPS outbound
    - protocol: TCP
      port: 53   # DNS
    - protocol: UDP
      port: 53   # DNS
```

### Question 35: Kubernetes Monitoring and Observability
**Scenario:** Design a comprehensive monitoring setup for applications deployed in this Kubernetes environment.

**Answer:**
**Prometheus Configuration:**
```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: mambu-metrics
  namespace: mambu
spec:
  selector:
    matchLabels:
      app: mambu
  endpoints:
  - port: metrics
    path: /actuator/prometheus
    interval: 30s
    scrapeTimeout: 10s
```

**Custom Metrics:**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: mambu-alerts
data:
  alerts.yaml: |
    groups:
    - name: mambu.rules
      rules:
      - alert: MambuHighErrorRate
        expr: rate(http_requests_total{job="mambu",status=~"5.."}[5m]) > 0.1
        for: 5m
        labels:
          severity: warning
          team: integration
        annotations:
          summary: "High error rate detected in Mambu service"
          description: "Mambu service has error rate of {{ $value }} requests per second"
      
      - alert: MambuHighLatency
        expr: histogram_quantile(0.95, rate(http_request_duration_seconds_bucket{job="mambu"}[5m])) > 2
        for: 10m
        labels:
          severity: critical
          team: integration
        annotations:
          summary: "High latency detected in Mambu service"
          description: "95th percentile latency is {{ $value }} seconds"
```

**Distributed Tracing with Jaeger:**
```yaml
apiVersion: jaegertracing.io/v1
kind: Jaeger
metadata:
  name: mambu-tracing
  namespace: observability
spec:
  strategy: production
  storage:
    type: elasticsearch
    options:
      es:
        server-urls: http://elasticsearch:9200
  ingress:
    enabled: true
    hosts:
      - jaeger.grandcentral.com
```

---

## Kubernetes Application Deployment

### Question 36: Helm Chart Structure
**Scenario:** You need to create a Helm chart for a new connector application. Design the chart structure based on the existing patterns in this environment.

**Answer:**
**Chart structure:**
```
gc-connector/
├── Chart.yaml
├── values.yaml
├── values-dev.yaml
├── values-staging.yaml
├── values-prod.yaml
├── templates/
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── configmap.yaml
│   ├── secret.yaml
│   ├── serviceaccount.yaml
│   ├── hpa.yaml
│   ├── pdb.yaml
│   ├── networkpolicy.yaml
│   ├── servicemonitor.yaml
│   └── tests/
│       └── test-connection.yaml
└── crds/
    └── customresource.yaml
```

**Chart.yaml:**
```yaml
apiVersion: v2
name: gc-connector
description: Central Connector Helm Chart
type: application
version: 1.0.0
appVersion: "1.2.3"
home: https://grandcentral.com
sources:
  - https://github.com/backbase-grand-central/gc-connector
maintainers:
  - name: Platform Team
    email: platform@grandcentral.com
dependencies:
  - name: postgresql
    version: "11.x.x"
    repository: "https://charts.bitnami.com/bitnami"
    condition: postgresql.enabled
```

**values.yaml with all configuration options:**
```yaml
replicaCount: 2

image:
  repository: acr.azurecr.io/gc-connector
  tag: ""
  pullPolicy: IfNotPresent

nameOverride: ""
fullnameOverride: ""

serviceAccount:
  create: true
  annotations: {}
  name: ""

podAnnotations: {}
podSecurityContext:
  fsGroup: 10001
  runAsNonRoot: true
  runAsUser: 10001

securityContext:
  allowPrivilegeEscalation: false
  readOnlyRootFilesystem: true
  runAsNonRoot: true
  runAsUser: 10001
  capabilities:
    drop:
    - ALL

service:
  type: ClusterIP
  port: 80
  targetPort: 8080

ingress:
  enabled: true
  className: "istio"
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
  hosts:
    - host: connector.grandcentral.com
      paths:
        - path: /
          pathType: Prefix
  tls:
    - secretName: connector-tls
      hosts:
        - connector.grandcentral.com

resources:
  requests:
    memory: "512Mi"
    cpu: "250m"
  limits:
    memory: "1Gi"
    cpu: "500m"

autoscaling:
  enabled: true
  minReplicas: 2
  maxReplicas: 10
  targetCPUUtilizationPercentage: 70
  targetMemoryUtilizationPercentage: 80

nodeSelector: {}
tolerations: []
affinity: {}

config:
  logLevel: INFO
  database:
    host: postgresql
    port: 5432
    name: connector_db
  external:
    api:
      timeout: 30000
      retries: 3
```

### Question 37: ArgoCD Application of Applications Pattern
**Scenario:** Implement an "App of Apps" pattern for managing multiple applications across environments using ArgoCD.

**Answer:**
**Root application:**
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: gc-platform-dev
  namespace: argocd
  finalizers:
    - resources-finalizer.argocd.argoproj.io
spec:
  project: platform
  source:
    repoURL: https://github.com/backbase-grand-central/gc-ecos-applications-live
    targetRevision: main
    path: runtimes/dev/platform
  destination:
    server: https://kubernetes.default.svc
    namespace: argocd
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
```

**Platform application manifests:**
```yaml
# runtimes/dev/platform/mambu-app.yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: gc-mambu
  namespace: argocd
spec:
  project: tenant-applications
  source:
    repoURL: https://charts.grandcentral.com
    chart: gc-connector
    targetRevision: "1.2.3"
    helm:
      valueFiles:
        - ../../values/mambu/dev.yaml
  destination:
    namespace: mambu
    server: https://kubernetes.default.svc
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
    syncOptions:
      - CreateNamespace=true
    retry:
      limit: 5
      backoff:
        duration: 5s
        factor: 2
        maxDuration: 3m
  ignoreDifferences:
    - group: v1
      kind: Secret
      jsonPointers:
        - /data
```

**Benefits of App of Apps:**
- Centralized management of all applications
- Environment promotion through Git
- Consistent application configurations
- Dependency management between apps
- Simplified disaster recovery

### Question 38: Blue-Green Deployment with ArgoCD
**Scenario:** Implement a blue-green deployment strategy for a critical application using ArgoCD and Istio.

**Answer:**
**ArgoCD Rollout configuration:**
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: mambu-rollout
  namespace: mambu
spec:
  replicas: 5
  strategy:
    blueGreen:
      activeService: mambu-active
      previewService: mambu-preview
      autoPromotionEnabled: false
      scaleDownDelaySeconds: 30
      prePromotionAnalysis:
        templates:
        - templateName: success-rate
        args:
        - name: service-name
          value: mambu-preview
      postPromotionAnalysis:
        templates:
        - templateName: success-rate
        args:
        - name: service-name
          value: mambu-active
  selector:
    matchLabels:
      app: mambu
  template:
    metadata:
      labels:
        app: mambu
    spec:
      containers:
      - name: mambu
        image: acr.azurecr.io/gc-mambu:v1.3.0
        ports:
        - containerPort: 8080
          protocol: TCP
```

**Services for blue-green:**
```yaml
apiVersion: v1
kind: Service
metadata:
  name: mambu-active
  namespace: mambu
spec:
  selector:
    app: mambu
  ports:
  - port: 80
    targetPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: mambu-preview
  namespace: mambu
spec:
  selector:
    app: mambu
  ports:
  - port: 80
    targetPort: 8080
```

**Analysis template for automated promotion:**
```yaml
apiVersion: argoproj.io/v1alpha1
kind: AnalysisTemplate
metadata:
  name: success-rate
spec:
  args:
  - name: service-name
  metrics:
  - name: success-rate
    interval: 60s
    count: 5
    successCondition: result[0] >= 0.95
    provider:
      prometheus:
        address: http://prometheus.monitoring.svc:9090
        query: |
          sum(rate(
            http_requests_total{job="{{args.service-name}}",status!~"5.*"}[5m]
          )) /
          sum(rate(
            http_requests_total{job="{{args.service-name}}"}[5m]
          ))
```

### Question 39: Application Configuration Management
**Scenario:** Design a configuration management strategy that supports environment-specific configurations while maintaining consistency.

**Answer:**
**Kustomize overlay structure:**
```
overlays/
├── base/
│   ├── kustomization.yaml
│   ├── deployment.yaml
│   ├── service.yaml
│   └── configmap.yaml
├── dev/
│   ├── kustomization.yaml
│   ├── config-patch.yaml
│   └── replica-patch.yaml
├── staging/
│   ├── kustomization.yaml
│   ├── config-patch.yaml
│   └── resource-patch.yaml
└── production/
    ├── kustomization.yaml
    ├── config-patch.yaml
    └── security-patch.yaml
```

**Base kustomization.yaml:**
```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - deployment.yaml
  - service.yaml
  - configmap.yaml

commonLabels:
  app: mambu
  version: v1.2.3

namespace: mambu

images:
  - name: mambu
    newTag: v1.2.3
```

**Environment-specific patches:**
```yaml
# dev/config-patch.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: mambu-config
data:
  LOG_LEVEL: DEBUG
  API_ENDPOINT: https://dev-mambu.api.com
  CACHE_TTL: "300"
  RATE_LIMIT: "1000"
---
# staging/config-patch.yaml  
apiVersion: v1
kind: ConfigMap
metadata:
  name: mambu-config
data:
  LOG_LEVEL: INFO
  API_ENDPOINT: https://staging-mambu.api.com
  CACHE_TTL: "600"
  RATE_LIMIT: "500"
```

**Production security patches:**
```yaml
# production/security-patch.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mambu
spec:
  template:
    spec:
      securityContext:
        runAsNonRoot: true
        runAsUser: 10001
        fsGroup: 10001
      containers:
      - name: mambu
        securityContext:
          allowPrivilegeEscalation: false
          readOnlyRootFilesystem: true
          capabilities:
            drop:
            - ALL
```

### Question 40: Container Security Scanning
**Scenario:** Implement container security scanning in the CI/CD pipeline for applications deployed to this Kubernetes environment.

**Answer:**
**GitHub Actions workflow with security scanning:**
```yaml
name: Container Security Scan
on:
  pull_request:
    paths:
      - 'Dockerfile'
      - 'src/**'
  push:
    branches:
      - main

jobs:
  security-scan:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    
    - name: Build container image
      run: |
        docker build -t mambu-connector:${{ github.sha }} .
        
    - name: Run Trivy vulnerability scanner
      uses: aquasecurity/trivy-action@master
      with:
        image-ref: 'mambu-connector:${{ github.sha }}'
        format: 'sarif'
        output: 'trivy-results.sarif'
        severity: 'CRITICAL,HIGH'
        
    - name: Upload Trivy scan results to GitHub Security
      uses: github/codeql-action/upload-sarif@v2
      if: always()
      with:
        sarif_file: 'trivy-results.sarif'
        
    - name: Run Snyk container scan
      uses: snyk/actions/docker@master
      env:
        SNYK_TOKEN: ${{ secrets.SNYK_TOKEN }}
      with:
        image: mambu-connector:${{ github.sha }}
        args: --severity-threshold=high
        
    - name: Push to ACR only if scans pass
      if: success()
      run: |
        echo ${{ secrets.ACR_PASSWORD }} | docker login acr.azurecr.io -u ${{ secrets.ACR_USERNAME }} --password-stdin
        docker tag mambu-connector:${{ github.sha }} acr.azurecr.io/gc-mambu:${{ github.sha }}
        docker push acr.azurecr.io/gc-mambu:${{ github.sha }}
```

**Kubernetes admission controller for image scanning:**
```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: require-scanned-images
spec:
  validationFailureAction: enforce
  background: false
  rules:
  - name: check-image-scan
    match:
      any:
      - resources:
          kinds:
          - Pod
          - Deployment
          - StatefulSet
          - DaemonSet
    validate:
      message: "Image must be from allowed registry and scanned"
      pattern:
        spec:
          containers:
          - image: "acr.azurecr.io/*"
      anyPattern:
      - spec:
          containers:
          - image: "*/*/gc-*:sha256:*"  # Require digest-based images
```

---

## Kubernetes Operators

### Question 41: ASO (Azure Service Operator) Implementation
**Scenario:** Use the Azure Service Operator to provision Azure resources from Kubernetes for one of the applications in this environment.

**Answer:**
**Install ASO operator:**
```bash
# Install using Helm
helm repo add aso2 https://raw.githubusercontent.com/Azure/azure-service-operator/main/v2/charts
helm upgrade --install aso2 aso2/azure-service-operator \
  --create-namespace \
  --namespace=azureserviceoperator-system \
  --set azureSubscriptionID=$AZURE_SUBSCRIPTION_ID \
  --set azureTenantID=$AZURE_TENANT_ID \
  --set azureClientID=$AZURE_CLIENT_ID
```

**Provision PostgreSQL using ASO:**
```yaml
apiVersion: dbforpostgresql.azure.com/v1api20210601
kind: FlexibleServer
metadata:
  name: mambu-postgres
  namespace: mambu
spec:
  location: East US
  owner:
    name: mambu-rg
  administratorLogin: pgadmin
  administratorLoginPassword:
    name: postgres-admin-password
    key: password
  storage:
    storageSizeGB: 32
  sku:
    name: Standard_B1ms
    tier: Burstable
  version: "13"
  highAvailability:
    mode: Disabled
  backup:
    backupRetentionDays: 7
    geoRedundantBackup: Disabled
---
apiVersion: dbforpostgresql.azure.com/v1api20210601
kind: FlexibleServersDatabase
metadata:
  name: mambu-database
  namespace: mambu
spec:
  owner:
    name: mambu-postgres
  charset: utf8
  collation: en_US.utf8
```

**Benefits of ASO:**
- Infrastructure as Code within Kubernetes
- GitOps workflow for both applications and infrastructure
- Kubernetes RBAC for infrastructure provisioning
- Consistent API across cloud resources
- Integration with Kubernetes lifecycle management

### Question 42: Knative Implementation
**Scenario:** Implement Knative for serverless workloads in this environment, specifically for event-driven processing applications.

**Answer:**
**Install Knative Serving:**
```bash
# Install Knative Serving
kubectl apply -f https://github.com/knative/serving/releases/download/knative-v1.8.0/serving-crds.yaml
kubectl apply -f https://github.com/knative/serving/releases/download/knative-v1.8.0/serving-core.yaml

# Install Istio networking
kubectl apply -f https://github.com/knative/net-istio/releases/download/knative-v1.8.0/net-istio.yaml
```

**Knative Service for event processing:**
```yaml
apiVersion: serving.knative.dev/v1
kind: Service
metadata:
  name: mambu-event-processor
  namespace: mambu
  annotations:
    autoscaling.knative.dev/minScale: "0"
    autoscaling.knative.dev/maxScale: "100"
    autoscaling.knative.dev/target: "10"
spec:
  template:
    metadata:
      annotations:
        autoscaling.knative.dev/class: "kpa.autoscaling.knative.dev"
        autoscaling.knative.dev/metric: "concurrency"
    spec:
      containerConcurrency: 10
      timeoutSeconds: 300
      containers:
      - image: acr.azurecr.io/gc-mambu-processor:v1.0.0
        ports:
        - containerPort: 8080
          protocol: TCP
        env:
        - name: MAMBU_API_KEY
          valueFrom:
            secretKeyRef:
              name: mambu-secrets
              key: api-key
        resources:
          requests:
            memory: "128Mi"
            cpu: "100m"
          limits:
            memory: "512Mi"
            cpu: "1000m"
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
```

**Knative Eventing with Azure Service Bus:**
```yaml
apiVersion: eventing.knative.dev/v1
kind: Trigger
metadata:
  name: mambu-transaction-trigger
  namespace: mambu
spec:
  broker: default
  filter:
    attributes:
      type: com.mambu.transaction.created
  subscriber:
    ref:
      apiVersion: serving.knative.dev/v1
      kind: Service
      name: mambu-event-processor
```

### Question 43: Custom Kubernetes Operator
**Scenario:** Design a custom Kubernetes operator for managing Mambu connector configurations and lifecycle.

**Answer:**
**Custom Resource Definition:**
```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: mambuconnectors.connectors.grandcentral.com
spec:
  group: connectors.grandcentral.com
  versions:
  - name: v1
    served: true
    storage: true
    schema:
      openAPIV3Schema:
        type: object
        properties:
          spec:
            type: object
            properties:
              mambuInstance:
                type: string
                description: "Mambu instance URL"
              apiVersion:
                type: string
                enum: ["v1", "v2"]
                default: "v2"
              resources:
                type: object
                properties:
                  cpu:
                    type: string
                    default: "500m"
                  memory:
                    type: string
                    default: "1Gi"
              scaling:
                type: object
                properties:
                  minReplicas:
                    type: integer
                    default: 2
                  maxReplicas:
                    type: integer
                    default: 10
                  targetCPU:
                    type: integer
                    default: 70
          status:
            type: object
            properties:
              phase:
                type: string
                enum: ["Pending", "Running", "Failed"]
              replicas:
                type: integer
              readyReplicas:
                type: integer
              lastUpdateTime:
                type: string
                format: date-time
  scope: Namespaced
  names:
    plural: mambuconnectors
    singular: mambuconnector
    kind: MambuConnector
    shortNames:
    - mc
```

**Operator implementation (using controller-runtime):**
```go
// controllers/mambuconnector_controller.go
func (r *MambuConnectorReconciler) Reconcile(ctx context.Context, req ctrl.Request) (ctrl.Result, error) {
    log := r.Log.WithValues("mambuconnector", req.NamespacedName)
    
    // Fetch the MambuConnector instance
    var connector connectorsv1.MambuConnector
    if err := r.Get(ctx, req.NamespacedName, &connector); err != nil {
        return ctrl.Result{}, client.IgnoreNotFound(err)
    }
    
    // Create or update Deployment
    deployment := r.deploymentForConnector(&connector)
    if err := r.createOrUpdate(ctx, deployment); err != nil {
        return ctrl.Result{}, err
    }
    
    // Create or update Service
    service := r.serviceForConnector(&connector)
    if err := r.createOrUpdate(ctx, service); err != nil {
        return ctrl.Result{}, err
    }
    
    // Create or update HPA
    hpa := r.hpaForConnector(&connector)
    if err := r.createOrUpdate(ctx, hpa); err != nil {
        return ctrl.Result{}, err
    }
    
    // Update status
    connector.Status.Phase = "Running"
    connector.Status.Replicas = deployment.Status.Replicas
    connector.Status.ReadyReplicas = deployment.Status.ReadyReplicas
    connector.Status.LastUpdateTime = metav1.Now()
    
    return ctrl.Result{RequeueAfter: time.Minute * 5}, r.Status().Update(ctx, &connector)
}

func (r *MambuConnectorReconciler) deploymentForConnector(connector *connectorsv1.MambuConnector) *appsv1.Deployment {
    labels := map[string]string{
        "app":        "mambu-connector",
        "connector":  connector.Name,
        "version":    "v1",
    }
    
    return &appsv1.Deployment{
        ObjectMeta: metav1.ObjectMeta{
            Name:      connector.Name,
            Namespace: connector.Namespace,
            Labels:    labels,
        },
        Spec: appsv1.DeploymentSpec{
            Replicas: &connector.Spec.Scaling.MinReplicas,
            Selector: &metav1.LabelSelector{
                MatchLabels: labels,
            },
            Template: corev1.PodTemplateSpec{
                ObjectMeta: metav1.ObjectMeta{
                    Labels: labels,
                },
                Spec: corev1.PodSpec{
                    Containers: []corev1.Container{
                        {
                            Name:  "mambu-connector",
                            Image: "acr.azurecr.io/gc-mambu-connector:latest",
                            Env: []corev1.EnvVar{
                                {
                                    Name:  "MAMBU_INSTANCE_URL",
                                    Value: connector.Spec.MambuInstance,
                                },
                                {
                                    Name:  "API_VERSION",
                                    Value: connector.Spec.APIVersion,
                                },
                            },
                            Resources: corev1.ResourceRequirements{
                                Requests: corev1.ResourceList{
                                    corev1.ResourceCPU:    resource.MustParse(connector.Spec.Resources.CPU),
                                    corev1.ResourceMemory: resource.MustParse(connector.Spec.Resources.Memory),
                                },
                            },
                        },
                    },
                },
            },
        },
    }
}
```

### Question 44: Operator Lifecycle Management
**Scenario:** Implement proper lifecycle management for operators in this Kubernetes environment including upgrades, rollbacks, and monitoring.

**Answer:**
**OLM (Operator Lifecycle Manager) configuration:**
```yaml
apiVersion: operators.coreos.com/v1alpha1
kind: ClusterServiceVersion
metadata:
  name: mambu-operator.v1.0.0
  namespace: mambu-operator-system
spec:
  displayName: Mambu Operator
  description: Manages Mambu connector instances
  version: 1.0.0
  replaces: mambu-operator.v0.9.0
  
  maturity: stable
  provider:
    name: Central Platform Team
  
  installModes:
  - type: OwnNamespace
    supported: true
  - type: SingleNamespace
    supported: true
  - type: MultiNamespace
    supported: false
  - type: AllNamespaces
    supported: true
    
  customresourcedefinitions:
    owned:
    - name: mambuconnectors.connectors.grandcentral.com
      version: v1
      kind: MambuConnector
      displayName: Mambu Connector
      description: Represents a Mambu connector instance
      
  install:
    strategy: deployment
    spec:
      deployments:
      - name: mambu-operator-controller-manager
        spec:
          replicas: 1
          selector:
            matchLabels:
              control-plane: controller-manager
          template:
            metadata:
              labels:
                control-plane: controller-manager
            spec:
              containers:
              - name: manager
                image: acr.azurecr.io/mambu-operator:v1.0.0
                command:
                - /manager
                args:
                - --leader-elect
                resources:
                  limits:
                    cpu: 500m
                    memory: 128Mi
                  requests:
                    cpu: 10m
                    memory: 64Mi
```

**Operator monitoring:**
```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: mambu-operator-metrics
  namespace: mambu-operator-system
spec:
  selector:
    matchLabels:
      control-plane: controller-manager
  endpoints:
  - port: https
    scheme: https
    tlsConfig:
      insecureSkipVerify: true
    bearerTokenFile: /var/run/secrets/kubernetes.io/serviceaccount/token
```

### Question 45: Istio Operator Configuration
**Scenario:** Configure Istio service mesh for this environment using the Istio operator with proper security and observability settings.

**Answer:**
**Install Istio Operator:**
```bash
istioctl operator init
```

**IstioOperator configuration:**
```yaml
apiVersion: install.istio.io/v1alpha1
kind: IstioOperator
metadata:
  name: control-plane
  namespace: istio-system
spec:
  components:
    pilot:
      k8s:
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
        hpaSpec:
          minReplicas: 2
          maxReplicas: 5
    ingressGateways:
    - name: istio-ingressgateway
      enabled: true
      k8s:
        service:
          type: LoadBalancer
          annotations:
            service.beta.kubernetes.io/azure-load-balancer-internal: "true"
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
        hpaSpec:
          minReplicas: 2
          maxReplicas: 5
    egressGateways:
    - name: istio-egressgateway
      enabled: true
      k8s:
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
  values:
    global:
      meshID: mesh1
      multiCluster:
        clusterName: gc-dev-aks
      network: network1
      proxy:
        resources:
          requests:
            cpu: 10m
            memory: 40Mi
    pilot:
      traceSampling: 1.0
    telemetry:
      v2:
        prometheus:
          configOverride:
            inbound_metric_relabeling:
            - source_labels: [__name__]
              regex: 'istio_.*'
              action: keep
```

**Enable automatic sidecar injection:**
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: mambu
  labels:
    istio-injection: enabled
    name: mambu
```

**Istio Gateway and VirtualService:**
```yaml
apiVersion: networking.istio.io/v1beta1
kind: Gateway
metadata:
  name: mambu-gateway
  namespace: mambu
spec:
  selector:
    istio: ingressgateway
  servers:
  - port:
      number: 443
      name: https
      protocol: HTTPS
    tls:
      mode: SIMPLE
      credentialName: mambu-tls
    hosts:
    - mambu.grandcentral.com
---
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: mambu-vs
  namespace: mambu
spec:
  hosts:
  - mambu.grandcentral.com
  gateways:
  - mambu-gateway
  http:
  - match:
    - uri:
        prefix: "/api/v1"
    route:
    - destination:
        host: mambu-service
        port:
          number: 80
    fault:
      delay:
        percentage:
          value: 0.1
        fixedDelay: 5s
    retries:
      attempts: 3
      perTryTimeout: 2s
```

**Authorization Policy:**
```yaml
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: mambu-authz
  namespace: mambu
spec:
  selector:
    matchLabels:
      app: mambu
  rules:
  - from:
    - source:
        principals: ["cluster.local/ns/api-gateway/sa/api-gateway"]
    to:
    - operation:
        methods: ["GET", "POST"]
        paths: ["/api/*"]
  - from:
    - source:
        namespaces: ["monitoring"]
    to:
    - operation:
        methods: ["GET"]
        paths: ["/metrics", "/health"]
```

---

## GitHub and GitHub Actions

### Question 46: CI/CD Pipeline for Kubernetes Applications
**Scenario:** Design a comprehensive CI/CD pipeline using GitHub Actions for the applications in this environment.

**Answer:**
**GitHub Actions workflow:**
```yaml
name: CI/CD Pipeline
on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

env:
  REGISTRY: acr.azurecr.io
  IMAGE_NAME: gc-mambu

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    
    - name: Set up JDK 17
      uses: actions/setup-java@v3
      with:
        java-version: '17'
        distribution: 'temurin'
        
    - name: Cache Maven dependencies
      uses: actions/cache@v3
      with:
        path: ~/.m2
        key: ${{ runner.os }}-m2-${{ hashFiles('**/pom.xml') }}
        
    - name: Run tests
      run: mvn clean test
      
    - name: Generate test report
      uses: dorny/test-reporter@v1
      if: success() || failure()
      with:
        name: Maven Tests
        path: target/surefire-reports/*.xml
        reporter: java-junit
        
    - name: Run SonarCloud analysis
      env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
      run: mvn sonar:sonar

  security-scan:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    
    - name: Run SAST with CodeQL
      uses: github/codeql-action/init@v2
      with:
        languages: java
        
    - name: Build for analysis
      run: mvn compile -DskipTests
      
    - name: Perform CodeQL Analysis
      uses: github/codeql-action/analyze@v2

  build-and-push:
    needs: [test, security-scan]
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    outputs:
      image-digest: ${{ steps.build.outputs.digest }}
    steps:
    - uses: actions/checkout@v3
    
    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v2
      
    - name: Log in to ACR
      uses: azure/docker-login@v1
      with:
        login-server: ${{ env.REGISTRY }}
        username: ${{ secrets.REGISTRY_USERNAME }}
        password: ${{ secrets.REGISTRY_PASSWORD }}
        
    - name: Extract metadata
      id: meta
      uses: docker/metadata-action@v4
      with:
        images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
        tags: |
          type=ref,event=branch
          type=ref,event=pr
          type=sha,prefix={{branch}}-
          
    - name: Build and push Docker image
      id: build
      uses: docker/build-push-action@v4
      with:
        context: .
        platforms: linux/amd64
        push: true
        tags: ${{ steps.meta.outputs.tags }}
        labels: ${{ steps.meta.outputs.labels }}
        cache-from: type=gha
        cache-to: type=gha,mode=max
        
    - name: Generate SBOM
      uses: anchore/sbom-action@v0
      with:
        image: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}@${{ steps.build.outputs.digest }}
        format: spdx-json
        output-file: sbom.spdx.json
        
    - name: Upload SBOM
      uses: actions/upload-artifact@v3
      with:
        name: sbom
        path: sbom.spdx.json

  deploy-dev:
    needs: build-and-push
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    environment: development
    steps:
    - uses: actions/checkout@v3
      with:
        repository: backbase-grand-central/gc-ecos-applications-live
        token: ${{ secrets.GITOPS_TOKEN }}
        
    - name: Update image tag
      run: |
        NEW_TAG="${{ needs.build-and-push.outputs.image-digest }}"
        sed -i "s|image:.*|image: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}@${NEW_TAG}|" \
          runtimes/dev/values/mambu/values.yaml
          
    - name: Commit and push changes
      run: |
        git config --local user.email "action@github.com"
        git config --local user.name "GitHub Action"
        git add .
        git commit -m "Update mambu image to ${{ github.sha }}"
        git push
```

### Question 47: GitHub Actions for Infrastructure
**Scenario:** Create GitHub Actions workflows for managing the Terraform infrastructure in the gc-infra repository.

**Answer:**
**Terraform CI/CD workflow:**
```yaml
name: Terraform Infrastructure
on:
  push:
    branches: [main]
    paths: ['shared/**', 'bootstrap/**']
  pull_request:
    paths: ['shared/**', 'bootstrap/**']

env:
  TF_VERSION: 1.5.0
  ARM_SUBSCRIPTION_ID: ${{ secrets.ARM_SUBSCRIPTION_ID }}
  ARM_CLIENT_ID: ${{ secrets.ARM_CLIENT_ID }}
  ARM_TENANT_ID: ${{ secrets.ARM_TENANT_ID }}

jobs:
  discover-changes:
    runs-on: ubuntu-latest
    outputs:
      directories: ${{ steps.changes.outputs.directories }}
    steps:
    - uses: actions/checkout@v3
      with:
        fetch-depth: 0
        
    - name: Detect changed directories
      id: changes
      run: |
        CHANGED_DIRS=$(git diff --name-only HEAD~1 HEAD | \
          grep -E '\.(tf|tfvars)$' | \
          xargs -I {} dirname {} | \
          sort | uniq | \
          jq -R -s -c 'split("\n")[:-1]')
        echo "directories=$CHANGED_DIRS" >> $GITHUB_OUTPUT

  terraform-plan:
    needs: discover-changes
    if: github.event_name == 'pull_request'
    runs-on: ubuntu-latest
    strategy:
      matrix:
        directory: ${{ fromJson(needs.discover-changes.outputs.directories) }}
    steps:
    - uses: actions/checkout@v3
    
    - name: Setup Terraform
      uses: hashicorp/setup-terraform@v2
      with:
        terraform_version: ${{ env.TF_VERSION }}
        
    - name: Azure CLI login
      uses: azure/login@v1
      with:
        creds: ${{ secrets.AZURE_CREDENTIALS }}
        
    - name: Terraform Init
      working-directory: ${{ matrix.directory }}
      run: terraform init
      
    - name: Terraform Validate
      working-directory: ${{ matrix.directory }}
      run: terraform validate
      
    - name: Terraform Plan
      working-directory: ${{ matrix.directory }}
      run: |
        terraform plan -out=tfplan -no-color
        terraform show -json tfplan > plan.json
        
    - name: Comment PR with Plan
      uses: actions/github-script@v6
      with:
        script: |
          const fs = require('fs');
          const path = '${{ matrix.directory }}';
          const plan = fs.readFileSync(`${path}/plan.json`, 'utf8');
          const body = `## Terraform Plan for \`${path}\`\n\`\`\`json\n${plan}\n\`\`\``;
          
          github.rest.issues.createComment({
            issue_number: context.issue.number,
            owner: context.repo.owner,
            repo: context.repo.repo,
            body: body.substring(0, 65536) // Limit comment size
          });

  terraform-apply:
    needs: discover-changes
    if: github.ref == 'refs/heads/main'
    runs-on: ubuntu-latest
    strategy:
      matrix:
        directory: ${{ fromJson(needs.discover-changes.outputs.directories) }}
    environment: production
    steps:
    - uses: actions/checkout@v3
    
    - name: Setup Terraform
      uses: hashicorp/setup-terraform@v2
      with:
        terraform_version: ${{ env.TF_VERSION }}
        
    - name: Azure CLI login
      uses: azure/login@v1
      with:
        creds: ${{ secrets.AZURE_CREDENTIALS }}
        
    - name: Terraform Init
      working-directory: ${{ matrix.directory }}
      run: terraform init
      
    - name: Terraform Apply
      working-directory: ${{ matrix.directory }}
      run: terraform apply -auto-approve
      
    - name: Upload Terraform State
      if: always()
      working-directory: ${{ matrix.directory }}
      run: |
        # Backup state to separate storage account
        az storage blob upload \
          --account-name tfstatebackup \
          --container-name terraform-state \
          --file terraform.tfstate \
          --name "${{ matrix.directory }}/terraform-$(date +%Y%m%d-%H%M%S).tfstate"
```

### Question 48: Security and Compliance in GitHub Actions
**Scenario:** Implement security best practices and compliance checks in the GitHub Actions workflows for this environment.

**Answer:**
**Security workflow:**
```yaml
name: Security and Compliance
on:
  push:
    branches: [main]
  pull_request:
  schedule:
    - cron: '0 2 * * 1'  # Weekly security scan

jobs:
  secret-scanning:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
      with:
        fetch-depth: 0
        
    - name: Run TruffleHog
      uses: trufflesecurity/trufflehog@main
      with:
        path: ./
        base: main
        head: HEAD
        
    - name: Run GitLeaks
      uses: gitleaks/gitleaks-action@v2
      env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}

  dependency-check:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    
    - name: Setup Java
      uses: actions/setup-java@v3
      with:
        java-version: '17'
        distribution: 'temurin'
        
    - name: OWASP Dependency Check
      uses: dependency-check/Dependency-Check_Action@main
      with:
        project: 'gc-mambu'
        path: '.'
        format: 'JSON'
        args: >
          --enableRetired
          --failOnCVSS 7
          --suppression suppressions.xml
          
    - name: Upload SARIF file
      uses: github/codeql-action/upload-sarif@v2
      with:
        sarif_file: reports/dependency-check-report.sarif

  infrastructure-security:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    
    - name: Run Checkov
      uses: bridgecrewio/checkov-action@master
      with:
        directory: .
        framework: terraform,kubernetes
        output_format: sarif
        output_file_path: reports/checkov.sarif
        
    - name: Upload Checkov results to GitHub Security
      uses: github/codeql-action/upload-sarif@v2
      if: always()
      with:
        sarif_file: reports/checkov.sarif
        
    - name: Run Terraform Security Scan
      uses: aquasecurity/tfsec-action@v1.0.0
      with:
        working_directory: shared/
        
  compliance-check:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    
    - name: Run Open Policy Agent Tests
      run: |
        opa test policies/ --verbose
        
    - name: Validate Kubernetes manifests
      run: |
        find . -name "*.yaml" -o -name "*.yml" | \
          xargs -I {} kubeval {}
          
    - name: Check resource quotas
      run: |
        # Ensure all namespaces have resource quotas
        grep -r "kind: Namespace" . | \
        while read -r namespace_file; do
          namespace_dir=$(dirname "$namespace_file")
          if ! find "$namespace_dir" -name "*quota*" | grep -q .; then
            echo "ERROR: No resource quota found for namespace in $namespace_file"
            exit 1
          fi
        done

  sbom-generation:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    
    - name: Generate SBOM for repository
      uses: anchore/sbom-action@v0
      with:
        path: ./
        format: spdx-json
        output-file: repo-sbom.spdx.json
        
    - name: Upload SBOM to release
      if: github.ref == 'refs/heads/main'
      run: |
        gh release create "sbom-$(date +%Y%m%d)" \
          --title "SBOM $(date +%Y-%m-%d)" \
          --notes "Software Bill of Materials" \
          repo-sbom.spdx.json
      env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

### Question 49: Multi-Environment Promotion Pipeline
**Scenario:** Design a promotion pipeline that automatically deploys applications through dev → staging → production environments.

**Answer:**
**Environment promotion workflow:**
```yaml
name: Environment Promotion
on:
  workflow_run:
    workflows: ["CI/CD Pipeline"]
    branches: [main]
    types: [completed]

jobs:
  deploy-dev:
    if: ${{ github.event.workflow_run.conclusion == 'success' }}
    runs-on: ubuntu-latest
    environment: 
      name: development
      url: https://dev.grandcentral.com
    steps:
    - uses: actions/checkout@v3
      with:
        repository: backbase-grand-central/gc-ecos-applications-live
        token: ${{ secrets.GITOPS_TOKEN }}
        
    - name: Update dev environment
      run: |
        IMAGE_TAG="${{ github.event.workflow_run.head_sha }}"
        yq e ".image.tag = \"$IMAGE_TAG\"" -i runtimes/dev/values/mambu/values.yaml
        git config --local user.email "action@github.com"
        git config --local user.name "GitHub Action"
        git add .
        git commit -m "Deploy $IMAGE_TAG to dev"
        git push
        
    - name: Wait for ArgoCD sync
      run: |
        argocd app sync gc-mambu-dev --prune
        argocd app wait gc-mambu-dev --timeout 600
        
    - name: Run smoke tests
      run: |
        curl -f https://dev.grandcentral.com/mambu/health || exit 1
        ./scripts/smoke-tests.sh dev

  deploy-staging:
    needs: deploy-dev
    runs-on: ubuntu-latest
    environment:
      name: staging
      url: https://staging.grandcentral.com
    steps:
    - uses: actions/checkout@v3
      with:
        repository: backbase-grand-central/gc-ecos-applications-live
        token: ${{ secrets.GITOPS_TOKEN }}
        
    - name: Update staging environment
      run: |
        IMAGE_TAG="${{ github.event.workflow_run.head_sha }}"
        yq e ".image.tag = \"$IMAGE_TAG\"" -i runtimes/stg/values/mambu/values.yaml
        git config --local user.email "action@github.com"
        git config --local user.name "GitHub Action"
        git add .
        git commit -m "Deploy $IMAGE_TAG to staging"
        git push
        
    - name: Run integration tests
      run: |
        ./scripts/integration-tests.sh staging
        
    - name: Performance tests
      run: |
        ./scripts/performance-tests.sh staging

  deploy-production:
    needs: deploy-staging
    runs-on: ubuntu-latest
    environment:
      name: production
      url: https://grandcentral.com
    steps:
    - uses: actions/checkout@v3
      with:
        repository: backbase-grand-central/gc-ecos-applications-live
        token: ${{ secrets.GITOPS_TOKEN }}
        
    - name: Create release PR
      uses: peter-evans/create-pull-request@v5
      with:
        token: ${{ secrets.GITOPS_TOKEN }}
        commit-message: "Deploy ${{ github.event.workflow_run.head_sha }} to production"
        title: "Production Deployment: ${{ github.event.workflow_run.head_sha }}"
        body: |
          ## Production Deployment
          
          **Commit**: ${{ github.event.workflow_run.head_sha }}
          **Dev Tests**: ✅ Passed
          **Staging Tests**: ✅ Passed
          **Performance Tests**: ✅ Passed
          
          ### Changes
          - Update mambu connector to ${{ github.event.workflow_run.head_sha }}
          
          ### Rollback Plan
          ```bash
          argocd app rollback gc-mambu-prod
          ```
        branch: deploy/prod/${{ github.event.workflow_run.head_sha }}
        delete-branch: true
```

### Question 50: GitHub Actions Self-Hosted Runners
**Scenario:** Set up self-hosted GitHub Actions runners in the Azure Kubernetes environment for secure CI/CD operations.

**Answer:**
**Self-hosted runner deployment:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: github-runner
  namespace: github-runners
spec:
  replicas: 3
  selector:
    matchLabels:
      app: github-runner
  template:
    metadata:
      labels:
        app: github-runner
    spec:
      serviceAccountName: github-runner
      containers:
      - name: runner
        image: sumologic/github-runner:latest
        env:
        - name: GITHUB_OWNER
          value: "backbase-grand-central"
        - name: GITHUB_REPOSITORY
          value: "gc-ecos-applications-live"
        - name: GITHUB_TOKEN
          valueFrom:
            secretKeyRef:
              name: github-token
              key: token
        - name: RUNNER_LABELS
          value: "self-hosted,azure,kubernetes"
        - name: RUNNER_NAME_PREFIX
          value: "k8s-runner"
        resources:
          requests:
            memory: "1Gi"
            cpu: "500m"
          limits:
            memory: "2Gi"
            cpu: "1000m"
        securityContext:
          runAsNonRoot: true
          runAsUser: 1000
          allowPrivilegeEscalation: false
          readOnlyRootFilesystem: true
        volumeMounts:
        - name: docker-sock
          mountPath: /var/run/docker.sock
        - name: workspace
          mountPath: /workspace
      volumes:
      - name: docker-sock
        hostPath:
          path: /var/run/docker.sock
          type: Socket
      - name: workspace
        emptyDir: {}
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: github-runner
  namespace: github-runners
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: github-runner
rules:
- apiGroups: [""]
  resources: ["pods", "services", "configmaps", "secrets"]
  verbs: ["get", "list", "create", "update", "patch", "delete"]
- apiGroups: ["apps"]
  resources: ["deployments", "replicasets"]
  verbs: ["get", "list", "create", "update", "patch", "delete"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: github-runner
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: github-runner
subjects:
- kind: ServiceAccount
  name: github-runner
  namespace: github-runners
```

**Runner auto-scaling with HPA:**
```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: github-runner-hpa
  namespace: github-runners
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: github-runner
  minReplicas: 1
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
```

**Network security for runners:**
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: github-runner-netpol
  namespace: github-runners
spec:
  podSelector:
    matchLabels:
      app: github-runner
  policyTypes:
  - Ingress
  - Egress
  egress:
  - to: []
    ports:
    - protocol: TCP
      port: 443  # HTTPS to GitHub
    - protocol: TCP
      port: 22   # Git SSH
    - protocol: TCP
      port: 53   # DNS
    - protocol: UDP
      port: 53   # DNS
  - to:
    - namespaceSelector:
        matchLabels:
          name: argocd
    ports:
    - protocol: TCP
      port: 8080  # ArgoCD API
```

## GitOps - ArgoCD/FluxCD

### Question 51: ArgoCD Configuration and Management
**Scenario:** Configure ArgoCD for managing the applications in the `gc-ecos-applications-live` repository with proper RBAC and security.

**Answer:**
**ArgoCD installation with Helm:**
```yaml
# argocd-values.yaml
global:
  domain: argocd.grandcentral.com

configs:
  params:
    server.insecure: false
    server.grpc.web: true
  cm:
    application.instanceLabelKey: argocd.argoproj.io/instance
    exec.enabled: true
    admin.enabled: false
    oidc.config: |
      name: Azure AD
      issuer: https://login.microsoftonline.com/tenant-id/v2.0
      clientId: azure-ad-client-id
      clientSecret: $oidc.azure.clientSecret
      requestedIDTokenClaims:
        groups:
          essential: true
      requestedScopes: ["openid", "profile", "email", "groups"]
  rbac:
    policy.default: role:readonly
    policy.csv: |
      p, role:admin, applications, *, */*, allow
      p, role:admin, clusters, *, *, allow
      p, role:admin, repositories, *, *, allow
      
      p, role:developer, applications, get, */*, allow
      p, role:developer, applications, sync, */*, allow
      p, role:developer, applications, action/*, *, */*, allow
      
      g, platform-team, role:admin
      g, developers, role:developer

server:
  ingress:
    enabled: true
    ingressClassName: istio
    annotations:
      kubernetes.io/ingress.class: istio
      cert-manager.io/cluster-issuer: letsencrypt-prod
    hosts:
      - argocd.grandcentral.com
    tls:
      - secretName: argocd-tls
        hosts:
          - argocd.grandcentral.com

repoServer:
  volumes:
    - name: custom-tools
      emptyDir: {}
  initContainers:
    - name: download-tools
      image: alpine:3.18
      command: [sh, -c]
      args:
        - |
          wget -O- https://github.com/mikefarah/yq/releases/latest/download/yq_linux_amd64 > /custom-tools/yq &&
          chmod +x /custom-tools/yq
      volumeMounts:
        - mountPath: /custom-tools
          name: custom-tools
  volumeMounts:
    - mountPath: /usr/local/bin/yq
      name: custom-tools
      subPath: yq

applicationSet:
  enabled: true
```

**ArgoCD Projects for multi-tenancy:**
```yaml
apiVersion: argoproj.io/v1alpha1
kind: AppProject
metadata:
  name: tenant-applications
  namespace: argocd
spec:
  description: Tenant applications project
  destinations:
  - namespace: '*'
    server: https://kubernetes.default.svc
  sourceRepos:
  - 'https://charts.grandcentral.com'
  - 'https://github.com/backbase-grand-central/gc-ecos-applications-live'
  clusterResourceWhitelist:
  - group: ''
    kind: Namespace
  - group: ''
    kind: PersistentVolume
  namespaceResourceWhitelist:
  - group: ''
    kind: ConfigMap
  - group: ''
    kind: Secret
  - group: ''
    kind: Service
  - group: apps
    kind: Deployment
  - group: networking.k8s.io
    kind: Ingress
  roles:
  - name: tenant-admin
    policies:
    - p, proj:tenant-applications:tenant-admin, applications, *, tenant-applications/*, allow
    groups:
    - tenant-admins
  - name: tenant-developer
    policies:
    - p, proj:tenant-applications:tenant-developer, applications, get, tenant-applications/*, allow
    - p, proj:tenant-applications:tenant-developer, applications, sync, tenant-applications/*, allow
    groups:
    - tenant-developers
```

### Question 52: ApplicationSets for Multi-Environment Management
**Scenario:** Use ArgoCD ApplicationSets to manage applications across multiple environments (dev, staging, production) efficiently.

**Answer:**
**ApplicationSet for environment management:**
```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: mambu-connector-environments
  namespace: argocd
spec:
  generators:
  - clusters:
      selector:
        matchLabels:
          environment: "dev"
      values:
        environment: dev
        revision: HEAD
        replicas: "2"
  - clusters:
      selector:
        matchLabels:
          environment: "staging"
      values:
        environment: staging
        revision: staging
        replicas: "3"
  - clusters:
      selector:
        matchLabels:
          environment: "production"
      values:
        environment: production
        revision: main
        replicas: "5"
  template:
    metadata:
      name: 'mambu-{{values.environment}}'
      labels:
        environment: '{{values.environment}}'
    spec:
      project: tenant-applications
      source:
        repoURL: https://charts.grandcentral.com
        chart: gc-connector
        targetRevision: "1.2.3"
        helm:
          valueFiles:
            - values.yaml
          parameters:
            - name: environment
              value: '{{values.environment}}'
            - name: replicaCount
              value: '{{values.replicas}}'
            - name: image.tag
              value: '{{metadata.annotations.image-tag}}'
      destination:
        server: '{{server}}'
        namespace: mambu-{{values.environment}}
      syncPolicy:
        automated:
          prune: true
          selfHeal: true
        syncOptions:
          - CreateNamespace=true
        retry:
          limit: 5
          backoff:
            duration: 5s
            factor: 2
            maxDuration: 3m
      ignoreDifferences:
        - group: v1
          kind: Secret
          jsonPointers:
            - /data
```

**ApplicationSet with Git generator for multiple applications:**
```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: tenant-applications
  namespace: argocd
spec:
  generators:
  - git:
      repoURL: https://github.com/backbase-grand-central/gc-ecos-applications-live
      revision: main
      directories:
      - path: runtimes/dev/apps/*
  template:
    metadata:
      name: '{{path.basename}}'
      labels:
        app: '{{path.basename}}'
        environment: dev
    spec:
      project: tenant-applications
      source:
        repoURL: https://github.com/backbase-grand-central/gc-ecos-applications-live
        path: '{{path}}'
        targetRevision: main
      destination:
        server: https://kubernetes.default.svc
        namespace: '{{path.basename}}'
      syncPolicy:
        automated:
          prune: true
          selfHeal: true
        syncOptions:
          - CreateNamespace=true
```

### Question 53: ArgoCD Sync Waves and Hooks
**Scenario:** Implement proper application deployment ordering using ArgoCD sync waves for applications with dependencies.

**Answer:**
**Sync waves for ordered deployment:**
```yaml
# Database deployment (Wave 0)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mambu-database
  annotations:
    argocd.argoproj.io/sync-wave: "0"
spec:
  # ... database deployment spec

---
# Database initialization job (Wave 1)
apiVersion: batch/v1
kind: Job
metadata:
  name: mambu-db-migration
  annotations:
    argocd.argoproj.io/sync-wave: "1"
    argocd.argoproj.io/hook: Sync
    argocd.argoproj.io/hook-delete-policy: BeforeHookCreation
spec:
  template:
    spec:
      restartPolicy: Never
      containers:
      - name: migrate
        image: migrate/migrate
        command: ["migrate"]
        args: ["-path", "/migrations", "-database", "postgres://...", "up"]

---
# Application deployment (Wave 2)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mambu-connector
  annotations:
    argocd.argoproj.io/sync-wave: "2"
spec:
  # ... application deployment spec

---
# Post-deployment tests (Wave 3)
apiVersion: batch/v1
kind: Job
metadata:
  name: mambu-smoke-tests
  annotations:
    argocd.argoproj.io/sync-wave: "3"
    argocd.argoproj.io/hook: PostSync
    argocd.argoproj.io/hook-delete-policy: HookSucceeded
spec:
  template:
    spec:
      restartPolicy: Never
      containers:
      - name: smoke-test
        image: appropriate/curl
        command: ["curl"]
        args: ["-f", "http://mambu-connector/health"]
```

**Health checks and sync options:**
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: mambu-connector
  annotations:
    argocd.argoproj.io/sync-options: SkipDryRunOnMissingResource=true
spec:
  source:
    # ... source configuration
  destination:
    # ... destination configuration
  syncPolicy:
    syncOptions:
      - CreateNamespace=true
      - PruneLast=true
      - ApplyOutOfSyncOnly=true
    automated:
      prune: true
      selfHeal: true
    retry:
      limit: 5
      backoff:
        duration: 5s
        factor: 2
        maxDuration: 3m
  ignoreDifferences:
    - group: apps
      kind: Deployment
      jsonPointers:
        - /spec/replicas  # Ignore HPA-managed replicas
    - group: v1
      kind: Secret
      jsonPointers:
        - /data
```

### Question 54: GitOps Security and Secret Management
**Scenario:** Implement secure secret management in a GitOps workflow without storing secrets in Git.

**Answer:**
**External Secrets Operator with Azure Key Vault:**
```yaml
apiVersion: external-secrets.io/v1beta1
kind: SecretStore
metadata:
  name: azure-keyvault
  namespace: mambu
spec:
  provider:
    azurekv:
      tenantId: "tenant-id"
      vaultUrl: "https://shared-kv.vault.azure.net/"
      authSecretRef:
        clientId:
          name: azure-secret
          key: client-id
        clientSecret:
          name: azure-secret
          key: client-secret

---
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: mambu-secrets
  namespace: mambu
spec:
  refreshInterval: 15s
  secretStoreRef:
    name: azure-keyvault
    kind: SecretStore
  target:
    name: mambu-app-secrets
    creationPolicy: Owner
  data:
  - secretKey: api-key
    remoteRef:
      key: mambu-api-key
  - secretKey: database-password
    remoteRef:
      key: mambu-db-password
```

**Sealed Secrets for GitOps:**
```yaml
# Install Sealed Secrets controller
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sealed-secrets-controller
  namespace: kube-system
spec:
  selector:
    matchLabels:
      name: sealed-secrets-controller
  template:
    metadata:
      labels:
        name: sealed-secrets-controller
    spec:
      serviceAccountName: sealed-secrets-controller
      containers:
      - name: sealed-secrets-controller
        image: bitnami/sealed-secrets-controller:v0.18.0
        command:
        - controller
        args:
        - --update-status
        - --rotate-keys
        - --key-rotation-period=720h
        ports:
        - containerPort: 8080
          name: http
```

**Create and use sealed secrets:**
```bash
# Create sealed secret
echo -n mypassword | kubectl create secret generic mambu-secret \
  --dry-run=client --from-file=password=/dev/stdin -o yaml | \
  kubeseal -f - -w mambu-sealed-secret.yaml

# Sealed secret manifest
apiVersion: bitnami.com/v1alpha1
kind: SealedSecret
metadata:
  name: mambu-secret
  namespace: mambu
spec:
  encryptedData:
    password: AgBy3i4OJSWK+PiTySYZZA9rO43cGDEQAx...
  template:
    metadata:
      name: mambu-secret
      namespace: mambu
    type: Opaque
```

### Question 55: ArgoCD Image Updater
**Scenario:** Configure ArgoCD Image Updater to automatically update application images when new versions are available.

**Answer:**
**ArgoCD Image Updater configuration:**
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-image-updater-config
  namespace: argocd
data:
  registries.conf: |
    registries:
    - name: Azure Container Registry
      api_url: https://acr.azurecr.io
      prefix: acr.azurecr.io
      ping: true
      credentials: secret:argocd/acr-credentials#username,secret:argocd/acr-credentials#password
      default: true

  applications.yaml: |
    applications:
    - name: mambu-connector
      image_name: acr.azurecr.io/gc-mambu
      image_tag: v1.2.3
      update_strategy: semantic
      ignore_tags:
        - latest
        - dev
        - staging
      write_back_method: git
      git_branch: auto-update/mambu
```

**Application annotation for image updater:**
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: mambu-connector
  namespace: argocd
  annotations:
    argocd-image-updater.argoproj.io/image-list: mambu=acr.azurecr.io/gc-mambu
    argocd-image-updater.argoproj.io/mambu.update-strategy: semver
    argocd-image-updater.argoproj.io/mambu.allow-tags: regexp:^v[0-9]+\.[0-9]+\.[0-9]+$
    argocd-image-updater.argoproj.io/mambu.ignore-tags: latest,dev,staging
    argocd-image-updater.argoproj.io/write-back-method: git
    argocd-image-updater.argoproj.io/write-back-target: kustomization
    argocd-image-updater.argoproj.io/git-branch: auto-update/mambu
spec:
  # ... application spec
```

**Automated PR creation for image updates:**
```yaml
# GitHub Action triggered by image updater
name: Auto-merge Image Updates
on:
  pull_request:
    branches: [main]
    paths: ['runtimes/*/values/*/values.yaml']

jobs:
  auto-merge:
    if: github.actor == 'argocd-image-updater'
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    
    - name: Validate image update
      run: |
        # Check if only image tags were updated
        CHANGED_FILES=$(git diff HEAD~1 HEAD --name-only)
        echo "$CHANGED_FILES" | grep -E '^runtimes/.*/values/.*/values\.yaml$' || exit 1
        
        # Check if changes are only image tag updates
        git diff HEAD~1 HEAD | grep -E '^\+.*image\.tag:' || exit 1
        
    - name: Run tests
      run: |
        ./scripts/validate-manifests.sh
        ./scripts/security-scan.sh
        
    - name: Auto-merge PR
      uses: pascalgn/merge-action@v0.15.6
      with:
        github_token: ${{ secrets.GITHUB_TOKEN }}
        merge_method: squash
      env:
        GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

---

## Troubleshooting

### Question 56: Kubernetes Pod Troubleshooting
**Scenario:** A Mambu connector pod is in `CrashLoopBackOff` state. Walk through your troubleshooting methodology.

**Answer:**
**Systematic troubleshooting approach:**

1. **Check pod status and events:**
```bash
# Get pod details and events
kubectl describe pod mambu-connector-xxx -n mambu

# Check recent events in namespace  
kubectl get events -n mambu --sort-by='.lastTimestamp' | tail -20

# Check pod logs
kubectl logs mambu-connector-xxx -n mambu
kubectl logs mambu-connector-xxx -n mambu --previous
```

2. **Analyze resource constraints:**
```bash
# Check resource usage
kubectl top pod mambu-connector-xxx -n mambu
kubectl describe node <node-name>

# Check resource quotas
kubectl describe resourcequota -n mambu

# Check if pod is being evicted
kubectl get events -n mambu | grep Evicted
```

3. **Check configuration issues:**
```bash
# Verify ConfigMaps and Secrets
kubectl get configmap -n mambu
kubectl describe configmap mambu-config -n mambu

kubectl get secrets -n mambu
kubectl describe secret mambu-secrets -n mambu

# Check environment variables
kubectl exec -it mambu-connector-xxx -n mambu -- env | grep MAMBU
```

4. **Network and service connectivity:**
```bash
# Test DNS resolution
kubectl exec -it mambu-connector-xxx -n mambu -- nslookup kubernetes.default

# Test service connectivity
kubectl exec -it mambu-connector-xxx -n mambu -- curl -I mambu-database:5432

# Check service endpoints
kubectl get endpoints mambu-database -n mambu
```

5. **Deep dive into application logs:**
```bash
# Stream logs with timestamps
kubectl logs -f mambu-connector-xxx -n mambu --timestamps

# Check for specific error patterns
kubectl logs mambu-connector-xxx -n mambu | grep -i "error\|exception\|failed"

# Multi-container pod logs
kubectl logs mambu-connector-xxx -c sidecar -n mambu
```

**Common fixes based on findings:**
- **Out of resources**: Increase resource requests/limits
- **Config issues**: Fix ConfigMap or Secret values
- **Image issues**: Check image tag and registry access
- **Network issues**: Verify Network Policies and DNS
- **Storage issues**: Check PVC status and storage class

### Question 57: Azure AKS Cluster Troubleshooting
**Scenario:** The AKS cluster is experiencing node issues and some pods are stuck in `Pending` state. How do you diagnose and resolve this?

**Answer:**
**AKS cluster diagnostics:**

1. **Check cluster health:**
```bash
# Get cluster status
az aks show --name gc-dev-aks --resource-group gc-dev-rg --query "powerState"

# Check cluster autoscaler status
az aks nodepool show --cluster-name gc-dev-aks --name default --resource-group gc-dev-rg

# Get cluster events
az aks get-upgrades --name gc-dev-aks --resource-group gc-dev-rg
```

2. **Node diagnostics:**
```bash
# Check node status
kubectl get nodes -o wide
kubectl describe node <node-name>

# Check node resource allocation
kubectl describe node <node-name> | grep -A 10 "Allocated resources"

# Check node conditions
kubectl get nodes -o custom-columns=NAME:.metadata.name,STATUS:.status.conditions[-1].type,REASON:.status.conditions[-1].reason
```

3. **Pod scheduling issues:**
```bash
# Get pending pods
kubectl get pods --all-namespaces --field-selector=status.phase=Pending

# Check scheduler events
kubectl describe pod <pending-pod> | grep -A 10 Events

# Check resource constraints
kubectl top nodes
kubectl describe namespace <namespace> | grep -A 5 "Resource Quotas"
```

4. **Azure diagnostics:**
```bash
# Check Azure Monitor for cluster
az monitor activity-log list --resource-group gc-dev-rg --start-time 2024-01-01T00:00:00Z

# Check node pool status
az aks nodepool list --cluster-name gc-dev-aks --resource-group gc-dev-rg --output table

# Check cluster autoscaler logs
kubectl logs -n kube-system -l app=cluster-autoscaler
```

5. **Network troubleshooting:**
```bash
# Check CNI plugin status
kubectl get pods -n kube-system | grep -E "(azure-cni|calico|cilium)"

# Check network policies
kubectl get networkpolicy --all-namespaces

# Test pod-to-pod connectivity
kubectl run test-pod --image=alpine:latest --rm -it -- sh
# From within the pod: ping <other-pod-ip>
```

**Common resolution steps:**
- **Scale node pool**: `az aks nodepool scale`
- **Add new node pool**: `az aks nodepool add`
- **Cordon and drain problematic nodes**: `kubectl cordon/drain`
- **Update cluster**: `az aks upgrade`
- **Check and adjust resource quotas**

### Question 58: Application Performance Issues
**Scenario:** The Mambu connector is experiencing high latency and timeout errors. How do you diagnose and optimize performance?

**Answer:**
**Performance investigation methodology:**

1. **Metrics analysis:**
```bash
# Check application metrics in Prometheus
kubectl port-forward svc/prometheus 9090:9090 -n monitoring

# Key queries for investigation:
# - Request rate: rate(http_requests_total[5m])
# - Latency: histogram_quantile(0.95, rate(http_request_duration_seconds_bucket[5m]))
# - Error rate: rate(http_requests_total{status=~"5.."}[5m])
# - CPU usage: rate(container_cpu_usage_seconds_total[5m])
# - Memory usage: container_memory_working_set_bytes
```

2. **Application profiling:**
```bash
# Java application profiling
kubectl exec -it mambu-connector-xxx -n mambu -- jstack 1 > thread-dump.txt
kubectl exec -it mambu-connector-xxx -n mambu -- jmap -histo 1 > heap-histogram.txt

# Get garbage collection stats
kubectl exec -it mambu-connector-xxx -n mambu -- jstat -gc 1 5s 10
```

3. **Database performance:**
```bash
# Check database connections
kubectl exec -it mambu-db-xxx -n mambu -- psql -U postgres -c "SELECT * FROM pg_stat_activity;"

# Check slow queries
kubectl exec -it mambu-db-xxx -n mambu -- psql -U postgres -c "SELECT query, mean_time, calls FROM pg_stat_statements ORDER BY mean_time DESC LIMIT 10;"

# Monitor database metrics
kubectl port-forward svc/postgres-exporter 9187:9187 -n mambu
```

4. **Network analysis:**
```bash
# Check service mesh metrics (Istio)
kubectl port-forward svc/grafana 3000:3000 -n istio-system
# View Istio dashboards for request flow and latency

# Check network policies impact
kubectl get networkpolicy -n mambu
kubectl describe networkpolicy mambu-netpol -n mambu
```

5. **Resource optimization:**
```bash
# Check current resource usage vs limits
kubectl top pod -n mambu
kubectl describe pod mambu-connector-xxx -n mambu | grep -A 10 "Limits\|Requests"

# Check HPA status
kubectl get hpa -n mambu
kubectl describe hpa mambu-hpa -n mambu
```

**Performance optimization strategies:**
```yaml
# Optimized deployment configuration
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mambu-connector-optimized
spec:
  template:
    spec:
      containers:
      - name: mambu
        image: acr.azurecr.io/gc-mambu:v1.2.3
        resources:
          requests:
            memory: "1Gi"      # Increased from 512Mi
            cpu: "500m"        # Increased from 250m
          limits:
            memory: "2Gi"      # Increased from 1Gi  
            cpu: "1000m"       # Increased from 500m
        env:
        - name: JAVA_OPTS
          value: "-XX:+UseG1GC -XX:MaxGCPauseMillis=200 -XX:+UseStringDeduplication -Xmx1536m"
        - name: SERVER_TOMCAT_MAX_THREADS
          value: "200"
        - name: HIKARI_MAXIMUM_POOL_SIZE
          value: "20"
        - name: HIKARI_CONNECTION_TIMEOUT
          value: "30000"
```

**Database optimization:**
```sql
-- Connection pooling optimization
ALTER SYSTEM SET max_connections = 100;
ALTER SYSTEM SET shared_buffers = '256MB';
ALTER SYSTEM SET effective_cache_size = '1GB';
ALTER SYSTEM SET maintenance_work_mem = '64MB';
ALTER SYSTEM SET checkpoint_completion_target = 0.9;
ALTER SYSTEM SET wal_buffers = '16MB';
ALTER SYSTEM SET default_statistics_target = 100;

-- Index optimization
CREATE INDEX CONCURRENTLY idx_transactions_date ON transactions(created_date);
CREATE INDEX CONCURRENTLY idx_accounts_customer_id ON accounts(customer_id);
```

### Question 59: ArgoCD Sync Issues
**Scenario:** ArgoCD applications are showing "OutOfSync" status and manual sync attempts are failing. How do you troubleshoot and resolve?

**Answer:**
**ArgoCD troubleshooting steps:**

1. **Check ArgoCD application status:**
```bash
# Get application sync status
argocd app get mambu-connector

# Get application details
argocd app describe mambu-connector

# Check application events
kubectl describe application mambu-connector -n argocd
```

2. **Examine sync errors:**
```bash
# Get sync operation details
argocd app get mambu-connector -o yaml | grep -A 20 "operation:"

# Check repo server logs
kubectl logs -n argocd -l app.kubernetes.io/component=repo-server

# Check application controller logs
kubectl logs -n argocd -l app.kubernetes.io/component=application-controller
```

3. **Resource comparison:**
```bash
# Show differences between Git and cluster
argocd app diff mambu-connector

# Get live resource state
kubectl get deployment mambu-connector -n mambu -o yaml

# Compare with desired state from Git
argocd app manifests mambu-connector
```

4. **Common sync issues and fixes:**

**Issue: Resource quota exceeded**
```bash
# Check resource quotas
kubectl describe resourcequota -n mambu

# Fix: Increase quotas or reduce resource requests
kubectl patch resourcequota mambu-quota -n mambu --type='merge' -p='{"spec":{"hard":{"limits.memory":"4Gi"}}}'
```

**Issue: RBAC permissions**
```bash
# Check ArgoCD service account permissions
kubectl auth can-i create pods --as=system:serviceaccount:argocd:argocd-application-controller -n mambu

# Fix: Update RBAC
kubectl create clusterrolebinding argocd-admin --clusterrole=cluster-admin --serviceaccount=argocd:argocd-application-controller
```

**Issue: Finalizer blocking deletion**
```bash
# Remove stuck finalizers
kubectl patch application mambu-connector -n argocd --type='merge' -p='{"metadata":{"finalizers":null}}'
```

5. **Repository access issues:**
```bash
# Test repository access
argocd repo list
argocd repo get https://github.com/backbase-grand-central/gc-ecos-applications-live

# Check Git credentials
kubectl get secret -n argocd | grep repo
kubectl describe secret repo-123456-secret -n argocd
```

**ArgoCD health check and recovery:**
```bash
# Restart ArgoCD components
kubectl rollout restart deployment argocd-application-controller -n argocd
kubectl rollout restart deployment argocd-repo-server -n argocd

# Refresh application
argocd app refresh mambu-connector

# Hard refresh (bypass cache)
argocd app refresh mambu-connector --hard-refresh

# Force sync with prune
argocd app sync mambu-connector --prune
```

### Question 60: Azure Resource Troubleshooting
**Scenario:** Terraform apply is failing when trying to create Azure resources. The error indicates permission issues, but the service principal seems to have the correct roles.

**Answer:**
**Azure permission troubleshooting:**

1. **Verify service principal permissions:**
```bash
# Check current token and permissions
az account show
az ad signed-in-user show

# Test specific permissions
az role assignment list --assignee <service-principal-id> --scope /subscriptions/<subscription-id>

# Check if permission is inherited or direct
az role assignment list --assignee <service-principal-id> --scope /subscriptions/<subscription-id>/resourceGroups/<rg-name>
```

2. **Terraform state and provider debugging:**
```bash
# Enable Terraform debugging
export TF_LOG=DEBUG
export TF_LOG_PATH=terraform.log

# Check provider configuration
terraform providers

# Validate configuration
terraform validate
terraform plan -detailed-exitcode
```

3. **Azure API call debugging:**
```bash
# Enable Azure CLI debugging
export AZURE_CLI_DEBUG=1

# Test Azure operations manually
az group create --name test-rg --location "East US"
az storage account create --name teststorage123 --resource-group test-rg --location "East US" --sku Standard_LRS
```

4. **Common Azure permission issues:**

**Issue: Missing subscription-level permissions**
```bash
# Fix: Add required role at subscription level
az role assignment create \
  --assignee <service-principal-id> \
  --role "Contributor" \
  --scope "/subscriptions/<subscription-id>"

# Or more specific roles
az role assignment create \
  --assignee <service-principal-id> \
  --role "Key Vault Administrator" \
  --scope "/subscriptions/<subscription-id>"
```

**Issue: Azure AD permissions for service principals**
```bash
# Check AAD permissions
az ad app permission list --id <application-id>

# Add required AAD permissions
az ad app permission add \
  --id <application-id> \
  --api 00000003-0000-0000-c000-000000000000 \
  --api-permissions 7ab1d382-f21e-4acd-a863-ba3e13f7da61=Role

# Grant admin consent
az ad app permission admin-consent --id <application-id>
```

5. **Resource-specific troubleshooting:**

**Key Vault access issues:**
```bash
# Check Key Vault access policies
az keyvault show --name shared-kv --resource-group shared-rg --query "properties.accessPolicies"

# Add access policy
az keyvault set-policy \
  --name shared-kv \
  --resource-group shared-rg \
  --spn <service-principal-id> \
  --secret-permissions get list set delete
```

**Network security group issues:**
```bash
# Check NSG rules
az network nsg show --name mambu-nsg --resource-group networking-rg --query "securityRules"

# Test network connectivity
az network test-connectivity \
  --source-resource <vm-resource-id> \
  --dest-address <destination-ip> \
  --dest-port 443
```

**Terraform state corruption:**
```bash
# Check state file integrity
terraform state list
terraform state show <resource-name>

# Fix state issues
terraform import <resource-type>.<resource-name> <azure-resource-id>

# Recreate problematic resources
terraform state rm <resource-name>
terraform apply -target=<resource-name>
```

---

## Networking

### Question 61: Azure Virtual Network Design
**Scenario:** Design a comprehensive networking strategy for the multi-tenant environment using the current Azure virtual network configuration.

**Answer:**
**Hub-and-spoke network architecture:**
```hcl
# Hub VNet for shared services
resource "azurerm_virtual_network" "hub" {
  name                = "${var.prefix}-hub-vnet"
  address_space       = ["10.0.0.0/16"]
  location           = azurerm_resource_group.networking.location
  resource_group_name = azurerm_resource_group.networking.name

  tags = local.common_tags
}

# Spoke VNet for tenant workloads
resource "azurerm_virtual_network" "spoke_workloads" {
  name                = "${var.prefix}-spoke-workloads-vnet"
  address_space       = ["10.1.0.0/16"]
  location           = azurerm_resource_group.networking.location
  resource_group_name = azurerm_resource_group.networking.name

  tags = local.common_tags
}

# Subnet design
resource "azurerm_subnet" "hub_gateway" {
  name                 = "GatewaySubnet"
  resource_group_name  = azurerm_resource_group.networking.name
  virtual_network_name = azurerm_virtual_network.hub.name
  address_prefixes     = ["10.0.1.0/27"]
}

resource "azurerm_subnet" "hub_firewall" {
  name                 = "AzureFirewallSubnet"
  resource_group_name  = azurerm_resource_group.networking.name
  virtual_network_name = azurerm_virtual_network.hub.name
  address_prefixes     = ["10.0.2.0/26"]
}

resource "azurerm_subnet" "hub_shared_services" {
  name                 = "shared-services"
  resource_group_name  = azurerm_resource_group.networking.name
  virtual_network_name = azurerm_virtual_network.hub.name
  address_prefixes     = ["10.0.3.0/24"]

  delegation {
    name = "aks-delegation"
    service_delegation {
      name    = "Microsoft.ContainerService/managedClusters"
      actions = ["Microsoft.Network/virtualNetworks/subnets/action"]
    }
  }
}

resource "azurerm_subnet" "spoke_aks" {
  name                 = "aks-nodes"
  resource_group_name  = azurerm_resource_group.networking.name
  virtual_network_name = azurerm_virtual_network.spoke_workloads.name
  address_prefixes     = ["10.1.0.0/22"]

  delegation {
    name = "aks-delegation"
    service_delegation {
      name    = "Microsoft.ContainerService/managedClusters"
      actions = ["Microsoft.Network/virtualNetworks/subnets/action"]
    }
  }
}

# VNet peering
resource "azurerm_virtual_network_peering" "hub_to_spoke" {
  name                      = "hub-to-spoke-workloads"
  resource_group_name       = azurerm_resource_group.networking.name
  virtual_network_name      = azurerm_virtual_network.hub.name
  remote_virtual_network_id = azurerm_virtual_network.spoke_workloads.id
  
  allow_virtual_network_access = true
  allow_forwarded_traffic      = true
  allow_gateway_transit        = true
  use_remote_gateways         = false
}

resource "azurerm_virtual_network_peering" "spoke_to_hub" {
  name                      = "spoke-workloads-to-hub"
  resource_group_name       = azurerm_resource_group.networking.name
  virtual_network_name      = azurerm_virtual_network.spoke_workloads.name
  remote_virtual_network_id = azurerm_virtual_network.hub.id
  
  allow_virtual_network_access = true
  allow_forwarded_traffic      = true
  allow_gateway_transit        = false
  use_remote_gateways         = true
}
```

### Question 62: DNS Configuration and Management
**Scenario:** Configure DNS for the multi-environment setup with proper domain resolution for internal and external services.

**Answer:**
**Azure Private DNS configuration:**
```hcl
# Private DNS zone for internal services
resource "azurerm_private_dns_zone" "internal" {
  name                = "gc.internal"
  resource_group_name = azurerm_resource_group.networking.name

  tags = local.common_tags
}

# Environment-specific DNS zones
resource "azurerm_private_dns_zone" "dev" {
  name                = "dev.gc.internal"
  resource_group_name = azurerm_resource_group.networking.name

  tags = local.common_tags
}

# Link DNS zones to VNets
resource "azurerm_private_dns_zone_virtual_network_link" "hub_internal" {
  name                  = "hub-link"
  resource_group_name   = azurerm_resource_group.networking.name
  private_dns_zone_name = azurerm_private_dns_zone.internal.name
  virtual_network_id    = azurerm_virtual_network.hub.id
  registration_enabled  = true
}

# A records for services
resource "azurerm_private_dns_a_record" "mambu_dev" {
  name                = "mambu"
  zone_name          = azurerm_private_dns_zone.dev.name
  resource_group_name = azurerm_resource_group.networking.name
  ttl                = 300
  records            = [azurerm_kubernetes_cluster.main.private_fqdn]
}

# CNAME records for aliases
resource "azurerm_private_dns_cname_record" "api_dev" {
  name                = "api"
  zone_name          = azurerm_private_dns_zone.dev.name
  resource_group_name = azurerm_resource_group.networking.name
  ttl                = 300
  record             = "mambu.dev.gc.internal"
}
```

**Kubernetes DNS configuration:**
```yaml
# External DNS for public domains
apiVersion: apps/v1
kind: Deployment
metadata:
  name: external-dns
  namespace: external-dns
spec:
  template:
    spec:
      serviceAccountName: external-dns
      containers:
      - name: external-dns
        image: k8s.gcr.io/external-dns/external-dns:v0.13.1
        args:
        - --source=service
        - --source=ingress
        - --provider=azure
        - --azure-resource-group=dns-rg
        - --azure-subscription-id=$(AZURE_SUBSCRIPTION_ID)
        - --txt-owner-id=external-dns
        - --domain-filter=grandcentral.com
        - --policy=sync
        env:
        - name: AZURE_SUBSCRIPTION_ID
          valueFrom:
            secretKeyRef:
              name: azure-config-file
              key: subscription-id
        - name: AZURE_TENANT_ID
          valueFrom:
            secretKeyRef:
              name: azure-config-file
              key: tenant-id
        - name: AZURE_CLIENT_ID
          valueFrom:
            secretKeyRef:
              name: azure-config-file
              key: client-id
        - name: AZURE_CLIENT_SECRET
          valueFrom:
            secretKeyRef:
              name: azure-config-file
              key: client-secret

---
# CoreDNS configuration for custom domains
apiVersion: v1
kind: ConfigMap
metadata:
  name: coredns-custom
  namespace: kube-system
data:
  gc.override: |
    gc.internal:53 {
        errors
        cache 30
        forward . 168.63.129.16
    }
  mambu.server: |
    mambu.external:53 {
        errors
        cache 30
        forward . 8.8.8.8 8.8.4.4
    }
```

### Question 63: SSL/TLS Certificate Management
**Scenario:** Implement automated SSL certificate management for all services using cert-manager and Azure Key Vault.

**Answer:**
**Cert-manager installation and configuration:**
```yaml
# Install cert-manager
apiVersion: v1
kind: Namespace
metadata:
  name: cert-manager
---
# ClusterIssuer for Let's Encrypt
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-prod
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: admin@grandcentral.com
    privateKeySecretRef:
      name: letsencrypt-prod
    solvers:
    - dns01:
        azureDNS:
          subscriptionID: "subscription-id"
          resourceGroupName: "dns-rg"
          hostedZoneName: "grandcentral.com"
          environment: AzurePublicCloud
          managedIdentity:
            clientID: "cert-manager-identity-client-id"

---
# ClusterIssuer for internal CA
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: internal-ca-issuer
spec:
  ca:
    secretName: internal-ca-secret
```

**Certificate resources:**
```yaml
# Public certificate for external services
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: mambu-tls
  namespace: mambu
spec:
  secretName: mambu-tls-secret
  issuerRef:
    name: letsencrypt-prod
    kind: ClusterIssuer
  dnsNames:
  - mambu.grandcentral.com
  - api.mambu.grandcentral.com

---
# Internal certificate for service-to-service communication
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: mambu-internal-tls
  namespace: mambu
spec:
  secretName: mambu-internal-tls
  issuerRef:
    name: internal-ca-issuer
    kind: ClusterIssuer
  dnsNames:
  - mambu.mambu.svc.cluster.local
  - mambu-internal.gc.internal
```

**Istio Gateway with TLS:**
```yaml
apiVersion: networking.istio.io/v1beta1
kind: Gateway
metadata:
  name: mambu-gateway
  namespace: mambu
spec:
  selector:
    istio: ingressgateway
  servers:
  - port:
      number: 443
      name: https
      protocol: HTTPS
    tls:
      mode: SIMPLE
      credentialName: mambu-tls-secret
    hosts:
    - mambu.grandcentral.com
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
    - mambu.grandcentral.com
    redirect:
      httpsRedirect: true

---
# VirtualService for routing
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: mambu-vs
  namespace: mambu
spec:
  hosts:
  - mambu.grandcentral.com
  gateways:
  - mambu-gateway
  http:
  - match:
    - uri:
        prefix: "/"
    route:
    - destination:
        host: mambu-service
        port:
          number: 80
```

### Question 64: Network Security Groups and Firewall Rules
**Scenario:** Configure comprehensive network security using Azure NSGs and Azure Firewall for the multi-tenant environment.

**Answer:**
**NSG configuration for different tiers:**
```hcl
# NSG for AKS subnet
resource "azurerm_network_security_group" "aks" {
  name                = "${var.prefix}-aks-nsg"
  location           = azurerm_resource_group.networking.location
  resource_group_name = azurerm_resource_group.networking.name

  # Allow inbound HTTPS from Application Gateway
  security_rule {
    name                       = "AllowHTTPS"
    priority                   = 1001
    direction                 = "Inbound"
    access                    = "Allow"
    protocol                  = "Tcp"
    source_port_range         = "*"
    destination_port_range    = "443"
    source_address_prefix     = "10.0.4.0/24"  # App Gateway subnet
    destination_address_prefix = "*"
  }

  # Allow AKS required ports
  security_rule {
    name                       = "AllowAKSAPI"
    priority                   = 1002
    direction                 = "Inbound"
    access                    = "Allow"
    protocol                  = "Tcp"
    source_port_range         = "*"
    destination_port_range    = "443"
    source_address_prefix     = "AzureCloud"
    destination_address_prefix = "*"
  }

  # Deny all other inbound traffic
  security_rule {
    name                       = "DenyAllInbound"
    priority                   = 4096
    direction                 = "Inbound"
    access                    = "Deny"
    protocol                  = "*"
    source_port_range         = "*"
    destination_port_range    = "*"
    source_address_prefix     = "*"
    destination_address_prefix = "*"
  }

  tags = local.common_tags
}

# Associate NSG with subnet
resource "azurerm_subnet_network_security_group_association" "aks" {
  subnet_id                 = azurerm_subnet.spoke_aks.id
  network_security_group_id = azurerm_network_security_group.aks.id
}
```

**Azure Firewall configuration:**
```hcl
# Azure Firewall
resource "azurerm_firewall" "main" {
  name                = "${var.prefix}-firewall"
  location           = azurerm_resource_group.networking.location
  resource_group_name = azurerm_resource_group.networking.name
  sku_name           = "AZFW_VNet"
  sku_tier           = "Standard"

  ip_configuration {
    name                 = "configuration"
    subnet_id           = azurerm_subnet.hub_firewall.id
    public_ip_address_id = azurerm_public_ip.firewall.id
  }
}

# Firewall rules
resource "azurerm_firewall_application_rule_collection" "web_apps" {
  name                = "web-applications"
  azure_firewall_name = azurerm_firewall.main.name
  resource_group_name = azurerm_resource_group.networking.name
  priority            = 100
  action              = "Allow"

  rule {
    name = "allow-mambu-api"
    source_addresses = ["10.1.0.0/22"]  # AKS subnet
    target_fqdns = [
      "*.mambu.com",
      "api.mambu.com"
    ]
    protocol {
      port = "443"
      type = "Https"
    }
  }

  rule {
    name = "allow-container-registry"
    source_addresses = ["10.1.0.0/22"]
    target_fqdns = [
      "*.azurecr.io",
      "mcr.microsoft.com",
      "*.data.mcr.microsoft.com"
    ]
    protocol {
      port = "443"
      type = "Https"
    }
  }
}

resource "azurerm_firewall_network_rule_collection" "outbound" {
  name                = "outbound-rules"
  azure_firewall_name = azurerm_firewall.main.name
  resource_group_name = azurerm_resource_group.networking.name
  priority            = 100
  action              = "Allow"

  rule {
    name = "allow-dns"
    source_addresses = ["10.1.0.0/22"]
    destination_ports = ["53"]
    destination_addresses = ["168.63.129.16"]  # Azure DNS
    protocols = ["TCP", "UDP"]
  }

  rule {
    name = "allow-ntp"
    source_addresses = ["10.1.0.0/22"]
    destination_ports = ["123"]
    destination_addresses = ["*"]
    protocols = ["UDP"]
  }
}
```

### Question 65: Private Endpoints and Service Endpoints
**Scenario:** Configure private endpoints for Azure services to ensure secure connectivity without exposing services to the public internet.

**Answer:**
**Private endpoint configuration:**
```hcl
# Private endpoint for Azure Container Registry
resource "azurerm_private_endpoint" "acr" {
  name                = "${var.prefix}-acr-pe"
  location           = azurerm_resource_group.networking.location
  resource_group_name = azurerm_resource_group.networking.name
  subnet_id          = azurerm_subnet.spoke_private_endpoints.id

  private_service_connection {
    name                           = "acr-connection"
    private_connection_resource_id = azurerm_container_registry.main.id
    subresource_names             = ["registry"]
    is_manual_connection          = false
  }

  private_dns_zone_group {
    name                 = "acr-dns-zone-group"
    private_dns_zone_ids = [azurerm_private_dns_zone.acr.id]
  }
}

# Private DNS zone for ACR
resource "azurerm_private_dns_zone" "acr" {
  name                = "privatelink.azurecr.io"
  resource_group_name = azurerm_resource_group.networking.name
}

resource "azurerm_private_dns_zone_virtual_network_link" "acr" {
  name                  = "acr-dns-link"
  resource_group_name   = azurerm_resource_group.networking.name
  private_dns_zone_name = azurerm_private_dns_zone.acr.name
  virtual_network_id    = azurerm_virtual_network.spoke_workloads.id
  registration_enabled  = false
}

# Private endpoint for Key Vault
resource "azurerm_private_endpoint" "keyvault" {
  name                = "${var.prefix}-kv-pe"
  location           = azurerm_resource_group.networking.location
  resource_group_name = azurerm_resource_group.networking.name
  subnet_id          = azurerm_subnet.spoke_private_endpoints.id

  private_service_connection {
    name                           = "keyvault-connection"
    private_connection_resource_id = azurerm_key_vault.shared.id
    subresource_names             = ["vault"]
    is_manual_connection          = false
  }

  private_dns_zone_group {
    name                 = "kv-dns-zone-group"
    private_dns_zone_ids = [azurerm_private_dns_zone.keyvault.id]
  }
}

# Private endpoint for PostgreSQL
resource "azurerm_private_endpoint" "postgres" {
  name                = "${var.prefix}-postgres-pe"
  location           = azurerm_resource_group.networking.location
  resource_group_name = azurerm_resource_group.networking.name
  subnet_id          = azurerm_subnet.spoke_private_endpoints.id

  private_service_connection {
    name                           = "postgres-connection"
    private_connection_resource_id = azurerm_postgresql_flexible_server.main.id
    subresource_names             = ["postgresqlServer"]
    is_manual_connection          = false
  }
}
```

**Service endpoints configuration:**
```hcl
# Subnet with service endpoints
resource "azurerm_subnet" "database" {
  name                 = "database-subnet"
  resource_group_name  = azurerm_resource_group.networking.name
  virtual_network_name = azurerm_virtual_network.spoke_workloads.name
  address_prefixes     = ["10.1.4.0/24"]

  service_endpoints = [
    "Microsoft.Storage",
    "Microsoft.KeyVault",
    "Microsoft.Sql"
  ]
}

# Storage account with VNet rules
resource "azurerm_storage_account" "logs" {
  name                     = "${var.prefix}logsstorage"
  resource_group_name      = azurerm_resource_group.shared.name
  location                = azurerm_resource_group.shared.location
  account_tier             = "Standard"
  account_replication_type = "LRS"

  network_rules {
    default_action             = "Deny"
    bypass                    = ["AzureServices"]
    virtual_network_subnet_ids = [
      azurerm_subnet.spoke_aks.id,
      azurerm_subnet.database.id
    ]
    ip_rules = [
      "203.0.113.0/24"  # Office IP range
    ]
  }

  tags = local.common_tags
}
```

---

*[Continuing to reach 100 questions...]*
