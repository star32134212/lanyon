---
layout: post
title: Graph 2 - Node Embedding
tags: 
  - "graph" 
  - "tech"
---
對於一張圖，我們可以獲得的資訊分成 :  
- node-level : node本身的feature和label
- edge-level : edge上的特徵
- graph-level : 圖結構，兩個node之間是否有連線

Node Embedding要討論的問題是如何把一個node的特徵map到一個embedding space好讓GNN可以運算。這個ENC就是要用來轉換的encoding function :  
![](https://i.imgur.com/bGAPUA9.png)  

## Encoder + Decoder Framework

一個node embedding的framework會包含三個部分 :  
- encoding function
- similarity function
- decoding function


### Shallow Encoding 
這個概念是指在算embedding時，會把embedding當成是查表，這種矩陣格式比較好運算及方便儲存。  

![](https://i.imgur.com/VX6ZHoE.png){: width="400" }  

Z的每個column都是一個node的embedding，v是indicator，代表特定的一個node，dot後可以得到特定node的embedding。  

### Similarity Function
encoding / decoding function是trainable function，所以整個framework最重要的是 similarity function，靠similarity function去更新encoding function和decoding function的參數。下面是similarity function與embedding之間的關係，右邊為node v與node u做dot product，還有其他種方式，例如在embedding space上取z_v和z_u的cos function。  

![](https://i.imgur.com/LOGBy9e.png){: width="400" }

接著是左邊的部分，有什麼方法可以表示兩個node很相似，是否有連接? 是否有夠多共同neighbors? 是否有相似的圖結構? 這是接下來要討論的問題  

### Node Embedding Task
首先Node Embedding是一個unsupervised/self-supervised task，因為  
1. 不使用node labels  
2. 不使用node features  
3. Task independent : 他們不是為了某個task train的，但可以用到任何task  
接著會介紹用來當作imilarity function的兩種random walk。  

## Random Walk 
首先定義 :  
- z_u是node u的embedding
- P(v|z_u)是從node u開始random walk，走到v的機率
- non-linear functions用來算機率，使用Softmax或Sigmoid

接著對於一張Graph，給定node u開始做random walk : 隨機挑一個neighbor，訪問他，然後再從他的neighbor隨機挑一個去訪問，不斷循環。  

Similarity function定為Random Walk中node u和node v同時出現的機率，如果兩個node很容易訪問到彼此，代表他們可能很相似：  
![](https://i.imgur.com/TH9IrjZ.png){: width="500" }  

這個方法的好處是只要考慮當前node的所有neighbors，不用考慮到整張圖。  

### 優化的目標 
![](https://i.imgur.com/702f4dZ.png){: width="400" }  
 
- 這裡的R是walk strategy，以這裡來講是random，我們要設法讓每個node的log(node u走到他鄰居的機率)總和越大越好。
- 我們給定random walk要走的步數，會得到一個路徑，這裡面是可以有重複node的，每個node不限只能訪問到一次，運氣好可能有兩次。
以下格式與上式相等 :  
![](https://i.imgur.com/oUnJzbv.png){: width="400" }  

而P的算法如下，是用softmax的機率，可以表示這個機率要越大越好  
![](https://i.imgur.com/arufzun.png){: width="400" }  

整合起來得到  
![](https://i.imgur.com/9kQCJ61.png)  

可以看到前後各出現一次要對所有V做一次的sigma，所以complexity = O(|V|^2)，這個演算法太重了不好用。根據一篇[paper](https://arxiv.org/pdf/1402.3722.pdf)研究，softmax的部分可以改寫成以下形式  
![](https://i.imgur.com/T6CDVfo.png)  

這裡的k是一個常數，相比於更新整個圖，這裡只拿一部分(k個)的node來更新，這方法稱為**Negative Sampling**，這樣成功去掉其中一個V，現在解決Loss function了，optimize function用Gradient Descent, Stochastic Gradient Descent就可以了。  

## node2vec
### Biased Walk
關於Walk strategy，以常見的BFS和DFS為例，兩個提供了不同角度的walk方式   
![](https://i.imgur.com/eeOreDd.png)  
- BFS提供Local microscopic view 
- DFS提供Global macroscopic view

兩個各有好處，都很重要，所以Biased Walk就是為了同時利用這兩個的好處而建的，他比Random Walk多了可控制性
他有兩個參數 :  
- return parameters p : 往回走
- In-out parameters q : 往外走

![](https://i.imgur.com/LmAu666.png)  

![](https://i.imgur.com/9sbdvMX.png)  

給定p q後每走到一個node都可以算出走到所有neighbor的機率，如果會遠離前一個來的地方，就用1/q代替，會靠近就用1/p代替，如果距離不變就用1。右邊是還沒有nomalize的結果，先得到所有的值在normalize，然後照著機率走到下一個node再重複同個步驟。  

在給出p,q以及u時就可以平行地計算所有node走到各neighbor的機率，一樣走完給定步數，然後用Stochastic Gradient Descent更新。  


## Summary
Embedding的核心目標就是讓原本兩個相似的node在embedding node space上也能反映出來，然後就是定義怎麼樣算是相似，至於相似的定義不固定，看要處理的task哪個表現好就用哪個  

