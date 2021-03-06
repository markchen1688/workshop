---
title: "存取報告 - Azure RBAC | Microsoft Docs"
description: "產生一份報告，其中列出您的 Azure 訂用帳戶 (採用角色型存取控制) 在過去 90 天內的所有存取權變更。"
services: active-directory
documentationcenter: 
author: kgremban
manager: femila
editor: 
ms.assetid: 2bc68595-145e-4de3-8b71-3a21890d13d9
ms.service: active-directory
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 02/27/2017
ms.author: kgremban
ms.custom: H1Hack27Feb2017
translationtype: Human Translation
ms.sourcegitcommit: 015cc28903bfd366c653a51b0f73512bf8b578ea
ms.openlocfilehash: 433dc731c342924d962e2f08e392556558a0168d
ms.lasthandoff: 02/28/2017

---
# <a name="create-an-access-report-for-role-based-access-control"></a>建立角色型存取控制的存取報告
每當有人授與或撤銷您訂用帳戶中的存取權時，變更就會記錄在 Azure 事件中。 您可以建立存取權變更歷程記錄報告，以查看過去 90 天內的所有變更。

## <a name="create-a-report-with-azure-powershell"></a>使用 Azure PowerShell 建立報告
若要在 PowerShell 中建立存取權變更歷程記錄報告，請使用 `Get-AzureRMAuthorizationChangeLog` 命令。 [PowerShell 資源庫](https://www.powershellgallery.com/packages/AzureRM.Storage/1.0.6/Content/ResourceManagerStartup.ps1)中有提供關於此 Cmdlet 的更多詳細資料。

呼叫此命令時，您可以指定要列出哪一個指派屬性，其中包括下列屬性︰

| 屬性 | 說明 |
| --- | --- |
| **Action** |已授與或已撤銷存取權 |
| **Caller** |負責存取權變更的擁有者 |
| **Date** |變更存取權的日期和時間 |
| **DirectoryName** |Azure Active Directory 目錄 |
| **PrincipalName** |使用者、群組或應用程式的名稱 |
| **PrincipalType** |指派對象為使用者、群組或應用程式 |
| **RoleId** |已授與或已撤銷之角色的 GUID |
| **RoleName** |已授與或已撤銷的角色 |
| **ScopeName** |訂用帳戶、資源群組或資源的名稱 |
| **ScopeType** |指派的範圍是訂用帳戶、資源群組或資源 |
| **SubscriptionId** |Azure 訂用帳戶的 GUID |
| **SubscriptionName** |Azure 訂用帳戶的名稱 |

此範例命令會列出過去&7; 天訂用帳戶中的所有存取權變更：

```
Get-AzureRMAuthorizationChangeLog -StartTime ([DateTime]::Now - [TimeSpan]::FromDays(7)) | FT Caller,Action,RoleName,PrincipalType,PrincipalName,ScopeType,ScopeName
```

![PowerShell Get-AzureRMAuthorizationChangeLog - 螢幕擷取畫面](./media/role-based-access-control-configure/access-change-history.png)

## <a name="create-a-report-with-azure-cli"></a>使用 Azure CLI 建立報告
若要在 Azure 命令列介面 (CLI) 中建立存取權變更歷程記錄報告，請使用 `azure role assignment changelog list` 命令。

## <a name="export-to-a-spreadsheet"></a>匯出為試算表
若要儲存報告或處理此資料，請將存取權變更匯出為 .csv 檔案。 您即可在試算表中檢閱此報告。

![以試算表形式來檢視的變更記錄 - 螢幕擷取畫面](./media/role-based-access-control-configure/change-history-spreadsheet.png)

## <a name="next-steps"></a>後續步驟
* 使用 [Azure RBAC 中的自訂角色](role-based-access-control-custom-roles.md)
* 了解如何[使用 PowerShell 管理 Azure RBAC](role-based-access-control-manage-access-powershell.md)


