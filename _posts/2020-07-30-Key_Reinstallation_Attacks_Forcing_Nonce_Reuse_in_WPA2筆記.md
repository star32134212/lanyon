---
layout: post
title: Key Reinstallation Attacks - Forcing Nonce Reuse in WPA2 筆記
tags: 
  - "tech"
---    
> 論文作者：Mathy Vanhoef、Frank Piessens  
> 中文筆記整理：Orange  
 
## 摘要
Mathy Vanhoef和Frank Piessens找出了現行Wifi通訊的重大漏洞，設計了KRACKs(Key Re-installation Attack)攻擊可以解密傳送的密文。針對各種裝置的通訊架構作者提出了各種實現KRACKs的攻擊方法。  

---

## 名詞解釋

### The 802.11i Amendment 
在802.11的WEP被證明不安全後，開始發展802.11i，[WPA](https://zh.wikipedia.org/wiki/WPA)是在802.11i還沒完全確定時先建立的，等到802.11i完整後才出現了WPA2。主要差別在WPA預設使用的是TKIP，WPA2預設使用的是CCMP，可以手動進行更改。不管是WPA還是WPA2都要經過4-way Handshak。  


### WEP 無線加密協定
[WEP wiki](https://zh.wikipedia.org/wiki/%E6%9C%89%E7%B7%9A%E7%AD%89%E6%95%88%E5%8A%A0%E5%AF%86)    
一個標準的64位元WEP使用40位元的鑰匙接上24位元的初向量（initialization vector，IV）成為RC4用的鑰匙。在起草原始的WEP標準的時候，美國政府在加密技術的輸出限制中限制了鑰匙的長度，一旦這個限制放寬之後，所有的主要業者都用104位元的鑰匙實作了128位元的WEP延伸協定。用戶輸入128位元的WEP鑰匙的方法一般都是用含有26個十六進位數（0-9和A-F）的字串來表示，每個字元代表鑰匙中的4個位元，4 * 26 = 104位元，再加上24位元的IV就成了所謂的"128位元WEP鑰匙"。有些廠商還提供256位元的WEP系統，就像上面講的，24位元是IV，實際上剩下232位元作為保護之用，典型的作法是用58個十六進位數來輸入，（58 * 4 = 232位元）+ 24個IV位元 = 256個WEP位元。  

同一個鑰匙絕不能使用二次，所以使用IV的目的就是要避免重複；然而24位元的IV並沒有長到足以擔保在忙碌的網路上不會重複，目前已被證實此種方法不安全，很容易就可以破解。  

### (WPA-)TKIP 臨時金鑰完整性協定 
**Temporal Key Integrity Protocol**，TKIP可以包在WEP外層，再幫WEP加一道鎖，主要目的就是補上WEP的已知弱點，例如：WEP使用的密鑰長度是40位元和128位元，40位元很容易被破解，而TKIP使用的密鑰長度為128位元，解決了WEP的密鑰長度過短的問題。另外WEP的同一區域網路內所有用戶都共享同一個密鑰，一個用戶洩漏將使整個區域網絡不安全，TKIP可以讓每個數據包所用的密鑰有變化(使用暫時的密鑰)，密鑰通過某種公式生成，參數包括基本密鑰、MAC地址及數據包的序列號。WEP還有個弱點是replay attacks，而TKIP傳送的每一個數據包都具有獨特的48位元序列號，由於48位元序列號需要數千年時間才會用完而出現重複，檢查序列號就可以免疫重送攻擊。  

當TKIP使用中，Temporal Key(TK)又被分為128位元的encryption key、兩個64位元的Message Integrity Check (MIC)，第一個MIC用在AP到STA間的通訊，第二個MIC則是反方向，前面的encryption key使用的是[RC4](https://zh.wikipedia.org/wiki/RC4)加密。  

### (AES-)CCMP 
[CCMP百度](https://baike.baidu.com/item/CCMP)    
CCMP主要分兩塊，分别是CTR mode和CBC-MAC mode。CTR mode是加密算法，CBC-MAC用於訊息完整性的檢查。CCMP的IV是由傳送方的MAC地址、一些flag和一些隨機值(nonce)產生，這特定條件下是不初始化的，這個nonce也被用在replay counter，傳送前回將nonce +1，直到刷新TK才讓他歸0。此種架構的好處，第一是讓IV不會重複，第二是讓雙向的傳輸都只要用同個TK。  


這種架構稱為[AEAD](https://zhuanlan.zhihu.com/p/28566058)，全名Authenticated Encryption with Associated Data，兼具加密、資料完整驗證和身份可認證性的加密形式。原本的加密法解密後雖可以得到明文，但這無法保證使用的密鑰就是正確的，如果解密後的明文是一組疑似明文的內容，要怎麼確定是密鑰用錯還是本身傳送的內容就有問題，AEAD結合了可以驗證身份的資訊(MAC)和確認資料完整性的資訊(MIC)來加密可以解決這件事。  

### GCMP
給WPA3使用的，和CCMP一樣是AEAD，IV是發送方MAC+48位隨機數，一樣有用replay counter計數器避免重複使用，也用TK雙向傳輸。  

---

## 解釋各種會用到的Key
下圖為一個AP所擁有的Key和分級：  
![](https://i.imgur.com/kgxNp0q.png){:height="300" width="400"}    

### MSK (Master Session Key)
由802.1X/EAP或PSK身分認證後生成的第一個密鑰。每個AP有獨立的MSK，MSK是用來生成GMK和PMK的，如果WPA沒有開啟PSA模式，而是安裝Radius伺服器，要用WPA/WPA2，那PMK就完全是由MSK產生的，應該會是[MSK的前256 bit](https://blog.csdn.net/lee244868149/article/details/52743302)。  
不過一般用戶都是[WPA-PSK/WPA2-PSK](https://kknews.cc/zh-tw/tech/89b293q.html)，這時候的PMK就會是256 bit的PSK，。  

### PMK (Pairwise Master Key)
PMK會存在AP和所有STA，不需要進行key交換，用它来生成加密單播數據的PTK。  

### PTK (Pairwise Transit Key)

PTK用來加密AP和STA之間通訊的單播封包，AP與每個STA通訊用的PTK都是唯一的。  

`PTK = PRF(PMK + ANonce + SNonce + AMac + SMac)`  

PRF：pseudo-random function，偽隨機函數  
ANonce：AP生成的隨機數(A = authenticator)  
SNonce：STA生成的隨機數(S = supplicant)  
AMac：AP的Mac地址  
SMac：STA的Mac地址  

PTK又可分為KCK、KEK和TK三部分，KCK用在MIC校驗，KEK用在加密GTK，TK在在為資料加密金鑰。4-way Handshak完成後，傳輸資料用TK進行加密。  

### GMK (Group Master Key)

GMK用來生成GTK，給所有連接到此AP的STA共享。  

### GTK (Group Temporal Key)

每過一段時間AP就更新一次GTK並傳給每個正在使用此AP的用戶，GTK用来加密AP和STA之間通訊的多播封包，連結同個AP的所有 STA共享一個GTK。AP可以選擇在發送後就把新的GTK安裝好也可以等到每個用戶都回傳確認安裝新GTK的訊息後。之後的GTK會存在每次傳送的EAPOL的RSC中。GTK本身是透過KEK加密的。GTK和PTK就可以對EAPOL架構的訊息包進行完整的保護。  

`GTK = PRF(GMK + ANonce + AMac)`

### MIC (Message Integrity Check)

訊息完整性檢查，用來防止訊息遭修改。  

---

## 四向交握（4-way Handshak）基本原理
4-way Handshak中的所有msg皆使用EAPOL的框架實作，這是一種廣泛使用在無線網路通訊的框架。  
![](https://i.imgur.com/j2aPBwP.png)  

* header定義了此EAPOL在四向交握中的msg類型，ex：msg1 msg2
* replay counter在每次傳送EAPOL FRAME時作為計數器增加
* nonce放AP和STA隨機產生的亂數

以`MsgN(r, Nonce; GTK)`為例，header是MsgN，replay counter是r，可能會有Nonce也可能不需要，`;`後面存在key data field，GTK用KEK加密。  

![](https://i.imgur.com/Obi9tlp.png){:height="400" width="400"}      


簡單版：  
>step 1：AP傳送ANonce給STA。  

>step 2：STA根據ANonce產生臨時的PTK和檢查碼。  

>step 3：AP比對PTK和檢查碼後，產生GTK和檢查碼。  

>step 4：STA成功安裝密鑰，回傳ok給AP，之後STA和AP的通訊就用這組密鑰加密。  

詳細版：  
STA進入PTK-INIT階段，產生PMK，AP把自己算出的ANonce給STA。收到後，STA進入PTK-START階段，自己也算一把 SNonce，然後用SNonce算出PTK，把SNonce/PTK附上MIC發給AP，AP收到SNonce，也可以算出PTK來比對，沒問題之後就把 用GMK產生GTK，連著MIC一起發給STA，STA收到訊息後先檢查是否有問題。如果合法，就進入PTK-NEGOTIATING階段，安裝PTK和GTK，然後和AP說裝好了，進入PTK-DONE階段。AP收到裝好的訊息後，安裝這個STA的PTK。完成之後，開啟port，雙方就可以用 802.1x 做通訊。  
![](https://i.imgur.com/CrcL5qW.png){:height="400" width="300"}     

---

## KRACKs-針對PTK
全名是Key Re-installation Attack。四向交握的第四次握手有可能會因網路不穩之類的原因，導致AP沒有收到STA回傳的ok，這時AP就只好再傳一次GTK給STA。當初制訂時只有pseudo code，沒有考慮到state machine。如果STA其實有成功安裝，使用**man-in-the-middle(MitM)攻擊**，讓STA回傳告知AP安裝完成的訊息被擋掉，AP重送了GTK，此時的replay counter一樣會加1，由於PTK是由Nonce生成的，replay counter改變，STA會重新安裝PTK，當初制定協議時沒有考慮密鑰安裝的次數，所以KRACKs就是讓AP重複第三次握手，多次之後會導致協議使用的Nonce和封包計數器歸零，而攻擊者會監聽拿到多個密文訊息。  

假設在Nonce(k)重複用的狀況下，攻擊者拿到c1和c2，就可以計算 c1 xor c2 的結果。因為 xor 符合交換率(A xor B = B xor A)，又是自反函數 (X xor X = 0)，所以還是可以倒推回去原本的明文。  
```
m = 明文; c = 密文; k = 種子; P = 算法
c1 = m1 xor P(k)
c2 = m2 xor P(k)
```

### 作者提出的三個KRACKs實作的困難點和解決辦法
1.Window和IOS不同意第三次握手的重傳，但仍然可以用FT Handshake(Fast Transmit Handshake)來實現reinstall。  
2.要用KRACKs，還必須能讓MitM成功，使用channel-based MitM attack可以偽造出一個和目標AP相同MAC的通道，有了一樣的MAC和可控制的counter就能弄出同一把session key。  
3.有些系統只允許在PTK安裝後進行加密傳輸，代表在傳送GTK時不用另外加密，所以沒有利用counter的機會，不過作者仍提出了可以繞過這個問題的方法。  

### Plaintext Retransmission of message 3 
![](https://i.imgur.com/upcIdCE.png){:height="400" width="300"}     
這種類型是安裝了PTK後仍然接受未被加密過的Msg3訊息，MitM直接擋Msg4就可以了。  

### Encrypted Retransmission of message 3 
![](https://i.imgur.com/XVrBpcr.png){:height="400" width="350"}  
這種類型是只要STA安裝了PTK就只接受PTK加密過的訊息，如上圖，MitM可以直接攔截Msg3，讓AP發好幾個Msg3，然後再一次丟給STA，STA收到一堆Msg3會排進receive queue，處理第一個Msg3的時候安裝PTK並回傳Msg4，然後處理第二個時又被要求再安裝一次PTK，以此類推。這種方法是利用CPU和NIC的race conditions原理。  

![](https://i.imgur.com/SsqOmkR.png){:height="400" width="350"}  
上圖是在講nonce的變動，第一個msg3讓STA安裝了PTK，假設此時nonce=0，然後用這個nonce=0去回傳msg4，接著處理第二個，這種只接收加密訊息的系統不會去檢查加密的鑰匙是否對不對，只要有加密就好，第二個msg3用的nonce=1，有加密就收，然後執行裡面帶的指令，是安裝PTK，所以又安裝了一次，nonce歸0，然後回傳msg4，此時nonce++變成1，以此類推，每次都用新的PTK回傳msg4，而且nonce都是1。  
  
### 不怕此種攻擊的案例：PeerKey Handshake 
**PeerKey**用在STA和STA直接溝通，使用的是**Station-To-Station Link (STSL) Master Key (SMK)**，Re-installation Attack對使用此種方法的設備沒什麼影響，原因在這種Master Key不是用data-confidentiality protocol，代表不會用到nonce和replay counters，但他一樣是使用四向交握的，但因為攻擊者無法控制nonce和replay counters而免疫。  

---
## KRACKs-針對GTK
要針對GTK攻擊有兩個條件，第一個是STA在重新安裝GTK後會刷新replay counter，第二個是要能攔截到目標STA會收的msg1且裡面已經包含目前AP正在使用的GTK。AP每過一段時間就重新算一個GTK，有的會延後安裝，等到所有STA都安裝好並回傳msg2時才安裝新的GTK。為了讓正在使用的STA這個群體獨立出來，AP會在有STA離開時就更新一次GTK，為的就是只有正在使用的這群用戶共享，因此可以透過假裝要離開的請求騙AP更新GTK。  


### Attacking Immediate Key Installation 
![](https://i.imgur.com/Wm7oAsM.png){:height="400" width="350"}  
此圖是AP直接安裝GTK的情況，把STA通知完成安裝GTK的msg2攔截，AP重發msg1，一樣把這個msg攔截，此時因為AP不延後安裝，他已經安裝好GTK了，開始廣播他要通知給各STA的資料，STA因為有安裝好GTK，也可以讀到這段資料，然後再把重發的msg1傳給STA，對STA來說他不知道這個msg1是補發的，把它當作AP又要求大家更新GTK，於是用剛才接收到的廣播資訊算出了新的GTK，以此類推可以讓STA重複安裝GTK。  

### Attacking Delayed Key Installation 
![](https://i.imgur.com/OFmQc2D.png){:height="400" width="350"}     
此圖是AP會延後安裝GTK的情況，跟左圖只有一點差別，一樣攔截了STA回傳的msg2，等到AP重新發了安裝GTK的要求時才回傳msg2，AP會無視counter不同這個問題(本來就會考慮網路不穩的狀況，這個無視算合理)，AP確認大家都裝好GTK後自己也安裝新GTK，此時可以開始廣播資料了，等STA收到廣播資料後再把剛剛的補發msg1傳給STA，STA根據最新的廣播資訊算出新的GTK並安裝，以此類推可以讓STA重複安裝GTK。  

### 不怕此種攻擊的案例： OpenBSD AP
最後作者提到只有**OpenBSD AP**可以免疫這種攻擊，他採用延後安裝GTK且要求counter要對上。  

---

## KRACKs-針對 802.11R FT HANDSHAKE 
[IEEE802.11r-2008](https://zh.wikipedia.org/wiki/IEEE_802.11r-2008)，又稱為快速BSS切換，也稱快速漫遊，目標是在解決使用者從一個AP移動到另一個AP的漫遊可以快速轉換。  
![](https://i.imgur.com/kNyLWLI.png){:height="400" width="300"}   
正常的快速漫遊過程如上圖的第1部分，AuthReq和AuthResp在功能上就和四向交握中的msg1和msg2一樣，且他們帶著Nonce可以產生fresh session key，ReassoReq和ReassoResp在功能上和msg3和msg4一樣，最終可以算出PTK和GTK並安裝。  

和四向交握不同的是FT**只有兩個msg有用到MIC**且**不需使用到replay counters**，也因此**PTK必須是在STA回應的訊息確實被送到AP時才安裝**，因此**雖然AuthReq完和AuthResp完就有能力裝PTK了也會等到最後握完才和GTK一起裝**。  

聽起來似乎不能用KRACKs攻擊，但作者仍然提出了可以攻擊的辦法，而且不需要MitM就可以達成了，只要能竊聽和發送frame就能達成。  

### 攻擊原理
竊聽AP傳送給STA的加密資料，拿走其中的Nonce和MIC，偽造一個ReassonResp傳給AP可以促使AP重新安裝PTK，多次以後就會用到重複的nonce，由於沒有counter，且本身就會考量到可能因background noise導致資料重傳，使這樣的偽造包可以有效果。  

### 強制使用FT handshake
由於FT handshake是用使用者在AP與AP間的轉換，使得可攻擊時間其實很短，他不會是一個長期使用的交握模式，因此要用一些方法強迫FT handshake。  

[蟲洞攻擊](https://baike.baidu.com/item/%E8%99%AB%E6%B4%9E%E6%94%BB%E5%87%BB)就是一招，利用 wormhole attack在目標AP附近製造出一個鄰近AP，接著發送 BSS Tran-sition Management Request給使用者，這是一個要求使用者漫遊到附近AP的請求，使用者發現附近有一個AP於是答應請求啟動FT handshake跳過去。  

---

## 總結
TKIP、CCMP和GCMP都是使用[stream cipher](https://zh.wikipedia.org/zh-tw/%E6%B5%81%E5%AF%86%E7%A0%81)，所以只要nonce重複就可以代表接下來的加密也會重複，可以被用來解密。可怕的是，AP通常不會是frame的終點，網路設備才是終點，KRACKs可以控制與目標AP連接的網路設備。  

實際上KRACKs能做到的傷害是可以再延伸的，包括TCP、NTP、NLS...Android 6.0也許是最嚴重的，KRACKs導致其有可能使用全是0的鑰匙進行加密，代表著將有31.2%的Android用戶處於傳輸資料不安全的狀態。  

有時候由於background noise的原因，不用攻擊者也有可能自動產生re-installation attack。  
 
有個有趣的研究方向是確定其他協定是否也容易受到KRACKs的攻擊。  

### 對策
作者提出了兩個對策：  
* 第一個是任何使用data-confidentiality protocol的裝置應該要檢查是否已經安裝了正在使用的密鑰。確保replay counter只能增加不要隨意歸0。
* 第二個是確保特定key在某次的握手過程中只能被安裝一次，例如可以多加一個布林欄位來表示目前進行到PTK的哪個階段，如果是PTK-DONE=True、非PTK-DONE=False。

關於改進的部分，作者認為：  
*  協定的規範應足夠精確和明確，四向交握的counter就有著模糊的定義
*  並不是協定已被證明是安全的，它的實作就也會是安全的
*  data-confidentiality protocol 應該要有一些防止nonce重用的機制，應使用抗隨機數濫用的加密法，例如AES-SIV、GCM-SIV或HS1-SIV





---
## 其他閱讀
[原論文](https://dl.acm.org/doi/abs/10.1145/3133956.3134027)
[WEP](http://www.tsnien.idv.tw/Network_WebBook/chap15/15-5%20%E8%AA%8D%E8%AD%89%E8%88%87%E4%BF%9D%E5%AF%86.html)  
[四向交握](https://read01.com/zh-tw/00QNOo.html#.XyApRfgzYyl)  
[四向交握分析與實作](https://kysonlok.gitbook.io/blog/wireless/4_way_handshake#gtk-group-temporal-key)