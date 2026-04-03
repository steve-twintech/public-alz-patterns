# Workload Repository & Agent Blueprint

> **Role:** You are assisting a Senior DevSecOps Engineer in an enterprise Azure Platform engineering environment.

> **Context:** This repository operates in **Workload Mode**. It does not handle the initial setup of its own Azure CI/CD identities or Terraform state infrastructure.

When generating code, writing Terraform, or creating CI/CD workflows for this repository, you must strictly adhere to the following architectural rules and constraints.

---

## Workload Bootstrapping & Environment Architecture

This repository is **pre-bootstrapped**. You must assume the following infrastructure and configuration items already exist and are provided by the platform:

### Target Environments
- `dev`
- `stage`
- `prod`

### Workload Mapping
A workload corresponds to exactly **one Resource Group per environment**.

> **Note:** These resource groups may reside in the same Azure Subscription or be spread across different subscriptions depending on the environment tier. The CI/CD pipeline handles this routing dynamically.

### State & Identity Management
The following are provided to the pipeline at runtime via GitHub Environment Variables/Secrets:

| Category | Variables |
|----------|-----------|
| OIDC Federated Identity | `AZURE_CLIENT_ID`, `AZURE_TENANT_ID`, `AZURE_SUBSCRIPTION_ID` |
| Terraform Backend Configuration | `TF_STATE_RG`, `TF_STATE_STORAGE_ACCOUNT`, `TF_STATE_CONTAINER` |

### Configuration Strategy (DRY)
- **NEVER** hardcode environment-specific values (names, capacities, SKUs) in root `.tf` files.
- All variables must be defined in `variables.tf` and populated exclusively via per-environment config files:
  - `config/dev.tfvars`
  - `config/stage.tfvars`
  - `config/prod.tfvars`

---

## Strict RBAC Limitations

We operate on a **zero-trust, least-privilege model**. These rules are **non-negotiable**.

### Self-Modification Denied
This repository **CANNOT** and **MUST NOT** generate code attempting to modify the RBAC assignments of its own CI/CD Principal (Service Principal or Managed Identity) at the subscription or management group level.

### Scoped Permissions
The CI/CD identity has `Contributor`/`User Access Administrator` (or equivalent custom roles) scoped **only** to the workload's specific Resource Group for the target environment.

### Permitted Actions
You may generate `azurerm_role_assignment` resources **only** to assign permissions between resources that exist entirely within this workload's designated Resource Group.

**Example:** Granting an AKS cluster identity access to an ACR within the same RG.

---

## Central Platform Resources Integration

Workloads must utilize central platform resources. **Do not create new instances of these services within the workload resource group.**

### A. Central Key Vault

Applications must use the central Subscription or Platform Key Vault for managing enterprise secrets.

#### Constraint
Do **not** write Terraform code to provision an `azurerm_key_vault`.

#### Procedure
1. Use Terraform data sources (`data "azurerm_key_vault"`) to reference the central Key Vault by name and resource group (passed via variables).
2. Inject secrets into the application using native Azure identity integrations:
   - App Service Key Vault references
   - AKS Secret Store CSI driver
3. **Avoid passing plain text secrets through Terraform state when possible.**

### B. Platform Service Bus

Applications requiring message queues or topics must use the central Platform Service Bus namespace.

#### Constraint
Do **not** write Terraform code to provision an `azurerm_servicebus_namespace`.

#### Procedure for New Queues/Topics
1. A **Pull Request** must be made to the central Platform Service Bus Repository to request a new queue.
2. The central repository must be updated to include the new queue and export its details via outputs.
3. This workload repository will then consume those details (via remote state, data sources, or variables).

#### Example Output (Required in the Central Service Bus Repo)

```hcl
# Example of what must be added to the Central Service Bus Repo's outputs.tf

output "workload_xyz_queue_id" {
  description = "The Resource ID of the Workload XYZ queue"
  value       = azurerm_servicebus_queue.workload_xyz_queue.id
}
```

---

## Quick Reference Checklist

When generating code for this workload repository, always verify:

- [ ] No hardcoded environment values in `.tf` files
- [ ] All variables populated via `config/{env}.tfvars`
- [ ] No `azurerm_key_vault` resource blocks
- [ ] No `azurerm_servicebus_namespace` resource blocks
- [ ] No RBAC modifications to CI/CD principal
- [ ] Role assignments scoped to workload Resource Group only
- [ ] Using data sources for central platform resources
