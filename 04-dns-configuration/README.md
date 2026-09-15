# Lab 04 — Azure DNS: Public Delegation & Private Resolution

## 🎯 Objective

Configure both a public-facing DNS zone (with real name-server delegation and record verification) and a private DNS zone scoped to an internal virtual network, to demonstrate the two distinct name-resolution models Azure DNS supports.

---

## 🌐 Public DNS Zone

| Setting | Value |
|---------|-------|
| Zone | `istiaktech.com` |
| Resource Group | `az104-rg4` |
| Record created | `www` (A record) → `10.1.1.4` |

![Create DNS Zone](./screenshots/01-create-dns-zone.png)

Azure automatically assigned four name servers to the zone on creation — this is the delegation set that a domain registrar would need to point to in order for the zone to actually serve public DNS traffic for the domain:

![DNS Zone Overview - Name Servers](./screenshots/02-dns-zone-overview-nameservers.png)

```
ns1-09.azure-dns.com.
ns2-09.azure-dns.net.
ns3-09.azure-dns.org.
ns4-09.azure-dns.info.
```

![Add A Record - www](./screenshots/03-add-a-record-www.png)

### Verification

Resolution was verified using `nslookup` directly against one of the Azure-assigned name servers:

![nslookup - Success](./screenshots/05-nslookup-success.png)

```
nslookup www.istiaktech.com ns1-09.azure-dns.com
Server:   ns1-09.azure-dns.com
Address:  13.107.236.9#53

Name:     www.istiaktech.com
Address:  10.1.1.4
```

For comparison, querying the same name server for a domain it doesn't host returns `REFUSED` rather than an answer — confirming that a name server responds authoritatively only for zones it actually serves:

![nslookup - Wrong Domain](./screenshots/04-nslookup-failed-wrong-domain.png)

```
nslookup www.contosoxyz104.com ns1-09.azure-dns.com
** server can't find www.contosoxyz104.com: REFUSED
```

---

## 🔒 Private DNS Zone

| Setting | Value |
|---------|-------|
| Zone | `private.istiaktech.com` |
| Linked VNet | `ManufacturingVnet` (link name: `manufacturing-link`) |
| Record created | `sensorvm` (A record) → `10.1.1.4` |

![Private DNS Zone Overview](./screenshots/06-private-dns-zone-overview.png)

Note what's *absent* from this overview compared to the public zone: no name servers are listed. A private zone is never delegated on the public internet — it's only resolvable from VNets it's explicitly linked to.

![Private DNS Virtual Network Link](./screenshots/07-private-dns-vnet-link.png)

The link to `ManufacturingVnet` shows status **Completed**, with auto-registration left disabled (records were added manually for this lab rather than relying on VM auto-registration).

![Private DNS Add Record](./screenshots/08-private-dns-add-record.png)

---

## 🧠 Key Concepts Demonstrated

- **Domain delegation model** — a public Azure DNS zone is only authoritative for a domain once a registrar's NS records point to Azure's assigned name servers; Azure hosts the zone, it does not sell or register the domain itself
- **Record management** — creating A records in both zone types and understanding TTL as a caching duration, not a resolution mechanism
- **Zone-type isolation** — private zones expose no public name servers and are unreachable from the internet by design; resolution is scoped entirely to linked VNets
- **Virtual network links** — the mechanism that makes a private zone resolvable from inside a specific VNet, with optional auto-registration for VM lifecycle-driven record management
- **DNS response codes** — distinguishing a `REFUSED` response (server reached, zone not hosted there) from a timeout or `NXDOMAIN`, which point to different underlying problems

---

## 💡 What I Learned

DNS response codes carry more specific meaning than a simple pass/fail. A `REFUSED` response means the name server was reached and responded, but explicitly declined to answer for that zone — a different signal than a timeout (server unreachable) or `NXDOMAIN` (zone exists but the record doesn't). Understanding that distinction is what actually matters when triaging DNS issues, since the response code itself points to where to look next.
