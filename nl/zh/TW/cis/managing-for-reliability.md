---
copyright:
  years: 2018
lastupdated: "2018-03-13"
---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}

# 管理 IBM CIS 部署以取得最佳可靠性

若要達到 IBM CIS 部署的最佳可靠性，您可以設定有用的 DNS 配置，而且可以設定「廣域負載平衡器」。為了額外的可靠性，您可以使用「頁面規則」確定將您的 Web 內容遞送給客戶，即使您的原點伺服器或快取發生問題也是一樣。此文件提供讓 IBM CIS 部署最為可靠之一些最佳作法的詳細資料。

一般而言，建議的最佳作法如下：

 * 設定 DNS 以利用 IBM CIS Proxy 伺服器及其他特性
 * 使用一個以上的「廣域負載平衡器」平均分散您的網站資料流量
 * 設定適當的「頁面規則」來管理快取及其他選項

所有這些項目都會提供您可以用來建立更可靠 CIS 部署的特定功能。

請注意，CIS 介面會組織成*安全*、*可靠性* 及*效能* 的區段。下圖會顯示主要導覽功能表，並顯示「廣域負載平衡器」及 DNS 功能表項目：

![左導覽 DNS](images/cis-left-navigation.png)


## 設定 DNS
 
 若要開始設定 DNS 配置，請從導覽功能表中選取 **DNS**（如前所示）。
 
 如需設定及管理 DNS 以取得可靠性的詳細資訊，請[參閱此文件](dns.html#setting-up-your-domain-name-system-dns-for-ibm-cis)。


## 設定廣域負載平衡器


若要開始設定「廣域負載平衡器」，請從導覽功能表中選取**廣域負載平衡器**。

如需設定及管理「廣域負載平衡器」的詳細資訊，請[參閱此文件](glb.html#global-load-balancer-glb-concepts)。

## 使用頁面規則來增加可靠性

以下是一些建議的「頁面規則」設定，讓您的網站具有最大可靠性：

 * 一律上線
 * 原點快取控制
 * 轉遞 URL

 ## 一律上線

您可以使用**一律上線**「頁面規則」設定，在伺服器關閉時將網站的有限版本保持為上線。

使用**一律上線**，您的伺服器會關閉，IBM CIS 會提供我們快取中的頁面，因此，訪客仍然會看到他們嘗試造訪的一些頁面。您的訪客會在頁面頂端看到一則訊息，告訴他們目前處於離線瀏覽模式。「一律上線」會傳回 HTTP 狀態 503，但許多其他 Web 應用程式也會使用 503。當您的伺服器重新上線時，IBM CIS 會讓使用者平順地回到一般瀏覽作業。

如果 IBM CIS 在其快取中沒有所要求的頁面，則您的訪客會看到一個錯誤頁面，讓他們知道其所要求的網站網頁已離線。

### 如何設定一律上線

若要啟用**一律上線**，請遵循下列步驟：

 * 使用導覽功能表，以選取「效能」下的「頁面規則」。
 * 使用您網域的 URL 型樣來建立「頁面規則」。
 * 新增已開啟該切換參數的**一律上線**設定。
 * 選取「佈建資源」。

 ### 一律上線的限制

 * **一律上線**會快取根 HTML 的前 10 個鏈結，接著只會快取所有這些頁面的第一個鏈結，最後是所有這些後續頁面的第一個鏈結。這表示只有在原點伺服器關閉時，才能檢視網站上的部分頁面。

 * 最近新增的網站不會有其網站的大型快取可用，這表示如果您只在幾天前新增網站，則**一律上線**無法運作。

 * 如果伺服器關閉，則 CIS 無法顯示專用內容或處理表單提交 (POST)。訪客將會在結帳頁面或需要檢視登入的項目上顯示錯誤。

 * 若要觸發**一律上線**，您的 Web 伺服器必須傳回標準 HTTP 錯誤碼 502 或 504 逾時。當我們遇到連接原點（錯誤 521 & 523）、逾時 (522 & 524)、SSL 錯誤 (525 & 526) 或不明錯誤 (520) 的問題時，「一律上線」也會運作。針對其他 HTTP 回應碼（例如 404、500、503）、資料庫連線錯誤、內部伺服器錯誤，或來自伺服器的空白回覆，不會觸發**一律上線**。

 * 如果啟用「邊緣快取到期 TTL」低於快取頻率（免費客戶：7 天、「專業」客戶：3 天，而「商務與企業」客戶：1 天）的「快取所有項目」頁面規則，則**一律上線**不會運作，因為「邊緣快取到期 TTL」會導致在對應的間隔中清除**一律上線**快取。

## 原點快取控制

您可以使用**原點快取控制**「頁面規則」設定來判定從您原點快取的內容及內容更新頻率，這會影響可靠性及效能。依預設，如果未變更設定，而且未從原點伺服器傳送防止快取的標頭，則 IBM CIS 會快取所有具有特定副檔名的靜態內容。這些類型的內容包括影像、CSS 及 JavaScript。這項快取主要是針對效能。

若要設定**原點快取控制**，您可以使用「頁面規則」來開啟特定標頭，以提供與內容之每一個資源相關的所需行為。若要瞭解如何使用**原點快取控制**，需要有「頁面規則」的某些一般說明及 CIS 的整體快取行為，才能提供環境定義，您會在接下來的幾節中看到此環境定義。有三種方法可以用來控制一般快取，而**原點快取控制**是第二種方法。

設定**原點快取控制**會呼叫快取規則，以嘗試緊密遵循網際網路最佳作法及 RFC，主要是與重新驗證有關。例如，`max-age=0` 的 CIS 預設行為根本不會進行快取，而設定**原點快取控制**會進行快取，但它一律會重新驗證。

### 如何設定原點快取控制

 * 使用導覽功能表，以選取「效能」下的「頁面規則」。
 * 使用參照網域的 URL 型樣來建立「頁面規則」。
 * 新增已開啟該切換參數的**原點快取控制**設定。
 * 選取「佈建資源」。

### 頁面規則優先順序

對於快取整體，會優先使用兩個特定「頁面規則」：

 * 如果「頁面規則」的**快取層次**設為`略過`，則不會快取符合該「頁面規則」的資源。IBM CIS 仍然是作為 Proxy，而其他效能特性仍然維持作用中。不過，不會從我們的快取中提供您的內容，而是直接從原點伺服器進行提取。

 * 如果「頁面規則」的**快取層次**設為`快取所有項目`，則會快取符合「頁面規則」的資源。**使用此「頁面規則」設定是讓我們知道快取視為靜態內容之外資源（包括 HTML）的唯一方式。**

如果未設定「頁面規則」，我們將使用根據資源副檔名的`標準`快取模式。我們只會快取靜態資源（如前所述）。

### 原點快取控制標頭

變更 IBM CIS 所快取內容的第二種方式是透過快取從原點傳送的標頭。CIS 將會遵循這些設定，但您可以指定**邊緣快取 TTL**「頁面規則」設定予以置換。以下是我們在決定要從原點快取哪些資源時考量的標頭：

 * 如果 **Cache-Control** 標頭設為 `private`、`no-store`、`no-cache` 或 `max-age=0`，或者回應中有 Cookie，則 IBM CIS 不會快取資源。請注意，不應該快取機密資料，因此，您可以考慮在該情況下使用其中一個標頭。

 * 如果 **Cache-Control** 標頭設為 `public` 且 `max-age` 大於 0，或者 `Expires` 標頭設為未來的任何時間，則我們會快取資源。

**附註：**根據 RFC 規則，`Cache-Control: max-age` 勝過 `Expires` 標頭。如果我們看到這兩者，但兩者不一致，則會使用 `max-age`。

### 使用 's-maxage' 標頭

同時控制快取行為及瀏覽器快取行為的第三種方式是使用 `s-maxage` Cache-Control 標頭。

通常，我們遵循 `max-age` 指引：

`Cache-Control: max-age=1000`

但是，如果您要指定與瀏覽器不同的快取逾時，則我們可以使用 `s-maxage`。以下範例告知 IBM CIS 快取物件 200 秒，並告知瀏覽器快取物件 60 秒。

`Cache-Control: s-maxage=200, max-age=60`

基本上，`s-maxage` 只要後面接著反向 Proxy（因此瀏覽器應該會忽略它），而另一方面，我們 (IBM CIS) 會優先使用 `s-maxage`（如果有的話）。我們將會遵循較高的值：瀏覽器快取設定或 `max-age` 標頭。

### 快取控制標頭及頁面規則以取得可靠性的摘要

總之，以下是要考量快取相關可靠性的一些主要區域：

 * 檢查您原點的快取標頭，確定沒有可快取資源的置換中標頭（`Cache-Control` 及 `Expires`）。

 * 依預設，CIS 一律會快取靜態內容，以及下列根據回覆碼的 TTL：

```
200 301    120m;
302 303    20m;
403        5m; for reliability
404        5m;
any        0s;
```

 * 若要快取更多內容，請在所需 URL 上建立**快取層次**設為「快取所有項目」的「頁面規則」（如果您的 Web 伺服器在要求此 URL 時傳回 404，則我們只會快取這個結果 5 分鐘）。

 * 若要避免在 URL 上進行快取，請建立**快取層次**設為「略過」的「頁面規則」。


 ## 轉遞 URL

為了確保內容一律可用 (HA)，您可以建立具有**轉遞 URL** 設定的「頁面規則」，以在網站無法使用時使用。

 **附註：**當您啟用**轉遞 URL** 時，會停用您的所有其他設定，因為您會將所有資料流量傳送至另一個 URL。

### 如何設定轉遞 URL

 * 使用導覽功能表，以選取「效能」下的「頁面規則」。
 * 使用參照網域的 URL 型樣來建立「頁面規則」。
 * 新增**轉遞 URL** 設定。
 * 選取轉遞類型，並輸入目的地 URL。
 * 選取「佈建資源」。

### 轉遞範例：

假設您要讓任何人都能輕鬆地到達 URL，例如：

    *www.example.com/+

    *example.com/+

此型樣會符合：

    http://example.com/+
    http://www.example.com/+
    https://www.example.com/+
    https://blog.example.com/+
    https://www.blog.example.com/+
    Etc...

它不會符合：

    http://www.example.com/blog/+  [extra directory before the +]
    http://www.example.com+  [no trailing slash]


建立符合您要的型樣之後，請新增**轉遞 URL** 設定，然後選取轉遞類型，並輸入目的地 URL。例如：

    https://plus.google.com/yourid

選取「佈建資源」。在幾秒內，任何符合型樣的要求都會轉遞至具有所指定重新導向的新 URL。

### 進階轉遞選項：

如果您使用基本重新導向（例如，將根網域轉遞至 www.yourdomain.com），則會遺失 URL 中的其他任何項目。例如，您可以設定型樣：

    example.com

並且將它轉遞至：

    http://www.example.com

但接著某個人輸入：

    example.com/some-particular-page.html

則會將他們重新導向至：

    www.example.com

而非

    www.example.com/some-particular-page.html

解決方案是使用變數。每一個萬用字元都會對應至可在轉遞位址中參照的變數。變數是以後接數字的 `$` 代表。若要參照第一個萬用字元，請使用 `$1`，若要參照第二個萬用字元，請使用 `$2`，依此類推。若要修正前一個範例中從根到 `www` 的轉遞，則可以使用相同的型樣：

    example.com/*

您接著設定將資料流量轉遞至其中的下列 URL：

    http://www.example.com/$1

在此情況下，如果有人前往：

    example.com/some-particular-page.html

則會將他們重新導向至：

    http://www.example.com/some-particular-page.html
