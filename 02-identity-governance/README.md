# Lab 02 — Identity & Governance

## 🎯 Objective

Implement core Azure identity and governance controls using Microsoft Entra ID, Role-Based Access Control (RBAC), management groups, and Azure Policy — establishing the access-control foundation that the networking labs in this repo run on top of.

---

## 🧱 What Was Built

### Users & Groups
- Created individual users directly in Microsoft Entra ID
- Created a security group with assigned membership, and explored dynamic membership rules
- Assigned a license to the group directly (group-based licensing) rather than per-user, to observe automatic license inheritance for group members

### Management Groups
- Created a management group above the subscription level
- Enabled "Access management for Azure resources" at the root scope, which is a prerequisite for managing subscription access from Entra ID

### Role-Based Access Control (RBAC)
- Assigned the built-in **Virtual Machine Contributor** role to a user at resource-group scope
- Cloned a built-in role to create a **custom role** scoped to a narrower set of permissions
- Reviewed role assignments in **Access control (IAM)**, distinguishing directly-assigned roles from roles inherited from a parent scope

### Azure Policy
- Reviewed built-in policy definitions and initiatives (policy sets)
- Worked through policy effect types (`deny`, `audit`, `modify`, `deployIfNotExists`) and how definition scope determines where a policy can later be assigned

---

## 🧠 Key Concepts Demonstrated

- **Least privilege** — granting a role scoped to exactly the resource group and permission set required, rather than defaulting to broader access
- **RBAC scope inheritance** — a role assigned at a parent scope (management group, subscription) flows down to every child resource group and resource beneath it
- **Custom roles** — cloning and trimming a built-in role definition when none of the built-ins matched the required permission set exactly
- **Management group hierarchy** — organizing subscriptions under a shared governance boundary before assigning policy or access at scale
- **Policy vs. RBAC** — RBAC controls *who can perform which actions*; Azure Policy controls *what configurations are allowed to exist*, regardless of who created them

---

## 💡 What I Learned

RBAC and Azure Policy solve different problems that are easy to conflate at first. RBAC is about **permission** — can this identity call this API. Policy is about **compliance** — even if an identity is permitted to create a resource, does that resource's configuration meet organizational standards. Seeing role inheritance in practice (a role assigned at the subscription automatically applying to every resource group beneath it, without a separate assignment at each one) also clarified why getting the *scope* right matters as much as getting the *role* right — the same role assigned one level too high grants far more access than intended.

*Screenshots for this lab were not captured during the original session — this report reflects the work completed. Screenshots from a future run-through of this lab (VM Contributor assignment, IAM role list, custom role JSON, management group hierarchy) will be added here.*
