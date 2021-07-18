---
layout: post
title: Basic Adversarial Attack
tags: 
  - "graph" 
  - "tech"
---

Adversarial Attack 是一種對 neural network 的攻擊，攻擊方式是對一張圖片加一些肉眼無法判斷的 noise ，讓模型做出錯誤的判斷。首先了解 Adversarial Attack 的歷史，根據我查到的資料，最早是在 2013 年 Christian Szegedy 等人發表的一篇論文 -- Intriguing properties of neural networks [[link]](https://arxiv.org/abs/1312.6199)，他們發現對於用  ImageNet 等圖片資料集訓練出的模型會因微小的改動使模型輸出結果有很大的變化。用關鍵字在學術搜尋中找，最早看到 Adversarial Attack 這個詞差不多在 2016 年，這時的 Adversarial Attack 是針對圖片做攻擊，到了 2018 年，Daniel Zügner等人在 KDD 2018 發表的論文 -- Graph Adversarial Attack - KDD18 Adversarial Attacks on Neural Networks for Graph Data [[link]](https://arxiv.org/abs/1805.07984) 被視為 Graph Adversarial Attack 的第一篇研究，他們提出了 Nettack 攻擊模型攻擊 Graph。

**讓熊貓被判斷成長臂猿**
![](https://i.imgur.com/Z4iVDMn.png){: width="500" }

**讓停止變成綠燈**
![](https://i.imgur.com/RDjMNFn.png){: width="500" }

**在 Graph 上可以是加減邊讓沒標 node 分類錯誤**
![](https://i.imgur.com/kELlaF4.png){: width="500" }

### Adversarial Attack 根據攻擊階段可以分成兩種  
- evasion attack : 改動要送進訓練好的 model 的資料，讓他判斷錯誤，例如 [2018 GeekPwn](http://2018.geekpwn.org/zh/index.html#21)的比賽是要讓參賽者製作一個帶有 noise 的口罩戴在臉上騙過人臉辨識
- poisoning attack : 注射 noise 到 model 用來訓練的 dataset，Grpah 中特別常見， Graph 常會對當下的 Graph 做 msg passing 來判斷特定 node 

### Adversarial Attack 根據攻擊目標可以分成兩種  
- targeted attack : 讓 model 對特定類別分類錯誤成另一個特定類別，例如讓人臉辨識誤判成特定人
- non-targeted attack : 讓 model 整體正確率下降，不管是誤判成什麼

### Adversarial Attack 根據掌握的攻擊對象訊息可以分成三種 
- White box  
    - Image : 有整個 model 的構造與參數和訓練資料
    - Graph : 有整個 model 的構造與參數和 Graph 的結構、特徵、label 
- Gray box  
    - Image : 只有訓練資料
    - Graph : 只有 Graph 的結構、特徵、label 
- Black box
    - Image : 只能拿到結果，完全不知道 model 內部資訊，透過修改觀察結果學習
    - Graph : 只有 Graph 沒有 特徵和 label 

### Graph Adversarial Attack 根據攻擊行為共有三種 
- Modify node feature : 改 node 的 X
- Add or delete edges : 改 Adjacency Matrix，經過 msg passing 後影響很大
- Add new nodes : 新增一個 node (刪除 node 可以看成是把那個 node 所有 edge 都刪掉)

### Graph Adversarial Attack 根據目標是否攻擊 target node 的周遭可以分成兩種  
- undirect : 改動的 edge 不能跟 target node 有關，改動的 node 不能是 target 鄰居 
- direct : 沒限制

### 其他在 Adversarial Attack 會用到的名詞
- budget : 為了讓攻擊不被發現，會做一些限制，用最少的改動來達成效果，在 Image 會注重 noise 的大小，也有不管整體大小，直接限制改動像素數量進行高 noise 攻擊的 [one pixel attack](https://star32134212.github.io/OrangeBlog/2020/10/07/One_pixel_attack%E7%AD%86%E8%A8%98/)，在 Graph 上比較難定義 noise ，目前沒有一個固定的定法，看論文作者怎麼定義，常見的做法是直接計算次數，改邊改點都算一次，不能超過特定次，這個次數會用一些 Graph 整體資訊來定，例如 average degree
- perturbation rate : 擾動率，指得是修改後的 Graph 中，非原始邊(改動邊)數量佔原始邊(正常邊)的比例