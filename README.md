# Secure Enterprise Hub-and-Spoke Network Architecture

**Date:** January 2026

## Aim of the Project
The goal of this project was to demonstrate a secure, scalable Azure network design using Microsoft’s recommended hub-and-spoke pattern. Specifically, I set out to:
* **Centralize security:** Ensure a single Firewall inspects all traffic and Bastion provides safe management access.
* **Isolate workloads:** Ensure Dev and Prod environments cannot communicate directly.
* **Control internet access:** Block unwanted sites at the application level.
* **Provide secure remote access:** Enable RDP management without using risky Public IPs.

## Executive Summary
In modern cloud environments, a "flat" network is a major security risk—if one server is compromised, the entire network is vulnerable. This architecture separates "Development" and "Production" into isolated spokes, ensuring traffic only flows through a central "Security Hub." This provides a single point of control for inspecting traffic and managing the environment securely.



## Key Technical Features
| Feature | Benefit |
| :--- | :--- |
| **Hub-and-Spoke Topology** | Centralizes security tools and reduces costs by sharing resources. |
| **Azure Firewall** | Acts as a "Gatekeeper," inspecting all traffic and blocking unauthorized sites. |
| **Azure Bastion** | Provides "Private" remote access; no public IP addresses are required on VMs. |
| **User-Defined Routing (UDR)** | Forces all traffic through the Firewall, ensuring no "backdoors" to the internet. |

---

## Implementation Phases

### Phase 1: Hub Infrastructure
I began by creating the central Hub network and the resource group to house all assets.
![Resource Group Created](phase1-resource-group-created.png)
![Hub VNet Overview](phase1-hub-vnet-overview.png)
![Hub Subnets (Firewall, Bastion, Gateway)](phase1-hub-vnet-subnets-list.png)

### Phase 2: Spoke VNets (Dev & Prod)
I deployed two separate VNets with non-overlapping address spaces to simulate environment isolation.
![Spoke Dev Overview](phase2-spoke-dev-vnet-overview.png)
![Spoke Dev Workload Subnet](phase2-spoke-dev-subnet-workload.png)
![Spoke Prod Overview](phase2-spoke-prod-vnet-overview.png)
![Spoke Prod Workload Subnet](phase2-spoke-prod-subnet-workload.png)

### Phase 3: VNet Peering
I connected the Spokes to the Hub. No direct peering exists between the spokes, forcing a Hub-and-Spoke traffic pattern.
![Hub Peerings Connected](phase3-hub-peerings-connected.png)
![Spoke Dev Peering](phase3-spoke-dev-peerings-connected.png)
![Spoke Prod Peering](phase3-spoke-prod-peerings-connected.png)

### Phase 4: Security Controls & Routing
This phase involved locking down the network using Azure Firewall, NSGs, and custom Route Tables (UDR).
![Azure Firewall Running](phase4-azure-firewall-overview-running.png)
![Firewall Application Rules](phase4-firewall-application-rules.png)
![NSG Overview](phase4-nsg-overview.png)
![NSG Inbound Rules](phase4-nsg-inbound-rules.png)
![NSG Outbound Rules](phase4-nsg-outbound-rules.png)
![Route Table Overview](phase4-route-table-overview.png)
![UDR 0.0.0.0/0 to Firewall](phase4-route-table-routes-0-0-0-0-firewall.png)
![Route Table Associated to Subnets](phase4-route-table-subnets-associated.png)
![Spoke Dev Subnet Associated](phase4-spoke-dev-subnet-nsg-associated.png)
![Spoke Prod Subnet Associated](phase4-spoke-prod-subnet-nsg-associated.png)

### Phase 5: Validation & Testing
I verified the architecture worked as intended through connectivity and security tests.

**Secure Access:**
![Bastion RDP Session](phase5-bastion-rdp-session-open.png)

**Egress Firewall Testing (Browser):**
![Microsoft.com Success](phase5-firewall-test-browser-microsoft-success.png)
![Facebook.com Blocked](phase5-firewall-test-browser-facebook-success.png)

**Network Command Line Validation:**
![Test-NetConnection Microsoft True](phase5-powershell-test-netconnection-microsoft-true.png)
![Test-NetConnection Facebook](phase5-powershell-test-netconnection-facebook-false.png)
![Invoke-WebRequest Microsoft](phase5-powershell-invoke-webrequest-microsoft-error.png)
![Invoke-WebRequest Facebook Fail](phase5-powershell-invoke-webrequest-facebook-fail.png)

---

## Technical Challenges Faced
* **The "Invisible" Internet Path:** Initially, VMs tried to bypass the firewall using default routes. I solved this by implementing **User-Defined Routes (UDRs)** to override Azure's default routing logic.
* **Secure Management:** Eliminated the risk of "Brute Force" attacks on RDP by removing all Public IPs and deploying **Azure Bastion**, ensuring all management happens over a secure SSL tunnel.

---

**Note:** All resources were deployed within a single resource group (**Networking-Lab-RG**) and deleted after validation to ensure cost-efficiency.
