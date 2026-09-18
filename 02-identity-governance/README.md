# 🛡️ Lab 02 — Identity & Governance: Users, Groups, RBAC, Custom Roles & Policy-Driven Tagging

Managing **identities** and enforcing **least-privilege access** across an Azure tenant — from creating internal and guest users, through group-based access management, to management-group-scoped role assignments and a purpose-built custom RBAC role. The second half moves from *who can act* to *what configurations are allowed*: using **Azure Policy** to make tagging mandatory, auto-remediate resources that are missing it, and locking a resource group against accidental deletion. Together, RBAC and Policy are the two halves of Azure governance.

---

## 🎯 Objective

Stand up the identity, access-control, and configuration-governance structure a growing organization needs:

1. Create internal users and invite external (guest) collaborators
2. Organize access through security groups rather than per-user grants
3. Introduce a management group to govern subscriptions centrally
4. Assign a built-in role at management-group scope so it inherits downward
5. Build a **custom RBAC role** that grants exactly the permissions required — no more
6. Verify every change through the Activity Log
7. Enforce mandatory resource tagging with a **deny**-effect Azure Policy
8. Auto-remediate missing tags with a **modify**-effect policy and a remediation task
9. Prove the enforcement by deploying a real resource against it
10. Protect the resource group from accidental deletion with a **resource lock**

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

### 7. Enforce mandatory tagging with a Deny policy

Moved from access control to configuration governance by assigning the built-in policy **Require a tag and its value on resources**, scoped to a dedicated resource group, `az104-rg2`:

![Resource group created with tag](./screenshots/14-resource-group-tag-created.png)

Like every Azure Policy resource, the definition is just JSON — a single `if/then` rule. If the resource's tag doesn't equal the required value, the effect is `deny`:

![Policy definition JSON](./screenshots/15-policy-definition-require-tag-json.png)

```json
"if": {
  "not": {
    "field": "[concat('tags[', parameters('tagName'), ']')]",
    "equals": "[parameters('tagValue')]"
  }
},
"then": { "effect": "deny" }
```

Assigned it at resource-group scope with `Tag Name = Cost Center` and `Tag Value = 000` — from this point on, any resource landing in `az104-rg2` without that exact tag gets rejected outright:

![Assign policy — scope](./screenshots/16-assign-policy-scope-rg.png)
![Assign policy — basics](./screenshots/17-assign-policy-basics-require-tag.png)
![Assign policy — parameters](./screenshots/18-assign-policy-parameters-costcenter.png)
![Assign policy — review + create](./screenshots/19-assign-policy-review-require-tag.png)

---

### 8. Auto-remediate missing tags with a Modify policy

A `deny` policy alone punishes anyone who simply forgets the tag. To close that gap, assigned a second built-in policy — **Inherit a tag from the resource group if missing** — using the `modify` effect, with a remediation task enabled so it can also fix existing non-compliant resources, not just new ones:

![Assign policy — inherit tag basics](./screenshots/23-assign-policy-basics-inherit-tag.png)
![Assign policy — remediation task](./screenshots/24-assign-policy-remediation-inherit-tag.png)

With both assignments active, `az104-rg2` is now governed by two policies working together — one fills the tag in, the other blocks anything still missing it after that:

![Both policy assignments active](./screenshots/25-policy-assignments-both-active.png)

---

### 9. Validate the enforcement with a real deployment

Created a storage account, `second12`, in `az104-rg2` **without manually setting any tag** — deliberately relying on the two policies to handle it. Deployment succeeded:

![Storage account deployment succeeded](./screenshots/26-storage-account-review-success.png)
![Deployment complete](./screenshots/27-storage-account-deployment-complete.png)

The account's Essentials pane confirms the design worked: `Cost Center: 000` is applied automatically. The `modify` policy added it before the `deny` policy ever evaluated the request, so nothing was rejected:

![Cost Center tag inherited automatically](./screenshots/28-storage-account-tag-inherited.png)

An earlier attempt at the same task, under a different account name, failed at Review + Create — but the cause was a client-side form validation error (the **Primary service** dropdown had been left on its placeholder), not a policy denial. That request never reached Azure Resource Manager, so Policy was never even evaluated:

![Storage account validation failed](./screenshots/21-storage-account-validation-failed.png)

A useful reminder for troubleshooting: a red ✗ on Review + Create can come from the portal form itself, before RBAC or Policy are ever in the picture.

---

### 10. Lock the resource group against accidental deletion

Added a **Delete** lock (`rg-lock`) on `az104-rg2` — the last layer of protection, sitting above both RBAC and Policy:

![Delete lock added to resource group](./screenshots/29-resource-group-add-delete-lock.png)

Confirmed it works by attempting to delete the resource group afterward — the deletion was blocked, exactly as designed. A `CanNotDelete` lock applies regardless of RBAC role, so even an Owner-level account can't remove a locked resource or resource group until the lock itself is taken off first.

The final state — tag, both policy assignments, the lock, and a fully compliant resource — reflects the finished governance setup:

![Resource group final overview](./screenshots/30-resource-group-final-overview.png)
![Policy compliance — 100%](./screenshots/31-policy-compliance-overview.png)

---

## 🔑 Key Technical Takeaways

- **Scope determines reach.** The same role assigned at management-group scope grants far more than at resource-group scope — getting the scope right matters as much as getting the role right.
- **RBAC inherits downward.** A role assigned at a management group flows to every subscription and resource group beneath it, with no separate assignment needed at each level.
- **Assign to groups, not individuals.** Group-based assignment means access is managed by group membership — cleaner, auditable, and less error-prone than per-user grants.
- **Custom roles enforce least privilege.** When a built-in role grants too much, clone it and move the unwanted permissions into `NotActions`. Every role is ultimately a JSON document of `Actions`, `NotActions`, and `AssignableScopes`.
- **RBAC ≠ Entra ID roles.** Azure RBAC governs access to *Azure resources* (VMs, storage, networks); Entra ID roles govern *directory* objects (users, groups, tenant settings). They are separate systems.
- **B2B guests extend access safely.** External collaborators get scoped access through invitation without a full internal account or a shared credential.
- **The Activity Log is the audit source of truth** for who changed access, what changed, and when.
- **Deny and Modify work as a pair.** A `deny` policy alone breaks any deployment that omits a required tag; pairing it with a `modify`/inherit policy closes the gap by filling the tag in automatically before the `deny` check ever runs.
- **Tags are not inherited by default.** A tag on a resource group does not automatically appear on the resources inside it — that requires a `modify` (or legacy `append`) policy to copy it down.
- **Portal validation runs before Policy.** A red ✗ on Review + Create can come from an incomplete form field, not a policy or RBAC denial — the request has to pass client-side validation before it ever reaches Azure Resource Manager.
- **Resource locks sit above RBAC.** A `CanNotDelete` lock blocks deletion for every principal at that scope — Owner included — until the lock itself is removed.

---

## 🧰 Tools & Services

`Microsoft Entra ID` · `Azure RBAC` · `Management Groups` · `Built-in & Custom Roles` · `Security Groups` · `B2B Guest Invitations` · `Azure Policy` · `Resource Tags` · `Resource Locks` · `Activity Log` · `Azure Portal`

---

## 💰 Cost Note

Identity and governance constructs — users, groups, management groups, role assignments, and custom roles — are **not billable**; that part of this lab ran at zero cost. The policy/tagging/lock section deployed one Standard GRS storage account to validate the enforcement. Following the same cleanup discipline as the rest of this portfolio, `az104-rg2` was removed once the configuration was verified, keeping this lab's total spend effectively zero.

---

*Part of my [Azure AZ-104 hands-on lab portfolio](../README.md).*
