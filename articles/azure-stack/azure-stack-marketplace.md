---
title: Publish a custom marketplace item in Azure Stack (service administrator) | Microsoft Docs
description: As a service administrator, learn how to publish a custom marketplace item in Azure Stack.
services: azure-stack
documentationcenter: 
author: rupisure
manager: byronr
editor: 
ms.assetid: 60871cbb-eed2-433c-a76d-d605c7aec06c
ms.service: azure-stack
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 09/26/2016
ms.author: rupisure
translationtype: Human Translation
ms.sourcegitcommit: 2ea002938d69ad34aff421fa0eb753e449724a8f
ms.openlocfilehash: f19d6fcabaadc0ba970c2143e32da3452d6fb947


---
# <a name="the-azure-stack-marketplace"></a>The Azure Stack Marketplace
The Marketplace is a collection of items customized for Azure Stack, like services, applications, and resources. It's the place where tenants come to create new resources and deploy new applications. Service administrators can add custom items to the Marketplace and tenants will see them right away.

To open the Marketplace, click **New**.

![](media/azure-stack-publish-custom-marketplace-item/image1.png)

The Marketplace is updated every five minutes.

## <a name="marketplace-items"></a>Marketplace items
Every Marketplace item has:

* An Azure Resource Manager template for resource provisioning
* Metadata, like strings, icons, and other marketing collateral
* Formatting information to display the item in the portal

Every item published to the Marketplace uses a format called the Azure Gallery Package (azpkg). Deployment or runtime resources (like code, zip files with software, or virtual machine images) should be added to Azure Stack separately, not as part of the Marketplace Item. 

## <a name="next-steps"></a>Next steps
[Create and publish a marketplace item](azure-stack-create-and-publish-marketplace-item.md)




<!--HONumber=Nov16_HO3-->


