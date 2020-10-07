---
layout: post
title: One Pixel Attack for fooling deep neural network 筆記
tags: 
  - "security"
  - "tech"
---    
作者：Jiawei Su, Danilo Vasconcellos Vargas, Sakurai Kouichi  
來源：CVPR2017 [paper link](https://arxiv.org/abs/1710.08864)  
與一般的adversarial attack不同的是adversarial attack要求是整張圖片的修改應小於某個值，one pixels attack則強調只能更動一個像素而沒有強度限制。只更動一個像素且半黑箱攻擊的性質使這種攻擊法在現實世界中的可行性提昇了不少，擾動效果也得到不錯的成績。  
![](https://i.imgur.com/xJkMuQe.png =300x)  

下圖為FGSM attack和one pixels attack各自的目標，左邊是限制全局變動不能超過某個值(norm infinite)，右邊是限制用了幾個pixel(norm 0)。  
![](https://i.imgur.com/qnoVagv.png =500x)  

### Contribution
* 提出了多點干擾與多點干擾的極端->單點干擾
* 使用Differential Evolution，與模型是分開的，不需要算梯度，因此可以做到半黑箱攻擊(要知道model的輸出分類機率)
* 由於不用算梯度，不用管網路架構，因此彈性很大


### Differential Evolution
重複以下三步驟：  
```
1.Intitialize Candidates 
2.Select and Generate 
3.Test and Substitute
```
例如：M = x^2 + y^2 + z^2 and –3 <= x,y,z <= 3, 要找最佳解  
```
Initial:
[x,y,z] = [1,1.2,-0.5], [1.7,-2.2,0.8]........
```
先隨機生成-3~3之間的x,y,z，將每個vertor都成是parents，再另外選三個vectors  
```
Candidate(Parents) = [1, 1.2, -0.5]
Three vectors = [ 0.4, 0.8, 0.9] , [ 0.2, 1.3, 2.9] , [ 1.3, -0.7, -2.1]
```
接著帶入以下公式，可以視為是交配過程，F是自己設定的參數，此篇論文設定為0.5  
![](https://i.imgur.com/MrNHF5u.png =300x)  
最後可得出下一代為`[-0.15, 1.8, 3.4]`，接著要做個clip，把超出範圍的限制在正負3，最後得到`[-0.15, 1.8, 3]`
然後要決定是否被遺傳下去，從0~1之間隨機選三個數字，設定一個值，若大於這個值則為True，反之為False，可以得到一個隨機的vector，例如`[True,False,True]`，接著看parent`[1, 1.2, -0.5]`若該位置為True則替換成子代，最終得到`[-0.48, 1.2, 3]`，帶回目標函式，如果是更佳解則取代原本parent的位置，如果沒有變更好則直接丟掉這個子代。
![](https://i.imgur.com/7kEVxSl.gif =350x)  

### How to use DE on one pixel attack ?
要找到一張圖片的最佳攻擊點太花時間了，因此要使用DE去找。那具體要如何應用？其實一張圖片是RGB三層疊在一起形成的，結合該點在圖片中的位置(x,y座標)，可以得到要求最佳化的vector是`[X,Y,R,G,B]`，此論文設定共跑100代，每代400人，有early stop，targeted attack設定為目標類超過9成的Confidence(後面會提到)，non-targeted attack設定為正確答案低於5%的Confidence。  
![](https://i.imgur.com/nR26cEX.png =200x)  


### Dataset
* [imageNET](http://www.image-net.org/)：此專案以手動注釋標註了1400多萬張圖像，並在至少100萬張圖像中提供了邊框。ImageNet包含2萬多個典型類別，例如「氣球」或「草莓」，每一類包含數百張圖像。實際圖像不歸ImageNet所有，但可以直接從ImageNet免費獲得標註的第三方圖像URL。目前CV的[成果](https://chtseng.wordpress.com/2017/11/20/ilsvrc-%E6%AD%B7%E5%B1%86%E7%9A%84%E6%B7%B1%E5%BA%A6%E5%AD%B8%E7%BF%92%E6%A8%A1%E5%9E%8B/)。  
* [CIFAR-10](https://www.kaggle.com/c/cifar-10)：Kaggle上的開源dataset，總共有六萬張圖片，共有十類，分別是飛機、車、鳥、貓、鹿、狗、青蛙、馬、船和卡車。目前CV的[成果](http://rodrigob.github.io/are_we_there_yet/build/classification_datasets_results.html#43494641522d3130)。  


### Targeted attacks & non-targeted attacks
Targeted attacks目標是讓某張圖片被辨認為指定類別，non-targeted attacks目標是讓某張圖片辨認錯誤。在後面的實驗中對imageNET使用non-targeted attack，對CIFAR-10使用targeted attack 和 non-target attack。  

關於target訓練的問題，假如今天要做出一個狗的目標攻擊，也就是將目標認成狗，那就是拿輸出vector狗的label機率來做差分演化法的回饋(找最大值)，而non-target問題可以求正確label的機率之最小值來做差分演化法的回饋。  

### Evaluation and Results
四個判斷效果的指標：  
* **Success Rate**：在非目標攻擊的情況下，被定義為對抗影象被成功分類成任意其他類別的比例。在目標攻擊的情況下，被定義為將影象擾動到指定目標類別的比例。
* **Adversarial Probability Label (Confidence)**：累加每次成功擾動為目標類別的機率值，除以成功擾動的總數。表示模型對對抗影象產生“誤分類”的confidence。例如模型認為是狗的機率(正確答案不是狗)的機率平均為p。
* **Number of Target Classes**：計算擾動到各類別的成功數量以及不被擾動的數量，可用來評估非目標攻擊。後面用熱點圖來表示。
* **Number of Original-Target Class Pairs**：計算原始目標類別對被攻擊的次數。

### Adversarial Object
All Convolution Network(2014)：
一般CNN包含三部分：卷積、池化、全連結輸出，此網路想探討池化的作用，嘗試用增加stride來代替池化，設計出了只使用卷積的網路，首先池化的理由可歸類為三點，其一是縮放不變性、其二是更加清楚抓到大特徵避免細節化、其三減少參數較好優化，ACN試圖解決其中最關鍵的第二點，藉由增加步長來代替。另外由於沒有池化，所以做deconvolution也比較容易。  
![](https://i.imgur.com/YmDIFhM.png =300x)  

Network in Network(2014)：
作者認為CNN是一個廣義線性模型，萃取的特徵都是抽象畫程度低的特徵，越抽象越能保有不變性，也就越不受外界干擾，但在現實中轉換後的資料多是以非線形流形存在，因此作者希望部分CNN之架構以非線性架構來取代。於是加入了MLPConv，試圖整合多個通道的特徵到一個全鏈結網路裡跑，每卷積一次就用MLPConv再整合一次各通道。
![](https://i.imgur.com/UkCtLZn.png =450x)  

另外一個特點是最後輸出不用全鏈結層，而是用MLpConv輸出成類別數，要分n類輸出就n維，然後採用Global Average Pooling(全局平均池化)，最後直接輸出每個feature map的平均去做softmax
![](https://i.imgur.com/Bjqgqrb.png =300x)  

VGG16 Network(2014)：
AlexNet(12年ILSVRC冠軍)的延伸，14年亞軍，主要差別就是加深AlexNet而已，間接帶出了加深網路這條概念，後面的ResNet就是以這方向創新的殘差學習。當年冠軍為GoogLeNet。  
![](https://i.imgur.com/aUEJCuU.png =300x)  
### Results
OriginAcc是該模型原本的準確率，因為參數的設定，可能沒有做優化(不是這篇論文的重點)，正確率不會像比賽時那麼高，不過85%以上的正確率也算是有一定判斷力，Non-targeted可以得到的成功率超過60%，NiN更有逼近73%的成功率，算是還不錯的結果。  

左邊三個網路使用的是CIFAR-10，BVLC是AlexNet，使用imageNet做non-targeted的攻擊，所以5.53的置信度是很厲害的，因為總共有1000類，代表模型認為是這個類別(其實他猜錯了)的機率是5.53%(這個機率足以讓他做出錯誤判斷)。  
![](https://i.imgur.com/0847cHZ.png =400x)  
3 5 pixels用的是NiN  
![](https://i.imgur.com/w4jCsh4.png =400x)  

### Is DE useful?
使用DE是否是必要的，作者用random的方式隨機選擇像素去進行隨機擾動量的攻擊，結果如下圖，沒有使用DE的效果遠不及使用DE找像素的效果，因此用DE找像素以及擾動量是必要的。  
![](https://i.imgur.com/1WdNrpF.png =400x)  

### Heat-maps
熱點圖表示了原類別變動到特定類別的數量，直排是原類別，橫排是誤判類別，由熱點圖可知對某些類別進行擾動有更高的成功率，且特別容易擾動到某個類別，例如類別5(dog)特別容易擾動到類別3(cat)，作者認為理論上應該要有對稱性，如果貓跟狗長很像，狗很容易誤認為貓，那貓應該也要很容易誤認為狗，大致上是有這樣的跡象，但仍有差很大的例外。  
![](https://i.imgur.com/gBbCbnZ.png =200x)![](https://i.imgur.com/PWXvE80.png =200x)![](https://i.imgur.com/009hYFX.png =200x)  

### Boundary
以這張熱點圖為例，類別8(ship)變動到類別0(airplane)有32個成功案例，但0變回8卻只有9個，作者認為是可能的因素。  
![](https://i.imgur.com/toYecnD.png =300x)  
可能是下面兩個因素：  
* boundary shape
* how close are natural images to the border

如果A的邊界普遍又長又細，且很靠近整張圖片的邊界，則很容易製造誤認為A的對抗圖像，而很難製造模型對應該是A而錯誤判斷成其他東西的對抗圖像。  

### Success Distribution
另外作者發現，增加pixel擾動數可以讓誤認的類別更多元，下圖可以很好的說明這種現象，對一定數量的圖片做non-targeted attack，模型誤判成的類別分佈：  
![](https://i.imgur.com/xr8tE6L.png =400x)  
![](https://i.imgur.com/xY5Book.png =400x)  
![](https://i.imgur.com/8hzXLow.png =400x)  

另外也有一些有趣的圖片剛好處在決策邊界，例如這隻狗，只要一個pixel，不同位置可以讓他被誤判成各種東西：  
![](https://i.imgur.com/DnDUMSq.png =300x)![](https://i.imgur.com/HvHfOln.png =300x)  

### Comparison
跟其他adversarial attack效果做比較，雖說效果比其他模型差，以只更動一個pixel來說，這個結果已經算不錯了。  
![](https://i.imgur.com/lgum7WX.png =400x)  
(LSA是另一種black box，FGSM是white box)  
### Success rate and image size
如果DE規模不變的話，圖片變大成功率會下降，可能是因為單個pixel的影響力就降低了。  
![](https://i.imgur.com/4k6PJIa.png =100x) ![](https://i.imgur.com/eCCNrDF.png =150x) ![](https://i.imgur.com/zMtVUP5.png =200x)  

### Future Work 
* 可以套用到其他進化演算法，可能會有更好的效果
* 把多像素攻擊也考慮進去，從不同數量的像素攻擊中找到最佳解
* 作者特別提到了在NLP上做到的應用，也許是希望可以做到曲解語意

### References 

[Adversarial Attacks名詞介紹](https://www.twblogs.net/a/5c1fbee9bd9eee16b4a73a7c)  
[Adversarial Attacks介紹](https://medium.com/trustableai/%E9%87%9D%E5%B0%8D%E6%A9%9F%E5%99%A8%E5%AD%B8%E7%BF%92%E7%9A%84%E6%83%A1%E6%84%8F%E8%B3%87%E6%96%99%E6%94%BB%E6%93%8A-%E4%B8%80-e94987742767)  
[知乎one pixel attack介紹](https://zhuanlan.zhihu.com/p/155971229)  
[Threat of Adversarial Attacks on Deep Learning in Computer Vision: A Survey](https://arxiv.org/abs/1801.00553)一篇Adversarial Attacks的survey型論文  
[上面這篇的導讀](https://kknews.cc/zh-tw/tech/gan5lk8.html)  
#### CV extended link
[NiN](https://www.cnblogs.com/shine-lee/p/11655836.html)  
[CNN演化史](https://medium.com/%E9%9B%9E%E9%9B%9E%E8%88%87%E5%85%94%E5%85%94%E7%9A%84%E5%B7%A5%E7%A8%8B%E4%B8%96%E7%95%8C/%E6%A9%9F%E5%99%A8%E5%AD%B8%E7%BF%92-ml-note-cnn%E6%BC%94%E5%8C%96%E5%8F%B2-alexnet-vgg-inception-resnet-keras-coding-668f74879306)  
[ILSVRC 歷屆的深度學習模型](https://chtseng.wordpress.com/2017/11/20/ilsvrc-%E6%AD%B7%E5%B1%86%E7%9A%84%E6%B7%B1%E5%BA%A6%E5%AD%B8%E7%BF%92%E6%A8%A1%E5%9E%8B/)  
[膠囊網路](https://www.itread01.com/content/1543983006.html)  

### Slide  
<iframe src="https://nctuitsc-my.sharepoint.com/personal/kutar4649_dif05_o365_nctu_edu_tw/_layouts/15/Doc.aspx?sourcedoc={039f5fb0-5ae0-4520-80fe-f16f00eb46e7}&amp;action=embedview&amp;wdAr=1.7777777777777777" width="610px" height="367px" frameborder="0">這是 <a target="_blank" href="https://office.com/webapps">Office</a> 提供的內嵌 <a target="_blank" href="https://office.com">Microsoft Office</a> 簡報。</iframe>