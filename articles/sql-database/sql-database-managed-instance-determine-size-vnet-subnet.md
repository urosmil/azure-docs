---
title: Managed instance determine VNet/subnet size
description: This topic describes how to calculate the size of the subnet where the Azure SQL Database Managed Instances will be deployed.
services: sql-database
ms.service: sql-database
ms.subservice: managed-instance
ms.custom: seo-lt-2019
ms.devlang: 
ms.topic: conceptual
author: srdan-bozovic-msft
ms.author: srbozovi
ms.reviewer: sstein, bonova, carlrab
ms.date: 02/22/2019
---
# Determine VNet subnet size for Azure SQL Database Managed Instance

Azure SQL Database Managed Instance must be deployed within an Azure [virtual network (VNet)](../virtual-network/virtual-networks-overview.md).

The number of Managed Instances that can be deployed in the subnet of VNet depends on the size of the subnet (subnet range).

When you create a Managed Instance, Azure allocates a number of virtual machines depending on the tier you selected during provisioning. Because these virtual machines are associated with your subnet, they require IP addresses. To ensure high availability during regular operations and service maintenance, Azure may allocate additional virtual machines. As a result, the number of required IP addresses in a subnet is larger than the number of Managed Instances in that subnet.

By design, a Managed Instance needs a minimum of 16 IP addresses in a subnet and may use up to 256 IP addresses. As a result, you can use a subnet masks between /28 and /24 when defining your subnet IP ranges. A network mask bit of /28 (14 hosts per network) is a good size for a single general purpose or business-critical deployment. A mask bit of /27 (30 hosts per network) is ideal for a multiple Managed Instance deployments within the same VNet. Mask bit settings of /26 (62 hosts) and /24 (254 hosts) allows further scaling out of the VNet to support additional Managed Instances.

> [!IMPORTANT]
> A subnet size with 16 IP addresses is the bare minimum with limited potential where a scaling operation like vCore size change is not supported. Choosing subnet with the prefix /27 or longest prefix is highly recommended.

## Determine subnet size

If you plan to deploy multiple Managed Instances inside the subnet and need to optimize on subnet size, use these parameters to form a calculation:

- Azure uses five IP addresses in the subnet for its own needs
- Each managed instance uses number of addreses that depends of pricing tier and hardware generation

| **Pricing tier** | **Hardware generation** | **Used addresses** |
| --- | --- | --- |
| General Purpose | Gen4 | 6 |
| General Purpose | Gen5 | 9 |
| Business Critical | Gen4 | 6 |
| Business Critical | Gen5 | 11 |

**Example 1**: You plan to have three General Purpose (Gen5 hardware) and two Business Critical Managed Instances (Gen5 hardware). That means you need 5 + 3 * 9 + 2 * 11 = 43 IP addresses. As IP ranges are defined in power of 2, you need the IP range of 64 (2^6) IP addresses. Therefore, you need to reserve the subnet with subnet mask of /26.

**Example 2**: You plan to have two General Purpose (Gen4 hardware) and one Business Critical Managed Instances (Gen5 hardware). That means you need 5 + 2 * 6 + 5 + 1 * 11 = 33 IP addresses. As IP ranges are defined in power of 2, you need the IP range of 64 (2^6) IP addresses. Therefore, you need to reserve the subnet with subnet mask of /26.

> [!IMPORTANT]
> Calculation displayed above will become obsolete with further improvements.

## Next steps

- For an overview, see [What is a Managed Instance](sql-database-managed-instance.md).
- Learn more about [Connectivity architecture for Managed Instance](sql-database-managed-instance-connectivity-architecture.md).
- See how to [create VNet where you will deploy Managed Instances](sql-database-managed-instance-create-vnet-subnet.md)
- For DNS issues, see [Configuring a Custom DNS](sql-database-managed-instance-custom-dns.md)
