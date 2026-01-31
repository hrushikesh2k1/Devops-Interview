# Azure VPN Gateway & ExpressRoute – AG104 Notes

## What is Azure VPN Gateway?

Azure VPN Gateway is a cloud networking service that allows secure communication between:
- On-premises networks
- Remote users
- Azure Virtual Networks (VNets)

It uses encrypted tunnels (IPsec/IKE) over the public internet to provide private connectivity to Azure.

In simple words:
Azure VPN Gateway allows your company’s office, laptops, or other Azure networks to connect securely to Azure.

---

## Why VPN Gateway is needed

Without VPN Gateway:
Azure VNets are isolated and cannot be accessed privately.

With VPN Gateway:
- Office network can access Azure
- Work-from-home users can access Azure
- Other Azure VNets can communicate securely

All traffic is encrypted and secure.

---

## Types of Azure VPN Gateway

### 1. Site-to-Site (S2S)
Connects an on-premises office network to an Azure VNet.

Example:
Company office firewall connects to Azure through VPN.

Used when:
You want the entire office network connected to Azure.

---

### 2. Point-to-Site (P2S)
Connects individual users (laptops) to Azure.

Example:
A developer working from home connects his laptop to Azure VNet using a VPN client.

Used when:
Remote users need secure access to Azure.

---

### 3. VNet-to-VNet
Connects one Azure VNet to another Azure VNet.

Example:
Production VNet in India connects to DR VNet in USA.

Used when:
Azure networks need secure communication between regions.

---

## What is Azure ExpressRoute?

Azure ExpressRoute provides a **private dedicated connection** between your on-premises network and Azure.

Unlike VPN Gateway, ExpressRoute does NOT use the public internet.

Traffic goes through:
Office → ISP → Private Fiber → Microsoft Network → Azure

This provides:
- Very high speed
- Very low latency
- Enterprise-grade reliability

---

## VPN Gateway vs ExpressRoute

| Feature | VPN Gateway | ExpressRoute |
|--------|-------------|---------------|
| Uses public internet | Yes | No |
| Encryption | Yes | Not required |
| Connection type | Encrypted tunnel | Private fiber link |
| Speed | Up to ~3 Gbps | Up to 100 Gbps |
| Reliability | Internet-based | Carrier-grade SLA |
| Cost | Low | Very high |
| Setup | Simple | Requires ISP |

---

## When to use VPN Gateway

Use VPN Gateway when:
- You want a low-cost solution
- You need secure connectivity
- Traffic volume is moderate
- Small or medium company

---

## When to use ExpressRoute

Use ExpressRoute when:
- High performance is required
- Large enterprise workloads
- Banking, healthcare, finance
- Compliance and SLA are required

---

## Exam One-Line Answers

Azure VPN Gateway:
Provides encrypted IPsec tunnels between on-premises networks, users, and Azure VNets over the internet.

Azure ExpressRoute:
Provides a private, high-speed, dedicated connection from on-premises networks directly into Microsoft’s Azure network.

---

End of Notes
