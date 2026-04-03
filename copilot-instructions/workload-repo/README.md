# Copilot Instructions - Workload Repository Example

This directory contains copilot instructions designed for a **bootstrapped workload solution repository** within an Azure Landing Zone architecture.

## Purpose

These instructions guide AI assistants (like GitHub Copilot) to generate code that adheres to enterprise Azure Landing Zone constraints and best practices, including:

- **Zero-trust RBAC model** - CI/CD identities have limited, scoped permissions
- **DRY configuration** - Environment-specific values in tfvars files only
- **Central platform integration** - Use shared Key Vault and Service Bus resources
- **Pre-bootstrapped infrastructure** - State storage and identities already exist

## Files

| File | Description |
|------|-------------|
| [copilot-instructions.md](copilot-instructions.md) | The main copilot instructions document |

## Usage

To use these instructions in your workload repository:

1. Copy the `copilot-instructions.md` file to your repository
2. Place it in one of these locations:
   - `.github/copilot-instructions.md` (repository-level)
   - `.github/copilot-instructions/` directory (for multiple instruction sets)
3. Customize the instructions based on your specific platform constraints

## Architecture Context

```
┌─────────────────────────────────────────────────────────────┐
│                     Azure Landing Zone                       │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌──────────────────────┐    ┌──────────────────────────┐   │
│  │  Platform Services   │    │   Workload Repository    │   │
│  │  (Central Repo)      │    │   (This Pattern)         │   │
│  ├──────────────────────┤    ├──────────────────────────┤   │
│  │ • Key Vault          │◄───│ • Data Sources Only      │   │
│  │ • Service Bus        │    │ • Scoped RBAC            │   │
│  │ • State Storage      │    │ • Environment tfvars     │   │
│  │ • OIDC Identities    │    │ • No Central Resources   │   │
│  └──────────────────────┘    └──────────────────────────┘   │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```
