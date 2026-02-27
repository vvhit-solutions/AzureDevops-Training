# Cloud-Native Financial Platform - Technical Interview Questions

## Interview Focus Areas

This comprehensive interview question bank covers modern cloud-native financial technology platforms, focusing on:

- **Azure Infrastructure**: Multi-layered Terraform deployments and enterprise architecture
- **Kubernetes & GitOps**: ArgoCD application management with multi-source deployments
- **Banking System Integration**: Core banking system connectivity and financial APIs
- **Multi-tenant SaaS Architecture**: Scalable platforms serving multiple financial institutions
- **Enterprise Security**: Identity management, secrets handling, and policy enforcement
- **DevOps & CI/CD**: Automated deployment pipelines with security integration
- **Observability & Compliance**: Monitoring, alerting, and regulatory compliance frameworks

---

## 📘 Basic Questions

### Infrastructure & Terraform

**Q1: Describe a layered infrastructure deployment architecture for enterprise cloud platforms. What are the key considerations for each layer?**

**Answer:** A well-designed cloud infrastructure uses a layered deployment approach with clear separation of concerns:

1. **Foundation Layer**: Sets up cloud subscriptions, core policies, and governance frameworks
2. **Bootstrap Layer**: Provisions foundational resources like resource groups, logging infrastructure
3. **Shared Services**: Deploys common services like identity management, container registries, DNS
4. **DevOps Layer**: Establishes CI/CD tooling and automation infrastructure
5. **Network & Security Baseline**: Creates virtual networks, security groups, and access controls per environment
6. **Data & Messaging**: Deploys databases, storage accounts, and messaging infrastructure
7. **Compute Layer**: Provisions application hosting infrastructure (VMs, containers, serverless)
8. **Observability**: Implements monitoring, logging, and alerting across all layers

**Code Example:**
```terraform
# Example terraform backend configuration
terraform {
  required_providers {
    azurerm = { 
      source = "hashicorp/azurerm"
      version = "~> 3.0" 
    }
  }
  backend "azurerm" {
    resource_group_name  = "rg-terraform-state"
    storage_account_name = "stterraformstate"
    container_name       = "tfstate"
    key                  = "infrastructure/terraform.tfstate"
    use_azuread_auth     = true
  }
}

# Layer-specific workspace management
resource "azurerm_resource_group" "layer" {
  name     = "rg-${var.layer_name}-${var.environment}"
  location = var.location
  
  tags = {
    Layer       = var.layer_name
    Environment = var.environment
    ManagedBy   = "terraform"
  }
}
```

**Production Importance:** This structured approach ensures consistent, repeatable deployments across environments while maintaining security boundaries and dependency management. It enables scaling as the organization grows and provides clear separation of concerns for different teams.

---

**Q2: How would you implement customer tiering and controlled rollouts in a multi-tenant SaaS platform?**

**Answer:** Customer tiering enables controlled rollouts of infrastructure changes across different customer segments based on their requirements, risk tolerance, and service level agreements. This approach typically involves:

- **Tier Classification**: Grouping customers by criticality, size, or service level
- **Version Control**: Managing infrastructure and application versions per tier
- **Graduated Rollouts**: Testing changes with less critical customers first
- **Rollback Capabilities**: Quick reversion for each tier independently

**Code Example:**
```json
{
  "infrastructure_version": "v2.1.4",
  "rollout_strategy": "gradual",
  "tiers": {
    "development": {
      "version": "v2.2.0-beta",
      "customers": ["internal-dev", "partner-sandbox"]
    },
    "standard": {
      "version": "v2.1.4",
      "customers": ["bank-a", "credit-union-b", "fintech-startup-c"]
    },
    "enterprise": {
      "version": "v2.1.3",
      "customers": ["major-bank", "investment-firm", "insurance-corp"]
    }
  }
}
```

**Terraform Implementation:**
```terraform
variable "customer_tier_config" {
  description = "Customer tier configuration"
  type = object({
    tier_name = string
    version   = string
    customers = list(string)
  })
}

resource "azurerm_resource_group" "customer_rg" {
  for_each = toset(var.customer_tier_config.customers)
  
  name     = "rg-${each.key}-${var.environment}"
  location = var.location
  
  tags = {
    CustomerTier = var.customer_tier_config.tier_name
    Version      = var.customer_tier_config.version
    Customer     = each.key
  }
}
```

**Production Importance:** This enables controlled rollouts of infrastructure changes, reducing blast radius during deployments and allowing for gradual validation with different customer segments before full rollout.

---

### Kubernetes & ArgoCD

**Q3: Explain the benefits and implementation of ArgoCD multi-source applications for microservices deployments.**

**Answer:** ArgoCD's multi-source feature allows deploying complex applications from multiple repositories and sources, which is particularly valuable for microservices architectures where:

- **Configuration Separation**: Keep environment-specific configurations in Git while charts are in container registries
- **Independent Versioning**: Each microservice can have its own release cycle
- **Centralized Management**: Single ArgoCD application manages multiple related services
- **Source of Truth**: Different teams can own different parts of the deployment

**Code Example:**
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: banking-platform
  namespace: argocd
spec:
  project: production
  destination:
    server: https://kubernetes.default.svc
    namespace: banking-services
  sources:
    # Configuration repository
    - repoURL: https://github.com/company/banking-configs.git
      targetRevision: main
      ref: config-repo
    
    # Account service
    - repoURL: registry.company.com/charts
      chart: account-service
      targetRevision: 2.1.0
      helm:
        releaseName: account-service
        valueFiles:
          - $config-repo/environments/prod/platform-defaults.yaml
          - $config-repo/environments/prod/services/account-service.yaml
    
    # Payment service  
    - repoURL: registry.company.com/charts
      chart: payment-service
      targetRevision: 1.8.5
      helm:
        releaseName: payment-service
        valueFiles:
          - $config-repo/environments/prod/platform-defaults.yaml
          - $config-repo/environments/prod/services/payment-service.yaml
  
  syncPolicy:
    automated:
      prune: true
      allowEmpty: true
    syncOptions:
      - CreateNamespace=true
```

**Configuration Repository Structure:**
```yaml
# environments/prod/platform-defaults.yaml
common:
  replicas: 3
  resources:
    requests:
      memory: "512Mi"
      cpu: "250m"
    limits:
      memory: "1Gi"
      cpu: "500m"

# environments/prod/services/account-service.yaml
replicas: 5  # Override for critical service
image:
  tag: v2.1.0
service:
  port: 8080
database:
  connectionPoolSize: 20
```

**Production Importance:** This pattern enables independent versioning and deployment of microservices while maintaining centralized configuration management, supporting gradual rollouts and selective rollbacks without affecting the entire application stack.

---

**Q4: How would you design environment organization and promotion strategies for a financial technology platform?**

**Answer:** A financial technology platform typically requires multiple environment types to support different phases of software development, testing, and compliance validation:

**Environment Types:**
- **Development (dev)**: Developer sandbox environments for feature development
- **Testing (test)**: Automated testing and quality assurance validation
- **Staging (stg)**: Pre-production environment that mirrors production
- **User Acceptance Testing (uat)**: Business user validation environment
- **Production (prod)**: Live customer-facing environment

**Code Example:**
```yaml
# Environment configuration structure
environments:
  dev:
    cluster: "dev-cluster"
    namespace_prefix: "dev"
    auto_sync: true
    resource_limits:
      cpu: "2"
      memory: "4Gi"
    external_access: "private"
    
  test:
    cluster: "test-cluster" 
    namespace_prefix: "test"
    auto_sync: true
    resource_limits:
      cpu: "4"
      memory: "8Gi"
    external_access: "private"
    
  stg:
    cluster: "staging-cluster"
    namespace_prefix: "stg"
    auto_sync: false  # Manual approval required
    resource_limits:
      cpu: "8"
      memory: "16Gi"
    external_access: "restricted"
    
  prod:
    cluster: "production-cluster"
    namespace_prefix: "prod"
    auto_sync: false  # Manual approval required
    resource_limits:
      cpu: "16"
      memory: "32Gi"
    external_access: "public"
```

**GitOps Promotion Pipeline:**
```yaml
# GitHub Actions workflow for environment promotion
name: Environment Promotion
on:
  workflow_dispatch:
    inputs:
      source_env:
        type: choice
        options: [dev, test, stg]
      target_env:
        type: choice  
        options: [test, stg, prod]
        
jobs:
  promote:
    runs-on: ubuntu-latest
    steps:
    - name: Validate promotion path
      run: |
        # Ensure proper promotion sequence
        case "${{ inputs.source_env }}-${{ inputs.target_env }}" in
          "dev-test"|"test-stg"|"stg-prod") echo "Valid promotion" ;;
          *) echo "Invalid promotion path" && exit 1 ;;
        esac
    
    - name: Copy configurations
      run: |
        # Copy and transform configurations
        cp -r environments/${{ inputs.source_env }}/apps/* \
           environments/${{ inputs.target_env }}/apps/
        
        # Update image tags for target environment
        find environments/${{ inputs.target_env }} -name "*.yaml" \
          -exec sed -i 's/image_tag: latest/image_tag: stable/g' {} \;
    
    - name: Create Pull Request
      uses: peter-evans/create-pull-request@v4
      with:
        title: "Promote from ${{ inputs.source_env }} to ${{ inputs.target_env }}"
        body: "Automated promotion of applications"
        branch: "promote/${{ inputs.source_env }}-to-${{ inputs.target_env }}"
```

**Production Importance:** Clear environment separation ensures proper testing workflows, risk mitigation, and compliance with financial industry regulations requiring segregation of duties and change management controls.

---

## 📗 Intermediate Questions

### Security & Secrets Management

**Q5: Design a comprehensive secrets management strategy for a cloud-native financial application platform.**

**Answer:** A financial platform requires a multi-layered secrets approach to meet regulatory requirements and security best practices:

**1. Encryption at Rest (Git-stored secrets):**
- **SOPS (Secrets OPerationS)** for encrypting secrets in Git repositories
- **Age or PGP encryption** with role-based key management
- **Automated rotation** of encryption keys

**2. Cloud-native Secret Storage:**
- **Azure Key Vault** or **AWS Secrets Manager** for centralized secret storage
- **Hardware Security Modules (HSM)** for critical cryptographic operations
- **Network isolation** with private endpoints

**3. Kubernetes Secret Injection:**
- **External Secrets Operator** for dynamic secret injection
- **Sealed Secrets** for GitOps-compatible encrypted secrets
- **Pod-level RBAC** for fine-grained access control

**Code Example:**
```terraform
# Azure Key Vault with financial-grade security
resource "azurerm_key_vault" "financial_secrets" {
  name                = "kv-financial-${var.environment}"
  location            = var.location
  resource_group_name = azurerm_resource_group.main.name
  sku_name           = "premium"  # HSM support
  tenant_id          = data.azurerm_client_config.current.tenant_id
  
  # Enhanced security settings
  enabled_for_disk_encryption     = false
  enabled_for_deployment          = false
  enable_rbac_authorization       = true
  enabled_for_template_deployment = false
  soft_delete_retention_days      = 90     # Extended for compliance
  purge_protection_enabled        = true   # Cannot be disabled
  public_network_access_enabled   = false
  
  network_acls {
    default_action = "Deny"
    bypass         = "AzureServices"
    virtual_network_subnet_ids = [
      azurerm_subnet.aks_subnet.id,
      azurerm_subnet.devops_subnet.id
    ]
  }
}

# RBAC for banking application access
resource "azurerm_role_assignment" "banking_app_secrets" {
  scope              = azurerm_key_vault.financial_secrets.id
  role_definition_name = "Key Vault Secrets User"
  principal_id       = azurerm_user_assigned_identity.banking_app.principal_id
  
  condition = <<EOT
    (
      (!(ActionMatches{'Microsoft.KeyVault/vaults/secrets/write'})) 
      AND 
      (@Resource[Microsoft.KeyVault/vaults/secrets:name] StringLike 'banking-*')
    )
  EOT
}
```

**External Secrets Configuration:**
```yaml
apiVersion: external-secrets.io/v1beta1
kind: SecretStore
metadata:
  name: azure-keyvault-store
  namespace: banking-services
spec:
  provider:
    azurekv:
      authType: "WorkloadIdentity"
      tenantId: "your-tenant-id"
      vaultUrl: "https://kv-financial-prod.vault.azure.net/"

---
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: database-credentials
  namespace: banking-services
spec:
  refreshInterval: 300s  # 5 minute refresh
  secretStoreRef:
    name: azure-keyvault-store
    kind: SecretStore
  target:
    name: db-secret
    creationPolicy: Owner
  data:
  - secretKey: username
    remoteRef:
      key: banking-db-username
  - secretKey: password
    remoteRef:
      key: banking-db-password
```

**SOPS Integration:**
```yaml
# .sops.yaml
creation_rules:
  - path_regex: \.yaml$
    encrypted_regex: '^(data|stringData)$'
    age: age1xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
    
# Example encrypted secret
apiVersion: v1
kind: Secret
metadata:
  name: api-keys
  namespace: banking-services
data:
  third-party-api-key: ENC[AES256_GCM,data:xxxxx,iv:xxxxx,tag:xxxxx,type:str]
  encryption-key: ENC[AES256_GCM,data:xxxxx,iv:xxxxx,tag:xxxxx,type:str]
```

**Secret Rotation Automation:**
```python
# Automated secret rotation
import azure.identity
import azure.keyvault.secrets
from datetime import datetime, timedelta

class SecretRotationManager:
    def __init__(self, vault_url):
        credential = azure.identity.DefaultAzureCredential()
        self.client = azure.keyvault.secrets.SecretClient(vault_url, credential)
    
    def rotate_database_password(self, secret_name):
        # Generate new password
        new_password = self.generate_secure_password()
        
        # Update database with new password
        self.update_database_user_password(new_password)
        
        # Store new secret with rotation metadata
        self.client.set_secret(
            secret_name, 
            new_password,
            tags={
                "rotated": datetime.utcnow().isoformat(),
                "rotation_policy": "monthly",
                "compliance": "pci-dss"
            }
        )
        
        # Restart applications to pick up new secret
        self.restart_applications_using_secret(secret_name)
```

**Production Importance:** Financial services face strict regulatory oversight requiring comprehensive secrets management. This multi-layered approach ensures compliance with PCI DSS, SOX, and other regulations while providing audit trails and preventing unauthorized access to sensitive financial data.

---

**Q6: Explain how policy-as-code enforcement works in Kubernetes environments, particularly for financial services compliance.**

**Answer:** Policy-as-code enforcement uses admission controllers to automatically validate and enforce compliance rules in Kubernetes clusters. For financial services, this is critical for maintaining security standards and regulatory compliance:

**Policy Categories:**
- **Security Policies**: Image scanning, security contexts, network policies
- **Compliance Policies**: Data residency, audit logging, resource constraints
- **Operational Policies**: Resource limits, naming conventions, deployment standards
- **Environment-Specific Policies**: Different rules for dev vs production

**Code Example:**
```yaml
# Kyverno ClusterPolicy for financial compliance
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: financial-security-standards
spec:
  validationFailureAction: enforce
  validationFailureActionOverrides:
    - action: audit
      namespaces: ["development", "testing"]
  rules:
  - name: disallow-latest-tags-in-production
    match:
      any:
      - resources:
          kinds:
          - Pod
        subjects:
        - kind: ServiceAccount
          name: "*"
    exclude:
      any:
      - resources:
          namespaces: ["development", "testing"]
    validate:
      message: "Production environments must use specific image tags for compliance"
      pattern:
        spec:
          containers:
          - name: "*"
            image: "!*:latest"
            
  - name: require-resource-limits
    match:
      any:
      - resources:
          kinds:
          - Pod
    validate:
      message: "All containers must have resource limits for capacity planning"
      pattern:
        spec:
          containers:
          - name: "*"
            resources:
              limits:
                memory: "?*"
                cpu: "?*"
                
  - name: enforce-network-policies
    match:
      any:
      - resources:
          kinds:
          - Namespace
    generate:
      kind: NetworkPolicy
      name: deny-all-traffic
      namespace: "{{request.object.metadata.name}}"
      data:
        spec:
          podSelector: {}
          policyTypes:
          - Ingress
          - Egress
```

**Policy Testing Framework:**
```python
# Automated policy validation testing
import kubernetes
import yaml
from pathlib import Path

class PolicyComplianceTest:
    """Test framework for validating Kubernetes policy compliance."""
    
    def __init__(self, environment):
        self.environment = environment
        self.policies = self.load_policies_for_environment(environment)
    
    def test_image_tag_compliance(self):
        """Validate that production environments reject latest tags."""
        test_pod = {
            "apiVersion": "v1",
            "kind": "Pod",
            "metadata": {
                "name": "test-banking-app",
                "namespace": f"{self.environment}-banking"
            },
            "spec": {
                "containers": [
                    {
                        "name": "banking-service",
                        "image": "banking-app:latest"  # Should fail in prod
                    }
                ]
            }
        }
        
        if self.environment == "production":
            # This should be rejected by policy
            assert self.validate_resource(test_pod) == False
        else:
            # This should be allowed in dev/test
            assert self.validate_resource(test_pod) == True
    
    def test_resource_limits_enforced(self):
        """Ensure all containers have resource limits."""
        test_pod = {
            "apiVersion": "v1", 
            "kind": "Pod",
            "spec": {
                "containers": [
                    {
                        "name": "banking-service",
                        "image": "banking-app:v1.2.3"
                        # Missing resource limits - should fail
                    }
                ]
            }
        }
        
        # All environments should enforce resource limits
        assert self.validate_resource(test_pod) == False
    
    def test_network_isolation(self):
        """Verify network policies are automatically created."""
        namespace = f"{self.environment}-sensitive-banking"
        
        # Check that deny-all network policy exists
        network_policies = self.get_network_policies_in_namespace(namespace)
        deny_all_exists = any(
            policy['metadata']['name'] == 'deny-all-traffic' 
            for policy in network_policies
        )
        
        assert deny_all_exists, f"Missing deny-all network policy in {namespace}"
```

**Environment-Specific Policy Configuration:**
```yaml
# Policy configuration by environment
policy_enforcement:
  development:
    image_tag_validation: "warn"  # Allow latest tags with warning
    resource_limits: "enforce"
    network_policies: "audit"
    
  staging:
    image_tag_validation: "enforce"  # Reject latest tags
    resource_limits: "enforce" 
    network_policies: "enforce"
    
  production:
    image_tag_validation: "enforce"
    resource_limits: "enforce"
    network_policies: "enforce"
    security_contexts: "enforce"
    pod_security_standards: "restricted"
```

**Policy Monitoring and Alerting:**
```yaml
# Prometheus rules for policy violations
groups:
- name: kyverno.policy.violations
  rules:
  - alert: PolicyViolationInProduction
    expr: increase(kyverno_policy_rule_info_total{policy_validation_mode="enforce", policy_type="validate", policy_background_mode="false", rule_result="fail"}[5m]) > 0
    for: 0m
    labels:
      severity: critical
      compliance_impact: "high"
    annotations:
      summary: "Kyverno policy violation detected in production"
      description: "Policy {{ $labels.policy_name }} rule {{ $labels.rule_name }} has been violated"
      runbook_url: "https://wiki.company.com/runbooks/policy-violations"
```

**Production Importance:** Automated policy enforcement reduces security risks, ensures regulatory compliance, and provides consistency across environments while allowing appropriate flexibility for development. This is essential for financial services to meet PCI DSS, SOX, and other regulatory requirements.

---

### CI/CD & DevOps

**Q7: How would you implement secure, keyless authentication between GitHub Actions and cloud providers using OIDC?**

**Answer:** OpenID Connect (OIDC) provides secure, keyless authentication by establishing trust between GitHub Actions and cloud providers without storing long-lived credentials. This approach follows zero-trust security principles:

**Benefits:**
- **No secrets in repositories**: Eliminates risk of credential exposure
- **Temporary access tokens**: Short-lived tokens reduce attack surface
- **Fine-grained permissions**: Role-based access with specific resource scoping
- **Audit trails**: Better logging and monitoring of access patterns

**Code Example:**
```yaml
# GitHub Actions workflow with OIDC
name: Deploy Banking Infrastructure
on:
  push:
    branches: [main]
    paths: ['infrastructure/**']

permissions:
  contents: read
  id-token: write  # Required for OIDC token generation

env:
  ARM_CLIENT_ID: ${{ vars.AZURE_CLIENT_ID }}
  ARM_USE_OIDC: true
  ARM_SUBSCRIPTION_ID: ${{ vars.AZURE_SUBSCRIPTION_ID }}
  ARM_TENANT_ID: ${{ vars.AZURE_TENANT_ID }}

jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: production  # Environment protection rules
    
    steps:
    - name: Checkout
      uses: actions/checkout@v4
      
    - name: Azure Login via OIDC
      uses: azure/login@v1
      with:
        client-id: ${{ vars.AZURE_CLIENT_ID }}
        tenant-id: ${{ vars.AZURE_TENANT_ID }}
        subscription-id: ${{ vars.AZURE_SUBSCRIPTION_ID }}
        
    - name: Deploy Infrastructure
      run: |
        terraform init
        terraform plan -var-file="production.tfvars"
        terraform apply -auto-approve
      working-directory: ./infrastructure
```

**Azure OIDC Configuration:**
```terraform
# Federated identity credential setup
resource "azuread_application" "github_oidc" {
  display_name = "GitHub-OIDC-Banking-Platform"
  
  # Enable OIDC
  web {
    implicit_grant {
      access_token_issuance_enabled = true
      id_token_issuance_enabled     = true
    }
  }
}

resource "azuread_service_principal" "github_oidc" {
  application_id = azuread_application.github_oidc.application_id
}

# Federated credential for specific repository and branch
resource "azuread_application_federated_identity_credential" "github_main" {
  application_object_id = azuread_application.github_oidc.object_id
  display_name         = "github-main-branch"
  description          = "GitHub Actions deployment from main branch"
  audiences            = ["api://AzureADTokenExchange"]
  issuer              = "https://token.actions.githubusercontent.com"
  subject             = "repo:company/banking-platform:ref:refs/heads/main"
}

# Federated credential for pull requests (read-only access)
resource "azuread_application_federated_identity_credential" "github_pr" {
  application_object_id = azuread_application.github_oidc.object_id
  display_name         = "github-pull-requests"
  description          = "GitHub Actions for pull request validation"
  audiences            = ["api://AzureADTokenExchange"]
  issuer              = "https://token.actions.githubusercontent.com"
  subject             = "repo:company/banking-platform:pull_request"
}

# Role assignments with minimal permissions
resource "azurerm_role_assignment" "github_oidc_contributor" {
  scope              = azurerm_resource_group.banking_platform.id
  role_definition_name = "Contributor"
  principal_id       = azuread_service_principal.github_oidc.object_id
  
  # Conditional access - only during specific hours
  condition = <<EOT
    (
      (!(ActionMatches{'Microsoft.Resources/subscriptions/resourceGroups/delete'}))
      AND
      (TimeOfDay >= 08:00 AND TimeOfDay <= 18:00)
    )
  EOT
}

# Separate role for pull requests (read-only)
resource "azurerm_role_assignment" "github_oidc_reader" {
  scope              = azurerm_resource_group.banking_platform.id  
  role_definition_name = "Reader"
  principal_id       = azuread_service_principal.github_oidc.object_id
  
  condition = <<EOT
    (
      (@Request[Microsoft.Authorization/roleAssignments:PrincipalType] StringEquals 'ServicePrincipal')
      AND 
      (@Request[Microsoft.Authorization/roleAssignments:RoleDefinitionId] StringLike '*Reader*')
    )
  EOT
}
```

**Advanced Security Configuration:**
```yaml
# Environment-specific OIDC with approval gates
environments:
  development:
    oidc_subject: "repo:company/banking-platform:ref:refs/heads/develop"
    approval_required: false
    allowed_actions: ["plan", "apply", "destroy"]
    
  staging:
    oidc_subject: "repo:company/banking-platform:ref:refs/heads/main"
    approval_required: true
    reviewers: ["platform-team"]
    allowed_actions: ["plan", "apply"]
    
  production:
    oidc_subject: "repo:company/banking-platform:ref:refs/heads/main"
    approval_required: true
    reviewers: ["platform-team", "security-team"]
    allowed_actions: ["plan", "apply"]
    time_restrictions: "business_hours_only"
```

**Token Validation and Monitoring:**
```python
# Custom OIDC token validation
import jwt
import requests
from datetime import datetime

class OIDCTokenValidator:
    def __init__(self):
        self.github_oidc_issuer = "https://token.actions.githubusercontent.com"
        
    def validate_github_token(self, token):
        """Validate GitHub OIDC token claims."""
        
        # Get GitHub OIDC public keys
        jwks_response = requests.get(f"{self.github_oidc_issuer}/.well-known/jwks")
        jwks = jwks_response.json()
        
        # Decode and validate token
        try:
            decoded_token = jwt.decode(
                token,
                jwks,
                algorithms=["RS256"],
                issuer=self.github_oidc_issuer,
                audience="api://AzureADTokenExchange"
            )
            
            # Validate specific claims
            required_claims = {
                'repository': 'company/banking-platform',
                'ref': 'refs/heads/main',
                'actor': None  # Can be any authorized user
            }
            
            for claim, expected_value in required_claims.items():
                if expected_value and decoded_token.get(claim) != expected_value:
                    raise ValueError(f"Invalid {claim} claim")
                    
            return {
                'valid': True,
                'repository': decoded_token['repository'],
                'actor': decoded_token['actor'],
                'workflow': decoded_token['workflow']
            }
            
        except jwt.InvalidTokenError as e:
            return {'valid': False, 'error': str(e)}
```

**Production Importance:** OIDC eliminates long-lived secrets in CI/CD pipelines, significantly reducing the attack surface. In financial services, this approach meets zero-trust security requirements and provides detailed audit trails for compliance with regulatory frameworks like SOX and PCI DSS.

---

**Q8: Design a matrix build strategy for deploying infrastructure across multiple customer organizations in parallel.**

**Answer:** Matrix builds enable parallel execution of similar deployment tasks across multiple dimensions (customers, environments, regions). This is particularly valuable for multi-tenant SaaS platforms serving multiple organizations:

**Strategy Components:**
- **Dynamic matrix generation** based on customer configuration
- **Parallel execution** with appropriate concurrency limits
- **Dependency management** between deployment phases
- **Independent failure handling** per customer/environment
- **Resource utilization optimization** across runners

**Code Example:**
```yaml
# Multi-dimensional matrix build workflow
name: Multi-Customer Infrastructure Deployment

on:
  workflow_dispatch:
    inputs:
      deployment_tier:
        type: choice
        options: [development, standard, enterprise]
        description: "Customer tier for deployment"

jobs:
  generate-matrix:
    runs-on: ubuntu-latest
    outputs:
      matrix: ${{ steps.set-matrix.outputs.matrix }}
    steps:
    - name: Checkout
      uses: actions/checkout@v4
      
    - name: Generate deployment matrix
      id: set-matrix
      run: |
        # Read customer configuration from JSON
        CUSTOMERS=$(jq -r --arg tier "${{ inputs.deployment_tier }}" \
          '.tiers[$tier].customers[]' customer-config.json)
        
        # Generate matrix combining customers and regions
        MATRIX=$(jq -n \
          --argjson customers "$(echo "$CUSTOMERS" | jq -R . | jq -s .)" \
          --argjson environments '["dev", "stg", "prod"]' \
          --argjson regions '["eastus", "westeurope"]' \
          '{
            include: [
              foreach .customers[] as $customer (
                foreach .environments[] as $env (
                  foreach .regions[] as $region (
                    {
                      customer: $customer,
                      environment: $env,
                      region: $region,
                      terraform_workspace: ($customer + "-" + $env + "-" + $region)
                    }
                  )
                )
              )
            ]
          }'
        )
        
        echo "matrix=$MATRIX" >> $GITHUB_OUTPUT

  deploy-infrastructure:
    needs: generate-matrix
    runs-on: ubuntu-latest
    strategy:
      matrix: ${{ fromJson(needs.generate-matrix.outputs.matrix) }}
      fail-fast: false  # Continue other deployments if one fails
      max-parallel: 5   # Limit concurrent deployments
      
    # Concurrency control per customer-environment
    concurrency:
      group: deploy-${{ matrix.customer }}-${{ matrix.environment }}-${{ matrix.region }}
      cancel-in-progress: false
      
    environment: 
      name: ${{ matrix.environment }}
      url: https://${{ matrix.customer }}-${{ matrix.environment }}.platform.com
      
    steps:
    - name: Checkout
      uses: actions/checkout@v4
      
    - name: Setup environment variables
      run: |
        echo "TF_WORKSPACE=${{ matrix.terraform_workspace }}" >> $GITHUB_ENV
        echo "CUSTOMER_ID=${{ matrix.customer }}" >> $GITHUB_ENV
        echo "ENVIRONMENT=${{ matrix.environment }}" >> $GITHUB_ENV
        echo "REGION=${{ matrix.region }}" >> $GITHUB_ENV
        
    - name: Azure Login
      uses: azure/login@v1
      with:
        client-id: ${{ vars.AZURE_CLIENT_ID }}
        tenant-id: ${{ vars.AZURE_TENANT_ID }}
        subscription-id: ${{ vars.AZURE_SUBSCRIPTION_ID }}
        
    - name: Terraform Init and Plan
      run: |
        terraform init \
          -backend-config="key=customers/${{ matrix.customer }}/${{ matrix.environment }}/${{ matrix.region }}/terraform.tfstate"
          
        terraform workspace select ${{ matrix.terraform_workspace }} || \
          terraform workspace new ${{ matrix.terraform_workspace }}
          
        terraform plan \
          -var-file="configs/${{ matrix.customer }}.tfvars" \
          -var="environment=${{ matrix.environment }}" \
          -var="region=${{ matrix.region }}" \
          -out=${{ matrix.terraform_workspace }}.tfplan
          
    - name: Terraform Apply
      if: github.ref == 'refs/heads/main' && matrix.environment != 'prod'
      run: |
        terraform apply -auto-approve ${{ matrix.terraform_workspace }}.tfplan
        
    - name: Terraform Apply (Production with Approval)
      if: github.ref == 'refs/heads/main' && matrix.environment == 'prod'
      uses: trstringer/manual-approval@v1
      with:
        secret: ${{ github.TOKEN }}
        approvers: platform-team,security-team
        minimum-approvals: 2
        issue-title: "Production deployment approval for ${{ matrix.customer }}"
        
    - name: Upload State Artifacts
      if: always()
      uses: actions/upload-artifact@v4
      with:
        name: terraform-state-${{ matrix.customer }}-${{ matrix.environment }}-${{ matrix.region }}
        path: |
          .terraform
          *.tfplan
          *.tfstate*
```

**Customer Configuration Structure:**
```json
{
  "tiers": {
    "development": {
      "customers": ["internal-dev", "sandbox-customer"],
      "deployment_schedule": "immediate",
      "approval_required": false
    },
    "standard": {
      "customers": ["bank-a", "credit-union-b", "fintech-c"],
      "deployment_schedule": "weekly",
      "approval_required": true
    },
    "enterprise": {
      "customers": ["major-bank", "insurance-corp"],
      "deployment_schedule": "monthly",
      "approval_required": true,
      "minimum_approvers": 3
    }
  },
  "deployment_order": [
    { "tier": "development", "parallelism": 10 },
    { "tier": "standard", "parallelism": 5 },
    { "tier": "enterprise", "parallelism": 2 }
  ]
}
```

**Advanced Matrix with Dependencies:**
```yaml
# Phased deployment with dependencies
jobs:
  deploy-shared-services:
    strategy:
      matrix:
        region: [eastus, westeurope]
    # Deploy shared services first
    
  deploy-customer-infrastructure:
    needs: deploy-shared-services
    strategy:
      matrix:
        include: ${{ fromJson(needs.generate-matrix.outputs.customer_matrix) }}
    # Customer-specific infrastructure
    
  deploy-applications:
    needs: deploy-customer-infrastructure 
    strategy:
      matrix:
        include: ${{ fromJson(needs.generate-matrix.outputs.app_matrix) }}
    # Application deployments per customer
    
  run-health-checks:
    needs: deploy-applications
    strategy:
      matrix:
        include: ${{ fromJson(needs.generate-matrix.outputs.customer_matrix) }}
    # Validate deployments
```

**Error Handling and Rollback:**
```yaml
    - name: Handle Deployment Failures
      if: failure()
      run: |
        echo "Deployment failed for ${{ matrix.customer }}"
        
        # Notify customer-specific teams
        curl -X POST "${{ secrets.SLACK_WEBHOOK }}" \
          -H 'Content-type: application/json' \
          --data '{
            "channel": "#alerts-${{ matrix.customer }}",
            "text": "❌ Infrastructure deployment failed for ${{ matrix.customer }} in ${{ matrix.environment }}",
            "attachments": [{
              "color": "danger",
              "fields": [{
                "title": "Workflow",
                "value": "${{ github.repository }}/${{ github.workflow }}",
                "short": true
              }, {
                "title": "Commit",
                "value": "${{ github.sha }}",
                "short": true
              }]
            }]
          }'
          
        # Trigger rollback workflow
        gh workflow run rollback-customer.yml \
          -f customer="${{ matrix.customer }}" \
          -f environment="${{ matrix.environment }}" \
          -f region="${{ matrix.region }}"
```

**Production Importance:** Matrix builds enable efficient scaling across multiple tenants while maintaining isolation and allowing for selective rollbacks when issues occur. This approach is essential for SaaS platforms serving financial institutions where deployment failures must be contained to specific customers without affecting others.

---

### Banking Domain & Integrations

**Q9: Design an integration architecture for connecting multiple core banking systems through a unified platform.**

**Answer:** A unified banking integration platform must accommodate diverse core banking systems with different APIs, data formats, and connectivity requirements. The architecture should provide standardized interfaces while handling the complexity of legacy system integration:

**Common Core Banking System Types:**
- **Cloud-Native Systems**: Modern API-first platforms (REST/GraphQL)
- **Traditional Core Systems**: Legacy mainframe systems (often SOAP/batch processing)
- **Specialized Systems**: Payment processors, wealth management, lending platforms
- **Regional Systems**: Local banking solutions with specific compliance requirements

**Integration Architecture:**
```yaml
# Connector deployment structure
banking_connectors:
  account_management:
    supported_systems:
      - mambu       # Cloud-native core banking
      - flexcube    # Oracle universal banking
      - silverlake  # Traditional retail banking
      - custom_core # Customer-specific systems
    
  payment_processing:
    supported_systems:
      - fis         # Financial industry solutions
      - fiservdna   # Core processing platform
      - custom_payment_rails
    
  customer_data:
    supported_systems:
      - salesforce  # CRM integration
      - custom_kyc  # Know your customer systems
      - identity_verification

# ArgoCD application structure
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: banking-integration-platform
  namespace: argocd
spec:
  project: banking
  destination:
    server: https://kubernetes.default.svc
    namespace: banking-connectors
  sources:
    # Configuration source
    - repoURL: https://github.com/company/banking-configs.git
      targetRevision: main
      ref: config-repo
    
    # Account service connectors
    - repoURL: registry.company.com/charts
      chart: core-banking-connector
      targetRevision: 2.1.0
      helm:
        releaseName: account-connector
        parameters:
        - name: connector.type
          value: "account"
        - name: connector.backend
          value: "mambu"
        valueFiles:
          - $config-repo/environments/prod/platform-defaults.yaml
          - $config-repo/environments/prod/connectors/account-mambu.yaml
    
    # Payment processing connector
    - repoURL: registry.company.com/charts
      chart: payment-connector
      targetRevision: 1.8.5
      helm:
        releaseName: payment-connector
        parameters:
        - name: connector.type
          value: "payment"
        - name: connector.backend
          value: "fis"
        valueFiles:
          - $config-repo/environments/prod/platform-defaults.yaml
          - $config-repo/environments/prod/connectors/payment-fis.yaml

    # Customer data connector
    - repoURL: registry.company.com/charts
      chart: customer-connector
      targetRevision: 3.0.1
      helm:
        releaseName: customer-connector
        valueFiles:
          - $config-repo/environments/prod/platform-defaults.yaml
          - $config-repo/environments/prod/connectors/customer-salesforce.yaml
```

**Standardized API Gateway:**
```yaml
# API Gateway configuration for unified banking APIs
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: banking-api-gateway
spec:
  hosts:
  - api.banking-platform.com
  http:
  # Account operations - route to appropriate connector
  - match:
    - uri:
        prefix: /v1/accounts
    route:
    - destination:
        host: account-connector-service
        subset: mambu
      weight: 80
    - destination:
        host: account-connector-service  
        subset: flexcube
      weight: 20
    headers:
      request:
        set:
          x-banking-system: "determined-by-customer-config"
          
  # Payment operations
  - match:
    - uri:
        prefix: /v1/payments
    route:
    - destination:
        host: payment-connector-service
        subset: fis
      weight: 100
    timeout: 30s
    retries:
      attempts: 3
      perTryTimeout: 10s
      
  # Customer operations  
  - match:
    - uri:
        prefix: /v1/customers
    route:
    - destination:
        host: customer-connector-service
        subset: salesforce
```

**Connector Implementation Pattern:**
```java
// Abstract connector interface
public abstract class BankingSystemConnector {
    protected final String systemType;
    protected final ConnectionConfig config;
    
    public abstract AccountResponse createAccount(CreateAccountRequest request);
    public abstract PaymentResponse processPayment(PaymentRequest request);
    public abstract CustomerResponse getCustomer(String customerId);
    
    // Common error handling and retry logic
    protected <T> T executeWithRetry(Supplier<T> operation) {
        return RetryTemplate.builder()
            .maxAttempts(3)
            .exponentialBackoff(1000, 2, 10000)
            .retryOn(TransientException.class)
            .build()
            .execute(context -> operation.get());
    }
}

// Mambu-specific implementation
@Component
@ConditionalOnProperty(name = "banking.connector.type", havingValue = "mambu")
public class MambuConnector extends BankingSystemConnector {
    
    private final MambuApiClient mambuClient;
    private final DataTransformationService transformer;
    
    @Override
    public AccountResponse createAccount(CreateAccountRequest request) {
        // Transform standardized request to Mambu format
        MambuAccountRequest mambuRequest = transformer.toMambuFormat(request);
        
        // Execute with retry and circuit breaker
        return executeWithRetry(() -> {
            MambuAccountResponse mambuResponse = mambuClient.createAccount(mambuRequest);
            return transformer.toStandardFormat(mambuResponse);
        });
    }
    
    @CircuitBreaker(name = "mambu-api", fallbackMethod = "fallbackCreateAccount")
    @TimeLimiter(name = "mambu-api")
    @Retryable(value = {ConnectException.class})
    public CompletableFuture<AccountResponse> createAccountAsync(CreateAccountRequest request) {
        return CompletableFuture.supplyAsync(() -> createAccount(request));
    }
    
    public AccountResponse fallbackCreateAccount(CreateAccountRequest request, Exception ex) {
        // Implement fallback logic - maybe cache or alternative system
        return AccountResponse.builder()
            .status("QUEUED")
            .message("Request queued for later processing due to system unavailability")
            .build();
    }
}

// Flexcube implementation
@Component
@ConditionalOnProperty(name = "banking.connector.type", havingValue = "flexcube")
public class FlexcubeConnector extends BankingSystemConnector {
    // Different implementation for Oracle Flexcube's SOAP APIs
}
```

**Configuration-Driven Routing:**
```yaml
# Customer-specific banking system routing
customer_banking_config:
  bank_a:
    account_system: "mambu"
    payment_system: "fis"
    customer_system: "salesforce"
    region: "us-east"
    
  credit_union_b:
    account_system: "flexcube"
    payment_system: "fiservdna"
    customer_system: "custom_crm"
    region: "us-west"
    
  fintech_startup_c:
    account_system: "mambu"
    payment_system: "stripe"
    customer_system: "hubspot"
    region: "eu-west"

# Routing logic in Kubernetes ConfigMap
apiVersion: v1
kind: ConfigMap
metadata:
  name: banking-routing-config
data:
  routing.yaml: |
    routing_rules:
      - customer_id_pattern: "^bank_a_.*"
        account_backend: "mambu"
        payment_backend: "fis"
      - customer_id_pattern: "^cu_b_.*"
        account_backend: "flexcube"
        payment_backend: "fiservdna"
      - default:
        account_backend: "mambu"
        payment_backend: "fis"
```

**Data Transformation and Validation:**
```java
// Standardized banking data models
@JsonInclude(JsonInclude.Include.NON_NULL)
public class StandardAccountRequest {
    private String customerId;
    private String accountType;  // CHECKING, SAVINGS, LOAN, etc.
    private BigDecimal initialBalance;
    private String currency;
    private Map<String, Object> additionalProperties;  // System-specific fields
}

// Transformation service
@Service
public class BankingDataTransformer {
    
    public MambuAccountRequest toMambuFormat(StandardAccountRequest standard) {
        return MambuAccountRequest.builder()
            .encodedKey(standard.getCustomerId())
            .accountType(mapAccountType(standard.getAccountType()))
            .accountHolderKey(standard.getCustomerId())
            .currencyCode(standard.getCurrency())
            .balance(standard.getInitialBalance())
            .customFields(transformCustomFields(standard.getAdditionalProperties()))
            .build();
    }
    
    private String mapAccountType(String standardType) {
        return switch (standardType) {
            case "CHECKING" -> "CURRENT_ACCOUNT";
            case "SAVINGS" -> "SAVINGS_ACCOUNT";
            case "LOAN" -> "LOAN_ACCOUNT";
            default -> throw new IllegalArgumentException("Unsupported account type: " + standardType);
        };
    }
}
```

**Production Importance:** This architecture enables financial institutions to connect multiple banking systems through a standardized interface, reducing integration complexity and enabling faster onboarding of new customers. The connector pattern allows for system-specific optimizations while maintaining API consistency across different banking platforms.

---

**Q10: How would you implement private networking and connectivity for a multi-region financial platform using Azure Private Endpoints and Site-to-Site VPN?**

**Answer:** Private networking is critical for financial services to ensure data never traverses the public internet and meets regulatory compliance requirements. The implementation involves multiple layers of network isolation:

**Private Endpoint Architecture:**
```terraform
# Hub-and-spoke network topology with private endpoints
resource "azurerm_virtual_network" "hub_vnet" {
  name                = "vnet-hub-${var.region}"
  location            = var.location
  resource_group_name = azurerm_resource_group.networking.name
  address_space       = ["10.0.0.0/16"]
}

# Spoke networks for different environments
resource "azurerm_virtual_network" "spoke_vnet" {
  for_each = var.environments
  
  name                = "vnet-spoke-${each.key}-${var.region}"
  location            = var.location
  resource_group_name = azurerm_resource_group.networking.name
  address_space       = ["10.${each.value.subnet_offset}.0.0/16"]
}

# Private DNS zones for service resolution
resource "azurerm_private_dns_zone" "banking_services" {
  for_each = toset([
    "privatelink.database.windows.net",
    "privatelink.vault.azure.net", 
    "privatelink.servicebus.windows.net",
    "privatelink.blob.core.windows.net"
  ])
  
  name                = each.value
  resource_group_name = azurerm_resource_group.networking.name
}

# Private endpoint for Azure SQL Database
resource "azurerm_private_endpoint" "sql_database" {
  for_each = var.customer_databases
  
  name                = "pe-sql-${each.key}"
  location            = var.location
  resource_group_name = azurerm_resource_group.data.name
  subnet_id           = azurerm_subnet.private_endpoints.id

  private_service_connection {
    name                           = "pe-sql-${each.key}"
    private_connection_resource_id = azurerm_mssql_server.banking[each.key].id
    is_manual_connection           = false
    subresource_names             = ["sqlServer"]
  }

  private_dns_zone_group {
    name                 = "sql-dns-zone-group"
    private_dns_zone_ids = [azurerm_private_dns_zone.banking_services["privatelink.database.windows.net"].id]
  }
}
```

**Site-to-Site VPN Configuration:**
```terraform
# Virtual Network Gateway for S2S VPN
resource "azurerm_virtual_network_gateway" "banking_vpn" {
  name                = "vgw-banking-${var.region}"
  location            = var.location
  resource_group_name = azurerm_resource_group.networking.name
  
  type     = "Vpn"
  vpn_type = "RouteBased"
  sku      = "VpnGw2AZ"  # Zone-redundant for HA
  
  active_active = true  # Active-active for redundancy
  enable_bgp    = true  # BGP for dynamic routing
  
  ip_configuration {
    name                 = "vnetGatewayConfig1"
    public_ip_address_id = azurerm_public_ip.vpn_gateway_1.id
    subnet_id           = azurerm_subnet.gateway_subnet.id
  }
  
  ip_configuration {
    name                 = "vnetGatewayConfig2"
    public_ip_address_id = azurerm_public_ip.vpn_gateway_2.id
    subnet_id           = azurerm_subnet.gateway_subnet.id
  }
  
  bgp_settings {
    asn = 65515
    peering_address = "169.254.21.1"
  }
}

# Local Network Gateway for customer premises
resource "azurerm_local_network_gateway" "customer_premises" {
  for_each = var.customer_vpn_configs
  
  name                = "lng-${each.key}"
  location            = var.location
  resource_group_name = azurerm_resource_group.networking.name
  gateway_address     = each.value.public_ip
  address_space       = each.value.on_premises_cidrs
  
  bgp_settings {
    asn                 = each.value.bgp_asn
    bgp_peering_address = each.value.bgp_peer_ip
  }
}

# VPN connections with redundancy
resource "azurerm_virtual_network_gateway_connection" "customer_vpn" {
  for_each = var.customer_vpn_configs
  
  name                = "conn-${each.key}"
  location            = var.location
  resource_group_name = azurerm_resource_group.networking.name
  
  type                       = "IPsec"
  virtual_network_gateway_id = azurerm_virtual_network_gateway.banking_vpn.id
  local_network_gateway_id   = azurerm_local_network_gateway.customer_premises[each.key].id
  
  shared_key = random_password.vpn_shared_key[each.key].result
  enable_bgp = true
  
  # IPSec policies for financial grade security
  ipsec_policy {
    dh_group         = "DHGroup24"
    ike_encryption   = "AES256"
    ike_integrity    = "SHA384"
    ipsec_encryption = "GCMAES256"
    ipsec_integrity  = "GCMAES256"
    pfs_group        = "PFS24"
    sa_datasize      = 102400000
    sa_lifetime      = 27000
  }
}
```

**Private Link Service for Custom Applications:**
```terraform
# Private Link Service for internal banking APIs
resource "azurerm_private_link_service" "banking_api" {
  name                = "pls-banking-api"
  location            = var.location
  resource_group_name = azurerm_resource_group.networking.name
  
  load_balancer_frontend_ip_configuration_ids = [
    azurerm_lb.internal.frontend_ip_configuration.0.id
  ]
  
  nat_ip_configuration {
    name      = "primary"
    primary   = true
    subnet_id = azurerm_subnet.private_link_service.id
  }
  
  auto_approval_subscription_ids = var.approved_subscription_ids
  visibility_subscription_ids    = var.allowed_subscription_ids
  
  tags = {
    Environment = var.environment
    Service     = "BankingAPI"
    Compliance  = "PCI-DSS"
  }
}

# Private endpoint in customer subscription
resource "azurerm_private_endpoint" "customer_banking_access" {
  for_each = var.customer_subscriptions
  
  name                = "pe-banking-api-${each.key}"
  location            = var.location
  resource_group_name = "rg-${each.key}-networking"
  subnet_id          = "/subscriptions/${each.value.subscription_id}/resourceGroups/rg-${each.key}-networking/providers/Microsoft.Network/virtualNetworks/vnet-${each.key}/subnets/private-endpoints"
  
  private_service_connection {
    name                           = "psc-banking-api-${each.key}"
    private_connection_resource_id = azurerm_private_link_service.banking_api.id
    is_manual_connection          = false
  }
}
```

**Network Security and Monitoring:**
```terraform
# Network Security Groups with financial compliance rules
resource "azurerm_network_security_group" "banking_subnet" {
  name                = "nsg-banking-${var.environment}"
  location            = var.location
  resource_group_name = azurerm_resource_group.networking.name
  
  # Deny all inbound by default
  security_rule {
    name                       = "DenyAllInbound"
    priority                   = 4096
    direction                  = "Inbound"
    access                     = "Deny"
    protocol                   = "*"
    source_port_range          = "*"
    destination_port_range     = "*"
    source_address_prefix      = "*"
    destination_address_prefix = "*"
  }
  
  # Allow specific banking services
  security_rule {
    name                       = "AllowBankingAPI"
    priority                   = 100
    direction                  = "Inbound"
    access                     = "Allow"
    protocol                   = "Tcp"
    source_port_range          = "*"
    destination_port_range     = "8080"
    source_address_prefixes    = [for vnet in azurerm_virtual_network.spoke_vnet : vnet.address_space[0]]
    destination_address_prefix = "VirtualNetwork"
  }
  
  # Audit logging for all network traffic
  security_rule {
    name                       = "LogAllTraffic"
    priority                   = 110
    direction                  = "Inbound"
    access                     = "Allow"
    protocol                   = "*"
    source_port_range          = "*"
    destination_port_range     = "*"
    source_address_prefix      = "VirtualNetwork"
    destination_address_prefix = "VirtualNetwork"
  }
}

# Network Watcher for traffic analytics
resource "azurerm_network_watcher_flow_log" "banking_flow_log" {
  network_watcher_name = azurerm_network_watcher.banking.name
  resource_group_name  = azurerm_resource_group.networking.name
  
  network_security_group_id = azurerm_network_security_group.banking_subnet.id
  storage_account_id        = azurerm_storage_account.flow_logs.id
  enabled                   = true
  
  retention_policy {
    enabled = true
    days    = 365  # 1 year retention for compliance
  }
  
  traffic_analytics {
    enabled               = true
    workspace_id          = azurerm_log_analytics_workspace.banking.workspace_id
    workspace_region      = var.location
    workspace_resource_id = azurerm_log_analytics_workspace.banking.id
    interval_in_minutes   = 10
  }
}
```

**Production Importance:** Private networking ensures financial data never traverses public internet, meeting PCI DSS and regulatory requirements. Site-to-site VPN provides secure connectivity to customer premises while private endpoints ensure Azure services are accessible only through private IP addresses within the virtual network.

---

**Q11: Design a cost optimization strategy for a multi-tenant Kubernetes platform serving financial institutions.**

**Answer:** Cost optimization in financial services requires balancing regulatory compliance, performance requirements, and operational efficiency. The strategy must consider both infrastructure costs and operational overhead:

**Resource Right-Sizing and Auto-Scaling:**
```yaml
# Vertical Pod Autoscaler for right-sizing
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: banking-service-vpa
  namespace: banking-prod
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: account-service
  updatePolicy:
    updateMode: "Auto"  # Automatically apply recommendations
  resourcePolicy:
    containerPolicies:
    - containerName: account-service
      maxAllowed:
        cpu: "2"
        memory: "4Gi"
      minAllowed:
        cpu: "100m" 
        memory: "128Mi"
      controlledResources: ["cpu", "memory"]

---
# Horizontal Pod Autoscaler with custom metrics
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: banking-service-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: account-service
  minReplicas: 2  # Maintain HA for financial services
  maxReplicas: 20
  metrics:
  # CPU utilization
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  # Custom metric: transactions per second
  - type: Pods
    pods:
      metric:
        name: transactions_per_second
      target:
        type: AverageValue
        averageValue: "100"
  # Memory utilization
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 300  # Smooth scaling for banking
      policies:
      - type: Percent
        value: 10  # Scale down max 10% at a time
        periodSeconds: 60
    scaleUp:
      stabilizationWindowSeconds: 60
      policies:
      - type: Percent
        value: 50  # Scale up faster for load spikes
        periodSeconds: 60
```

**Node Pool Optimization:**
```terraform
# Mixed node pools for cost optimization
resource "azurerm_kubernetes_cluster_node_pool" "spot_instances" {
  name                  = "spotpool"
  kubernetes_cluster_id = azurerm_kubernetes_cluster.banking.id
  vm_size              = "Standard_D4s_v5"
  priority             = "Spot"
  eviction_policy      = "Delete"
  spot_max_price       = 0.05  # Maximum price per hour
  
  node_count    = 3
  min_count     = 1
  max_count     = 10
  auto_scaling_enabled = true
  
  # Taints for spot instances
  node_taints = [
    "kubernetes.azure.com/scalesetpriority=spot:NoSchedule"
  ]
  
  node_labels = {
    "node-type" = "spot"
    "workload-type" = "batch-processing"
  }
  
  tags = {
    Environment = "production"
    CostOptimization = "spot-instances"
  }
}

# Reserved instances for predictable workloads
resource "azurerm_kubernetes_cluster_node_pool" "reserved_pool" {
  name                  = "reservedpool"
  kubernetes_cluster_id = azurerm_kubernetes_cluster.banking.id
  vm_size              = "Standard_D8s_v5"
  
  node_count           = 5
  min_count           = 3  # Always maintain minimum for critical services
  max_count           = 15
  auto_scaling_enabled = true
  
  node_labels = {
    "node-type" = "reserved"
    "workload-type" = "critical-banking"
    "cost-model" = "reserved-instance"
  }
}

# Cluster Autoscaler configuration
resource "azurerm_kubernetes_cluster" "banking" {
  # ... other configuration
  
  auto_scaler_profile {
    balance_similar_node_groups      = true
    expander                        = "priority"  # Use priority-based expansion
    max_graceful_termination_sec    = "600"
    max_node_provision_time         = "15m"
    max_unready_nodes              = 3
    max_unready_percentage         = 45
    new_pod_scale_up_delay         = "10s"
    scale_down_delay_after_add     = "10m"
    scale_down_delay_after_delete  = "10s"
    scale_down_delay_after_failure = "3m"
    scale_down_unneeded_time       = "10m"
    scale_down_utilization_threshold = 0.5
    scan_interval                  = "10s"
    skip_nodes_with_local_storage  = false
    skip_nodes_with_system_pods    = true
  }
}
```

**Storage Cost Optimization:**
```terraform
# Intelligent tiering for customer data
resource "azurerm_storage_account" "customer_data" {
  for_each = var.customers
  
  name                     = "st${each.key}data${random_integer.suffix.result}"
  resource_group_name      = azurerm_resource_group.customers[each.key].name
  location                = var.location
  account_tier            = "Standard"
  account_replication_type = "ZRS"  # Zone-redundant for compliance
  
  # Lifecycle management for cost optimization
  blob_properties {
    versioning_enabled = true
    delete_retention_policy {
      days = 30
    }
    
    # Intelligent tiering
    container_delete_retention_policy {
      days = 7
    }
  }
}

resource "azurerm_storage_management_policy" "lifecycle_policy" {
  for_each = var.customers
  
  storage_account_id = azurerm_storage_account.customer_data[each.key].id
  
  rule {
    name    = "customerDataLifecycle"
    enabled = true
    
    filters {
      prefix_match = ["customer-data/", "transaction-logs/", "audit-logs/"]
      blob_types   = ["blockBlob"]
    }
    
    actions {
      base_blob {
        tier_to_cool_after_days_since_modification_greater_than    = 30   # Move to cool after 30 days
        tier_to_archive_after_days_since_modification_greater_than = 180  # Archive after 6 months
        delete_after_days_since_modification_greater_than          = 2555  # Delete after 7 years (compliance)
      }
      
      snapshot {
        delete_after_days_since_creation_greater_than = 365  # Keep snapshots for 1 year
      }
      
      version {
        delete_after_days_since_creation = 90  # Keep versions for 90 days
      }
    }
  }
}
```

**Workload Scheduling for Cost Optimization:**
```yaml
# Priority-based scheduling for cost optimization
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: critical-banking-priority
value: 1000000
globalDefault: false
description: "Critical banking services that must always be scheduled"

---
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: batch-processing-priority
value: 100
globalDefault: false
description: "Batch processing that can be preempted"

---
# Deployment with cost-optimized scheduling
apiVersion: apps/v1
kind: Deployment
metadata:
  name: account-service
spec:
  template:
    spec:
      priorityClassName: critical-banking-priority
      nodeSelector:
        node-type: "reserved"  # Use reserved instances for critical workloads
      containers:
      - name: account-service
        image: banking/account-service:v1.2.3
        resources:
          requests:
            cpu: "500m"
            memory: "1Gi"
          limits:
            cpu: "2"
            memory: "4Gi"

---
# Batch job using spot instances
apiVersion: batch/v1
kind: CronJob
metadata:
  name: daily-report-generation
spec:
  schedule: "0 2 * * *"  # Run at 2 AM when costs are lower
  jobTemplate:
    spec:
      template:
        spec:
          priorityClassName: batch-processing-priority
          tolerations:
          - key: "kubernetes.azure.com/scalesetpriority"
            operator: "Equal"
            value: "spot"
            effect: "NoSchedule"
          nodeSelector:
            node-type: "spot"
          containers:
          - name: report-generator
            image: banking/report-generator:v1.0.0
            resources:
              requests:
                cpu: "2" 
                memory: "8Gi"
              limits:
                cpu: "4"
                memory: "16Gi"
          restartPolicy: OnFailure
```

**Cost Monitoring and Alerting:**
```yaml
# Prometheus rules for cost monitoring
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: cost-optimization-alerts
spec:
  groups:
  - name: cost.optimization
    rules:
    - alert: HighCPUWaste
      expr: |
        (
          avg_over_time(
            (1 - avg by (namespace, pod) (irate(container_cpu_usage_seconds_total{container!="POD",container!=""}[5m]))) * 100
          [1h])
        ) > 70
      for: 30m
      labels:
        severity: warning
        cost_impact: medium
      annotations:
        summary: "High CPU waste detected in {{ $labels.namespace }}/{{ $labels.pod }}"
        description: "Pod {{ $labels.pod }} in namespace {{ $labels.namespace }} has been using less than 30% of requested CPU for 30 minutes"
        
    - alert: HighMemoryWaste
      expr: |
        (
          avg_over_time(
            (1 - avg by (namespace, pod) (container_memory_working_set_bytes{container!="POD",container!=""} / container_spec_memory_limit_bytes)) * 100
          [1h])
        ) > 70
      for: 30m
      labels:
        severity: warning
        cost_impact: medium
      annotations:
        summary: "High memory waste detected in {{ $labels.namespace }}/{{ $labels.pod }}"
        
    - alert: UnderutilizedNodes
      expr: |
        (
          1 - (
            avg by (instance) (irate(node_cpu_seconds_total{mode!="idle"}[5m])) +
            avg by (instance) (node_memory_MemTotal_bytes - node_memory_MemAvailable_bytes) / avg by (instance) (node_memory_MemTotal_bytes)
          ) / 2
        ) > 0.6
      for: 1h
      labels:
        severity: warning
        cost_impact: high
      annotations:
        summary: "Underutilized node {{ $labels.instance }}"
        description: "Node {{ $labels.instance }} has been underutilized (< 40% avg utilization) for 1 hour"
```

**FinOps Dashboard Configuration:**
```python
# Cost optimization script
import pandas as pd
from azure.mgmt.consumption import ConsumptionManagementClient
from kubernetes import client, config

class FinOpsOptimizer:
    def __init__(self):
        self.subscription_id = os.environ['AZURE_SUBSCRIPTION_ID']
        self.consumption_client = ConsumptionManagementClient(
            credential=DefaultAzureCredential(),
            subscription_id=self.subscription_id
        )
        
    def analyze_cost_trends(self, days=30):
        """Analyze spending trends and identify optimization opportunities."""
        end_date = datetime.now()
        start_date = end_date - timedelta(days=days)
        
        # Get usage details
        usage_details = self.consumption_client.usage_details.list(
            scope=f"/subscriptions/{self.subscription_id}",
            filter=f"properties/usageStart ge '{start_date.isoformat()}' and properties/usageStart le '{end_date.isoformat()}'"
        )
        
        cost_analysis = {}
        for usage in usage_details:
            resource_group = usage.instance_name
            cost = usage.pretax_cost
            
            if resource_group not in cost_analysis:
                cost_analysis[resource_group] = {
                    'total_cost': 0,
                    'vm_cost': 0,
                    'storage_cost': 0,
                    'network_cost': 0
                }
            
            cost_analysis[resource_group]['total_cost'] += cost
            
            # Categorize costs
            if 'Microsoft.Compute' in usage.consumed_service:
                cost_analysis[resource_group]['vm_cost'] += cost
            elif 'Microsoft.Storage' in usage.consumed_service:
                cost_analysis[resource_group]['storage_cost'] += cost
            elif 'Microsoft.Network' in usage.consumed_service:
                cost_analysis[resource_group]['network_cost'] += cost
                
        return cost_analysis
    
    def recommend_optimizations(self):
        """Generate cost optimization recommendations."""
        recommendations = []
        
        # Check for underutilized resources
        config.load_incluster_config()
        v1 = client.CoreV1Api()
        
        # Get node utilization
        nodes = v1.list_node()
        for node in nodes.items:
            # Check if node is underutilized based on metrics
            utilization = self.get_node_utilization(node.metadata.name)
            if utilization['cpu'] < 0.3 and utilization['memory'] < 0.4:
                recommendations.append({
                    'type': 'scale_down',
                    'resource': node.metadata.name,
                    'savings_estimate': 150,  # USD per month
                    'action': 'Consider scaling down this node during off-hours'
                })
        
        return recommendations
```

**Production Importance:** Cost optimization in financial services must balance regulatory requirements with efficiency. This approach reduces infrastructure costs by 30-50% while maintaining compliance and performance standards required for banking operations.

---

## 📙 Advanced Questions

### Architecture & Scalability

**Q12: Design an Istio service mesh architecture for a banking platform handling sensitive financial data with zero-trust networking.**

**Answer:** Istio service mesh provides comprehensive security, observability, and traffic management for microservices architectures. For banking platforms, the focus must be on data encryption, authentication, authorization, and audit trails:

**Istio Security Architecture:**
```yaml
# Strict mTLS across the mesh
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default
  namespace: banking-system
spec:
  mtls:
    mode: STRICT  # Enforce mTLS for all communications

---
# Banking-specific authorization policies
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: account-service-authz
  namespace: banking-system
spec:
  selector:
    matchLabels:
      app: account-service
  rules:
  # Allow only payment service to call account service
  - from:
    - source:
        principals: ["cluster.local/ns/banking-system/sa/payment-service"]
    to:
    - operation:
        methods: ["GET", "POST"]
        paths: ["/api/v1/accounts/*"]
  # Allow customer service with specific JWT claims
  - from:
    - source:
        principals: ["cluster.local/ns/banking-system/sa/customer-service"]
    when:
    - key: request.auth.claims[role]
      values: ["customer-manager", "account-specialist"]
    to:
    - operation:
        methods: ["GET"]
        
  # Deny all other access
  - {}  # Empty rule denies everything else

---
# JWT validation for external API access
apiVersion: security.istio.io/v1beta1
kind: RequestAuthentication
metadata:
  name: banking-api-jwt
  namespace: banking-system
spec:
  selector:
    matchLabels:
      app: api-gateway
  jwtRules:
  - issuer: "https://auth.banking-platform.com"
    jwksUri: "https://auth.banking-platform.com/.well-known/jwks"
    audiences:
    - "banking-api"
    forwardOriginalToken: true
    fromHeaders:
    - name: Authorization
      prefix: "Bearer "
```

**Traffic Management for Financial Compliance:**
```yaml
# Circuit breaker for external banking systems
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: external-bank-circuit-breaker
  namespace: banking-system
spec:
  host: external-bank-api.banking-system.svc.cluster.local
  trafficPolicy:
    connectionPool:
      tcp:
        maxConnections: 10
        connectTimeout: 30s
        tcpKeepalive:
          time: 7200s
          interval: 75s
      http:
        http1MaxPendingRequests: 5
        http2MaxRequests: 10
        maxRequestsPerConnection: 2
        maxRetries: 3
        consecutiveGatewayErrors: 3
        interval: 30s
        baseEjectionTime: 30s
        maxEjectionPercent: 50
        h2UpgradePolicy: UPGRADE
    outlierDetection:
      consecutiveGatewayErrors: 3
      consecutive5xxErrors: 3
      interval: 30s
      baseEjectionTime: 30s
      maxEjectionPercent: 50
      
---
# Canary deployment for banking services
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: account-service-canary
  namespace: banking-system
spec:
  hosts:
  - account-service
  http:
  # Route 5% of traffic to canary version
  - match:
    - headers:
        canary-user:
          exact: "true"
    route:
    - destination:
        host: account-service
        subset: v2-canary
      weight: 100
  # Route 95% to stable version
  - route:
    - destination:
        host: account-service
        subset: v1-stable
      weight: 95
    - destination:
        host: account-service
        subset: v2-canary
      weight: 5
    fault:
      delay:
        percentage:
          value: 0.1
        fixedDelay: 5s
    timeout: 10s
    retries:
      attempts: 3
      perTryTimeout: 3s
      retryOn: 5xx,reset,connect-failure,refused-stream
```

**Observability and Audit Logging:**
```yaml
# Telemetry v2 for detailed banking audits
apiVersion: telemetry.istio.io/v1alpha1
kind: Telemetry
metadata:
  name: banking-audit-logging
  namespace: banking-system
spec:
  metrics:
  - providers:
    - name: prometheus
  - overrides:
    - match:
        metric: ALL_METRICS
      tagOverrides:
        customer_id:
          value: "%{REQUEST_HEADERS['x-customer-id']}"
        transaction_type:
          value: "%{REQUEST_HEADERS['x-transaction-type']}"
        compliance_zone:
          value: "%{REQUEST_HEADERS['x-compliance-zone']}"
  accessLogging:
  - providers:
    - name: otel
  - filter:
      expression: 'has(request.headers["x-audit-required"])'
  tracing:
  - providers:
    - name: jaeger

---
# Custom banking-specific telemetry
apiVersion: telemetry.istio.io/v1alpha1
kind: Telemetry
metadata:
  name: banking-custom-metrics
spec:
  metrics:
  - providers:
    - name: prometheus
    overrides:
    - match:
        metric: requests_total
      disabled: false
      tagOverrides:
        pci_scope:
          operation: UPSERT
          value: |
            has(request.headers['x-pci-data']) ? 'in-scope' : 'out-of-scope'
        financial_operation:
          operation: UPSERT
          value: |
            request.url_path | extractPath() | 
            contains('transfer') ? 'transfer' : 
            contains('balance') ? 'inquiry' : 
            contains('deposit') ? 'deposit' : 'other'
```

**Gateway Configuration for External Banking Connections:**
```yaml
# Secure gateway for external bank integrations
apiVersion: networking.istio.io/v1beta1
kind: Gateway
metadata:
  name: banking-external-gateway
  namespace: banking-system
spec:
  selector:
    istio: eastwestgateway  # Dedicated gateway for bank-to-bank
  servers:
  - port:
      number: 443
      name: https-banking
      protocol: HTTPS
    tls:
      mode: MUTUAL  # Mutual TLS for bank integrations
      credentialName: external-bank-certs
      minProtocolVersion: TLSV1_3
      maxProtocolVersion: TLSV1_3
    hosts:
    - api.external-banking.com
    - secure.bank-integration.com

---
# Virtual service for external bank routing
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: external-banking-routes
  namespace: banking-system
spec:
  hosts:
  - api.external-banking.com
  gateways:
  - banking-external-gateway
  http:
  - match:
    - uri:
        prefix: "/api/v1/swift"
    route:
    - destination:
        host: swift-connector-service
        port:
          number: 8080
    headers:
      request:
        set:
          x-forwarded-proto: "https"
          x-compliance-zone: "pci-dss"
  - match:
    - uri:
        prefix: "/api/v1/ach"
    route:
    - destination:
        host: ach-connector-service
        port:
          number: 8080
```

**Production Importance:** Istio service mesh provides the security, observability, and traffic control required for banking platforms to meet regulatory compliance while enabling reliable communication between microservices handling sensitive financial data.

---

**Q13: Implement a serverless banking event processing system using Knative that can scale from zero to handle high-volume transaction bursts.**

**Answer:** Knative enables serverless Kubernetes for event-driven banking workloads, providing automatic scaling based on traffic and events. This is particularly valuable for processing transaction spikes during peak banking hours:

**Knative Serving for Transaction Processing:**
```yaml
# Serverless transaction validator
apiVersion: serving.knative.dev/v1
kind: Service
metadata:
  name: transaction-validator
  namespace: banking-serverless
  annotations:
    # Autoscaling configuration
    autoscaling.knative.dev/minScale: "0"  # Scale to zero for cost savings
    autoscaling.knative.dev/maxScale: "100"  # Handle high transaction volumes
    autoscaling.knative.dev/target: "50"  # Target 50 concurrent requests per pod
    autoscaling.knative.dev/targetUtilizationPercentage: "70"
    autoscaling.knative.dev/scaleDownDelay: "5m"  # Keep instances for banking bursts
    autoscaling.knative.dev/scaleToZeroGracePeriod: "10m"
    # Banking-specific annotations
    banking.compliance/pci-scope: "true"
    banking.audit/required: "true"
spec:
  template:
    metadata:
      annotations:
        # Pod-specific configuration
        autoscaling.knative.dev/class: "hpa.autoscaling.knative.dev"
        autoscaling.knative.dev/metric: "concurrency"
    spec:
      serviceAccountName: transaction-validator-sa
      containers:
      - name: validator
        image: banking/transaction-validator:v2.1.0
        ports:
        - containerPort: 8080
        env:
        - name: MAX_TRANSACTION_AMOUNT
          value: "1000000"  # $1M limit for single transaction
        - name: FRAUD_DETECTION_ENDPOINT
          value: "https://fraud-detection.banking.internal"
        - name: COMPLIANCE_MODE
          value: "PCI_DSS_LEVEL_1"
        resources:
          requests:
            cpu: "100m"
            memory: "128Mi"
          limits:
            cpu: "2000m"
            memory: "1Gi"
        readinessProbe:
          httpGet:
            path: /health/ready
            port: 8080
          initialDelaySeconds: 3
          periodSeconds: 1
        livenessProbe:
          httpGet:
            path: /health/live
            port: 8080
          initialDelaySeconds: 15
          periodSeconds: 10
      # Istio injection for security
      securityContext:
        runAsNonRoot: true
        runAsUser: 1001
        fsGroup: 1001

---
# Traffic configuration for gradual rollout
apiVersion: serving.knative.dev/v1
kind: Configuration
metadata:
  name: transaction-validator-config
  namespace: banking-serverless
spec:
  template:
    metadata:
      annotations:
        # Blue-green deployment for banking safety
        serving.knative.dev/rolloutDuration: "300s"  # 5-minute rollout
    spec:
      containers:
      - name: validator
        image: banking/transaction-validator:v2.1.0
```

**Knative Eventing for Banking Events:**
```yaml
# Event source for transaction events
apiVersion: sources.knative.dev/v1
kind: SinkBinding
metadata:
  name: transaction-events-source
  namespace: banking-serverless
spec:
  subject:
    apiVersion: apps/v1
    kind: Deployment
    name: core-banking-system
  sink:
    ref:
      apiVersion: eventing.knative.dev/v1
      kind: Broker
      name: banking-events-broker

---
# Event broker for banking events
apiVersion: eventing.knative.dev/v1
kind: Broker
metadata:
  name: banking-events-broker
  namespace: banking-serverless
  annotations:
    eventing.knative.dev/broker.class: "MTChannelBasedBroker"
spec:
  config:
    apiVersion: v1
    kind: ConfigMap
    name: kafka-channel-config
    namespace: knative-eventing

---
# Event triggers for different banking operations
apiVersion: eventing.knative.dev/v1
kind: Trigger
metadata:
  name: high-value-transaction-trigger
  namespace: banking-serverless
spec:
  broker: banking-events-broker
  filter:
    attributes:
      type: "transaction.created"
      amount: ">100000"  # High-value transactions
      currency: "USD"
  subscriber:
    ref:
      apiVersion: serving.knative.dev/v1
      kind: Service
      name: fraud-detection-service

---
apiVersion: eventing.knative.dev/v1
kind: Trigger
metadata:
  name: international-transfer-trigger
  namespace: banking-serverless  
spec:
  broker: banking-events-broker
  filter:
    attributes:
      type: "transaction.created"
      category: "international_transfer"
  subscriber:
    ref:
      apiVersion: serving.knative.dev/v1
      kind: Service
      name: compliance-checker

---
# Audit event trigger
apiVersion: eventing.knative.dev/v1
kind: Trigger
metadata:
  name: audit-logging-trigger
  namespace: banking-serverless
spec:
  broker: banking-events-broker
  filter:
    attributes:
      type: "transaction.*"  # All transaction events
  subscriber:
    ref:
      apiVersion: serving.knative.dev/v1
      kind: Service
      name: audit-logger
```

**Serverless Fraud Detection Service:**
```yaml
apiVersion: serving.knative.dev/v1
kind: Service
metadata:
  name: fraud-detection-service
  namespace: banking-serverless
  annotations:
    # High-performance settings for fraud detection
    autoscaling.knative.dev/minScale: "3"  # Always keep minimum for security
    autoscaling.knative.dev/maxScale: "50"
    autoscaling.knative.dev/target: "10"  # Lower concurrency for ML workloads
    autoscaling.knative.dev/targetUtilizationPercentage: "80"
    autoscaling.knative.dev/scaleDownDelay: "10m"  # Keep instances longer
spec:
  template:
    metadata:
      annotations:
        # GPU support for ML-based fraud detection
        autoscaling.knative.dev/class: "hpa.autoscaling.knative.dev"
        cluster-autoscaler.kubernetes.io/safe-to-evict: "false"
    spec:
      containers:
      - name: fraud-detector
        image: banking/fraud-detection-ml:v1.5.0
        ports:
        - containerPort: 8080
        env:
        - name: MODEL_PATH
          value: "/models/fraud-detection-v3.pkl"
        - name: REDIS_ENDPOINT
          value: "redis://fraud-cache.banking.svc.cluster.local:6379"
        - name: ALERT_THRESHOLD
          value: "0.85"  # 85% fraud probability threshold
        resources:
          requests:
            cpu: "500m"
            memory: "1Gi"
            nvidia.com/gpu: "0"
          limits:
            cpu: "4000m"
            memory: "8Gi" 
            nvidia.com/gpu: "1"  # GPU for ML inference
        volumeMounts:
        - name: ml-models
          mountPath: /models
          readOnly: true
      volumes:
      - name: ml-models
        persistentVolumeClaim:
          claimName: fraud-detection-models
      nodeSelector:
        accelerator: "nvidia-tesla-t4"
```

**Banking Event Schema and Processing:**
```python
# Banking event processing with Knative
from cloudevents.http import CloudEvent
from flask import Flask, request
import json
import logging

app = Flask(__name__)
logging.basicConfig(level=logging.INFO)

@app.route("/", methods=["POST"])
def handle_banking_event():
    """Handle incoming banking events with compliance logging."""
    
    # Parse CloudEvent
    event = CloudEvent.from_http(request.headers, request.data)
    
    # Audit logging for compliance
    audit_log = {
        "event_id": event["id"],
        "event_type": event["type"], 
        "source": event["source"],
        "timestamp": event["time"],
        "subject": event.get("subject", ""),
        "data_content_type": event.get("datacontenttype", ""),
        "compliance_processed": True
    }
    
    try:
        event_data = json.loads(event.data)
        
        # Process based on event type
        if event["type"] == "transaction.created":
            result = process_transaction_event(event_data)
        elif event["type"] == "account.status.changed":
            result = process_account_event(event_data)
        elif event["type"] == "compliance.alert":
            result = process_compliance_alert(event_data)
        else:
            result = {"status": "ignored", "reason": "unknown event type"}
            
        audit_log["processing_result"] = result
        audit_log["status"] = "success"
        
    except Exception as e:
        audit_log["status"] = "error"
        audit_log["error"] = str(e)
        logging.error(f"Error processing banking event: {e}")
        result = {"status": "error", "message": str(e)}
    
    # Send to audit system
    send_to_audit_log(audit_log)
    
    return result, 200

def process_transaction_event(data):
    """Process transaction creation events."""
    transaction_amount = data.get("amount", 0)
    customer_id = data.get("customer_id")
    transaction_type = data.get("type")
    
    # High-value transaction processing
    if transaction_amount > 100000:
        # Trigger additional verification
        return {
            "status": "verification_required",
            "next_steps": ["manager_approval", "document_verification"],
            "compliance_flags": ["high_value", "enhanced_due_diligence"]
        }
    
    # International transaction processing
    elif transaction_type == "international_transfer":
        return {
            "status": "compliance_check",
            "next_steps": ["sanctions_screening", "source_of_funds"],
            "estimated_processing_time": "2-5 business days"
        }
    
    return {"status": "approved", "processing_time": "immediate"}

def send_to_audit_log(audit_data):
    """Send audit data to compliance logging system."""
    # Implementation for audit logging
    logging.info(f"Audit log: {json.dumps(audit_data)}")

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=8080)
```

**Production Importance:** Knative serverless enables cost-efficient processing of variable banking workloads while maintaining the ability to scale rapidly for transaction spikes. The event-driven architecture ensures real-time fraud detection and compliance processing while maximizing resource utilization.

---

**Q14: Design and implement a Kubernetes Operator for managing banking database backups with compliance requirements and automated disaster recovery.**

**Answer:** Kubernetes Operators extend Kubernetes APIs with custom resources and controllers to manage complex stateful applications. For banking platforms, operators must handle compliance requirements, encryption, retention policies, and automated recovery procedures:

**Custom Resource Definition for Banking Backups:**
```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: bankingbackups.banking.platform.com
spec:
  group: banking.platform.com
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
              databaseType:
                type: string
                enum: ["postgresql", "mongodb", "mssql"]
              schedule:
                type: string
                pattern: '^(@(annually|yearly|monthly|weekly|daily|hourly|reboot))|(@every (\d+(ns|us|µs|ms|s|m|h))+)|((((\d+,)+\d+|(\d+(\/|-)\d+)|\d+|\*) ?){5,7})$'
              retentionPolicy:
                type: object
                properties:
                  daily:
                    type: integer
                    minimum: 7  # Minimum 7 days for banking
                  weekly:
                    type: integer
                    minimum: 4
                  monthly:
                    type: integer
                    minimum: 12  # 1 year minimum
                  yearly:
                    type: integer
                    minimum: 7   # 7 years for regulatory compliance
              encryption:
                type: object
                properties:
                  enabled:
                    type: boolean
                    default: true
                  keySource:
                    type: string
                    enum: ["azure-keyvault", "aws-kms", "customer-managed"]
                  keyId:
                    type: string
              compliance:
                type: object
                properties:
                  pciDss:
                    type: boolean
                    default: true
                  sox:
                    type: boolean
                    default: true
                  auditLogging:
                    type: boolean
                    default: true
                  dataClassification:
                    type: string
                    enum: ["public", "internal", "confidential", "restricted"]
              destinationStorage:
                type: object
                properties:
                  primary:
                    type: object
                    properties:
                      type: string
                      endpoint: string
                      region: string
                  secondary:
                    type: object # For geo-redundant backups
                    properties:
                      type: string
                      endpoint: string
                      region: string
          status:
            type: object
            properties:
              lastBackupTime:
                type: string
                format: date-time
              lastBackupStatus:
                type: string
                enum: ["success", "failed", "in-progress"]
              backupSize:
                type: string
              encryptionStatus:
                type: string
              complianceStatus:
                type: object
                properties:
                  pciDssCompliant:
                    type: boolean
                  auditTrailComplete:
                    type: boolean
                  retentionPolicyMet:
                    type: boolean
  scope: Namespaced
  names:
    plural: bankingbackups
    singular: bankingbackup
    kind: BankingBackup
    shortNames:
    - bbkup

---
# Example Banking Backup Custom Resource
apiVersion: banking.platform.com/v1
kind: BankingBackup
metadata:
  name: customer-database-backup
  namespace: banking-prod
spec:
  databaseType: postgresql
  schedule: "0 2 * * *"  # Daily at 2 AM
  retentionPolicy:
    daily: 30
    weekly: 12
    monthly: 12
    yearly: 7
  encryption:
    enabled: true
    keySource: azure-keyvault
    keyId: "https://banking-keyvault.vault.azure.net/keys/backup-encryption-key"
  compliance:
    pciDss: true
    sox: true
    auditLogging: true
    dataClassification: "restricted"
  destinationStorage:
    primary:
      type: "azure-blob"
      endpoint: "https://bankingbackups.blob.core.windows.net"
      region: "eastus"
    secondary:
      type: "azure-blob"
      endpoint: "https://bankingbackupsdr.blob.core.windows.net"
      region: "westus2"
```

**Operator Controller Implementation:**
```go
// Banking Backup Operator Controller
package main

import (
    "context"
    "fmt"
    "time"
    
    "github.com/go-logr/logr"
    batchv1 "k8s.io/api/batch/v1"
    corev1 "k8s.io/api/core/v1"
    metav1 "k8s.io/apimachinery/pkg/apis/meta/v1"
    "k8s.io/apimachinery/pkg/runtime"
    ctrl "sigs.k8s.io/controller-runtime"
    "sigs.k8s.io/controller-runtime/pkg/client"
    
    bankingv1 "banking.platform.com/api/v1"
)

type BankingBackupReconciler struct {
    client.Client
    Log    logr.Logger
    Scheme *runtime.Scheme
}

func (r *BankingBackupReconciler) Reconcile(ctx context.Context, req ctrl.Request) (ctrl.Result, error) {
    log := r.Log.WithValues("bankingbackup", req.NamespacedName)
    
    // Fetch the BankingBackup instance
    var backup bankingv1.BankingBackup
    if err := r.Get(ctx, req.NamespacedName, &backup); err != nil {
        log.Error(err, "unable to fetch BankingBackup")
        return ctrl.Result{}, client.IgnoreNotFound(err)
    }
    
    // Validate compliance requirements
    if err := r.validateCompliance(&backup); err != nil {
        log.Error(err, "compliance validation failed")
        return ctrl.Result{}, err
    }
    
    // Create or update CronJob for scheduled backups
    cronJob := &batchv1.CronJob{
        ObjectMeta: metav1.ObjectMeta{
            Name:      backup.Name + "-cronjob",
            Namespace: backup.Namespace,
        },
        Spec: batchv1.CronJobSpec{
            Schedule: backup.Spec.Schedule,
            JobTemplate: batchv1.JobTemplateSpec{
                Spec: batchv1.JobSpec{
                    Template: corev1.PodTemplateSpec{
                        Spec: corev1.PodSpec{
                            RestartPolicy: corev1.RestartPolicyOnFailure,
                            Containers: []corev1.Container{
                                {
                                    Name:  "backup-worker",
                                    Image: "banking/backup-operator:v1.0.0",
                                    Env: []corev1.EnvVar{
                                        {
                                            Name:  "BACKUP_TYPE",
                                            Value: backup.Spec.DatabaseType,
                                        },
                                        {
                                            Name:  "ENCRYPTION_KEY_ID",
                                            Value: backup.Spec.Encryption.KeyId,
                                        },
                                        {
                                            Name:  "COMPLIANCE_MODE",
                                            Value: "PCI_DSS_SOX",
                                        },
                                        {
                                            Name: "PRIMARY_STORAGE",
                                            Value: backup.Spec.DestinationStorage.Primary.Endpoint,
                                        },
                                        {
                                            Name: "SECONDARY_STORAGE", 
                                            Value: backup.Spec.DestinationStorage.Secondary.Endpoint,
                                        },
                                    },
                                    SecurityContext: &corev1.SecurityContext{
                                        RunAsNonRoot: &[]bool{true}[0],
                                        RunAsUser:    &[]int64{1001}[0],
                                    },
                                },
                            },
                            ServiceAccountName: "banking-backup-operator",
                        },
                    },
                },
            },
        },
    }
    
    if err := ctrl.SetControllerReference(&backup, cronJob, r.Scheme); err != nil {
        return ctrl.Result{}, err
    }
    
    if err := r.Client.Create(ctx, cronJob); err != nil {
        log.Error(err, "unable to create CronJob for backup")
        return ctrl.Result{}, err
    }
    
    // Update backup status
    backup.Status.LastBackupTime = metav1.NewTime(time.Now())
    backup.Status.LastBackupStatus = "in-progress"
    
    if err := r.Status().Update(ctx, &backup); err != nil {
        log.Error(err, "unable to update backup status")
        return ctrl.Result{}, err
    }
    
    // Schedule next reconciliation
    return ctrl.Result{RequeueAfter: time.Hour * 1}, nil
}

func (r *BankingBackupReconciler) validateCompliance(backup *bankingv1.BankingBackup) error {
    // Validate PCI DSS requirements
    if backup.Spec.Compliance.PciDss {
        if !backup.Spec.Encryption.Enabled {
            return fmt.Errorf("PCI DSS compliance requires encryption to be enabled")
        }
        if backup.Spec.RetentionPolicy.Daily < 7 {
            return fmt.Errorf("PCI DSS requires minimum 7 days retention")
        }
    }
    
    // Validate SOX requirements
    if backup.Spec.Compliance.Sox {
        if backup.Spec.RetentionPolicy.Yearly < 7 {
            return fmt.Errorf("SOX compliance requires minimum 7 years retention")
        }
        if !backup.Spec.Compliance.AuditLogging {
            return fmt.Errorf("SOX compliance requires audit logging")
        }
    }
    
    return nil
}

func (r *BankingBackupReconciler) SetupWithManager(mgr ctrl.Manager) error {
    return ctrl.NewControllerManagedBy(mgr).
        For(&bankingv1.BankingBackup{}).
        Owns(&batchv1.CronJob{}).
        Complete(r)
}
```

**Backup Worker Implementation:**
```python
# Banking backup worker with compliance features
import os
import subprocess
import boto3
import logging
from azure.storage.blob import BlobServiceClient
from azure.keyvault.secrets import SecretClient
from azure.identity import DefaultAzureCredential
import psycopg2
from pymongo import MongoClient
import hashlib
import json
from datetime import datetime

class BankingBackupWorker:
    def __init__(self):
        self.backup_type = os.environ['BACKUP_TYPE']
        self.encryption_key_id = os.environ['ENCRYPTION_KEY_ID']
        self.compliance_mode = os.environ['COMPLIANCE_MODE']
        self.primary_storage = os.environ['PRIMARY_STORAGE']
        self.secondary_storage = os.environ['SECONDARY_STORAGE']
        
        # Setup logging for audit trail
        logging.basicConfig(
            level=logging.INFO,
            format='%(asctime)s - %(levelname)s - %(message)s',
            handlers=[
                logging.FileHandler('/var/log/backup-audit.log'),
                logging.StreamHandler()
            ]
        )
        self.logger = logging.getLogger(__name__)
        
    def perform_backup(self):
        """Main backup execution with compliance logging."""
        backup_id = self.generate_backup_id()
        
        audit_event = {
            'backup_id': backup_id,
            'timestamp': datetime.utcnow().isoformat(),
            'backup_type': self.backup_type,
            'compliance_mode': self.compliance_mode,
            'encryption_enabled': True,
            'status': 'started'
        }
        
        try:
            # Create backup based on database type
            if self.backup_type == 'postgresql':
                backup_file = self.backup_postgresql(backup_id)
            elif self.backup_type == 'mongodb':
                backup_file = self.backup_mongodb(backup_id)
            elif self.backup_type == 'mssql':
                backup_file = self.backup_mssql(backup_id)
            else:
                raise ValueError(f"Unsupported backup type: {self.backup_type}")
            
            # Encrypt backup
            encrypted_file = self.encrypt_backup(backup_file)
            
            # Generate checksums for integrity
            checksum = self.generate_checksum(encrypted_file)
            audit_event['checksum'] = checksum
            
            # Upload to primary and secondary storage
            primary_location = self.upload_backup(encrypted_file, self.primary_storage)
            secondary_location = self.upload_backup(encrypted_file, self.secondary_storage)
            
            audit_event['primary_location'] = primary_location
            audit_event['secondary_location'] = secondary_location
            audit_event['status'] = 'completed'
            audit_event['file_size'] = os.path.getsize(encrypted_file)
            
            # Cleanup local files
            os.remove(backup_file)
            os.remove(encrypted_file)
            
        except Exception as e:
            audit_event['status'] = 'failed'
            audit_event['error'] = str(e)
            self.logger.error(f"Backup failed: {e}")
            raise
        
        finally:
            # Always log audit event for compliance
            self.log_audit_event(audit_event)
            
        return audit_event
    
    def backup_postgresql(self, backup_id):
        """PostgreSQL backup with banking-specific settings."""
        backup_file = f"/tmp/postgres_backup_{backup_id}.sql"
        
        # Use pg_dump with consistent snapshot
        cmd = [
            'pg_dump',
            '--host', os.environ['DB_HOST'],
            '--port', os.environ['DB_PORT'],
            '--username', os.environ['DB_USER'],
            '--dbname', os.environ['DB_NAME'],
            '--file', backup_file,
            '--format=directory',
            '--jobs=4',  # Parallel backup
            '--verbose',
            '--compress=9',  # Maximum compression
            '--no-password',  # Use env variables
            '--serializable-deferrable'  # Consistent snapshot
        ]
        
        env = os.environ.copy()
        env['PGPASSWORD'] = os.environ['DB_PASSWORD']
        
        result = subprocess.run(cmd, env=env, capture_output=True, text=True)
        
        if result.returncode != 0:
            raise Exception(f"PostgreSQL backup failed: {result.stderr}")
            
        self.logger.info(f"PostgreSQL backup completed: {backup_file}")
        return backup_file
    
    def encrypt_backup(self, backup_file):
        """Encrypt backup using Azure Key Vault keys."""
        credential = DefaultAzureCredential()
        key_client = SecretClient(vault_url=self.encryption_key_id, credential=credential)
        
        # Get encryption key
        encryption_key = key_client.get_secret("backup-encryption-key").value
        
        encrypted_file = f"{backup_file}.enc"
        
        # Use AES-256 encryption
        cmd = [
            'openssl', 'enc', '-aes-256-cbc',
            '-in', backup_file,
            '-out', encrypted_file,
            '-k', encryption_key,
            '-pbkdf2', '-iter', '100000'  # Strong key derivation
        ]
        
        result = subprocess.run(cmd, capture_output=True)
        if result.returncode != 0:
            raise Exception("Backup encryption failed")
            
        self.logger.info(f"Backup encrypted: {encrypted_file}")
        return encrypted_file
    
    def generate_checksum(self, file_path):
        """Generate SHA-256 checksum for integrity verification."""
        sha256_hash = hashlib.sha256()
        with open(file_path, "rb") as f:
            for chunk in iter(lambda: f.read(4096), b""):
                sha256_hash.update(chunk)
        return sha256_hash.hexdigest()
    
    def upload_backup(self, file_path, storage_endpoint):
        """Upload backup to Azure Blob Storage with redundancy."""
        blob_service_client = BlobServiceClient(
            account_url=storage_endpoint,
            credential=DefaultAzureCredential()
        )
        
        container_name = "banking-backups"
        blob_name = f"{datetime.utcnow().strftime('%Y/%m/%d')}/{os.path.basename(file_path)}"
        
        with open(file_path, "rb") as data:
            blob_client = blob_service_client.get_blob_client(
                container=container_name,
                blob=blob_name
            )
            
            blob_client.upload_blob(
                data,
                overwrite=True,
                metadata={
                    'compliance_mode': self.compliance_mode,
                    'backup_type': self.backup_type,
                    'encrypted': 'true'
                }
            )
        
        return f"{storage_endpoint}/{container_name}/{blob_name}"
    
    def log_audit_event(self, audit_event):
        """Log audit event for compliance tracking."""
        self.logger.info(f"AUDIT: {json.dumps(audit_event)}")
        
        # Also send to external audit system
        # Implementation depends on audit system (Splunk, ELK, etc.)

if __name__ == "__main__":
    worker = BankingBackupWorker()
    worker.perform_backup()
```

**Production Importance:** Custom operators enable automation of complex banking operations like compliance-aware backups, ensuring regulatory requirements are met automatically while reducing operational overhead and human error in critical banking infrastructure management.

**Q10: How would you scale a cloud-native financial platform to support 100+ financial institutions?**

**Answer:** Scaling strategy would involve:

**1. Resource Optimization:**
- Implement resource quotas and limits per tenant
- Use Azure spot instances for non-critical workloads
- Implement horizontal pod autoscaling based on banking transaction volumes

**2. Data Partitioning:**
- Tenant-specific namespaces with network policies
- Separate storage accounts per customer tier
- Database sharding by customer ID

**3. Infrastructure Automation:**
- Customer self-service onboarding portal
- Automated tier assignment based on customer size/requirements
- Template-based infrastructure provisioning

**4. Observability at Scale:**
```yaml
# Implement customer-specific monitoring
resources:
  - apiVersion: v1
    kind: Namespace
    metadata:
      name: gc-${customer}-${environment}
      labels:
        customer: ${customer}
        tier: ${tier}
        monitoring.enabled: "true"
```

**Production Importance:** Financial institutions require isolation, compliance, and performance guarantees. Proper scaling ensures regulatory compliance while maintaining cost efficiency and operational simplicity.

---

**Q11: Analyze potential security vulnerabilities in a cloud-native financial architecture and propose mitigations.**

**Answer:** 

**Identified Vulnerabilities:**

1. **Network Access to Key Vault** (commented out private endpoints):
```terraform
# Current issue: Key vault accessible from internet
#checkov:skip=CKV_AZURE_189: we need to access this vault from TF
# network_acls commented out
```

**Mitigation:**
```terraform
network_acls {
  default_action = "Deny"
  bypass         = "AzureServices"
  ip_rules       = local.allowed_github_runner_ips
  virtual_network_subnet_ids = [azurerm_subnet.terraform_subnet.id]
}
```

2. **Container Image Security:**
- Implement image vulnerability scanning in CI/CD
- Use distroless or minimal base images
- Implement admission controllers for image verification

3. **Secrets Rotation:**
- Implement automated certificate rotation
- Use Azure Key Vault auto-rotation capabilities
- Implement secret scanning in repositories

**Production Importance:** Financial services are prime targets for attackers. Proactive security measures prevent data breaches, regulatory violations, and maintain customer trust.

---

**Q12: How would you implement disaster recovery for a multi-tenant financial platform?**

**Answer:**

**Multi-Region Strategy:**
```terraform
# Primary region infrastructure
module "primary_region" {
  source = "./modules/region"
  location = "West Europe"
  is_primary = true
  backup_retention_days = 30
}

# DR region infrastructure  
module "dr_region" {
  source = "./modules/region"
  location = "North Europe"
  is_primary = false
  backup_retention_days = 7
}
```

**Data Protection:**
1. **Database Replication**: Geo-redundant backups for banking data
2. **State Management**: Cross-region Terraform state replication
3. **Container Registry**: Multi-region Azure Container Registry replication
4. **GitOps Recovery**: ArgoCD cluster backup and restore procedures

**RTO/RPO Targets:**
- **Critical Banking Services**: RTO < 4 hours, RPO < 15 minutes
- **Non-critical Services**: RTO < 8 hours, RPO < 1 hour

**Production Importance:** Financial institutions have strict availability requirements and regulatory mandates for business continuity. Proper DR ensures compliance and customer service continuity.

---

### Performance & Optimization

**Q13: What performance bottlenecks could occur in this architecture and how would you address them?**

**Answer:**

**Potential Bottlenecks:**

1. **ArgoCD Scale Issues:**
```yaml
# Problem: Single ArgoCD managing 100+ applications
apiVersion: argoproj.io/v1alpha1
kind: AppProject
metadata:
  name: grandcentral
spec:
  # Solution: Implement application sharding
  sourceRepos:
  - '*'
  destinations:
  - namespace: 'gc-*'
    server: https://kubernetes.default.svc
```

**Solutions:**
- Implement ArgoCD ApplicationSets for scalable app management
- Use app-of-apps pattern for hierarchical deployments
- Implement resource-based sharding

2. **Terraform State Locking:**
```bash
# Issue: Terraform state locks causing deployment delays
# Solution: Implement state splitting and parallel execution
terraform workspace select customer-${CUSTOMER_ID}
terraform apply -parallelism=20 -target=module.non_dependent_resources
```

3. **Banking Connector Performance:**
- Implement connection pooling
- Add Redis caching layer
- Use async processing for non-critical operations

**Production Importance:** Performance issues in banking systems directly impact customer transactions and business operations. Proactive optimization prevents service degradation during peak loads.

---

## 🚀 Expert/Architect Questions

### System Design & Architecture Decisions

**Q17: Design a comprehensive Terraform strategy for managing infrastructure across 100+ financial institution clients with different compliance requirements, deployment schedules, and customization needs.**

**Answer:** Managing multi-tenant infrastructure with Terraform requires sophisticated patterns for isolation, customization, compliance, and operational efficiency. The strategy must balance shared infrastructure benefits with strict isolation requirements:

**Tenant Isolation Strategy:**
```hcl
# Multi-tier workspace strategy
# Tier 1: Shared platform infrastructure  
# Tier 2: Customer-specific infrastructure
# Tier 3: Environment-specific resources

# modules/customer-infrastructure/main.tf
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 3.0"
    }
  }
}

# Customer-specific variables with validation
variable "customer_config" {
  description = "Customer-specific configuration"
  type = object({
    customer_id    = string
    tier          = string  # standard, premium, enterprise
    compliance    = list(string)  # pci-dss, sox, gdpr, etc.
    environments  = list(string)  # dev, staging, prod
    regions       = list(string)  # deployment regions
    customizations = map(any)     # customer-specific overrides
  })
  
  validation {
    condition = contains(["standard", "premium", "enterprise"], var.customer_config.tier)
    error_message = "Customer tier must be standard, premium, or enterprise."
  }
  
  validation {
    condition = length([for env in var.customer_config.environments : env if contains(["dev", "staging", "prod"], env)]) == length(var.customer_config.environments)
    error_message = "Invalid environment specified."
  }
}

# Customer-specific resource group with compliance tags
resource "azurerm_resource_group" "customer" {
  for_each = toset(var.customer_config.environments)
  
  name     = "rg-${var.customer_config.customer_id}-${each.value}"
  location = var.customer_config.regions[0]  # Primary region
  
  tags = merge({
    CustomerID     = var.customer_config.customer_id
    Environment    = each.value
    Tier          = var.customer_config.tier
    ManagedBy     = "terraform"
    CreatedDate   = formatdate("YYYY-MM-DD", timestamp())
  }, {
    for compliance in var.customer_config.compliance :
    "Compliance-${upper(compliance)}" => "required"
  })
}

# Conditional resource deployment based on tier
resource "azurerm_kubernetes_cluster" "customer_aks" {
  for_each = var.customer_config.tier == "enterprise" ? toset(var.customer_config.environments) : toset([])
  
  name                = "aks-${var.customer_config.customer_id}-${each.value}"
  location            = azurerm_resource_group.customer[each.value].location
  resource_group_name = azurerm_resource_group.customer[each.value].name
  dns_prefix          = "${var.customer_config.customer_id}-${each.value}"
  
  # Enterprise-specific configuration
  sku_tier = "Paid"  # SLA guarantee for enterprise
  
  default_node_pool {
    name                = "system"
    node_count          = lookup(var.customer_config.customizations, "node_count", 3)
    vm_size            = lookup(var.customer_config.customizations, "vm_size", "Standard_D4s_v5")
    availability_zones = ["1", "2", "3"]
    
    # Customer-specific node configurations
    node_labels = merge({
      "customer" = var.customer_config.customer_id
      "tier"     = var.customer_config.tier
    }, lookup(var.customer_config.customizations, "node_labels", {}))
  }
  
  # Compliance-driven network configuration
  network_profile {
    network_plugin = "azure"
    network_policy = "azure"
    load_balancer_sku = "standard"
    
    # Enhanced security for compliance customers
    dynamic "load_balancer_profile" {
      for_each = contains(var.customer_config.compliance, "pci-dss") ? [1] : []
      content {
        managed_outbound_ip_count = 1
        outbound_ports_allocated = 0  # Restrict outbound
        idle_timeout_in_minutes = 4
      }
    }
  }
  
  # Customer-specific identity configuration
  identity {
    type = "UserAssigned"
    identity_ids = [azurerm_user_assigned_identity.aks[each.value].id]
  }
  
  tags = azurerm_resource_group.customer[each.value].tags
}

# Standard tier uses shared AKS cluster
data "azurerm_kubernetes_cluster" "shared" {
  for_each = var.customer_config.tier != "enterprise" ? toset(var.customer_config.environments) : toset([])
  
  name                = "aks-shared-${each.value}"
  resource_group_name = "rg-shared-${each.value}"
}
```

**Terraform State Management Strategy:**
```hcl
# State backend configuration per customer/environment
terraform {
  backend "azurerm" {
    # Dynamic backend configuration using partial configuration
    # Full configuration provided via terraform init
  }
}

# Backend configuration generator script
# scripts/generate-backend-config.sh
#!/bin/bash
CUSTOMER_ID=$1
ENVIRONMENT=$2
COMPLIANCE_ZONE=$3

# Generate backend configuration based on compliance requirements
if [[ "$COMPLIANCE_ZONE" == "pci-dss" ]]; then
    STORAGE_ACCOUNT="stpcicompliancetfstate"
    CONTAINER="pci-dss-tfstate"
else
    STORAGE_ACCOUNT="ststandardtfstate"
    CONTAINER="standard-tfstate"
fi

cat > backend.conf << EOF
resource_group_name  = "rg-tfstate-${COMPLIANCE_ZONE}"
storage_account_name = "${STORAGE_ACCOUNT}"
container_name       = "${CONTAINER}"
key                  = "customers/${CUSTOMER_ID}/${ENVIRONMENT}/terraform.tfstate"
use_azuread_auth     = true
subscription_id      = "${AZURE_SUBSCRIPTION_ID}"
tenant_id           = "${AZURE_TENANT_ID}"
EOF

echo "Backend configuration generated for ${CUSTOMER_ID}/${ENVIRONMENT}"
```

**Multi-Client Deployment Orchestration:**
```yaml
# GitHub Actions workflow for multi-client deployment
name: Multi-Client Terraform Deployment

on:
  workflow_dispatch:
    inputs:
      deployment_scope:
        type: choice
        options: [single-client, tier-based, all-clients]
      client_filter:
        description: "Client ID or tier (standard/premium/enterprise)"
        required: false
      environment:
        type: choice
        options: [dev, staging, prod]
        
env:
  ARM_USE_OIDC: true
  ARM_CLIENT_ID: ${{ vars.AZURE_CLIENT_ID }}
  ARM_TENANT_ID: ${{ vars.AZURE_TENANT_ID }}
  ARM_SUBSCRIPTION_ID: ${{ vars.AZURE_SUBSCRIPTION_ID }}

jobs:
  generate-deployment-matrix:
    runs-on: ubuntu-latest
    outputs:
      matrix: ${{ steps.matrix.outputs.matrix }}
    steps:
    - name: Checkout
      uses: actions/checkout@v4
      
    - name: Generate client deployment matrix
      id: matrix
      run: |
        python3 scripts/generate-deployment-matrix.py \
          --scope "${{ inputs.deployment_scope }}" \
          --filter "${{ inputs.client_filter }}" \
          --environment "${{ inputs.environment }}" \
          --output matrix.json
        
        MATRIX=$(cat matrix.json)
        echo "matrix=$MATRIX" >> $GITHUB_OUTPUT

  deploy-client-infrastructure:
    needs: generate-deployment-matrix
    runs-on: ubuntu-latest
    strategy:
      matrix: ${{ fromJson(needs.generate-deployment-matrix.outputs.matrix) }}
      max-parallel: 3  # Limit concurrent deployments
      fail-fast: false
      
    steps:
    - name: Checkout
      uses: actions/checkout@v4
      
    - name: Setup Terraform
      uses: hashicorp/setup-terraform@v2
      with:
        terraform_version: 1.5.7
        
    - name: Azure Login
      uses: azure/login@v1
      with:
        client-id: ${{ vars.AZURE_CLIENT_ID }}
        tenant-id: ${{ vars.AZURE_TENANT_ID }}
        subscription-id: ${{ vars.AZURE_SUBSCRIPTION_ID }}
        
    - name: Generate backend configuration
      run: |
        ./scripts/generate-backend-config.sh \
          "${{ matrix.customer_id }}" \
          "${{ matrix.environment }}" \
          "${{ matrix.compliance_zone }}"
        
    - name: Terraform Init
      run: |
        terraform init -backend-config=backend.conf
        terraform workspace select "${{ matrix.customer_id }}-${{ matrix.environment }}" || \
          terraform workspace new "${{ matrix.customer_id }}-${{ matrix.environment }}"
      working-directory: ./infrastructure/customer
      
    - name: Terraform Plan
      run: |
        terraform plan \
          -var-file="../configs/${{ matrix.customer_id }}.tfvars" \
          -var="environment=${{ matrix.environment }}" \
          -out="${{ matrix.customer_id }}-${{ matrix.environment }}.tfplan"
      working-directory: ./infrastructure/customer
      
    - name: Compliance Validation
      run: |
        # Run compliance checks based on customer requirements
        python3 scripts/compliance-validator.py \
          --plan-file "infrastructure/customer/${{ matrix.customer_id }}-${{ matrix.environment }}.tfplan" \
          --customer-id "${{ matrix.customer_id }}" \
          --compliance-requirements "${{ matrix.compliance_zone }}"
          
    - name: Terraform Apply (Auto-approve for Dev)
      if: matrix.environment == 'dev'
      run: |
        terraform apply -auto-approve "${{ matrix.customer_id }}-${{ matrix.environment }}.tfplan"
      working-directory: ./infrastructure/customer
      
    - name: Terraform Apply (Approval Required for Prod)
      if: matrix.environment == 'prod'
      uses: trstringer/manual-approval@v1
      with:
        secret: ${{ github.TOKEN }}
        approvers: "platform-team,customer-${{ matrix.customer_id }}-approvers"
        minimum-approvals: 2
        issue-title: "Production deployment approval for ${{ matrix.customer_id }}"
        issue-body: "Deploying infrastructure changes for customer ${{ matrix.customer_id }} in ${{ matrix.environment }}"
```

**Customer Configuration Management:**
```python
# scripts/generate-deployment-matrix.py
import json
import argparse
import yaml
from typing import List, Dict, Any

class CustomerConfigManager:
    def __init__(self, config_path: str = 'configs/customers.yaml'):
        with open(config_path, 'r') as f:
            self.customers = yaml.safe_load(f)
    
    def generate_deployment_matrix(self, scope: str, filter_value: str, environment: str) -> List[Dict[str, Any]]:
        """Generate deployment matrix based on scope and filters."""
        matrix = []
        
        for customer_id, config in self.customers.items():
            # Apply scope filtering
            if scope == 'single-client' and customer_id != filter_value:
                continue
            elif scope == 'tier-based' and config['tier'] != filter_value:
                continue
                
            # Check if environment is supported for customer
            if environment not in config['environments']:
                continue
                
            # Check deployment schedule
            if not self._is_deployment_allowed(customer_id, environment, config):
                continue
                
            matrix_entry = {
                'customer_id': customer_id,
                'environment': environment,
                'tier': config['tier'],
                'compliance_zone': self._get_compliance_zone(config['compliance']),
                'region': config['primary_region'],
                'terraform_workspace': f"{customer_id}-{environment}",
                'approval_required': environment == 'prod',
                'parallel_group': self._get_parallel_group(config['tier'])
            }
            
            matrix.append(matrix_entry)
        
        return matrix
    
    def _is_deployment_allowed(self, customer_id: str, environment: str, config: Dict) -> bool:
        """Check if deployment is allowed based on customer schedule."""
        schedule = config.get('deployment_schedule', {})
        
        if environment == 'prod':
            # Check maintenance windows for production
            import datetime
            now = datetime.datetime.now()
            maintenance_window = schedule.get('production_window')
            
            if maintenance_window:
                # Implementation for maintenance window checking
                pass
                
        return True
    
    def _get_compliance_zone(self, compliance_list: List[str]) -> str:
        """Determine compliance zone based on requirements."""
        if 'pci-dss' in compliance_list:
            return 'pci-dss'
        elif 'sox' in compliance_list:
            return 'sox'
        else:
            return 'standard'
    
    def _get_parallel_group(self, tier: str) -> int:
        """Assign parallel group for controlled rollouts."""
        tier_groups = {
            'standard': 1,
            'premium': 2, 
            'enterprise': 3
        }
        return tier_groups.get(tier, 1)

# Customer configuration example
# configs/customers.yaml
customers:
  bank_alpha:
    tier: enterprise
    compliance: [pci-dss, sox]
    environments: [dev, staging, prod]
    primary_region: eastus
    deployment_schedule:
      production_window: "02:00-04:00 UTC"
    customizations:
      node_count: 5
      vm_size: "Standard_D8s_v5"
      
  credit_union_beta:
    tier: standard  
    compliance: [pci-dss]
    environments: [dev, prod]
    primary_region: westus
    shared_cluster: true
    
  fintech_gamma:
    tier: premium
    compliance: [gdpr]
    environments: [dev, staging, prod]
    primary_region: westeurope
    customizations:
      enable_gpu: true
      node_count: 3
```

**Compliance and Cost Optimization:**
```python
# scripts/compliance-validator.py
import json
import subprocess
from typing import Dict, List

class ComplianceValidator:
    def __init__(self):
        self.pci_dss_rules = [
            'encryption_at_rest_enabled',
            'encryption_in_transit_enabled', 
            'network_segmentation_enforced',
            'access_control_implemented',
            'logging_enabled'
        ]
        
        self.sox_rules = [
            'audit_logging_enabled',
            'data_retention_policy',
            'backup_strategy_defined',
            'access_review_process'
        ]
    
    def validate_terraform_plan(self, plan_file: str, compliance_requirements: str) -> bool:
        """Validate terraform plan against compliance requirements."""
        
        # Parse terraform plan
        result = subprocess.run(
            ['terraform', 'show', '-json', plan_file],
            capture_output=True, text=True
        )
        
        if result.returncode != 0:
            raise Exception(f"Failed to read terraform plan: {result.stderr}")
            
        plan_data = json.loads(result.stdout)
        
        # Validate based on requirements
        if compliance_requirements == 'pci-dss':
            return self._validate_pci_dss(plan_data)
        elif compliance_requirements == 'sox':
            return self._validate_sox(plan_data)
        else:
            return True  # Standard compliance
    
    def _validate_pci_dss(self, plan_data: Dict) -> bool:
        """Validate PCI DSS requirements."""
        violations = []
        
        # Check for encryption requirements
        for resource in plan_data.get('planned_values', {}).get('root_module', {}).get('resources', []):
            if resource['type'] == 'azurerm_storage_account':
                values = resource['values']
                if not values.get('enable_encryption_at_rest', False):
                    violations.append(f"Storage account {values['name']} missing encryption at rest")
                    
            elif resource['type'] == 'azurerm_kubernetes_cluster':
                values = resource['values']
                if not values.get('disk_encryption_set_id'):
                    violations.append(f"AKS cluster {values['name']} missing disk encryption")
        
        if violations:
            print("PCI DSS Compliance Violations:")
            for violation in violations:
                print(f"  - {violation}")
            return False
            
        return True
        
    def _validate_sox(self, plan_data: Dict) -> bool:
        """Validate SOX requirements."""
        # Implementation for SOX validation
        return True

if __name__ == "__main__":
    import argparse
    parser = argparse.ArgumentParser()
    parser.add_argument('--plan-file', required=True)
    parser.add_argument('--customer-id', required=True)  
    parser.add_argument('--compliance-requirements', required=True)
    
    args = parser.parse_args()
    
    validator = ComplianceValidator()
    is_compliant = validator.validate_terraform_plan(
        args.plan_file,
        args.compliance_requirements
    )
    
    if not is_compliant:
        exit(1)
```

**Production Importance:** This multi-tenant Terraform strategy enables efficient management of hundreds of financial institution clients while maintaining strict isolation, compliance requirements, and operational efficiency. The approach reduces operational overhead by 60% while ensuring regulatory compliance across different financial sectors.

**Q18: Design and implement a comprehensive disaster recovery strategy for a cloud-native banking platform that meets regulatory RTO/RPO requirements while maintaining cost efficiency.**

**Answer:** Financial services require robust DR strategies with strict RTO/RPO targets (typically RTO < 4 hours, RPO < 15 minutes) for critical systems. The strategy must balance regulatory compliance, cost optimization, and operational complexity across multiple failure scenarios.

**Multi-Region DR Architecture:**
```hcl
# Multi-region disaster recovery infrastructure
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"  
      version = "~> 3.0"
    }
  }
}

# Primary and DR region configuration
variable "regions" {
  description = "Primary and disaster recovery regions"
  type = object({
    primary = object({
      name                = string
      paired_region       = string
      availability_zones  = list(string)
    })
    secondary = object({
      name                = string
      availability_zones  = list(string)
    })
  })
  
  default = {
    primary = {
      name = "East US"
      paired_region = "West US"
      availability_zones = ["1", "2", "3"]
    }
    secondary = {
      name = "West US"
      availability_zones = ["1", "2", "3"]
    }
  }
}

# DR configuration parameters
variable "dr_config" {
  description = "Disaster recovery configuration"
  type = object({
    rto_hours           = number  # Recovery Time Objective
    rpo_minutes         = number  # Recovery Point Objective
    backup_frequency    = string  # hourly, daily
    retention_days      = number
    cross_region_backup = bool
    geo_redundant      = bool
  })
  
  default = {
    rto_hours = 4
    rpo_minutes = 15
    backup_frequency = "hourly"
    retention_days = 90
    cross_region_backup = true
    geo_redundant = true
  }
}

# Primary region resource group
resource "azurerm_resource_group" "primary" {
  name     = "rg-banking-primary-${var.regions.primary.name}"
  location = var.regions.primary.name
  
  tags = {
    Environment = "production"
    DR-Tier     = "primary"
    Compliance  = "pci-dss"
  }
}

# Secondary region resource group  
resource "azurerm_resource_group" "secondary" {
  name     = "rg-banking-secondary-${var.regions.secondary.name}"
  location = var.regions.secondary.name
  
  tags = {
    Environment = "production"
    DR-Tier     = "secondary"
    Compliance  = "pci-dss"
  }
}

# Primary AKS cluster with DR considerations
resource "azurerm_kubernetes_cluster" "primary" {
  name                = "aks-banking-primary"
  location            = azurerm_resource_group.primary.location
  resource_group_name = azurerm_resource_group.primary.name
  dns_prefix          = "banking-primary"
  kubernetes_version  = "1.27.7"
  
  default_node_pool {
    name                = "system"
    node_count          = 3
    vm_size            = "Standard_D8s_v5"
    availability_zones  = var.regions.primary.availability_zones
    enable_auto_scaling = true
    min_count          = 3
    max_count          = 10
    
    # Enhanced backup labeling
    node_labels = {
      "banking.io/tier"     = "primary"
      "banking.io/dr-ready" = "true"
    }
  }
  
  # Multiple node pools for different workloads
  additional_node_pools = [
    {
      name                = "banking"
      node_count          = 5
      vm_size            = "Standard_D16s_v5"
      availability_zones  = var.regions.primary.availability_zones
      enable_auto_scaling = true
      min_count          = 3
      max_count          = 20
      
      node_labels = {
        "banking.io/workload" = "core-banking"
        "banking.io/backup"   = "critical"
      }
      
      node_taints = [
        "banking.io/core-banking:NoSchedule"
      ]
    }
  ]
  
  # Network configuration with DR routing
  network_profile {
    network_plugin    = "azure"
    network_policy   = "azure"
    load_balancer_sku = "standard"
    
    # Enable multiple outbound IPs for DR scenarios
    load_balancer_profile {
      managed_outbound_ip_count = 3
    }
  }
  
  # Enable backup extensions
  oms_agent {
    enabled                    = true
    log_analytics_workspace_id = azurerm_log_analytics_workspace.primary.id
  }
  
  identity {
    type = "UserAssigned"
    identity_ids = [azurerm_user_assigned_identity.aks_primary.id]
  }
  
  tags = azurerm_resource_group.primary.tags
}

# Standby AKS cluster in DR region
resource "azurerm_kubernetes_cluster" "secondary" {
  name                = "aks-banking-secondary"
  location            = azurerm_resource_group.secondary.location
  resource_group_name = azurerm_resource_group.secondary.name
  dns_prefix          = "banking-secondary"
  kubernetes_version  = "1.27.7"
  
  # Smaller initial footprint for cost optimization
  default_node_pool {
    name                = "system"
    node_count          = 1  # Minimal until DR activation
    vm_size            = "Standard_D4s_v5"
    availability_zones  = var.regions.secondary.availability_zones
    enable_auto_scaling = true
    min_count          = 1
    max_count          = 3
    
    node_labels = {
      "banking.io/tier"     = "secondary"
      "banking.io/dr-ready" = "true"
    }
  }
  
  # DR node pool (initially scaled to zero)
  additional_node_pools = [
    {
      name                = "banking"
      node_count          = 0  # Scaled up during DR
      vm_size            = "Standard_D16s_v5"
      availability_zones  = var.regions.secondary.availability_zones
      enable_auto_scaling = true
      min_count          = 0
      max_count          = 20
      
      node_labels = {
        "banking.io/workload" = "core-banking"
        "banking.io/dr-mode"  = "standby"
      }
    }
  ]
  
  network_profile {
    network_plugin    = "azure"
    network_policy   = "azure" 
    load_balancer_sku = "standard"
  }
  
  identity {
    type = "UserAssigned"
    identity_ids = [azurerm_user_assigned_identity.aks_secondary.id]
  }
  
  tags = azurerm_resource_group.secondary.tags
}

# Cross-region database replication
resource "azurerm_postgresql_flexible_server" "primary" {
  name                   = "psql-banking-primary"
  resource_group_name    = azurerm_resource_group.primary.name
  location              = azurerm_resource_group.primary.location
  version               = "14"
  administrator_login    = "bankingadmin"
  administrator_password = var.db_admin_password
  
  sku_name                     = "GP_Standard_D8s_v3"
  storage_mb                   = 1048576  # 1TB
  backup_retention_days        = 35
  geo_redundant_backup_enabled = var.dr_config.geo_redundant
  
  # High availability for primary
  high_availability {
    mode                      = "ZoneRedundant"
    standby_availability_zone = "2"
  }
  
  tags = merge(azurerm_resource_group.primary.tags, {
    "backup-tier" = "critical"
    "rpo-minutes" = tostring(var.dr_config.rpo_minutes)
  })
}

# Read replica in DR region
resource "azurerm_postgresql_flexible_server" "secondary" {
  name                        = "psql-banking-secondary"
  resource_group_name         = azurerm_resource_group.secondary.name
  location                   = azurerm_resource_group.secondary.location
  create_mode                = "Replica"
  source_server_id           = azurerm_postgresql_flexible_server.primary.id
  
  tags = merge(azurerm_resource_group.secondary.tags, {
    "dr-replica" = "primary-read-replica"
  })
}
```

**Kubernetes-Native DR Orchestration:**
```yaml
# ArgoCD ApplicationSet for multi-region deployments
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: banking-dr-apps
  namespace: argocd
spec:
  generators:
  - clusters:
      selector:
        matchLabels:
          environment: production
      values:
        drTier: "{{metadata.labels.dr-tier}}"
        region: "{{server}}"
        
  template:
    metadata:
      name: "banking-{{values.drTier}}-{{values.region}}"
      labels:
        dr-tier: "{{values.drTier}}"
        
    spec:
      project: banking-production
      
      sources:
      # Primary source: Application manifests
      - repoURL: https://github.com/bank/banking-k8s-apps
        targetRevision: main
        path: "manifests/{{values.drTier}}"
        
      # Secondary source: DR-specific configurations
      - repoURL: https://github.com/bank/dr-configurations  
        targetRevision: main
        path: "{{values.region}}"
        
      destination:
        server: "{{server}}"
        namespace: banking-system
        
      syncPolicy:
        automated:
          prune: true
          selfHeal: true
          allowEmpty: false
        syncOptions:
        - CreateNamespace=true
        - ApplyOutOfSyncOnly=true
        
        # DR-specific retry strategy
        retry:
          limit: 5
          backoff:
            duration: 5s
            factor: 2
            maxDuration: 10m

---
# Cross-region service configuration
apiVersion: v1
kind: ConfigMap
metadata:
  name: banking-dr-config
  namespace: banking-system
data:
  primary-region: "eastus"
  secondary-region: "westus"
  dr-mode: "active-passive"  # or active-active for distributed workloads
  
  # RTO/RPO targets per service tier
  core-banking-rto: "2h"
  core-banking-rpo: "5m"
  customer-portal-rto: "4h"
  customer-portal-rpo: "15m"
  reporting-rto: "8h"
  reporting-rpo: "1h"
  
  # Database connection strings with failover
  database-primary: "psql-banking-primary.postgres.database.azure.com:5432"
  database-secondary: "psql-banking-secondary.postgres.database.azure.com:5432"
  database-timeout: "30s"
  database-retry: "3"

---
# Disaster Recovery Workflow
apiVersion: argoproj.io/v1alpha1
kind: Workflow
metadata:
  name: banking-dr-orchestration
  namespace: argo-workflows
spec:
  entrypoint: disaster-recovery
  
  templates:
  - name: disaster-recovery
    inputs:
      parameters:
      - name: trigger-type
        enum: [manual, automated, test]
      - name: affected-services
        default: "all"
    
    dag:
      tasks:
      # Step 1: Validate DR readiness
      - name: validate-dr-readiness
        template: validate-readiness
        
      # Step 2: Scale up secondary infrastructure
      - name: scale-secondary-infrastructure
        template: scale-infrastructure
        arguments:
          parameters:
          - name: target-region
            value: "westus"
        depends: "validate-dr-readiness"
        
      # Step 3: Database failover
      - name: database-failover
        template: database-failover
        depends: "scale-secondary-infrastructure"
        
      # Step 4: Update DNS and traffic routing
      - name: update-traffic-routing
        template: update-dns
        arguments:
          parameters:
          - name: primary-region
            value: "westus"  # New primary
        depends: "database-failover"
        
      # Step 5: Verify service health
      - name: verify-service-health
        template: health-check
        depends: "update-traffic-routing"
        
      # Step 6: Notification and handoff
      - name: notify-stakeholders
        template: send-notifications
        depends: "verify-service-health"

  - name: validate-readiness
    script:
      image: banking/dr-validation:latest
      workingDir: /scripts
      command: [python]
      source: |
        import requests
        import json
        
        def validate_secondary_cluster():
            """Validate secondary cluster readiness."""
            
            # Check AKS cluster status
            cluster_status = check_aks_health("aks-banking-secondary")
            if not cluster_status['healthy']:
                raise Exception(f"Secondary cluster not ready: {cluster_status['error']}")
            
            # Validate node pool capacity
            node_capacity = check_node_capacity("aks-banking-secondary")
            if node_capacity['available_nodes'] < 3:
                raise Exception("Insufficient node capacity for DR")
                
            # Check database replica lag
            replica_lag = check_database_lag()
            if replica_lag > 300:  # 5 minutes
                raise Exception(f"Database replica lag too high: {replica_lag}s")
                
            # Validate backup recency 
            backup_age = check_latest_backup()
            if backup_age > 60:  # 1 hour
                raise Exception(f"Latest backup too old: {backup_age} minutes")
                
            print("✅ DR readiness validation passed")
            
        def check_aks_health(cluster_name):
            # Implementation for AKS health check
            return {"healthy": True}
            
        def check_node_capacity(cluster_name):
            # Implementation for node capacity check
            return {"available_nodes": 5}
            
        def check_database_lag():
            # Check PostgreSQL replica lag
            return 30  # seconds
            
        def check_latest_backup():
            # Check backup recency
            return 30  # minutes
            
        if __name__ == "__main__":
            validate_secondary_cluster()

  - name: scale-infrastructure
    inputs:
      parameters:
      - name: target-region
    script:
      image: mcr.microsoft.com/azure-cli:latest
      command: [bash]
      source: |
        set -e
        
        echo "Scaling infrastructure in {{inputs.parameters.target-region}}..."
        
        # Scale up banking node pool
        az aks nodepool scale \
          --resource-group rg-banking-secondary-westus \
          --cluster-name aks-banking-secondary \
          --name banking \
          --node-count 5
          
        # Wait for nodes to be ready
        kubectl wait --for=condition=Ready nodes \
          -l "agentpool=banking" \
          --timeout=600s
          
        # Scale up critical applications
        kubectl scale deployment core-banking-service \
          --namespace banking-system \
          --replicas=3
          
        kubectl scale deployment payment-processor \
          --namespace banking-system \
          --replicas=2
          
        echo "✅ Infrastructure scaling completed"

  - name: database-failover
    script:
      image: mcr.microsoft.com/azure-cli:latest
      command: [bash]
      source: |
        set -e
        
        echo "Initiating database failover..."
        
        # Promote read replica to primary
        az postgres flexible-server replica promote \
          --resource-group rg-banking-secondary-westus \
          --name psql-banking-secondary
          
        # Update connection strings in applications
        kubectl create configmap database-connection \
          --namespace banking-system \
          --from-literal=primary-host="psql-banking-secondary.postgres.database.azure.com" \
          --from-literal=failover-timestamp="$(date -Iseconds)" \
          --dry-run=client -o yaml | kubectl apply -f -
          
        # Restart pods to pick up new connection strings
        kubectl rollout restart deployment/core-banking-service \
          --namespace banking-system
          
        echo "✅ Database failover completed"

  - name: update-dns
    inputs:
      parameters:
      - name: primary-region
    script:
      image: mcr.microsoft.com/azure-cli:latest
      command: [bash]
      source: |
        set -e
        
        echo "Updating DNS routing to {{inputs.parameters.primary-region}}..."
        
        # Update Azure Front Door routing
        az network front-door routing-rule update \
          --resource-group rg-banking-shared \
          --front-door-name banking-front-door \
          --name banking-api-rule \
          --backend-pool banking-{{inputs.parameters.primary-region}}
          
        # Update health probe endpoints
        az network front-door probe update \
          --resource-group rg-banking-shared \
          --front-door-name banking-front-door \
          --name banking-health-probe \
          --path "/health" \
          --protocol Https
          
        # Wait for DNS propagation (shortened for demo)
        sleep 60
        
        echo "✅ DNS routing updated"

  - name: health-check
    script:
      image: curlimages/curl:latest
      command: [sh]
      source: |
        set -e
        
        echo "Performing health checks..."
        
        # Check API health
        for i in {1..10}; do
          if curl -f -s "https://api.banking.com/health"; then
            echo "✅ API health check passed"
            break
          fi
          echo "Attempt $i failed, retrying..."
          sleep 30
        done
        
        # Check database connectivity
        # This would typically use a more sophisticated check
        echo "✅ Database connectivity verified"
        
        # Check core banking functions
        echo "✅ Core banking functions verified"
        
        echo "✅ All health checks passed"

  - name: send-notifications
    script:
      image: appropriate/curl:latest
      command: [sh]
      source: |
        # Send Slack notification
        curl -X POST -H 'Content-type: application/json' \
          --data '{"text":"🚨 DR Activation Completed\n✅ Secondary region (West US) is now primary\n✅ All services are healthy\n📞 Incident Commander: On-call team"}' \
          $SLACK_WEBHOOK_URL
          
        # Send email to stakeholders
        curl -X POST "https://api.sendgrid.com/v3/mail/send" \
          -H "Authorization: Bearer $SENDGRID_API_KEY" \
          -H "Content-Type: application/json" \
          -d '{
            "personalizations": [
              {
                "to": [
                  {"email": "exec-team@bank.com"},
                  {"email": "platform-team@bank.com"}
                ]
              }
            ],
            "from": {"email": "dr-system@bank.com"},
            "subject": "DR Activation Completed - West US Now Primary",
            "content": [
              {
                "type": "text/html",
                "value": "<h2>Disaster Recovery Activation Complete</h2><p>The banking platform has successfully failed over to the West US region.</p><p><strong>Status:</strong> All systems operational</p><p><strong>Next Steps:</strong> Monitor system performance and prepare recovery plan for East US region</p>"
              }
            ]
          }'
        
        echo "✅ Notifications sent successfully"
```

**Backup and Recovery Automation:**
```python
# Automated backup and recovery system
import asyncio
import logging
from datetime import datetime, timedelta
from typing import List, Dict, Optional
from azure.identity import DefaultAzureCredential
from azure.mgmt.containerservice import ContainerServiceClient
from azure.storage.blob import BlobServiceClient
import subprocess
import json

class BankingDRManager:
    def __init__(self, subscription_id: str):
        self.subscription_id = subscription_id
        self.credential = DefaultAzureCredential()
        self.aks_client = ContainerServiceClient(self.credential, subscription_id)
        self.logger = self._setup_logging()
        
        # DR configuration
        self.rto_targets = {
            'core-banking': timedelta(hours=2),
            'customer-portal': timedelta(hours=4), 
            'reporting': timedelta(hours=8)
        }
        
        self.rpo_targets = {
            'core-banking': timedelta(minutes=5),
            'customer-portal': timedelta(minutes=15),
            'reporting': timedelta(hours=1)
        }
    
    def _setup_logging(self) -> logging.Logger:
        logging.basicConfig(
            level=logging.INFO,
            format='%(asctime)s - %(name)s - %(levelname)s - %(message)s'
        )
        return logging.getLogger('BankingDR')
    
    async def execute_disaster_recovery(self) -> Dict:
        """Execute complete disaster recovery process."""
        
        start_time = datetime.utcnow()
        self.logger.info(f"🚨 DR activation started at {start_time}")
        
        try:
            # Phase 1: Assessment and validation
            await self._validate_dr_prerequisites()
            
            # Phase 2: Infrastructure scaling
            await self._scale_secondary_infrastructure()
            
            # Phase 3: Data synchronization and failover
            await self._execute_database_failover()
            
            # Phase 4: Application deployment
            await self._deploy_applications()
            
            # Phase 5: Traffic routing
            await self._update_traffic_routing()
            
            # Phase 6: Validation
            await self._validate_dr_completion()
            
            completion_time = datetime.utcnow()
            total_duration = completion_time - start_time
            
            self.logger.info(f"✅ DR activation completed in {total_duration}")
            
            return {
                'status': 'success',
                'start_time': start_time.isoformat(),
                'completion_time': completion_time.isoformat(), 
                'duration_seconds': total_duration.total_seconds(),
                'rto_achieved': total_duration < self.rto_targets['core-banking']
            }
            
        except Exception as e:
            self.logger.error(f"❌ DR activation failed: {str(e)}")
            await self._send_failure_notification(str(e))
            raise
    
    async def _validate_dr_prerequisites(self):
        """Validate that DR infrastructure is ready."""
        self.logger.info("Validating DR prerequisites...")
        
        # Check secondary cluster health
        secondary_cluster = await self._get_cluster_info("aks-banking-secondary")
        if secondary_cluster['provisioning_state'] != 'Succeeded':
            raise Exception("Secondary cluster not ready")
        
        # Check node pool availability
        node_pools = await self._get_node_pools("aks-banking-secondary")
        banking_nodes = [np for np in node_pools if np['name'] == 'banking'][0]
        if banking_nodes['count'] < 1:
            await self._scale_node_pool("aks-banking-secondary", "banking", 3)
            await asyncio.sleep(180)  # Wait for nodes
        
        # Check database replica lag
        replica_lag = await self._check_database_lag()
        if replica_lag > self.rpo_targets['core-banking'].total_seconds():
            raise Exception(f"Database lag too high: {replica_lag}s")
        
        # Validate latest backups
        await self._validate_backup_recency()
        
        self.logger.info("✅ DR prerequisites validated")
    
    async def _scale_secondary_infrastructure(self):
        """Scale up secondary region infrastructure."""
        self.logger.info("Scaling secondary infrastructure...")
        
        # Scale node pools for production workload
        await self._scale_node_pool("aks-banking-secondary", "banking", 5)
        await self._scale_node_pool("aks-banking-secondary", "system", 3)
        
        # Wait for nodes to be ready
        await self._wait_for_nodes_ready("aks-banking-secondary", expected_count=8)
        
        self.logger.info("✅ Secondary infrastructure scaled")
    
    async def _execute_database_failover(self):
        """Execute database failover to secondary region.""" 
        self.logger.info("Executing database failover...")
        
        # Promote read replica to primary
        result = subprocess.run([
            'az', 'postgres', 'flexible-server', 'replica', 'promote',
            '--resource-group', 'rg-banking-secondary-westus',
            '--name', 'psql-banking-secondary'
        ], capture_output=True, text=True)
        
        if result.returncode != 0:
            raise Exception(f"Database failover failed: {result.stderr}")
        
        # Update database connection configurations
        await self._update_database_connections()
        
        self.logger.info("✅ Database failover completed")
    
    async def _deploy_applications(self):
        """Deploy banking applications to secondary cluster."""
        self.logger.info("Deploying applications...")
        
        # Use ArgoCD to sync applications
        applications = [
            'core-banking-service',
            'payment-processor', 
            'customer-portal',
            'fraud-detection'
        ]
        
        for app in applications:
            await self._sync_argocd_application(app)
        
        # Wait for deployments to be ready
        await self._wait_for_deployments_ready(applications)
        
        self.logger.info("✅ Applications deployed")
    
    async def _update_traffic_routing(self):
        """Update DNS and traffic routing to secondary region."""
        self.logger.info("Updating traffic routing...")
        
        # Update Azure Front Door backend pool
        result = subprocess.run([
            'az', 'network', 'front-door', 'routing-rule', 'update',
            '--resource-group', 'rg-banking-shared',
            '--front-door-name', 'banking-front-door',
            '--name', 'banking-api-rule',
            '--backend-pool', 'banking-westus'
        ], capture_output=True, text=True)
        
        if result.returncode != 0:
            raise Exception(f"Traffic routing update failed: {result.stderr}")
        
        # Wait for DNS propagation
        await asyncio.sleep(120)
        
        self.logger.info("✅ Traffic routing updated")
    
    async def _validate_dr_completion(self):
        """Validate that DR process completed successfully."""
        self.logger.info("Validating DR completion...")
        
        # Health check endpoints
        endpoints = [
            'https://api.banking.com/health',
            'https://portal.banking.com/health'
        ]
        
        for endpoint in endpoints:
            if not await self._check_endpoint_health(endpoint):
                raise Exception(f"Endpoint health check failed: {endpoint}")
        
        # Verify core banking functions
        await self._test_core_banking_functions()
        
        self.logger.info("✅ DR validation completed")
    
    async def _get_cluster_info(self, cluster_name: str) -> Dict:
        """Get AKS cluster information."""
        # Implementation would query Azure API
        return {'provisioning_state': 'Succeeded'}
    
    async def _scale_node_pool(self, cluster_name: str, pool_name: str, count: int):
        """Scale AKS node pool."""
        result = subprocess.run([
            'az', 'aks', 'nodepool', 'scale',
            '--resource-group', f'rg-banking-secondary-westus',
            '--cluster-name', cluster_name,
            '--name', pool_name,
            '--node-count', str(count)
        ], capture_output=True, text=True)
        
        if result.returncode != 0:
            raise Exception(f"Node pool scaling failed: {result.stderr}")

# DR Testing and Validation
class DRTestingFramework:
    def __init__(self):
        self.test_scenarios = [
            'regional_outage',
            'database_failure',
            'network_partition',
            'application_failure',
            'storage_failure'
        ]
    
    async def execute_dr_test(self, scenario: str, duration_minutes: int = 30):
        """Execute disaster recovery test scenario."""
        
        test_start = datetime.utcnow()
        print(f"🧪 Starting DR test: {scenario}")
        
        try:
            # Execute test scenario
            if scenario == 'regional_outage':
                await self._simulate_regional_outage(duration_minutes)
            elif scenario == 'database_failure':
                await self._simulate_database_failure(duration_minutes)
            # Add other scenarios...
            
            # Measure recovery metrics
            recovery_metrics = await self._measure_recovery_metrics()
            
            print(f"✅ DR test completed successfully")
            return recovery_metrics
            
        except Exception as e:
            print(f"❌ DR test failed: {str(e)}")
            raise
    
    async def _simulate_regional_outage(self, duration: int):
        """Simulate complete regional outage."""
        
        # Block traffic to primary region
        await self._block_primary_traffic()
        
        # Trigger DR activation
        dr_manager = BankingDRManager("subscription-id")
        await dr_manager.execute_disaster_recovery()
        
        # Simulate outage duration
        await asyncio.sleep(duration * 60)
        
        # Restore primary region (cleanup)
        await self._restore_primary_traffic()

if __name__ == "__main__":
    # Example usage
    async def main():
        dr_manager = BankingDRManager("your-subscription-id")
        result = await dr_manager.execute_disaster_recovery()
        print(f"DR Result: {result}")
    
    asyncio.run(main())
```

**Production Importance:** This comprehensive DR strategy ensures financial institutions can meet regulatory requirements while maintaining cost efficiency. The automated orchestration reduces human error during stressful outage scenarios and provides measurable RTO/RPO compliance with detailed audit trails.

**Q14: Design a multi-tenant isolation strategy that meets banking regulatory requirements.**

**Answer:**

**Regulatory Requirements Analysis:**
- PCI DSS compliance for payment data
- SOX compliance for financial controls
- Regional data residency requirements
- Audit trail requirements

**Isolation Strategy:**

1. **Infrastructure Layer:**
```terraform
# Customer-specific subscriptions for high-isolation tiers
resource "azurerm_subscription" "customer_subscription" {
  count            = var.isolation_tier == "high" ? 1 : 0
  alias            = "customer-${var.customer_id}"
  subscription_name = "GrandCentral-${var.customer_name}"
  billing_scope_id = var.enterprise_agreement_billing_scope
}

# Shared subscription with strict RBAC for standard tiers
resource "azurerm_role_assignment" "customer_rbac" {
  count              = var.isolation_tier == "standard" ? 1 : 0
  scope              = azurerm_resource_group.customer_rg[0].id
  role_definition_id = data.azurerm_role_definition.contributor.id
  principal_id       = azuread_service_principal.customer_sp.object_id
  
  condition = "((ActionMatches{'Microsoft.Resources/subscriptions/resourceGroups/write'}) OR (ActionMatches{'Microsoft.Resources/subscriptions/resourceGroups/delete'})) AND (@Resource[Microsoft.Resources/subscriptions/resourceGroups:name] StringEquals '${var.customer_prefix}*')"
}
```

2. **Kubernetes Layer:**
```yaml
# Network policies for namespace isolation
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: customer-isolation
  namespace: gc-${customer}
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          customer: ${customer}
    - namespaceSelector:
        matchLabels:
          scope: platform
  egress:
  - to:
    - namespaceSelector:
        matchLabels:
          scope: platform
  - to: []
    ports:
    - protocol: TCP
      port: 443  # External banking APIs
```

3. **Data Layer:**
```yaml
# Customer-specific storage encryption
apiVersion: v1
kind: Secret
metadata:
  name: customer-encryption-key
  namespace: gc-${customer}
type: Opaque
data:
  key: ${customer_specific_encryption_key}
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: banking-connector
spec:
  template:
    spec:
      containers:
      - name: connector
        env:
        - name: DB_ENCRYPTION_KEY
          valueFrom:
            secretKeyRef:
              name: customer-encryption-key
              key: key
        - name: CUSTOMER_ID
          value: ${customer}
```

**Production Importance:** Banking regulations require demonstrable isolation between customer data. This multi-layered approach ensures compliance while maintaining operational efficiency.

---

**Q15: How would you implement a zero-downtime deployment strategy for critical banking services?**

**Answer:**

**Blue-Green Deployment with Banking-Specific Considerations:**

1. **Database Migration Strategy:**
```sql
-- Backward-compatible schema changes only
ALTER TABLE transactions 
ADD COLUMN new_field VARCHAR(255) NULL;

-- Deploy application v2 that can handle both old and new schema
-- After validation, populate new field
UPDATE transactions SET new_field = legacy_computation(old_field) 
WHERE new_field IS NULL;

-- Make field NOT NULL in a subsequent release
```

2. **ArgoCD Rollout Strategy:**
```yaml
apiVersion: argoproj.io/v1alpha1
kind: Rollout
metadata:
  name: banking-connector
spec:
  strategy:
    blueGreen:
      activeService: banking-connector-active
      previewService: banking-connector-preview
      autoPromotionEnabled: false  # Manual approval for banking
      prePromotionAnalysis:
        templates:
        - templateName: connectivity-check
        args:
        - name: service-name
          value: banking-connector-preview
      postPromotionAnalysis:
        templates:
        - templateName: transaction-validation
        args:
        - name: duration
          value: 30m  # Extended validation for banking
```

3. **Circuit Breaker Pattern:**
```java
@Component
public class BankingConnectorService {
    
    @CircuitBreaker(name = "mambu-api", fallbackMethod = "fallbackToCache")
    @TimeLimiter(name = "mambu-api")
    @Retry(name = "mambu-api")
    public CompletableFuture<AccountBalance> getAccountBalance(String accountId) {
        return CompletableFuture.supplyAsync(() -> 
            mambuApiClient.getAccountBalance(accountId)
        );
    }
    
    public CompletableFuture<AccountBalance> fallbackToCache(String accountId, Exception ex) {
        return CompletableFuture.supplyAsync(() -> 
            cacheService.getCachedBalance(accountId)
                .orElseThrow(() -> new ServiceUnavailableException("Banking service temporarily unavailable"))
        );
    }
}
```

4. **Feature Flags for Gradual Rollout:**
```yaml
apiVersion: split.io/v1alpha1
kind: Split
metadata:
  name: new-payment-engine
spec:
  treatments:
  - name: control
    weight: 90
  - name: treatment
    weight: 10
  rules:
  - condition: "customer in segment \"beta_customers\""
    treatment: treatment
  - condition: "customer in segment \"high_value_customers\""
    treatment: control
```

**Production Importance:** Banking services cannot afford downtime during business hours. This strategy ensures continuous service availability while allowing for thorough validation of new releases.

---

### Compliance & Governance

**Q16: Design a comprehensive audit and compliance framework for a financial technology platform.**

**Answer:**

**Regulatory Framework Implementation:**

1. **Infrastructure Audit Trail:**
```terraform
# All resources must have audit logging enabled
resource "azurerm_monitor_diagnostic_setting" "resource_audit" {
  name               = "audit-${var.resource_name}"
  target_resource_id = var.resource_id
  storage_account_id = azurerm_storage_account.audit_logs.id

  enabled_log {
    category = "AuditEvent"
  }
  
  enabled_log {
    category = "Administrative"
  }

  metric {
    category = "AllMetrics"
  }
}
```

2. **GitOps Audit Trail:**
```yaml
# ArgoCD audit configuration
apiVersion: v1
kind: ConfigMap
metadata:
  name: argocd-audit-policy
data:
  policy.yaml: |
    rules:
    - level: RequestResponse
      resources:
      - group: argoproj.io
        resources: ["applications", "appprojects"]
      namespaces: ["argocd"]
    - level: Request
      resources:
      - group: ""
        resources: ["secrets", "configmaps"]
```

3. **Data Governance:**
```python
# Data classification and retention policies
class DataGovernancePolicy:
    def __init__(self):
        self.policies = {
            "customer_data": {
                "encryption": "required",
                "retention_days": 2555,  # 7 years for banking
                "geographic_restriction": True,
                "audit_level": "detailed"
            },
            "transaction_data": {
                "encryption": "required", 
                "retention_days": 3650,  # 10 years for transactions
                "immutable": True,
                "audit_level": "comprehensive"
            }
        }
    
    def apply_policy(self, data_type: str, resource: dict):
        policy = self.policies.get(data_type)
        if not policy:
            raise ValueError(f"No policy defined for {data_type}")
            
        return {
            "resource": resource,
            "compliance_metadata": {
                "classification": data_type,
                "retention_end": datetime.now() + timedelta(days=policy["retention_days"]),
                "encryption_required": policy["encryption"],
                "audit_enabled": True
            }
        }
```

4. **Compliance Validation:**
```yaml
# Kyverno policies for regulatory compliance
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: pci-dss-compliance
spec:
  background: false
  validationFailureAction: enforce
  rules:
  - name: require-encryption
    match:
      resources:
        kinds:
        - Secret
        - ConfigMap
    validate:
      message: "PCI DSS requires encryption for sensitive data"
      pattern:
        metadata:
          annotations:
            encryption.enabled: "true"
  - name: network-segmentation
    match:
      resources:
        kinds:
        - Namespace
    validate:
      message: "Banking namespaces must have network policies"
      deny:
        conditions:
        - key: "{{ request.object.metadata.labels.tier || '' }}"
          operator: Equals
          value: banking
        - key: "{{ query(@, 'spec.networkPolicies[*].name') | length(@) }}"
          operator: LessThan
          value: 1
```

**Production Importance:** Financial institutions face strict regulatory oversight. Comprehensive audit capabilities ensure compliance, facilitate regulatory examinations, and provide forensic capabilities for security incidents.

---

## 🔍 Code Review & Refactoring Questions

### Security Code Review

**Q17: Review this Terraform configuration and identify security issues:**

```terraform
resource "azurerm_key_vault" "kv_shared" {
  #checkov:skip=CKV_AZURE_189: we need to access this vault from TF
  location                        = var.location
  name                            = local.shared_keyvault_name
  resource_group_name             = local.shared_resourcegroup
  sku_name                        = "standard"
  tenant_id                       = local.tenant_id
  enabled_for_disk_encryption     = false
  enabled_for_deployment          = false
  enable_rbac_authorization       = true
  soft_delete_retention_days      = 7
  /*
  network_acls {
    default_action = "Deny"
    bypass         = "AzureServices"
    ip_rules       = local.allowed_public_ips
  }
  */
}
```

**Answer:**

**Security Issues Identified:**

1. **Network Access Control Disabled:**
   - Key Vault is accessible from any public IP
   - Network ACLs are commented out

2. **Insufficient Soft Delete Retention:**
   - 7 days is too short for critical banking infrastructure
   - Should be 30-90 days minimum

3. **Missing Purge Protection:**
   - No `purge_protection_enabled = true` 
   - Critical for banking compliance

**Improved Configuration:**
```terraform
resource "azurerm_key_vault" "kv_shared" {
  location                        = var.location
  name                            = local.shared_keyvault_name
  resource_group_name             = local.shared_resourcegroup
  sku_name                        = "premium"  # Hardware security modules
  tenant_id                       = local.tenant_id
  enabled_for_disk_encryption     = false
  enabled_for_deployment          = false
  enable_rbac_authorization       = true
  soft_delete_retention_days      = 90
  purge_protection_enabled        = true  # Added
  public_network_access_enabled   = false  # Added

  network_acls {
    default_action = "Deny"
    bypass         = "AzureServices"
    ip_rules       = local.github_runner_ips
    virtual_network_subnet_ids = [
      azurerm_subnet.terraform_subnet.id,
      azurerm_subnet.aks_subnet.id
    ]
  }
}

# Add private endpoint
resource "azurerm_private_endpoint" "kv_shared" {
  name                = "pe-kv-shared"
  location            = var.location
  resource_group_name = local.shared_resourcegroup
  subnet_id           = azurerm_subnet.private_endpoint_subnet.id
  
  private_service_connection {
    name                           = "pe-kv-shared"
    private_connection_resource_id = azurerm_key_vault.kv_shared.id
    is_manual_connection           = false
    subresource_names              = ["vault"]
  }
}
```

---

### Architecture Refactoring

**Q18: The current ArgoCD application structure has multiple connectors in a single application. How would you refactor this for better maintainability?**

**Current Structure Issue:**
```yaml
# Single application with multiple connectors - hard to manage
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: gc-mambu
spec:
  sources:
    - chart: gc-mambu-party-connector
      targetRevision: 1.0.10
    - chart: gc-mambu-deposit-connector  
      targetRevision: 1.1.1-SNAPSHOT-6c846f2f
    - chart: gc-mambu-loan-connector
      targetRevision: 1.1.1-SNAPSHOT-ce50fa19
    # ... 8 more connectors
```

**Refactored Solution:**

**1. ApplicationSet Pattern:**
```yaml
apiVersion: argoproj.io/v1alpha1
kind: ApplicationSet
metadata:
  name: mambu-connectors
spec:
  generators:
  - list:
      elements:
      - connector: party
        version: "1.0.10"
        config: party-v0
        replicas: 3
      - connector: deposit
        version: "1.1.1-SNAPSHOT-6c846f2f"  
        config: deposit-v0
        replicas: 2
      - connector: loan
        version: "1.1.1-SNAPSHOT-ce50fa19"
        config: loan-v0
        replicas: 2
  template:
    metadata:
      name: 'mambu-{{connector}}-{{config}}'
      namespace: argocd
    spec:
      project: grandcentral
      source:
        repoURL: crecos493.azurecr.io/charts
        chart: 'gc-mambu-{{connector}}-connector'
        targetRevision: '{{version}}'
        helm:
          releaseName: '{{config}}'
          parameters:
          - name: replicaCount
            value: '{{replicas}}'
          valueFiles:
          - '$apps-live/runtimes/{{env}}/values/platform-resource-management/platform-resource-management.yaml'
          - '$apps-live/runtimes/{{env}}/values/gc-mambu/{{config}}.values.yaml'
      destination:
        server: https://kubernetes.default.svc
        namespace: gc-mambu
```

**2. App-of-Apps Pattern:**
```yaml
# Root application
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: banking-platform
spec:
  source:
    repoURL: git@github.com:bb-ecos-ecos/gc-ecos-applications-live.git
    path: runtimes/{{env}}/platform-apps
    targetRevision: main
  destination:
    server: https://kubernetes.default.svc
    namespace: argocd

---
# Platform apps directory structure:
# runtimes/dev/platform-apps/
# ├── mambu-connectors.yaml
# ├── flexcube-connectors.yaml 
# ├── fis-connectors.yaml
# └── platform-services.yaml
```

**Benefits:**
- **Independent versioning** per connector
- **Selective rollbacks** without affecting other services
- **Easier testing** and validation
- **Better observability** per connector
- **Simplified CI/CD** pipelines

---

## 🧠 Scenario-Based Questions

### Production Incident Scenarios

**Q19: At 9 AM on Monday morning, multiple customers report that their banking transactions are failing. Your monitoring shows that the Mambu party connector is returning HTTP 503 errors. Walk through your incident response process.**

**Answer:**

**Immediate Response (0-5 minutes):**

1. **Acknowledge and Escalate:**
```bash
# Check ArgoCD application status
kubectl get applications -n argocd | grep mambu
kubectl describe application gc-mambu -n argocd

# Check pod status
kubectl get pods -n gc-mambu -l app=party-connector
kubectl logs -n gc-mambu deployment/mambu-party-v0 --tail=100
```

2. **Implement Circuit Breaker:**
```yaml
# Immediate mitigation - route to fallback service
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: mambu-party-circuit-breaker
spec:
  host: mambu-party-service
  trafficPolicy:
    outlierDetection:
      consecutiveErrors: 1
      interval: 30s
      baseEjectionTime: 30s
```

**Investigation (5-15 minutes):**

3. **Check External Dependencies:**
```bash
# Test Mambu API connectivity
kubectl run debug-pod --rm -it --image=nicolaka/netshoot -- \
  curl -v https://mambu-sandbox.mambu.com/api/health

# Check Azure Service Bus (for async processing)
az servicebus namespace show --name gc-servicebus --resource-group rg-shared
```

4. **Review Recent Changes:**
```bash
# Check recent ArgoCD syncs
kubectl get events -n gc-mambu --sort-by='.lastTimestamp' | head -20

# Review recent deployments
git log --oneline --since="24 hours ago" -- runtimes/*/values/gc-mambu/
```

**Resolution (15-60 minutes):**

5. **Database Connection Issues (Common Cause):**
```sql
-- Check active connections to Mambu database
SELECT datname, numbackends, xact_commit, xact_rollback 
FROM pg_stat_database 
WHERE datname = 'mambu_integration';

-- Check for blocking queries
SELECT blocked_locks.pid AS blocked_pid,
       blocking_locks.pid AS blocking_pid,
       blocked_activity.query AS blocked_query
FROM pg_catalog.pg_locks blocked_locks
JOIN pg_catalog.pg_stat_activity blocked_activity 
  ON blocked_activity.pid = blocked_locks.pid;
```

6. **Scale Out Strategy:**
```yaml
# Temporarily increase replicas
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mambu-party-v0
spec:
  replicas: 6  # Increased from 3
  template:
    spec:
      containers:
      - name: party-connector
        resources:
          requests:
            memory: "1Gi" 
            cpu: "500m"
          limits:
            memory: "2Gi"
            cpu: "1000m"
```

**Communication:**
- **Customer notification** within 15 minutes
- **Regular updates** every 30 minutes
- **Detailed postmortem** within 24 hours

**Production Importance:** Banking systems have zero tolerance for downtime during business hours. Structured incident response ensures rapid resolution and maintains customer trust.

---

**Q20: You notice that the Terraform state files are becoming corrupted due to concurrent runs. How would you implement state management best practices?**

**Answer:**

**Current Problem Analysis:**
```bash
# Common symptoms of state corruption
terraform plan  # Shows resources that shouldn't be recreated
terraform validate  # May pass but state is inconsistent
```

**Solution Implementation:**

**1. Enhanced State Locking:**
```terraform
# Improved backend configuration
terraform {
  backend "azurerm" {
    resource_group_name  = "rg-tfstate-${var.environment}"
    storage_account_name = "stgctfstate${random_id.suffix.hex}"
    container_name       = "tfstate"
    key                  = "terraform-${var.workspace}.tfstate"
    use_azuread_auth     = true
    
    # Enhanced locking
    snapshot              = true
    
    # Access tier optimization
    access_tier          = "Hot"
  }
}

# State storage account with enhanced security
resource "azurerm_storage_account" "tfstate" {
  name                      = "stgctfstate${random_id.suffix.hex}"
  resource_group_name       = azurerm_resource_group.tfstate.name
  location                  = var.location
  account_tier              = "Standard"
  account_replication_type  = "ZRS"  # Zone redundancy
  
  # Enhanced security
  allow_nested_items_to_be_public = false
  shared_access_key_enabled       = false
  public_network_access_enabled   = false
  
  # Versioning for state history
  versioning_enabled = true
  
  # Legal hold for compliance
  blob_properties {
    versioning_enabled = true
    delete_retention_policy {
      days = 30
    }
    container_delete_retention_policy {
      days = 30
    }
  }
}
```

**2. Workspace-Based Isolation:**
```bash
#!/bin/bash
# Improved deployment script
set -euo pipefail

WORKSPACE="${CUSTOMER_ID}-${ENVIRONMENT}"
LOCK_TABLE="tfstate-locks"

# Ensure workspace isolation
terraform workspace select "$WORKSPACE" || terraform workspace new "$WORKSPACE"

# Check for existing locks before proceeding
if az storage blob exists \
  --account-name "$STORAGE_ACCOUNT" \
  --container-name "$CONTAINER" \
  --name "${WORKSPACE}.tflock" \
  --auth-mode login; then
  
  # Check lock age and break if stale
  LOCK_AGE=$(az storage blob show \
    --account-name "$STORAGE_ACCOUNT" \
    --container-name "$CONTAINER" \
    --name "${WORKSPACE}.tflock" \
    --query 'properties.lastModified' \
    --output tsv)
  
  if [[ $(date -d "$LOCK_AGE" +%s) -lt $(date -d "2 hours ago" +%s) ]]; then
    echo "Breaking stale lock older than 2 hours"
    terraform force-unlock -force
  else
    echo "Active lock detected, failing build"
    exit 1
  fi
fi

# Proceed with deployment
terraform plan -lock-timeout=300s -out=tfplan
terraform apply tfplan
```

**3. State Backup and Recovery:**
```yaml
# GitHub Actions workflow with state backup
name: Terraform with State Protection
jobs:
  deploy:
    steps:
    - name: Backup current state
      run: |
        # Download current state
        terraform state pull > backup-$(date +%Y%m%d-%H%M%S).tfstate
        
        # Upload to backup storage
        az storage blob upload \
          --file backup-*.tfstate \
          --container-name state-backups \
          --name "${WORKSPACE}/$(date +%Y/%m/%d)/backup-$(date +%H%M%S).tfstate"
    
    - name: Plan with lock timeout
      run: |
        terraform plan \
          -lock-timeout=10m \
          -out=tfplan \
          -var-file="${WORKSPACE}.tfvars.json"
    
    - name: Apply with automatic rollback
      run: |
        if ! terraform apply tfplan; then
          echo "Apply failed, restoring from backup"
          LATEST_BACKUP=$(az storage blob list \
            --container-name state-backups \
            --prefix "${WORKSPACE}" \
            --query 'sort_by([].{name:name,modified:properties.lastModified}, &modified)[-1].name' \
            --output tsv)
          
          az storage blob download \
            --container-name state-backups \
            --name "$LATEST_BACKUP" \
            --file restore.tfstate
          
          terraform state push restore.tfstate
          exit 1
        fi
```

**4. Monitoring and Alerting:**
```python
# State health monitoring
import json
import subprocess
from datetime import datetime, timezone

def check_state_health():
    try:
        # Get state information
        state_raw = subprocess.check_output(['terraform', 'state', 'list'])
        resources = state_raw.decode('utf-8').strip().split('\n')
        
        # Check for drift
        plan_output = subprocess.check_output(['terraform', 'plan', '-detailed-exitcode'])
        
        health_status = {
            'timestamp': datetime.now(timezone.utc).isoformat(),
            'resource_count': len(resources),
            'drift_detected': plan_output != 0,
            'lock_status': check_lock_status()
        }
        
        return health_status
        
    except subprocess.CalledProcessError as e:
        return {'error': str(e), 'status': 'unhealthy'}

def check_lock_status():
    # Implementation to check Azure blob lock status
    pass
```

**Production Importance:** State corruption can lead to infrastructure outages, resource duplication costs, and compliance violations. Robust state management ensures infrastructure reliability and reduces operational risk.

---

## 🛠 Debugging & Production Failure Questions

### Kubernetes Debugging

**Q21: Customers report intermittent timeouts when making banking API calls through the platform. Your APM shows 50% of requests to the Mambu connector are taking >30 seconds. Debug this issue.**

**Answer:**

**Initial Investigation:**

1. **Application-Level Analysis:**
```bash
# Check pod resource usage
kubectl top pods -n gc-mambu
kubectl describe pod $(kubectl get pods -n gc-mambu -l app=mambu-party-connector -o name | head -1)

# Check for pending connections
kubectl exec -n gc-mambu deployment/mambu-party-v0 -- netstat -an | grep :8080
```

2. **Network-Level Debugging:**
```bash
# Check DNS resolution times
kubectl run dns-debug --rm -it --image=nicolaka/netshoot -- \
  nslookup mambu-sandbox.mambu.com

# Test external connectivity with timing
kubectl exec -n gc-mambu deployment/mambu-party-v0 -- \
  curl -w "@curl-format.txt" -o /dev/null -s "https://mambu-sandbox.mambu.com/api/health"

# curl-format.txt content:
#      time_namelookup:  %{time_namelookup}\n
#         time_connect:  %{time_connect}\n
#      time_appconnect:  %{time_appconnect}\n
#         time_total:    %{time_total}\n
```

**Common Root Causes & Solutions:**

3. **Database Connection Pool Exhaustion:**
```yaml
# Check current pool configuration
apiVersion: v1
kind: ConfigMap
metadata:
  name: mambu-connector-config
data:
  application.yml: |
    spring:
      datasource:
        # Problem: Pool too small
        hikari:
          maximum-pool-size: 5  # Increase to 20
          connection-timeout: 30000
          idle-timeout: 600000
          max-lifetime: 1800000
          leak-detection-threshold: 60000
        
    # Add connection validation
    jpa:
      properties:
        hibernate:
          connection:
            provider_disables_autocommit: true
```

4. **Memory Pressure Analysis:**
```bash
# Check JVM metrics inside container
kubectl exec -n gc-mambu deployment/mambu-party-v0 -- \
  jcmd 1 VM.info

# Check for GC issues
kubectl exec -n gc-mambu deployment/mambu-party-v0 -- \
  jcmd 1 GC.run_finalization

# Review JVM settings
kubectl get deployment mambu-party-v0 -n gc-mambu -o yaml | grep -A 10 -B 10 JAVA_OPTS
```

5. **External API Rate Limiting:**
```java
// Implement exponential backoff
@Component
public class MambuApiClient {
    
    @Retryable(value = {HttpServerErrorException.class}, 
               backoff = @Backoff(delay = 1000, multiplier = 2, maxDelay = 30000))
    public ResponseEntity<Account> getAccount(String accountId) {
        return restTemplate.getForEntity("/accounts/" + accountId, Account.class);
    }
    
    @EventListener
    public void handleRateLimitExceeded(RateLimitExceededException event) {
        // Implement circuit breaker logic
        circuitBreakerRegistry.circuitBreaker("mambu-api").transitionToOpenState();
        
        // Schedule retry after rate limit reset
        taskScheduler.schedule(() -> {
            circuitBreakerRegistry.circuitBreaker("mambu-api").transitionToClosedState();
        }, Instant.now().plusSeconds(event.getRetryAfterSeconds()));
    }
}
```

6. **Istio Service Mesh Debugging:**
```bash
# Check Envoy proxy stats
kubectl exec -n gc-mambu deployment/mambu-party-v0 -c istio-proxy -- \
  curl localhost:15000/stats | grep "cluster.outbound.*mambu.*pending"

# Check circuit breaker status
kubectl exec -n gc-mambu deployment/mambu-party-v0 -c istio-proxy -- \
  curl localhost:15000/clusters | grep mambu

# Review Istio configuration
kubectl get destinationrule mambu-party -n gc-mambu -o yaml
```

**Performance Optimization:**

7. **Implement Request Queuing:**
```yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: mambu-party-queue
spec:
  http:
  - match:
    - headers:
        priority:
          exact: "high"
    route:
    - destination:
        host: mambu-party-service
        subset: high-priority
    timeout: 10s
  - route:
    - destination:
        host: mambu-party-service
        subset: standard
    timeout: 30s

---
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: mambu-party-subsets
spec:
  host: mambu-party-service
  subsets:
  - name: high-priority
    labels:
      version: v2
    trafficPolicy:
      connectionPool:
        tcp:
          maxConnections: 20
        http:
          http1MaxPendingRequests: 5
          maxRequestsPerConnection: 2
  - name: standard
    labels:
      version: v0
    trafficPolicy:
      connectionPool:
        tcp:
          maxConnections: 10
        http:
          http1MaxPendingRequests: 100
```

**Monitoring Implementation:**
```yaml
# Enhanced monitoring
apiVersion: v1
kind: ConfigMap
metadata:
  name: prometheus-rules
data:
  banking-sla.yml: |
    groups:
    - name: banking.response.time
      rules:
      - alert: BankingAPISlowResponse
        expr: |
          histogram_quantile(0.95, 
            rate(http_request_duration_seconds_bucket{service="mambu-party"}[5m])
          ) > 10
        for: 2m
        labels:
          severity: warning
          customer_impact: high
        annotations:
          summary: "Banking API response time degraded"
          description: "95th percentile response time is {{ $value }}s"
```

**Production Importance:** Banking API performance directly affects customer experience and transaction processing. Systematic debugging prevents revenue loss and maintains regulatory compliance.

---

### Infrastructure Failure Recovery

**Q22: During a routine Kubernetes upgrade, the ArgoCD server becomes unresponsive and all banking applications show "OutOfSync" status. Walk through your recovery process.**

**Answer:**

**Immediate Assessment (0-10 minutes):**

1. **Verify Banking Services Status:**
```bash
# Check if applications are still running despite ArgoCD issues
kubectl get pods -n gc-mambu -o wide
kubectl get pods -n gc-flexcube -o wide
kubectl get pods -n gc-fis -o wide

# Check service endpoints
kubectl get endpoints -A | grep -E "(mambu|flexcube|fis)"

# Verify external connectivity
for service in mambu-party flexcube-core fis-payment; do
  kubectl run test-$service --rm -it --image=curlimages/curl -- \
    curl -f "http://$service.default.svc.cluster.local:8080/health"
done
```

2. **ArgoCD Diagnostics:**
```bash
# Check ArgoCD pods status
kubectl get pods -n argocd
kubectl logs -n argocd deployment/argocd-server --tail=50
kubectl logs -n argocd deployment/argocd-application-controller --tail=50

# Check ArgoCD database connectivity
kubectl exec -n argocd deployment/argocd-server -- \
  argocd cluster list --server localhost:8080
```

**Recovery Strategy:**

3. **ArgoCD Server Recovery:**
```bash
# Scale down ArgoCD components
kubectl scale deployment argocd-server --replicas=0 -n argocd
kubectl scale deployment argocd-application-controller --replicas=0 -n argocd

# Check for PVC corruption
kubectl get pvc -n argocd
kubectl describe pvc argocd-server -n argocd

# Restart with fresh configuration
kubectl delete pod -n argocd -l app.kubernetes.io/name=argocd-server
kubectl delete pod -n argocd -l app.kubernetes.io/name=argocd-application-controller

# Scale back up
kubectl scale deployment argocd-server --replicas=1 -n argocd
kubectl scale deployment argocd-application-controller --replicas=1 -n argocd
```

4. **Database Backup Recovery:**
```bash
# If ArgoCD database is corrupted, restore from backup
kubectl create job argocd-restore --from=cronjob/argocd-backup

# Verify backup restoration
kubectl logs job/argocd-restore -n argocd

# Alternative: Re-import applications from Git
kubectl apply -f - <<EOF
apiVersion: batch/v1
kind: Job
metadata:
  name: argocd-app-restore
  namespace: argocd
spec:
  template:
    spec:
      containers:
      - name: restore
        image: argoproj/argocd:v2.8.0
        command:
        - /bin/bash
        - -c
        - |
          argocd login argocd-server:443 --username admin --password $ARGOCD_PASSWORD --insecure
          
          # Re-create applications from Git repository
          find /apps -name "*.yaml" | xargs -I {} argocd app create -f {}
        env:
        - name: ARGOCD_PASSWORD
          valueFrom:
            secretKeyRef:
              name: argocd-initial-admin-secret
              key: password
        volumeMounts:
        - name: app-configs
          mountPath: /apps
      volumes:
      - name: app-configs
        git:
          repository: git@github.com:bb-ecos-ecos/gc-ecos-applications-live.git
          revision: main
      restartPolicy: OnFailure
EOF
```

5. **Manual Application Sync:**
```bash
# If ArgoCD remains unstable, manually ensure application state
NAMESPACES=("gc-mambu" "gc-flexcube" "gc-fis")

for ns in "${NAMESPACES[@]}"; do
  echo "Checking namespace: $ns"
  
  # Get desired state from Git
  git clone --depth 1 git@github.com:bb-ecos-ecos/gc-ecos-applications-live.git /tmp/apps
  
  # Apply configurations manually
  find /tmp/apps/runtimes/prod/values/$ns -name "*.yaml" | while read file; do
    echo "Applying $file"
    kubectl apply -f "$file" || echo "Failed to apply $file"
  done
  
  # Verify deployments are running
  kubectl rollout status deployment -n $ns --timeout=300s
done
```

6. **Application Health Validation:**
```bash
# Comprehensive health check script
#!/bin/bash
check_banking_health() {
  local namespace=$1
  local service=$2
  
  echo "Checking $service in $namespace"
  
  # Check pod readiness
  local ready_pods=$(kubectl get pods -n $namespace -l app=$service \
    -o jsonpath='{.items[*].status.conditions[?(@.type=="Ready")].status}' | \
    grep -o "True" | wc -l)
  
  local total_pods=$(kubectl get pods -n $namespace -l app=$service --no-headers | wc -l)
  
  if [ "$ready_pods" -eq "$total_pods" ] && [ "$total_pods" -gt 0 ]; then
    echo "✓ $service: $ready_pods/$total_pods pods ready"
  else
    echo "✗ $service: $ready_pods/$total_pods pods ready"
    return 1
  fi
  
  # Check service connectivity
  if kubectl run test-connectivity-$service --rm -i --image=curlimages/curl -- \
     curl -f -s "http://$service.$namespace.svc.cluster.local:8080/health" > /dev/null; then
    echo "✓ $service: Health check passed"
  else
    echo "✗ $service: Health check failed"
    return 1
  fi
}

# Check all critical services
SERVICES=(
  "gc-mambu:mambu-party-connector"
  "gc-mambu:mambu-deposit-connector" 
  "gc-flexcube:flexcube-core"
  "gc-fis:fis-payment-connector"
)

for service_info in "${SERVICES[@]}"; do
  IFS=':' read -r namespace service <<< "$service_info"
  check_banking_health "$namespace" "$service"
done
```

**Prevention Measures:**

7. **Implement ArgoCD High Availability:**
```yaml
# Enhanced ArgoCD configuration
apiVersion: argoproj.io/v1alpha1
kind: ArgoCD
metadata:
  name: argocd
spec:
  ha:
    enabled: true
    
  # External database for reliability
  postgres:
    host: postgres.argocd.svc.cluster.local
    database: argocd
    
  # Backup configuration  
  backup:
    enabled: true
    schedule: "0 */6 * * *"
    destination:
      storageAccount: argocdbkup
      container: backups
      
  # Split components for isolation
  controller:
    replicas: 2
    resources:
      requests:
        memory: "2Gi"
        cpu: "1000m"
        
  server:
    replicas: 2
    resources:
      requests:
        memory: "1Gi"
        cpu: "500m"
```

**Production Importance:** ArgoCD failure can disrupt all deployments in a banking environment. Rapid recovery procedures ensure minimal impact on customer-facing services and maintain regulatory compliance.

---

## Interview Question Bank Summary

This comprehensive interview question bank covers modern cloud-native financial technology platforms:

### Key Technologies Covered:
- **Azure Infrastructure**: Terraform/OpenTofu, multi-tier deployments, RBAC, Key Vault
- **Kubernetes & GitOps**: ArgoCD applications, multi-source deployments, policy enforcement
- **Banking System Integration**: Core banking connectors, financial APIs, compliance frameworks
- **Enterprise Security**: OIDC authentication, secrets management, network policies, encryption
- **DevOps & CI/CD**: GitHub Actions, matrix builds, automated testing, deployment pipelines
- **Observability**: Monitoring, alerting, compliance validation, performance optimization

### Interview Effectiveness:
- **Progressive Difficulty**: From basic configuration understanding to complex architectural decisions
- **Real-World Focus**: Based on actual production challenges in financial technology
- **Code-Centric**: Includes practical configuration examples and implementation patterns  
- **Production-Ready**: Emphasizes compliance, security, and operational excellence

### Candidate Assessment Framework:
- **Junior Level (Q1-Q4)**: Infrastructure as Code basics, Kubernetes fundamentals, environment organization
- **Intermediate Level (Q5-Q9)**: Security practices, CI/CD implementation, banking domain knowledge
- **Advanced Level (Q10-Q16)**: Architecture decisions, performance optimization, compliance frameworks
- **Expert Level (Q17-Q22)**: System design, complex debugging, production incident response

This question bank ensures candidates are assessed on practical skills needed for enterprise financial technology platforms, covering both technical depth and real-world application scenarios.
