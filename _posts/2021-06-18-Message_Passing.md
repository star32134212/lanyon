---
layout: post
title: Graph 4 - Message Passing
tags: 
  - "graph" 
  - "tech"
---
前面介紹了Node Embeding，讓node的特徵可以在embedding space中表示，這次要討論的問題是  

> **如果拿到一張Graph以及一部分已標示label的node，要怎麼利用node embedding和現有的label找出剩下未被標示的node?**

## Homophily
研究發現，相似的node之間會有edge連接，反過來說，兩個node之間有edge代表他們的相似度可能高過沒有edge的兩個node。  

下圖是一張real world的Graph，調查一群人的興趣，node代表人，edge代表兩人是朋友關係，不同顏色代表不同興趣，可以發現同顏色的會聚集在一起。  
![](https://i.imgur.com/FT48Qvd.png){: width="500" }   
(Easley and Kleinberg,2010)  

#### Guilt-by-association
如果一個node A與一個label為X的node B有edge，那node A label也更有可能是X。  


根據Homophily的性質，以下要介紹三個方法 :  
1. **Relational classification**
2. **Iterative classification**
3. **Belief propagation**

# Relational Classification
Y_v 表示 node v 的 class probability，node v 的 Y_v 是他neighbors class probability Y 的 weighted average。  
![](https://i.imgur.com/FYutrYx.png){: width="120" }  
把label node用 ground-truth label Y_V表示(例如分兩類，那label 1 = 1, label 2 = 0)，unlabel node用 0.5表示，例如下圖 :  
![](https://i.imgur.com/Z9fd7g7.png){: width="500" }  
每個node用他的neighbor feature依序更新直到每個node都converge :  
![](https://i.imgur.com/8ZpM19u.png){: width="500" }  

等全部都converdge後(更新前後數值不變)以0.5為界分為兩類 :  
![](https://i.imgur.com/zvGeh2L.png){: width="500" }  


#### Relational Classification有兩個缺點 : 
1. 無法保證收斂
2. 無法利用node feature information 

# Iterative Classification
為了改善Relational Classification無法利用node feature而提出的方法，以下是一些會用到的定義 : 
- f_v表示node v的feature
- z_v表示summary of neighbor，也是vector，可以是鄰居的各label數、鄰居最多的label數、鄰居出現的label數種類....自訂
- φ1(f_v)表示以vector feature預測的node label
- φ2(f_v,z_v)表示以vector feature和summary of neighbors預測的node label

Iterative Classifiers 分成兩階段，以下是一個範例，要對一張有向圖使用Iterative Classification  

## phase 1
首先把label node切成training set 和 testing set，用training set訓練classifier，例如linear classifier，求出兩個model，分別求φ1(f_v)和φ2(f_v,z_v)  

以下是我們切出的一個trainning set，可以看到每個node有一個3x2的matrix，第一行是node本身的特徵，也就是f_v，第二第三行是我們自訂的z_v，因為這個範例是有向圖，這裏z_v的算法是一個二維vector，一個是input、一個是output，columns 0代表label 0，columns 1代表label 1。  

![](https://i.imgur.com/PN2JwPH.png){: width="500" }  

例如紫色匡起來的node，有兩條指向他的edge源頭分別是label 0和label 1的node，所以input vector = (1,1)，他有兩條分別指向label 0和label 1 node的edges，所以output vector = (1,1)。  

綠色圈起來的區域是f_v，用這行train一個預測φ1(f_v)的model，紅色圈起來的區域是f_v+z_v，用這三行train一過預測φ2(f_v,z_v)的model。  

## phase 2
接著使用testing set，先拿已有的model算出φ1(f_v)拿到每個node的Y，利用Y求，再用z算出φ2(f_v,z_v)，重複更新z和Y直到收斂或指定次數，此方法一樣不保證收斂。  

以下是testing set，我們有每個node的f_v和ground true(正確類別)，但還沒有label。  
![](https://i.imgur.com/9xuwANf.png){: width="500" }  

根據φ1(f_v) model得出每個node label，下圖中有一個node判斷錯誤 :  
![](https://i.imgur.com/NvfMK7b.png){: width="500" }  

把z_v拉出來看，開始調整z_v，讓用φ1(f_v) model判斷錯誤的node用φ2(f_v,z_v) model判斷正確，更新z_v，由於Y_v = φ2(f_v,z_v)，所以也會得到新的Y_v  
![](https://i.imgur.com/VHQoIz8.png){: width="500" }  

![](https://i.imgur.com/yBpePQl.png){: width="500" }  

直到converge (所有node都判斷正確) 或者是到達指定interation數 (因為Iterative Classification不保證converge)  
![](https://i.imgur.com/Xtcu2Zi.png){: width="500" }  

# Belief Propagation 
每個neighbor node互相交談，傳輸訊息，node只聽neighbor的，不管其他node，每個node收到neighbor傳來的訊息 (message)，匯集好後加上自己的feature在當成訊息傳給下一個node （要先指定好傳輸的方向)  

## Loopy BP Algorithm
![](https://i.imgur.com/K4pTlba.png){: width="500" }  
首先定義以下幾個機率 :  
- mi->j (Yj ): node i覺得 node j 屬於 class Yj 的機率 
- Ψ(Yi ,Yj ): node j屬於class Yj (Given neighbor i 屬於 class Yi)的機率 
- φi (Yi ): node i 屬於Yi 的機率
- bi (Yi): node i 被認為屬於 class Yi 的機率

一開始會將所有message初始化為1，然後開始傳遞message，以下是他們之間的關係如下 :   
硬要描述的話，node i屬於各種class的情況下node j 屬於 class j的機率乘以node i 屬於那個class的機率，再把這些值加總，這部分我覺得格式蠻像期望值的，雖然意義不一樣，後面要乘以所有鄰居傳出的message(覺得node i屬於Yi的機率)。  
![](https://i.imgur.com/iW46GBt.png){: width="200" }  

Convergence後，可以求bi (Yi)  
![](https://i.imgur.com/1u3LmnJ.png){: width="200" }  



前面的是沒有Cycle的範例，以下是如果Graph有Cycle時的狀況，這時我們無法再決定Graph傳遞message的方向，因為會在Cycle中不斷循環出不來，這會導致node i不斷地接收msg，現實中我們可以限制傳遞的message的長度來解決這件事。  
![](https://i.imgur.com/ikVoZ1B.png){: width="350" }  

Belief Propagation在有很多closed cycle的Graph中不保證converge，但其他的狀況下還是有不錯的效果。  