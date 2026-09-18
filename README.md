# ☁️ Azure AZ-104 — Hands-On Labs Portfolio

A collection of hands-on Microsoft Azure labs completed as part of my preparation for the **AZ-104: Microsoft Azure Administrator** certification. Every lab in this repository was built and verified in a **live Azure subscription** — real deployments, real configuration, real troubleshooting — and documented with portal screenshots and technical write-ups.

---

## 👤 About Me

IT professional with a background in enterprise support and cloud infrastructure, building hands-on expertise in Microsoft Azure administration.

- 🎓 **B.Sc. Computing Science** — Saint Mary's University
- 📜 **Certifications:** SC-900 (Security, Compliance & Identity Fundamentals), AZ-104 *(in progress)*
- 💼 Focused on cloud administration, networking, and information security

---

## 📚 Labs

| # | Lab | What It Covers | Report |
|---|-----|----------------|--------|
| 01 | Virtual Networking | VNets, subnetting, ARM template authoring & deployment | [Full report →](./01-virtual-networking/README.md) |
| 02 | Identity & Governance | Users, guests, groups, management groups, RBAC & custom roles | [Full report →](./02-identity-governance/README.md) |
| 03 | Network Security | NSGs, Application Security Groups, rule prioritization | [Full report →](./03-nsg-asg/README.md) |
| 04 | DNS | Public & private DNS zones, delegation, record management | [Full report →](./04-dns-configuration/README.md) |
| 05 | Intersite Connectivity | VNet peering, Network Watcher verification, user-defined routing | [Full report →](./05-intersite-connectivity/README.md) |

**Environment used:** A live Azure Pay-As-You-Go subscription. Each resource group was deleted immediately after its lab was verified, keeping total cloud spend across all labs to a few dollars — with per-lab cost governed by a budget alert and disciplined cleanup.

> Storage, Compute, and Monitoring labs are in progress and will be added here as they're completed.

---

## 🎯 Skills Demonstrated

**Networking**
- CIDR-based address space planning across multiple VNets
- Subnet segmentation and Azure's reserved-address behavior (5 IPs/subnet)
- VNet peering — bidirectional links, non-transitivity, backbone routing
- User-defined routes (UDRs) and network virtual appliance (NVA) traffic steering
- Infrastructure as Code — exporting, editing, and redeploying ARM templates
- Diagnosing real ARM template errors (malformed JSON, incorrect subnet `id` references)

**Security**
- Network Security Group rule design (priority ordering, allow/deny logic)
- Application Security Groups for maintainable, IP-independent security rules
- Understanding and safely overriding Azure's default outbound-allow behavior
- Control-plane connectivity testing without exposing public inbound ports

**DNS**
- Public DNS zone creation and domain delegation model (NS records, SOA)
- Private DNS zones and virtual network link-based internal resolution
- Using `nslookup` to verify resolution and diagnose a real failed query

**Identity & Governance**
- Entra ID user lifecycle and B2B guest collaboration
- Group-based access management (assign roles to groups, not individuals)
- Management groups for centralized, inheriting governance
- Azure RBAC scope model and role inheritance
- Custom role authoring with `NotActions` for least-privilege access
- Activity Log auditing of access changes

**Diagnostics & Verification**
- Network Watcher Connection Troubleshoot for before/after connectivity proof
- PowerShell `Test-NetConnection` for TCP-level validation across peered networks

---

## 🛠️ Technologies & Tools

`Microsoft Azure` · `Azure Portal` · `ARM Templates (JSON)` · `Azure Cloud Shell` · `VS Code` · `Virtual Networks` · `VNet Peering` · `Network Security Groups` · `Application Security Groups` · `Route Tables / UDRs` · `Network Watcher` · `Azure DNS` · `Azure Run Command` · `PowerShell` · `Git`

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

- LinkedIn: https://www.linkedin.com/in/kazi-istiak/
- Email: kaziistiak.dev@gmail.com
