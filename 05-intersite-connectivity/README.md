# 🔗 Lab 05 — Intersite Connectivity: VNet Peering & User-Defined Routing

Implementing and verifying communication between two isolated Azure virtual networks using **VNet peering**, then controlling traffic flow with a **user-defined route (UDR)**. This lab demonstrates the full lifecycle of intersite connectivity — proving default network isolation, establishing peering, validating the connection at the packet level, and layering custom routing on top.

---

## 🎯 Objective

Two business units — **core IT services** and a **manufacturing division** — run in separate virtual networks for isolation. Certain workloads need to communicate across that boundary. The goal was to:

1. Deploy two VMs in two independently addressed virtual networks
2. Confirm the networks are isolated by default (no connectivity)
3. Establish bidirectional VNet peering
4. Verify cross-network connectivity at the TCP level
5. Add a user-defined route to steer subnet traffic through a network virtual appliance (NVA)

---

## 🏗️ Environment

```
Resource Group: az104-rg5  (East US)
│
├── CoreServicesVnet          10.0.0.0/16
│   ├── Core        subnet     10.0.0.0/24   → CoreServicesVM   (10.0.0.4)
│   └── perimeter   subnet     10.0.1.0/24   → route table attached
│
├── ManufacturingVnet         172.16.0.0/16
│   └── Manufacturing subnet   172.16.0.0/24 → ManufacturingVM  (172.16.0.4)
│
├── Peering (bidirectional, Fully Synchronized)
│   ├── CoreServicesVnet-to-ManufacturingVnet
│   └── ManufacturingVnet-to-CoreServicesVnet
│
└── rt-CoreServices (Route Table)
    └── Route: PerimetertoCore → 10.0.0.0/16 via Virtual Appliance @ 10.0.1.7
        associated with: perimeter subnet
```

Non-overlapping address spaces (`10.0.0.0/16` and `172.16.0.0/16`) were used deliberately — **overlapping CIDR ranges make peering impossible**, so address planning is a prerequisite, not an afterthought.

---

## 🛠️ Implementation

### 1. Deploy two VMs in two separate virtual networks

Both VMs were Windows Server 2025 Datacenter (Standard_D2s_v7), with **no public inbound ports** — connectivity testing is done entirely through the Azure control plane (Run Command / Network Watcher), so no RDP exposure to the internet was required.

`CoreServicesVM` was placed in a new `CoreServicesVnet` (`10.0.0.0/16`) with a `Core` subnet (`10.0.0.0/24`):

![CoreServicesVM basics](./screenshots/01-vm-basics-coreservices.png)

![CoreServicesVM networking — CoreServicesVnet/Core](./screenshots/02-vm-networking-corevnet.png)

Deployed and running, it received private IP **10.0.0.4** from the Core subnet:

![CoreServicesVM running](./screenshots/03-coreservicesvm-running.png)

`ManufacturingVM` was placed in a separate `ManufacturingVnet` (`172.16.0.0/16`) with a `Manufacturing` subnet (`172.16.0.0/24`):

![ManufacturingVM networking — ManufacturingVnet/Manufacturing](./screenshots/04-vm-networking-mfgvnet.png)

It received private IP **172.16.0.4** — an entirely different address space, confirming the two VMs live in isolated networks:

![ManufacturingVM running](./screenshots/05-manufacturingvm-running.png)

Both virtual networks, side by side:

![Both VNets](./screenshots/06-both-vnets-created.png)

---

### 2. Prove default isolation with Network Watcher

Before peering, I used **Network Watcher → Connection troubleshoot** to test a TCP connection from `CoreServicesVM` to `ManufacturingVM` on port 3389:

![Connection troubleshoot setup](./screenshots/07-connection-troubleshoot-setup.png)

The result — **Unreachable**, 316 probes sent, 316 failed. This is the expected and correct outcome: resources in different virtual networks cannot communicate by default. Establishing this baseline is what makes the "after" result meaningful.

![Connectivity unreachable](./screenshots/08-connectivity-unreachable.png)

The diagnostic also flagged the NSG deny on both ends and showed the next hop as a **System Route** with no path to the destination network — a precise, layer-by-layer confirmation of *why* the traffic failed, not just *that* it failed.

---

### 3. Establish bidirectional VNet peering

I created a peering link from `CoreServicesVnet`, which provisions **both directions** in a single operation — a local link (`CoreServicesVnet-to-ManufacturingVnet`) and a remote link (`ManufacturingVnet-to-CoreServicesVnet`):

![Add peering configuration](./screenshots/09-add-peering-config.png)

Both peering links reached **Connected / Fully Synchronized** status. Verified from the CoreServicesVnet side:

![CoreServicesVnet peering connected](./screenshots/10-peering-core-connected.png)

…and from the ManufacturingVnet side:

![ManufacturingVnet peering connected](./screenshots/11-peering-mfg-connected.png)

---

### 4. Verify cross-network connectivity

To test at the TCP level, I first enabled the Remote Desktop firewall rule on `CoreServicesVM` via **Run Command** (this returns no output — it silently enables the rule group):

![Enable RDP firewall rule](./screenshots/12-enable-rdp-firewall.png)

Then, from `ManufacturingVM`, I ran `Test-NetConnection` against CoreServicesVM's private IP:

```powershell
Test-NetConnection 10.0.0.4 -port 3389
```

**Result: `TcpTestSucceeded : True`** — with `SourceAddress 172.16.0.4` reaching `RemoteAddress 10.0.0.4` across the peered networks. The exact same test returned `False` before peering; it now succeeds, over Microsoft's backbone, with no public IPs involved.

![Test-NetConnection success](./screenshots/13-testnetconnection-success.png)

---

### 5. Control traffic flow with a user-defined route

To demonstrate traffic steering, I created a route table and a custom route directing traffic bound for the core network through a network virtual appliance (NVA) in a perimeter subnet — the standard pattern for inserting a firewall or inspection appliance into the path.

Created the route table `rt-CoreServices`:

![Create route table](./screenshots/14-create-route-table.png)

Added a user-defined route sending all `10.0.0.0/16` traffic to a **Virtual appliance** next hop at `10.0.1.7`:

![Add UDR route](./screenshots/15-add-udr-route.png)

Then associated the route table with the `perimeter` subnet, so the route applies to traffic originating there:

![Associate perimeter subnet](./screenshots/16-associate-perimeter-subnet.png)

A UDR **overrides Azure's default system routes** — any traffic matching the destination prefix is redirected to the specified next hop instead of following the built-in path. This is the mechanism behind forced tunneling and NVA-based inspection designs.

---

## 🔑 Key Technical Takeaways

- **Isolation is the default.** Two VNets cannot communicate until explicitly connected — the "unreachable" baseline is a feature, not a failure.
- **Peering is non-transitive.** If A peers with B and B peers with C, A still cannot reach C without a direct A–C peering. Each relationship is point-to-point.
- **Peering appears as one network for connectivity**, and traffic between peered VNets stays on Microsoft's backbone — never traversing the public internet.
- **Address planning gates peering.** Overlapping CIDR ranges make two VNets un-peerable, so non-overlapping design must happen up front.
- **System routes vs. user-defined routes.** Azure auto-creates system routes for every subnet; a UDR overrides or supplements them, and is what enables NVA insertion, forced tunneling, and custom traffic inspection.
- **Control-plane testing beats exposing ports.** Network Watcher Connection Troubleshoot and Run Command validate connectivity without opening a single public inbound port — verification without attack surface.

---

## 🧰 Tools & Services

`Microsoft Azure` · `Virtual Network Peering` · `Network Watcher (Connection Troubleshoot)` · `Route Tables / User-Defined Routes` · `Azure Run Command` · `PowerShell (Test-NetConnection)` · `Windows Server 2025`

---

## 💰 Cost Note

Two Windows VMs ran only for the duration of the lab, and the entire `az104-rg5` resource group was deleted immediately after verification — keeping actual spend to a fraction of a dollar despite the default monthly estimate shown in the portal.

---

*Part of my [Azure AZ-104 hands-on lab portfolio](../README.md).*
