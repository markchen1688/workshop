---
title: "在 Azure CDN 中壓縮檔案以改善效能 | Microsoft Docs"
description: "了解如何藉由在 Azure CDN 中壓縮檔案來改善檔案傳輸速度並增加頁面載入效能。"
services: cdn
documentationcenter: 
author: zhangmanling
manager: erikre
editor: 
ms.assetid: af1cddff-78d8-476b-a9d0-8c2164e4de5d
ms.service: cdn
ms.workload: tbd
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 01/23/2017
ms.author: mazha
translationtype: Human Translation
ms.sourcegitcommit: 57d00f2192fed7a2e89ac94e110ebb7e84c83b72
ms.openlocfilehash: e80136d096ba83ab5050c8d1d95a9e2abb7a3646


---
# <a name="improve-performance-by-compressing-files-in-azure-cdn"></a>在 Azure CDN 中壓縮檔案以改善效能
壓縮是簡單且有效的方法，可提升檔案傳輸速度，並且在檔案從伺服器傳送出去之前先減少其大小，以增加頁面載入效能。 它會降低頻寬成本，並提供回應速度更快的體驗給使用者。

有兩種方式可啟用壓縮︰

* 您可以在原始伺服器上啟用壓縮，此時 CDN 會透過壓縮的檔案傳遞，並將壓縮的檔案傳送到提出要求的用戶端。
* 您可以直接在 CDN Edge Server 上啟用壓縮，此時 CDN 會壓縮檔案並將其提供給使用者，即使原始伺服器未壓縮這些檔案也是如此。

> [!IMPORTANT]
> CDN 組態變更會需要一些時間才能傳播至整個網路。  若為 <b>來自 Akamai 的 Azure CDN</b> 設定檔，通常會在一分鐘之內完成傳播。  若為 <b>來自 Verizon 的 Azure CDN</b> 設定檔，您通常會在 90 分鐘之內看到變更套用。  如果這是您第一次設定 CDN 端點壓縮，您應該考慮先等候 1-2 小時，確定壓縮設定已傳播至 POP 再進行疑難排解
> 
> 

## <a name="enabling-compression"></a>啟用壓縮
> [!NOTE]
> 標準和高階 CDN 層提供相同的壓縮功能，但兩者的使用者介面不同。  如需有關標準和高階 CDN 層之間的差異的詳細資訊，請參閱 [Azure CDN 概觀](cdn-overview.md)。
> 
> 

### <a name="standard-tier"></a>標準層
> [!NOTE]
> 本節適用於**來自 Verizon 的 Azure CDN 標準**和**來自 Akamai 的 Azure CDN 標準**設定檔。
> 
> 

1. 在 [CDN 設定檔] 刀鋒視窗中，按一下您要管理的 CDN 端點。
   
    ![[CDN 設定檔] 刀鋒視窗端點](./media/cdn-file-compression/cdn-endpoints.png)
   
    隨即開啟 [CDN 端點] 刀鋒視窗。
2. 按一下 [設定]  按鈕。
   
    ![[CDN 設定檔] 刀鋒視窗管理按鈕](./media/cdn-file-compression/cdn-config-btn.png)
   
    隨即開啟 [CDN 組態] 刀鋒視窗。
3. 開啟 [ **壓縮**]。
   
    ![CDN 壓縮的選項](./media/cdn-file-compression/cdn-compress-standard.png)
4. 請使用預設的類型，或者移除或新增檔案類型以修改清單。
   
   > [!TIP]
   > 如果可能，不建議對 ZIP、MP3、MP4、JPG 等壓縮格式套用壓縮。
   > 
   > 
5. 完成變更之後，按一下 [ **儲存** ] 按鈕。

### <a name="premium-tier"></a>高階層
> [!NOTE]
> 本節適用於 **來自 Verizon 的 Azure CDN 進階** 設定檔。
> 
> 

1. 在 [CDN 設定檔] 刀鋒視窗中，按一下 [管理]  按鈕。
   
    ![[CDN 設定檔] 刀鋒視窗的 [管理] 按鈕](./media/cdn-file-compression/cdn-manage-btn.png)
   
    隨即開啟 CDN 管理入口網站。
2. 將滑鼠移至 [HTTP 大型] 索引標籤上，然後將滑鼠移至 [快取設定] 飛出視窗上。  按一下 [ **壓縮**]。
   
    隨即顯示 [壓縮] 的選項。
   
    ![檔案壓縮](./media/cdn-file-compression/cdn-compress-files.png)
3. 按一下 [啟用壓縮]  選項按鈕以啟用壓縮。  在 [檔案類型]  文字方塊中，輸入您想要壓縮成逗號分隔清單 (無空格) 的 MIME 類型。
   
   > [!TIP]
   > 如果可能，不建議對 ZIP、MP3、MP4、JPG 等壓縮格式套用壓縮。 
   > 
   > 
4. 完成變更之後，按一下 [更新]  按鈕。

## <a name="compression-rules"></a>壓縮規則
這些資料表描述每個案例的 Azure CDN 壓縮行為。

> [!IMPORTANT]
> 若為 **來自 Verizon 的 Azure CDN** (標準和進)，只會壓縮合格的檔案。  若要符合壓縮，檔案必須︰
> 
> * 超過 128 個位元組。
> * 小於 1 MB。
> 
> 若為 **來自 Akamai 的 Azure CDN**，所有檔案都符合壓縮資格。
> 
> 對於所有的 Azure CDN 產品，檔案必須為已 [設定壓縮](#enabling-compression)的 MIME 類型。
> 
> **來自 Verizon 的 Azure CDN** 設定檔 (標準和進階) 支援 **gzip**、**deflate** 或 **bzip2** 編碼。  **來自 Akamai 的 Azure CDN** 設定檔只支援 **gzip** 編碼。
> 
> **來自 Akamai 的 Azure CDN** 端點一律要求來自原點的 **gzip** 編碼，不論用戶端是否提出要求。
> 
> 

### <a name="compression-disabled-or-file-is-ineligible-for-compression"></a>已停用壓縮或檔案不適合進行壓縮
| 用戶端要求的格式 (透過 Accept-encoding 標頭) | 快取的檔案格式 | CDN 對用戶端的回應 | 注意事項 |
| --- | --- | --- | --- |
| 已壓縮 |已壓縮 |已壓縮 | |
| 已壓縮 |未壓縮 |未壓縮 | |
| 已壓縮 |不快取 |已壓縮或未壓縮 |取決於原始回應 |
| 未壓縮 |已壓縮 |未壓縮 | |
| 未壓縮 |未壓縮 |未壓縮 | |
| 未壓縮 |不快取 |未壓縮 | |

### <a name="compression-enabled-and-file-is-eligible-for-compression"></a>啟用壓縮和檔案適合進行壓縮
| 用戶端要求的格式 (透過 Accept-encoding 標頭) | 快取的檔案格式 | CDN 對用戶端的回應 | 注意事項 |
| --- | --- | --- | --- |
| 已壓縮 |已壓縮 |已壓縮 |支援格式之間的 CDN 轉碼 |
| 已壓縮 |未壓縮 |已壓縮 |CDN 執行壓縮 |
| 已壓縮 |不快取 |已壓縮 |如果來源傳回未壓縮，則 CDN 會執行壓縮。  **來自 Verizon 的 Azure CDN** 會傳遞第一次要求中的未壓縮檔案，然後再壓縮及快取檔案以供後續要求之需。  具有 `Cache-Control: no-cache` 標頭的檔案永遠不會經過壓縮。 |
| 未壓縮 |已壓縮 |未壓縮 |CDN 執行解壓縮 |
| 未壓縮 |未壓縮 |未壓縮 | |
| 未壓縮 |不快取 |未壓縮 | |

## <a name="media-services-cdn-compression"></a>媒體服務 CDN 壓縮
對於啟用媒體服務 CDN 的串流端點，壓縮功能依預設會針對下列內容類型啟用：application/vnd.ms-sstr+xml, application/dash+xml,application/vnd.apple.mpegurl, application/f4m+xml。 您不能使用 Azure 入口網站啟用/停用上述類型的壓縮。  

## <a name="see-also"></a>另請參閱
* [CDN 檔案壓縮疑難排解](cdn-troubleshoot-compression.md)    




<!--HONumber=Jan17_HO4-->


