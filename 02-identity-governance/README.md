# 🛡️ Lab 02 — Identity & Governance: Users, Groups, RBAC & Custom Roles

Managing **identities** and enforcing **least-privilege access** across an Azure tenant — from creating internal and guest users, through group-based access management, to management-group-scoped role assignments and a purpose-built custom RBAC role. This lab covers the identity and governance foundation that underpins every other Azure administration task.

---

## 🎯 Objective

Stand up the identity and access-control structure a growing organization needs:

1. Create internal users and invite external (guest) collaborators
2. Organize access through security groups rather than per-user grants
3. Introduce a management group to govern subscriptions centrally
4. Assign a built-in role at management-group scope so it inherits downward
5. Build a **custom RBAC role** that grants exactly the permissions required — no more
6. Verify every change through the Activity Log

---

## 🛠️ Implementation

### 1. Create and configure user accounts

Created an internal member user, `az104-user1`, with job title and department populated on the Properties tab (usage location set to support future license assignment):

![az104-user1 created](./screenshots/01-user-created-az104user1.png)

Then invited an **external guest** via B2B collaboration — the standard way to grant an outside contractor or partner access without creating a full internal account. The guest shows as `User type: Guest`, `Invitation state: Pending acceptance`:

![Guest user invited](./screenshots/02-guest-user-invited.png)

The invitation email arrives in the guest's inbox with an **Accept invitation** link that redirects to the tenant — confirming the B2B flow works end to end:

![Invite email received](./screenshots/03-invite-email-received.png)

---

### 2. Create a security group and add members

Created a **security group** (`IT Lab Administrators`), assigned membership type and an owner:

![New group basics](./screenshots/04-new-group-basics.png)

…and added members directly (Assigned membership type):

![Add group members](./screenshots/05-group-add-members.png)

Group-based access is the recommended practice — you assign roles to a group **once**, then control access simply by adding or removing members, rather than re-assigning roles per person.

---

### 3. Implement a management group

Created a management group, `az104-mg1`, which sits directly under the built-in **Tenant Root Group**:

![Management group created](./screenshots/06-management-group-created.png)

Management groups let you apply RBAC and Azure Policy **once at the group level** and have them **inherit** to every subscription beneath — the core mechanism for governing many subscriptions consistently.

---

### 4. Assign a built-in role at management-group scope

Created a `helpdesk` security group and assigned it the built-in **Virtual Machine Contributor** role, scoped to `az104-mg1`. This role lets the help desk manage VMs but *not* touch the OS, networking, or storage they connect to — a deliberate least-privilege choice:

![Assign VM Contributor to helpdesk](./screenshots/07-assign-vmcontributor-helpdesk.png)

Because the assignment is at management-group scope, it automatically applies to every subscription in the group — no per-subscription assignment needed.

---

### 5. Create a custom RBAC role

Built-in roles sometimes grant more than a scenario needs. To implement true least privilege, I created a custom role, `Custom Support Request`, by **cloning** the built-in *Support Request Contributor*:

![Custom role basics — clone](./screenshots/08-custom-role-basics-clone.png)

Then **excluded** a specific permission — the ability to register the Support resource provider — so it lands in the role's `NotActions`. The help desk can raise support tickets but cannot register providers:

![Custom role permissions](./screenshots/09-custom-role-permissions.png)

The role's **assignable scope** was pinned to the management group, controlling where it can be used:

![Custom role assignable scope](./screenshots/10-custom-role-assignable-scope.png)

The generated **JSON** shows the full role definition — `Actions`, `NotActions`, and `AssignableScopes` — which is exactly how Azure stores and evaluates every RBAC role:

![Custom role JSON](./screenshots/11-custom-role-json.png)

```jsonc
"actions": [
  "Microsoft.Authorization/*/read",
  "Microsoft.Resources/subscriptions/resourceGroups/read",
  "Microsoft.Support/*"
],
"notActions": [
  "Microsoft.Support/register/action"   // explicitly denied
],
"assignableScopes": [
  "/providers/Microsoft.Management/managementGroups/az104-mg1"
]
```

---

### 6. Verify assignments and audit with the Activity Log

The management group's **Role assignments** blade confirms the final access state — the Owner (inherited) plus the `helpdesk` group with Virtual Machine Contributor:

![Role assignments verified](./screenshots/12-role-assignments-verified.png)

Finally, the **Activity Log** provides the audit trail — the custom role definition, the role assignment, and the management group creation all recorded with status, timestamp, and initiator:

![Activity log audit](./screenshots/13-activity-log-audit.png)

---

## 🔑 Key Technical Takeaways

- **Scope determines reach.** The same role assigned at management-group scope grants far more than at resource-group scope — getting the scope right matters as much as getting the role right.
- **RBAC inherits downward.** A role assigned at a management group flows to every subscription and resource group beneath it, with no separate assignment needed at each level.
- **Assign to groups, not individuals.** Group-based assignment means access is managed by group membership — cleaner, auditable, and less error-prone than per-user grants.
- **Custom roles enforce least privilege.** When a built-in role grants too much, clone it and move the unwanted permissions into `NotActions`. Every role is ultimately a JSON document of `Actions`, `NotActions`, and `AssignableScopes`.
- **RBAC ≠ Entra ID roles.** Azure RBAC governs access to *Azure resources* (VMs, storage, networks); Entra ID roles govern *directory* objects (users, groups, tenant settings). They are separate systems.
- **B2B guests extend access safely.** External collaborators get scoped access through invitation without a full internal account or a shared credential.
- **The Activity Log is the audit source of truth** for who changed access, what changed, and when.

---

## 🧰 Tools & Services

`Microsoft Entra ID` · `Azure RBAC` · `Management Groups` · `Built-in & Custom Roles` · `Security Groups` · `B2B Guest Invitations` · `Activity Log` · `Azure Portal`

---

## 💰 Cost Note

Identity and governance constructs — users, groups, management groups, role assignments, and custom roles — are **not billable**. This entire lab ran at zero cost.

---

*Part of my [Azure AZ-104 hands-on lab portfolio](../README.md).*
