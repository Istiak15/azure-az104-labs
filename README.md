# ☁️ Azure Virtual Networking — Hands-On Implementation

A hands-on Azure networking project completed as part of my preparation for the **AZ-104: Microsoft Azure Administrator** certification. This repository documents the design, deployment, and verification of a multi-network Azure environment — covering virtual networks, subnetting, Infrastructure-as-Code deployment, network security, and DNS — all built and tested in a live Azure subscription.

---

## 👤 About Me

IT professional with a background in enterprise support and cloud infrastructure, building hands-on expertise in Microsoft Azure administration.

- 🎓 **B.Sc. Computing Science** — Saint Mary's University
- 📜 **Certifications:** SC-900 (Security, Compliance & Identity Fundamentals), AZ-104 *(in progress)*
- 💼 Focused on cloud administration, networking, and security

---

## 📋 Project Summary

The goal of this project was to stand up a realistic two-network Azure environment for a growing organization — one network for core business services, one for a manufacturing division — and connect them with the appropriate security and name-resolution infrastructure. The work was split into four labs, each targeting a specific AZ-104 exam domain:

| # | Lab | What It Covers | Report |
|---|-----|----------------|--------|
| 01 | Virtual Networking | VNets, subnetting, ARM template authoring & deployment | [Full report →](./01-virtual-networking/README.md) |
| 02 | Identity & Governance | Entra ID users/groups, RBAC, Azure Policy, management groups | [Full report →](./02-identity-governance/README.md) |
| 03 | Network Security | NSGs, Application Security Groups, rule prioritization | [Full report →](./03-nsg-asg/README.md) |
| 04 | DNS | Public & private DNS zones, delegation, record management | [Full report →](./04-dns-configuration/README.md) |

**Environment used:** A live Azure Pay-As-You-Go subscription. Every resource group was deleted immediately after each lab was verified, keeping total cloud spend for this entire project under $1 CAD.

---

## 🏗️ Architecture

```
Resource Group: az104-rg4  (East US)
│
├── CoreServicesVnet          10.20.0.0/16
│   ├── SharedServicesSubnet  10.20.10.0/24  ← NSG: myNSGSecure attached
│   └── DatabaseSubnet        10.20.20.0/24
│
├── ManufacturingVnet         10.30.0.0/16   ← deployed via ARM template
│   ├── SensorSubnet1         10.30.20.0/24
│   └── SensorSubnet2         10.30.21.0/24
│
├── myNSGSecure (NSG)
│   ├── Inbound:  Allow 80/443 from ASG "asg-web"      (priority 100)
│   └── Outbound: Deny all to Internet service tag       (priority 4096)
│
├── istiaktech.com (Public DNS Zone)
│   └── www.istiaktech.com  →  10.1.1.4  (verified via nslookup)
│
└── private.istiaktech.com (Private DNS Zone)
    ├── Linked to: ManufacturingVnet (link: manufacturing-link)
    └── sensorvm.private.istiaktech.com  →  10.1.1.4
```

Two independently addressed networks (10.20.x and 10.30.x) were deliberately chosen with non-overlapping CIDR ranges — a prerequisite for VNet peering, even though peering itself wasn't part of this lab set.

---

## 🎯 Skills Demonstrated

**Networking**
- CIDR-based address space planning across multiple VNets
- Subnet segmentation and Azure's reserved-address behavior (5 IPs/subnet)
- Infrastructure as Code — exporting, editing, and redeploying ARM templates
- Diagnosing and fixing real ARM template errors (malformed JSON, incorrect subnet `id` references)

**Security**
- Network Security Group rule design (priority ordering, allow/deny logic)
- Application Security Groups for maintainable, IP-independent security rules
- Understanding and safely overriding Azure's default outbound-allow behavior

**DNS**
- Public DNS zone creation and domain delegation model (NS records, SOA)
- Private DNS zones and virtual network link-based internal resolution
- Using `nslookup` to verify resolution and diagnose a real failed query

**Identity & Governance**
- Microsoft Entra ID users, groups, and group-based licensing
- Role-Based Access Control — built-in roles, custom roles, scope inheritance
- Azure Policy structure (definitions, initiatives, effects) and management group hierarchy

---

## 🛠️ Technologies & Tools

`Microsoft Azure` · `Azure Portal` · `ARM Templates (JSON)` · `Azure Cloud Shell` · `VS Code` · `Microsoft Entra ID` · `Virtual Networks` · `Network Security Groups` · `Application Security Groups` · `Azure DNS` · `RBAC` · `Azure Policy` · `Git`

---

## 📚 Certification Roadmap

Working toward **AZ-104: Microsoft Azure Administrator**, covering all five exam domains:

- ✅ Manage Azure identities and governance (20–25%)
- 🔄 Implement and manage storage (15–20%)
- 🔄 Deploy and manage Azure compute resources (20–25%)
- ✅ Implement and manage virtual networking (15–20%)
- 🔄 Monitor and maintain Azure resources (10–15%)

---

## 📩 Contact

- **LinkedIn:** [Add your LinkedIn URL here]
- **Email:** [Add your professional email here]
