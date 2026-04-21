# Connect two Azure Virtual Networks using Global Virtual Network Peering  

# Overview

This exercise demonstrates how to connect two Azure Virtual Networks across different regions using **Global Virtual Network Peering**. Unlike regional peering, global peering links VNets across Azure regions without requiring a gateway, enabling low-latency private connectivity over the Microsoft backbone network.

# Learning Objectives

- Create two Virtual Networks in **different Azure regions**
- Configure **Global VNet Peering** in both directions (bidirectional)
- Verify connectivity between resources in peered networks
- Understand peering states: *Initiated → Connected*

---

# Architecture

```mermaid
flowchart TD
    subgraph RG["Resource Group: ContosoResourceGroup"]

        subgraph RegionA["East US2"]
            VNet1["CoreServicesVnet\n10.20.0.0/16\nEast US"]
            Subnet1["SharedServicesSubnet\n10.20.10.0/24"]
            VM1["TestVM1\n10.20.10.4"]
            VNet1 --> Subnet1
            Subnet1 --> VM1
        end

        subgraph RegionB["West Europe"]
            VNet2["ManufacturingVnet\n10.30.0.0/16\nWest Europe"]
            Subnet2["ManufacturingSystemSubnet\n10.30.10.0/24"]
            VM2[" TestVM2\n10.30.10.4"]
            VNet2 --> Subnet2
            Subnet2 --> VM2
        end

        VNet1 -- "Global VNet Peering\n(CoreServicesVnet-to-ManufacturingVnet)" --- VNet2
        VNet2 -- "Global VNet Peering\n(ManufacturingVnet-to-CoreServicesVnet)" --- VNet1

    end

    Internet(("Internet"))
    Internet -.->|"Azure Portal / Cloud Shell"| RG
```

---

# 1. Create a VM to test the configuration

Test VM on the Manufacturing VNet

Azure CLI / PowerShell Commands:

```
PS $RGName = "ContosoResourceGroup"
PS New-AzResourceGroupDeployment -ResourceGroupName $RGName -TemplateFile ManufacturingVMazuredeploy.json -TemplateParameterFile ManufacturingVMazuredeploy.parameters.json

```

<img width="1870" height="76" alt="image" src="https://github.com/user-attachments/assets/353877a1-f1db-420a-9b17-ab6ee9f8b392" />
Figure 1 - VM successfully created

---

# 2. Connect to the test VMs using RDP

<img width="1557" height="590" alt="image" src="https://github.com/user-attachments/assets/432b886f-a640-4285-bb41-441875abb18c" />
Figure 2 - Connected remotely to both VMs

---

# 3. Test the connection between the VMs

Testing the network connection between the two VMs over port 3389 (RDP)

Azure CLI / PowerShell Command:

```
PS Test-NetConnection 10.20.20.4 -port 3389
```

<img width="510" height="114" alt="image" src="https://github.com/user-attachments/assets/3e837465-9ac0-4ced-bde2-b344a4431645" />
Figure 3 - Failed connection test between the VMs

---

# 4. Create VNet peerings between CoreServicesVnet and ManufacturingVnet

CoreServicesVNet to ManufacturingVNet successfully, fully synchronised and connected

<img width="1959" height="319" alt="image" src="https://github.com/user-attachments/assets/20056b0b-2f43-446a-a0c9-4f5f4b4c0ce5" />
Figure 4 - VNet Peerings configured between CoreServicesVnet & ManufacturingVnet

---

# 5. Test connection between the VMs

Azure CLI / PowerShell Command:

```
PS Test-NetConnection 10.20.20.4 -port 3389
```
<img width="543" height="257" alt="image" src="https://github.com/user-attachments/assets/a4787aae-3ef5-465e-b7d3-d8bded235a56" />
Figure 5  - Connection test succeeded

---

# Key Concepts Demonstrated

| Concept | Detail |
|---|---|
| **Global VNet Peering** | Links VNets across different Azure regions |
| **Peering Direction** | Must be configured on **both** VNets (bidirectional) |
| **Address Space** | Must be non-overlapping (`10.20.x.x` vs `10.30.x.x`) |
| **Traffic Path** | Travels over Microsoft backbone — not the public internet |
| **Peering States** | `Initiated` → `Connected` (both sides must connect) |
| **No Gateway Required** | Global peering doesn't need a VPN or ExpressRoute gateway |

---

# Cleanup

>  Remember to delete resources after the exercise to avoid charges.

Azure CLI / PowerShell Command:

```
Remove-AzResourceGroup -Name 'ContosoResourceGroup' -Force -AsJob
```

---

# References

- [Azure VNet Peering — Microsoft Docs](https://learn.microsoft.com/en-us/azure/virtual-network/virtual-network-peering-overview)
- [AZ-700 Learning Path on Microsoft Learn](https://learn.microsoft.com/en-us/training/paths/design-implement-microsoft-azure-networking-solutions-az-700/)
- [Global VNet Peering FAQ](https://learn.microsoft.com/en-us/azure/virtual-network/virtual-networks-faq#what-are-the-constraints-related-to-global-vnet-peering-and-load-balancers)
