---
title: Use On-premises data gateway for Azure Virtual Network datasources | Microsoft Docs
description: Learn how to configure a server to use a gateway for datasources on VNet.
author: minewiskan
manager: kfile
ms.service: analysis-services
ms.topic: conceptual
ms.date: 04/20/2018
ms.author: owend
ms.reviewer: minewiskan

---
# Use gateway for datasources on an Azure Virtual Network (VNet)

This article describes the **AlwaysUseGateway** server property for use when data sources are on an [Azure Virtual Network (VNet)](../virtual-network/virtual-networks-overview.md).

## Server access to VNet datasources

If your data sources are accessed through a VNet, your Azure Analysis Services server must connect to those data sources as if they are on-premises, in your own environment. You can configure the **AlwaysUseGateway** server property to specify the server to access all datasource data through an [On-premises gateway](analysis-services-gateway.md). 

> [!NOTE]
> This property is effective only when an [On-premises data gateway](analysis-services-gateway.md) is installed and configured. The gateway can be on the VNet.

## Configure AlwaysUseGateway property

1. In SSMS > server > **Properties** > **General**, select **Show Advanced (All) Properties**.
2. In the **ASPaaS\AlwaysUseGateway**, select **true**.

    ![Always use gateway property](media/analysis-services-vnet-gateway/aas-ssms-always-property.png)


## See also
[Connecting to on-premises data sources](analysis-services-gateway.md)   
[Install and configure an on-premises data gateway](analysis-services-gateway-install.md)   
[Azure Virtual Network (VNET)](../virtual-network/virtual-networks-overview.md)   

