---
title: "刪除虛擬網路閘道：PowerShell：Azure 傳統 | Microsoft Docs"
description: "在傳統部署模型中使用 PowerShell 刪除虛擬網路閘道。"
services: vpn-gateway
documentationcenter: na
author: cherylmc
manager: timlt
editor: 
tags: azure-service-management
ms.assetid: 
ms.service: vpn-gateway
ms.devlang: na
ms.topic: 
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 03/20/2017
ms.author: cherylmc
translationtype: Human Translation
ms.sourcegitcommit: 0d8472cb3b0d891d2b184621d62830d1ccd5e2e7
ms.openlocfilehash: b5d90aa60de4a24b6d76414204ceae24670e48a3
ms.lasthandoff: 03/21/2017


---
# <a name="delete-a-virtual-network-gateway-using-powershell-classic"></a>使用 PowerShell (傳統) 刪除虛擬網路閘道
> [!div class="op_single_selector"]
> * [Resource Manager - PowerShell](vpn-gateway-delete-vnet-gateway-powershell.md)
> * [傳統 - PowerShell](vpn-gateway-delete-vnet-gateway-classic-powershell.md)
>
>

您可以使用 PowerShell 刪除傳統部署模型中的 VPN 閘道。 在刪除虛擬網路閘道之後，請修改網路設定檔，以移除您不再使用的項目。

##<a name="part-1-connect-to-azure"></a>第 1 部分。 連接到 Azure

### <a name="1-install-the-latest-powershell-cmdlets"></a>1.安裝最新的 PowerShell Cmdlet。

下載並安裝最新版的 Azure 服務管理 (SM) PowerShell Cmdlet。 如需詳細資訊，請參閱 [如何安裝和設定 Azure PowerShell](/powershell/azureps-cmdlets-docs)。

### <a name="2-connect-to-your-azure-account"></a>2.連線至您的 Azure 帳戶。 

以提高的權限開啟 PowerShell 主控台並連接到您的帳戶。 使用下列範例來協助您連接：

    Login-AzureRmAccount

檢查帳戶的訂用帳戶。

    Get-AzureRmSubscription

如果您有多個訂用帳戶，請選取您要使用的訂用帳戶。

    Select-AzureRmSubscription -SubscriptionName "Replace_with_your_subscription_name"

接下來，使用下列 Cmdlet，將您的 Azure 訂用帳戶新增到 PowerShell，以供傳統部署模型使用。

    Add-AzureAccount

## <a name="part-2-export-and-view-the-network-configuration-file"></a>第 2 部分。 匯出並檢視網路組態檔

在您的電腦上建立目錄，然後將網路組態檔匯出到該目錄。 您可以使用此檔案來檢視目前的組態資訊，也可以修改網路組態。

在此範例中，會將網路組態檔匯出到 C:\AzureNet。

     Get-AzureVNetConfig -ExportToFile C:\AzureNet\NetworkConfig.xml

使用文字編輯器開啟檔案，並檢視傳統 VNet 的名稱。 當您在 Azure 入口網站中建立 VNet 時，不會在 Azure 入口網站中顯示 Azure 所使用的完整名稱。 例如，在 Azure 入口網站中名稱顯示為 'ClassicVNet1' 的 VNet，在網路組態檔中的名稱可能更長。 名稱可能如下︰'Group ClassicRG1 ClassicVNet1'。 VNet 名稱會列為 'VirtualNetworkSite name ='。<br>執行 PowerShell Cmdlet 時，請使用網路組態檔中的名稱。

## <a name="part-3-delete-the-virtual-network-gateway"></a>第 3 部分。 刪除虛擬網路閘道

當您刪除虛擬網路閘道時，不會中斷連接所有透過閘道的 VNet 連接。 如果您的 P2S 用戶端連接到 VNet，則會將它們中斷連接而不發出警告。

這個範例會移除虛擬網路閘道。 當您執行此範例時，請使用網路組態檔中的虛擬網路完整名稱。

    Remove-AzureVNetGateway -VNetName "Group ClassicRG1 ClassicVNet1"

如果成功，傳回便會顯示：

    Status : Successful

## <a name="part-4-modify-the-network-configuration-file"></a>第 4 部分。 修改網路組態檔

當您刪除虛擬網路閘道時，Cmdlet 不會修改網路設定檔。 您需要修改檔案，以移除不再使用的項目。 下列各節可協助修改您所下載的網路組態檔。

###<a name="local-network-site-references"></a>區域網路網站參考

若要移除網站參考資訊，請對 ConnectionsToLocalNetwork/LocalNetworkSiteRef 進行組態變更。 移除區域網站參考會觸發 Azure 刪除通道。 根據您所建立的組態而定，您可能並未列出 LocalNetworkSiteRef。

    <Gateway>
       <ConnectionsToLocalNetwork>
         <LocalNetworkSiteRef name="D1BFC9CB_Site2">
           <Connection type="IPsec" />
         </LocalNetworkSiteRef>
       </ConnectionsToLocalNetwork>
    </Gateway>

範例：

    <Gateway>
       <ConnectionsToLocalNetwork>
       </ConnectionsToLocalNetwork>
     </Gateway>

###<a name="local-network-sites"></a>區域網路網站

移除所有您不再使用的區域網站。 根據您所建立的組態而定，您可能並未列出區域網路網站。

    <LocalNetworkSites>
      <LocalNetworkSite name="Site1">
        <AddressSpace>
          <AddressPrefix>192.168.0.0/16</AddressPrefix>
        </AddressSpace>
        <VPNGatewayAddress>5.4.3.2</VPNGatewayAddress>
      </LocalNetworkSite>
      <LocalNetworkSite name="Site3">
        <AddressSpace>
          <AddressPrefix>192.168.0.0/16</AddressPrefix>
        </AddressSpace>
        <VPNGatewayAddress>57.179.18.164</VPNGatewayAddress>
      </LocalNetworkSite>
    </LocalNetworkSites>

在此範例中，我們只會移除 Site3。

    <LocalNetworkSites>
        <LocalNetworkSite name="Site1">
        <AddressSpace>
          <AddressPrefix>192.168.0.0/16</AddressPrefix>
        </AddressSpace>
        <VPNGatewayAddress>5.4.3.2</VPNGatewayAddress>
      </LocalNetworkSite>
    </LocalNetworkSites>

### <a name="client-addresspool"></a>用戶端 AddressPool

如果您擁有連至 VNet 的 P2S 連接，就必須具備 VPNClientAddressPool。 移除對應至您已刪除之虛擬網路閘道的用戶端位址集區。

    <Gateway>
       <VPNClientAddressPool>
         <AddressPrefix>10.1.0.0/24</AddressPrefix>
       </VPNClientAddressPool>
       <ConnectionsToLocalNetwork />
    </Gateway>

範例：

     <Gateway>
       <ConnectionsToLocalNetwork />
     </Gateway>

### <a name="gatewaysubnet"></a>GatewaySubnet

刪除對應至 VNet 的 GatewaySubnet。

    <Subnets>
       <Subnet name="FrontEnd">
         <AddressPrefix>10.11.0.0/24</AddressPrefix>
       </Subnet>
       <Subnet name="GatewaySubnet">
         <AddressPrefix>10.11.1.0/29</AddressPrefix>
       </Subnet>
     </Subnets>

範例：

    <Subnets>
       <Subnet name="FrontEnd">
         <AddressPrefix>10.11.0.0/24</AddressPrefix>
       </Subnet>
     </Subnets>

## <a name="part-5-upload-the-network-configuration-file"></a>第 5 部分。 上傳網路組態檔

儲存您的變更並將網路組態檔上傳至 Azure。 確定您會視需要變更環境的檔案路徑。

     Set-AzureVNetConfig -ConfigurationPath C:\AzureNet\NetworkConfig.xml

如果成功，傳回就會顯示類似此範例的內容：

     OperationDescription        OperationId                      OperationStatus                                                
     --------------------        -----------                      ---------------                                                
     Set-AzureVNetConfig        e0ee6e66-9167-cfa7-a746-7casb9    Succeeded
