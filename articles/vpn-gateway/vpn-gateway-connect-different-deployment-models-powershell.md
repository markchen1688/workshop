---
title: "將傳統虛擬網路連接到 Azure Resource Manager VNet：PowerShell | Microsoft Docs"
description: "了解如何使用 VPN 閘道和 PowerShell 在傳統 VNet 和 Resource Manager VNet 之間建立 VPN 連線"
services: vpn-gateway
documentationcenter: na
author: cherylmc
manager: timlt
editor: 
tags: azure-service-management,azure-resource-manager
ms.assetid: f17c3bf0-5cc9-4629-9928-1b72d0c9340b
ms.service: vpn-gateway
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 01/12/2017
ms.author: cherylmc
translationtype: Human Translation
ms.sourcegitcommit: 10985eaebd9148ef4fbd22eac0ba50076ca71ef5
ms.openlocfilehash: 430d2c02537e6907da09926db52a3d07b975bb64


---
# <a name="connect-virtual-networks-from-different-deployment-models-using-powershell"></a>使用 PowerShell 從不同的部署模型連接虛擬網路
> [!div class="op_single_selector"]
> * [入口網站](vpn-gateway-connect-different-deployment-models-portal.md)
> * [PowerShell](vpn-gateway-connect-different-deployment-models-powershell.md)
> 
> 

Azure 目前有兩種管理模型：傳統和 Resource Manager (RM)。 如果您已使用 Azure 一段時間，則可能具有傳統 VNet 上執行的 Azure VM 和執行個體角色。 較新的 VM 和角色執行個體可能會在 Resource Manager 中建立的 VNet 中執行。 本文將逐步引導您將傳統 Vnet 連接到 Resource Manager Vnet，以允許位於不同部署模型中的資源透過閘道連線彼此通訊。

您可以在不同訂用帳戶和不同區域中的 VNet 之間建立連線。 只要設定的閘道是動態或路由式，您也可以連接已連線到內部部署網路的 Vnet。 如需 VNet 對 VNet 連線的詳細資訊，請參閱本文結尾處的 [VNet 對 VNet 常見問題集](#faq) 。

### <a name="deployment-models-and-methods-for-connecting-vnets-in-different-deployment-models"></a>連接不同的部署模型中的 VNet 所用的部署模型和方法
[!INCLUDE [vpn-gateway-clasic-rm](../../includes/vpn-gateway-classic-rm-include.md)]

當此組態有新文章和額外工具可以使用時，我們就會更新下表。 當文章可用時，我們會直接從資料表連結至該文章。<br><br>

[!INCLUDE [vpn-gateway-table-vnet-vnet](../../includes/vpn-gateway-table-vnet-to-vnet-include.md)]

**VNet 對等互連**

[!INCLUDE [vpn-gateway-vnetpeeringlink](../../includes/vpn-gateway-vnetpeeringlink-include.md)]

## <a name="before-beginning"></a>開始之前
下列步驟引導您完成為每個 VNet 設定動態或路由式閘道，以及建立閘道之間的 VPN 連線所需的設定。 此組態不支援靜態或路由式閘道。

### <a name="prerequisites"></a>必要條件
* 已建立兩個 Vnet。
* Vnet 的位址範圍不會彼此重疊，或與閘道可能連接的任何其他連線範圍重疊。
* 您已安裝最新的 PowerShell Cmdlet (1.0.2 或更新版本)。 如需詳細資訊，請參閱 [如何安裝和設定 Azure PowerShell](/powershell/azureps-cmdlets-docs) 。 確定安裝服務管理 (SM) 和 Resource Manager (RM) Cmdlet。 

### <a name="a-nameexamplerefaexample-settings"></a><a name="exampleref"></a>設定範例
您可以使用範例設定為參考。

**傳統 VNet 設定**

VNet 名稱 = ClassicVNet  <br>
位置 = West US <br>
虛擬網路位址空間 = 10.0.0.0/24 <br>
子網路 1 = 10.0.0.0/27 <br>
GatewaySubnet = 10.0.0.32/29 <br>
區域網路名稱 = RMVNetLocal <br>
GatewayType = DynamicRouting

**Resource Manager VNet 設定**

VNet 名稱 = RMVNet  <br>
資源群組 = RG1 <br>
虛擬網路的 IP 位址空間 = 192.168.0.0/16 <br>
子網路 1 = 192.168.1.0/24 <br>
GatewaySubnet = 192.168.0.0/26 <br>
位置 = 美國東部 <br>
閘道公用 IP 名稱 = gwpip <br>
區域網路閘道 = ClassicVNetLocal <br>
虛擬網路閘道名稱 = RMGateway <br>
閘道 IP 位址組態 = gwipconfig

## <a name="a-namecreatesmgwasection-1---configure-the-classic-vnet"></a><a name="createsmgw"></a>區段 1 - 設定傳統 VNet
### <a name="part-1---download-your-network-configuration-file"></a>第 1 部份 - 下載您的網路組態檔
1. 在 PowerShell 主控台中以提高的權限登入您的 Azure 帳戶。 下列 Cmdlet 會提示您輸入 Azure 帳戶的登入認證。 登入之後，它會下載您的帳戶設定以供 Azure PowerShell 使用。 您將使用 SM PowerShell Cmdlet 才能完成這個部分的設定。
   
        Add-AzureAccount
2. 執行下列命令以匯出 Azure 網路組態檔。 您可以視需要變更此檔案的位置，以匯出至不同的位置。 您將編輯此檔案，然後將它匯入至 Azure。
   
        Get-AzureVNetConfig -ExportToFile C:\AzureNet\NetworkConfig.xml
3. 開啟您下載的 .xml 檔案加以編輯。 如需網路組態檔的範例，請參閱 [網路組態結構描述](https://msdn.microsoft.com/library/jj157100.aspx)。

### <a name="part-2--verify-the-gateway-subnet"></a>第 2 部份 - 驗證閘道子網路
在 **VirtualNetworkSites** 元素中，將閘道子網路 (若已建立) 加入至您的 VNet。 使用網路組態檔時，閘道子網路必須命名為 "GatewaySubnet"，否則 Azure 無法辨識並將它當作閘道子網路。

[!INCLUDE [vpn-gateway-no-nsg-include](../../includes/vpn-gateway-no-nsg-include.md)]

**範例：**

    <VirtualNetworkSites>
      <VirtualNetworkSite name="ClassicVNet" Location="West US">
        <AddressSpace>
          <AddressPrefix>10.0.0.0/24</AddressPrefix>
        </AddressSpace>
        <Subnets>
          <Subnet name="Subnet-1">
            <AddressPrefix>10.0.0.0/27</AddressPrefix>
          </Subnet>
          <Subnet name="GatewaySubnet">
            <AddressPrefix>10.0.0.32/29</AddressPrefix>
          </Subnet>
        </Subnets>
      </VirtualNetworkSite>
    </VirtualNetworkSites>

### <a name="part-3---add-the-local-network-site"></a>步驟 3 - 新增區域網路站台
您新增的區域網路站台代表您要連接的 RM VNet。 您必須將 **LocalNetworkSites** 元素 (如果尚未存在) 加入至檔案。 此時在組態中，VPNGatewayAddress 可以是任何有效的公用 IP 位址，因為我們尚未建立 Resource Manager VNet 的閘道。 一旦建立閘道，我們會以指派給 RM 閘道的正確公用 IP 位址取代此預留位置 IP 位址。

    <LocalNetworkSites>
      <LocalNetworkSite name="RMVNetLocal">
        <AddressSpace>
          <AddressPrefix>192.168.0.0/16</AddressPrefix>
        </AddressSpace>
        <VPNGatewayAddress>13.68.210.16</VPNGatewayAddress>
      </LocalNetworkSite>
    </LocalNetworkSites>

### <a name="part-4---associate-the-vnet-with-the-local-network-site"></a>第 4 部份 – 建立 VNet 與區域網路站台的關聯
在此區段中，我們會指定您要 VNet 連接的區域網路站台。 在此例中，這是您稍早參考的 Resource Manager VNet。 確定名稱相符。 此步驟不會建立閘道。 它會指定閘道將要連接的區域網路。

        <Gateway>
          <ConnectionsToLocalNetwork>
            <LocalNetworkSiteRef name="RMVNetLocal">
              <Connection type="IPsec" />
            </LocalNetworkSiteRef>
          </ConnectionsToLocalNetwork>
        </Gateway>

### <a name="part-5---save-the-file-and-upload"></a>第 5 部份 - 儲存檔案並上傳
儲存檔案，然後執行下列命令，將它匯入至 Azure。 確定您會視需要變更環境的檔案路徑。

        Set-AzureVNetConfig -ConfigurationPath C:\AzureNet\NetworkConfig.xml

您應會看到類似以下的結果顯示匯入成功。

        OperationDescription        OperationId                      OperationStatus                                                
        --------------------        -----------                      ---------------                                                
        Set-AzureVNetConfig        e0ee6e66-9167-cfa7-a746-7casb9    Succeeded 

### <a name="part-6---create-the-gateway"></a>第 6 部分 - 建立閘道
您可以使用傳統入口網站或 PowerShell 來建立 VNet 閘道。

執行此範例之前，請參閱您針對 Azure 預期看到的確切名稱所下載的網路組態檔。 網路組態檔包含傳統虛擬網路的值。 因為部署模型中的差異，當在 Azure 入口網站中建立傳統 VNet 設定時，有時候傳統 VNet 的名稱會變更。 例如，如果您使用 Azure 入口網站在名稱為 'ClassicRG' 的資源群組中，建立一個名稱為 'Classic VNet' 的傳統 VNet，網路組態檔中所包含的名稱會轉換為 'Group ClassicRG Classic VNet'。 當指定包含空格的 VNet 名稱時，請使用引號括住值。


使用以下範例來建立動態路由閘道︰

    New-AzureVNetGateway -VNetName ClassicVNet -GatewayType DynamicRouting

您可以使用 `Get-AzureVNetGateway` Cmdlet 檢查閘道的狀態。

## <a name="a-namecreatermgwasection-2-configure-the-rm-vnet-gateway"></a><a name="creatermgw"></a>區段 2 - 設定 RM VNet 閘道
若要建立 ARM VNet 的 VPN 閘道，請遵循下列指示。 直到您已擷取傳統 VNet 閘道的公用 IP 位址以後，才可開始下列步驟。 

1. **登入您的 Azure 帳戶** 。 下列 Cmdlet 會提示您輸入 Azure 帳戶的登入認證。 登入之後，便會下載您的帳戶設定，以供 Azure PowerShell 使用。
   
        Login-AzureRmAccount 
   
     如果您有多個訂用帳戶，請取得 Azure 訂用帳戶的清單。
   
        Get-AzureRmSubscription
   
    指定您要使用的訂用帳戶。 
   
        Select-AzureRmSubscription -SubscriptionName "Name of subscription"
2. **建立區域網路閘道**。 在虛擬網路中，區域網路閘道通常是指您的內部部署位置。 在此例中，區域網路閘道會參考您的傳統 VNet。 賦予它一個可供 Azure 參考的名稱，並且指定位址空間前置詞。 Azure 會使用您指定的 IP 位址前置詞來識別要傳送至內部部署位置的流量。 如果您稍後需要先在此調整資訊，再建立您的閘道，您可以修改下列值並再次執行範例。<br><br>
   
   * `-Name` 是您要指派給區域網路閘道的參考名稱。<br>
   * `-AddressPrefix` 是傳統 VNet 的位址空間。<br>
   * `-GatewayIpAddress` 是傳統 VNet 閘道的公用 IP 位址。 請務必變更下列範例，以反映正確的 IP 位址。
     
           New-AzureRmLocalNetworkGateway -Name ClassicVNetLocal `
           -Location "West US" -AddressPrefix "10.0.0.0/24" `
           -GatewayIpAddress "n.n.n.n" -ResourceGroupName RG1
3. **要求將公用 IP 位址**配置到 Resource Manager VNet 的虛擬網路閘道。 您無法指定想要使用的 IP 位址。 IP 位址是動態地配置到虛擬網路閘道。 但是，這不代表 IP 位址會變更。 只有在刪除及重新建立閘道時，虛擬網路閘道 IP 位址才會變更。 它不會因為重新調整、重設或閘道的其他內部維護/升級而變更。<br>在此步驟中，我們也會設定用於後續步驟中的變數。

        $ipaddress = New-AzureRmPublicIpAddress -Name gwpip `
        -ResourceGroupName RG1 -Location 'EastUS' `
        -AllocationMethod Dynamic

1. **確認您的虛擬網路有閘道子網路**。 如果閘道器子網路不存在，請新增一個。 確定閘道子網路命名為 GatewaySubnet 。
2. **擷取用於閘道的子網路** 。 在此步驟中，我們也會設定要用於下一個步驟中的變數。
   
   * `-Name` 是 Resource Manager VNet 的名稱。
   * `-ResourceGroupName` 與 VNet 相關聯的資源群組。 此閘道子網路必須已為此 VNet 存在且命名為 GatewaySubnet  ，才能正常運作。

            $subnet = Get-AzureRmVirtualNetworkSubnetConfig -Name GatewaySubnet `
            -VirtualNetwork (Get-AzureRmVirtualNetwork -Name RMVNet -ResourceGroupName RG1) 

1. **建立閘道器 IP 位址組態**。 閘道器組態定義要使用的子網路和公用 IP 位址。 使用下列範例來建立閘道組態。<br>在這個步驟中，必須分別從子網路和 IP 位址物件傳遞 id 屬性給 `-SubnetId` 和 `-PublicIpAddressId` 參數。 您無法使用簡單的字串。 這些變數設定於要求公用 IP 的步驟以及擷取子網路的步驟中。
   
        $gwipconfig = New-AzureRmVirtualNetworkGatewayIpConfig `
        -Name gwipconfig -SubnetId $subnet.id `
        -PublicIpAddressId $ipaddress.id
2. **建立 Resource Manager 虛擬網路閘道** 。 `-VpnType` 必須為 *RouteBased*。 可能需要 45 分鐘以上才能完成。
   
        New-AzureRmVirtualNetworkGateway -Name RMGateway -ResourceGroupName RG1 `
        -Location "EastUS" -GatewaySKU Standard -GatewayType Vpn `
        -IpConfigurations $gwipconfig `
        -EnableBgp $false -VpnType RouteBased
3. **複製公用 IP 位址** 。 您會在進行傳統 VNet 的區域網路設定時用到它。 您可以使用下列 Cmdlet 來擷取公用 IP 位址。 公用 IP 位址會在傳回資料中列為 IpAddress 。
   
        Get-AzureRmPublicIpAddress -Name gwpip -ResourceGroupName RG1

## <a name="section-3-modify-the-classic-vnet-local-site-settings"></a>第 3 節：修改傳統 VNet 本機站台設定

在本節中，您將會處理傳統 VNet。 您將會取代您在指定本機站台設定 (將用於連線至 Resource Manager VNet 閘道) 時所使用的預留位置 IP 位址。 

1. 匯出網路組態檔。

        Get-AzureVNetConfig -ExportToFile C:\AzureNet\NetworkConfig.xml
2. 使用文字編輯器修改 VPNGatewayAddress 的值。 使用 Resource Manager 閘道的公用 IP 位址取代預留位置 IP 位址，然後儲存變更。
   
        <VPNGatewayAddress>13.68.210.16</VPNGatewayAddress>
3. 將修改過的網路組態檔匯入至 Azure。

        Set-AzureVNetConfig -ConfigurationPath C:\AzureNet\NetworkConfig.xml

## <a name="a-nameconnectasection-4-create-a-connection-between-the-gateways"></a><a name="connect"></a>區段 4：在閘道之間建立連線
在閘道之間建立連線需要 PowerShell。 您可以需要加入您的 Azure 帳戶，才能使用傳統 PowerShell Cmdlet。 若要這樣做，請使用 `Add-AzureAccount`。

1. 在 PowerShell 主控台中，設定您的共用金鑰。 執行 Cmdlet 之前，請參閱您針對 Azure 預期看到的確切名稱所下載的網路組態檔。 當指定包含空格的 VNet 名稱時，請使用單引號括住值。<br><br>在下列範例中，`-VNetName` 是傳統 VNet 的名稱，`-LocalNetworkSiteName` 是您針對區域網路站台指定的名稱。 `-SharedKey` 是您產生和指定的值。 在範例中，我們使用的是 'abc123'，但是您可以產生並使用更為複雜的值。 重要的是，您在此指定的值必須與您在下一個步驟中建立連線時指定的值相同。 傳回應顯示 [狀態: 成功]。 
   
        Set-AzureVNetGatewayKey -VNetName ClassicVNet `
        -LocalNetworkSiteName RMVNetLocal -SharedKey abc123
2. 執行下列命令來建立 VPN 連線。
   
    **設定變數**
   
        $vnet01gateway = Get-AzureRMLocalNetworkGateway -Name ClassicVNetLocal -ResourceGroupName RG1
        $vnet02gateway = Get-AzureRmVirtualNetworkGateway -Name RMGateway -ResourceGroupName RG1
   
    **建立連線**<br> 請注意，`-ConnectionType` 是 IPsec，而不是 Vnet2Vnet。
   
        New-AzureRmVirtualNetworkGatewayConnection -Name RM-Classic -ResourceGroupName RG1 `
        -Location "East US" -VirtualNetworkGateway1 `
        $vnet02gateway -LocalNetworkGateway2 `
        $vnet01gateway -ConnectionType IPsec -RoutingWeight 10 -SharedKey 'abc123'

## <a name="section-5-verify-your-connections"></a>第 5 節︰驗證您的連線

### <a name="to-verify-the-connection-from-your-classic-vnet-to-your-resource-manager-vnet"></a>驗證從傳統 VNet 到 Resource Manager VNet 的連線

####<a name="verify-your-connection-using-powershell"></a>使用 PowerShell 驗證您的連線


[!INCLUDE [vpn-gateway-verify-connection-ps-classic](../../includes/vpn-gateway-verify-connection-ps-classic-include.md)]


####<a name="verify-your-connection-using-the-azure-portal"></a>使用 Azure 入口網站驗證您的連線

[!INCLUDE [vpn-gateway-verify-connection-azureportal-classic](../../includes/vpn-gateway-verify-connection-azureportal-classic-include.md)]


###<a name="to-verify-the-connection-from-your-resource-manager-vnet-to-your-classic-vnet"></a>驗證從 Resource Manager VNet 到傳統 VNet 的連線

####<a name="verify-your-connection-using-powershell"></a>使用 PowerShell 驗證您的連線

[!INCLUDE [vpn-gateway-verify-ps-rm](../../includes/vpn-gateway-verify-connection-ps-rm-include.md)]

####<a name="verify-your-connection-using-the-azure-portal"></a>使用 Azure 入口網站驗證您的連線

[!INCLUDE [vpn-gateway-verify-connection-portal-rm](../../includes/vpn-gateway-verify-connection-portal-rm-include.md)]

## <a name="a-namefaqavnet-to-vnet-considerations"></a><a name="faq"></a>VNet 對 VNet 的考量

[!INCLUDE [vpn-gateway-vnet-vnet-faq](../../includes/vpn-gateway-vnet-vnet-faq-include.md)]




<!--HONumber=Jan17_HO4-->


