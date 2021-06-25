---
layout: post
title: Graph 3 - Graph Embedding
tags: 
  - "graph" 
  - "tech"
---
這是從之前的 Node Embedding 延伸出的議題，如果要把一張subgraph或是整張graph做embedding，應該怎麼做?  
![](https://i.imgur.com/r9WyXOb.png){: width="500" }    

### 1. Embed nodes and sum/avg
直接把subgraph中的所有node embedding相加或取平均。  
![](https://i.imgur.com/OR0I7cP.png){: width="120" }  

### 2. Create super-node and embed that node
加入一個virtual node，與subgraph的每個node都有edge，然後求這個virtual node的embedding   
![](https://i.imgur.com/cgKFEL6.png){: width="500" }    


### 3. [Anonymous Walk Embeddings ](http://proceedings.mlr.press/v80/ivanov18a.html) (Sergey Ivanov, Evgeny Burnaev, ICML 2018 )
以之前提到的Random Walk為基礎，我們將經過的node分別編號，以下圖Graph為例，若以編號表示，則ABCBC與CDBDB是相同的，把編號形式的路徑稱為Anonymous Walk。  
![](https://i.imgur.com/R1kixNi.png){: width="400" }  
![](https://i.imgur.com/KNgKpef.png){: width="500" }    
以此規則延伸，下圖為決定Walk步數後可以走出的編號可能數 :   
![](https://i.imgur.com/kUhXnZV.png){: width="500" }  
以3為例，有111,112,121,122,123種組合   
重複走m次random walk可以得到一個anonymous walks的distribution，這個m應該如何訂呢？  
![](https://i.imgur.com/gmnRCbN.png =350x)  
η (eta) 是特定長度的anonymous walks總數，用上面的圖表找到對應，例如l=7時η=877，如果ε(epsilon)設0.1，δ設0.01，則m為122500，代表有走122500次random walks來完成anonymous walks distribution。  

我們用trainable model學習圖的embedding Z_G而不是單個anonymous walk，任務定為預測某個walk是否在一個wondow內發生Co-occurrence，這裏是把每個Anonymous Walks當成單詞，window當成句子，以NLP的角度來看這任務，把所有anonymous walks排列，例如下圖是四個從node 1出發的anonymous walk排列  
![](https://i.imgur.com/Ivf7GI9.png){: width="400" }  
window是指以某個walk前後Δ的範圍，以上圖為例，若定為1，則共有四個句子，分別為{W1,W2}, {W1,W2,W3}, {W2,W3,W4}, {W3,W4}，其中在中心為W2時發生Co-occurrence，有兩個以上的1232同時出現了，以下是objective function :  
![](https://i.imgur.com/blkYPdB.png){: width="500" }   

P的算法是softmax機率  
![](https://i.imgur.com/MGHrnfF.png){: width="500" }   

整個網路架構，藉由W1,W2,W3預測W4的Anonynous Walk，然後更新weight :  
![](https://i.imgur.com/jaTRAyW.png){: width="500" }  


