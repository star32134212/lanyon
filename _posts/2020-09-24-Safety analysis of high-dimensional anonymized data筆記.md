---
layout: post
title: Safety analysis of high-dimensional anonymized data 筆記
tags: 
  - "security"
  - "tech"
---    

由於ML跟DL得到越來越多的應用，關於資料集的保護也值得重視，最近看論文偶然發現一個在探討資料保護的領域，對象是dataset而不是database，感覺這是一個蠻不錯的研究方向，如Kaggle這類的公開資料平台也需要面對敏感性資料處理的問題，這篇是在討論目前常見的幾種資料匿名化方式以及評估匿名效果，技術難度不到非常高，但卻是一個蠻重要的議題。  

## 資料匿名化
#### 差別隱私 noise addition
各attributes加上某個bias作為noise。  

#### K匿名 K-anonymity
假設要把一比代表某人的特定query藏在資料集中，那只要不要讓那筆query有明顯特徵就行了，這就是K匿名的主旨，假設k=30，則資料集中如果某個特徵是有30人都有的就保留，不符合的就去除或脫敏(歲數從23變成2X就是一種脫敏)，這樣即使攻擊者想利用此特徵抓出特定人物，也需要分辨這30人，k越高匿名性越好，但是也會對資料集的效能造成破壞，尤其不適合高維特徵資料集(應該是指多變化的特徵，性別算是少變化的特徵)，ID與QI就是要移除的目標特徵。  
[K 匿名特性與優缺點分析](https://www.itread01.com/content/1540995607.html)   

* 顯式識別符號ID：可直接分辨出這筆資料是誰的特徵
* 準識別符號QI：有高概率可知道是誰的特徵


#### 矩陣分解 Matrix decomposition
對高維資料集先進行K匿名或noise匿名後再加一層矩陣分解可以在保持其Utility的同時提升anonymization。  

## 一些反匿名攻擊
#### 背景知識攻擊
攻擊者透過其背景知識提高未被K匿名刪除的特徵之可用性。  

#### 未排序匹配攻擊
如果資料集有某種排序，依照排序判斷特定用戶的特徵。  

#### 補充資料攻擊
另外加入其他類似資料集或匿名法不同的同資料集的資料進行追蹤。  

## 匿名化效果指標
#### F-measure
>Utility = Fano / Fori

用logistic regression 和 random forests等機器學習方法得到的預期結果F，將匿名和原始資料得到的F相除可以用來評估此匿名化的效果如何  

#### Distribution distance
這裡用到了information entropy和KL divergence，透過算出每個attribute的marginal distribution可以得到該attribute的information entropy，接著可以推算KL divergence，如果匿名化資料集與原始資料集的KL divergence越大代表兩個資料集分布相差越大，也就越難被破解，這是一個評估數據匿名的指標。  
![](https://i.imgur.com/b9AG4h2.png =150x)  

## 攻擊者危險等級
#### Level 1
攻擊者拿到匿名資料以及擁有關於資料集內容的相關知識。  
#### Level 2
攻擊者拿到匿名資料以及擁有關於資料集內容的相關知識，而且還拿到了敏感attributes的[marginal distribution](https://zh.wikipedia.org/wiki/%E8%BE%B9%E7%BC%98%E5%88%86%E5%B8%83)，marginal distribution是在一個多維的概率分布中，只包含特定維度(變量)的概率分布。如果攻擊者拿到這個資料會讓匿名化效果起不到太大作用。    
![](https://i.imgur.com/H32JQQt.png =400x)  
