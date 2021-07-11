---
layout: post
title: Graph 10 - Community Detection
tags: 
  - "graph" 
  - "tech"
---

Community 是 Graph 中會出現的結構，是一群有相似特徵或高度連結的群體，有時會需要找出一張 Graph 中有幾個 Community ，或是可以藉由 Community 將 Graph 分解成幾個部分，分別去計算以增加演算法效率，總之， Community 不管是本身作為任務目標還是幫助演算法，都非常有用，以下介紹如何找出 Graph 中的 Community 。

###  Eage Strong
Graph 中 edge 又分 Strong 和 Weak ， Strong 的關係大家同質性高，能得到的資訊低，反而是Weak的關係能學到更多資訊，認識不同群體的人。
![](https://i.imgur.com/vEAUju6.png){: width="400" }
根據美國社會學家 Granovetter 的研究，提出了 **Triadic closure (High clustering coefficient)** ，如果B跟C有共同好友A : 
1. 那B認識C的機率會大過其他人
2. B C 兩人會更容易相信對方
3. A 讓 B C互相變成好友的機率比較高

### Eage Overlap O
定義 O_ij 表示 node i 和 node j 的 Eage Overlap ，算法是兩點所有潛在的鄰居數分之與兩點都有連線的鄰居數，例如除了 i,j 兩點以外有六個點，而 i j 除了直接想連的邊外有兩個鄰居與 i j 都有邊，所以`2/6` = `1/3`
![](https://i.imgur.com/ewZy7P5.png){: width="400" }


![](https://i.imgur.com/iyW3VJr.png){: width="340" } ![](https://i.imgur.com/T5CWEuw.png){: width="340" }
左圖是根據 edge strength 刪除 edge，右圖是根據 overlap 刪除 edge，紅色代表從low刪到high，黑色代表從high刪到low，在刪除edge strength 中，從 edge strength 低的開始刪最大的component 下降的速度較快，因為這通常會刪到跨 community 的 edge ，這種 edge 少導致兩個 community 容易斷開。同理在刪除 overlap 低的更明顯。

### Community and Modularity Q
定義 Modularity Q 代表分群效果的好壞，以下為公式 : 
Q 與某個 subgraph 的edge 總數減掉用 null model (random) 產生的 Graph 的 subgraph edge 總數，也就是**越偏離隨機生成模型生成的subgraph，越有結構性!**
![](https://i.imgur.com/R8Qdfw4.png){: width="400" }


### Null Model
node 和 degree 總數都不變，只是 edge 其中一邊從原本連接的 node 變成一個隨機 node ，node i 的 degree 是 ki ， node j 的 degree 是 kj ，則 node i 連到 node j 的機率是 kj / 2m ， edge 數的期望值為 ki * kj / 2m。(m 條 edges = 2m 條 directed edges)
![](https://i.imgur.com/LaS7ody.png){: width="400" }

整個 Graph 的 edge 數量的期望值 : 
![](https://i.imgur.com/sSqQeF4.png){: width="400" }
有此可知用 Null Model 產生的 Graph 可以保留 degree distrubution 和 total edge 數 

### Modularity Q
![](https://i.imgur.com/mFZrUic.png){: width="400" }
![](https://i.imgur.com/sx5O17b.png){: width="400" }

定義一個 group 中 edge 數與期望值的差距，以 Q 表示，範圍會在 [-1,1] ，如果是正的代表某個 group 的 edge 超過期望值，通常0.3~0.7代表有很明顯的 community structure  

可以化簡成以下形式 : 
![](https://i.imgur.com/wW2ngNl.png){: width="400" }
Aij 是 i j 之間 edge 的 attention，後面有一個indicator，ci代表 node i 所在的 community。


## Louvain Algorithm
- Greedy Algorithm : maximize modularity
- 複雜度 O(n log n)
- 可用在 weighted graph
- 可找 hierarchical communities

### Phase 1
先把每個 node 都算成是一個獨立的 community，然後對每個 node 重複做一件事，考慮把 node 與他的每個鄰居都合併成一個 community ，可以算出 modularity delta (∆𝑄) ，找出最大的∆𝑄合併，假設某個 node 已經在某個 community ，還是可以算∆𝑄，如下圖，先把他從原來 community 分開，接著對所有鄰居的 community 算 modularity 決定要去哪個， node i 有三個鄰居，現階段這三個鄰居共有兩個 community ，所以就拿這兩個做比較。
![](https://i.imgur.com/kbNqjRq.png){: width="400" }

接下來的問題是要如何算 `∆𝑄(D->i)` 和 `∆𝑄(i->C)`?

- 定義 Σin 表示 edge 兩端都在 community 內的 edge
- 定義 Σout 表示 edge 其中一端在 community 外的 edge
`Q(C)` 可以用 Σin 和 Σout 表示
![](https://i.imgur.com/eL5dcEt.png){: width="400" }

- 定義 ki,in 表示 node i 連接到特定 community 的 edge 總合，來回算兩次
![](https://i.imgur.com/w700USf.png){: width="400" }

計算`∆𝑄(i->C)` (`∆𝑄(D->i)`可以遇類似方法算)
![](https://i.imgur.com/ylWxBTH.png){: width="400" }

不斷重複以上步驟，因為是 greedy 且不做 back check ，所以可能會錯過 optimal ，但優點是速度快。

### Phase 2
將 phase 1 求得的 community 換成 super nodes ， super nodes 之間的 weight 是兩個 community 之間所有 edge 的 weight 總和。
![](https://i.imgur.com/2XQvBad.png){: width="400" }
再把 super nodes 當成一般 nodes 進行 Phase 1 的操作，重複進行到指定 community (例如一開始希望整張圖被分成 2 個 community 就進行到剩兩個為止)
![](https://i.imgur.com/nnbZ1MN.png){: width="400" }


## Community-affiliation Graph Model (AGM)
前面的方法沒有考慮到的情況是某個 node 同時屬於兩個 communities 的情況，如果有這種狀況， Louvain Algorithm 是去最大化 Modularity ，但通常 Graph 很大，這種方法會跑很久，還有一種是直接用 Louvain Algorithm 分好群後去某個 modulity 找他有沒有其他 Modularity 也很大的 community 可以去，判斷他同時屬於兩個 communities 。

有人提出了另一種想法，先定義一個產生 Graph 的 Model (Community-affiliation Graph Model)，裡面有參數可以學，然後給定 Graph，讓 AGM 去學習參數使他可以產生像 Graph 的模型
![](https://i.imgur.com/OgUVxTJ.png){: width="400" }
其中可學習的參數分別為 P, C, M ， PA 是 community A 中 node 互相有 edge 的機率，紅色 node 同時屬於 community A 和 B ，所以會做兩次，先用 PA 判定有沒有 edge 再用 PB ，最後會隨機生成右邊的 Graph ，跟餵進去的 Graph 算 Loss，最後可以從得到的參數知道 Community ，也知道有哪些同時屬於兩類的 nodes 。
![](https://i.imgur.com/b7uDvtB.png){: width="400" }

具體的算 Loss 方法如下，經過 AGM 分群後所有的 edge 應該可以得到一個生成機率，拿他跟餵進去的 Graph 比較
![](https://i.imgur.com/d60Rqx9.png){: width="400" }


## BigCLAM Model (Cluster Affiliation Model for Big Networks)
之後同個作者針對 P 做了改良，原本是同個 community 下用同個 P ，現在要改成讓他有點區別，會更好 model ，首先定義 F_uA 為 node u 與 community 的關係強度
![](https://i.imgur.com/CvNLX3u.png){: width="400" }
將 Pc(u,v) 定義為在 community 中兩點有 edge 的機率，如果u,v不在同個 C ，那其中一個 F 會是 0，P 也會變成 0。
![](https://i.imgur.com/VKbPmFl.png){: width="400" }
可以推廣到 P(u,v) ，圖中任兩點的連接機率 1 - `(兩點與所有 Community 的內積)` : 
![](https://i.imgur.com/UUfR62c.png){: width="400" }

Model 生成 input Graph 的機率，要 maximize 這個值，由於是機率，做很多次會變成浮點數不夠或很難存，所以加個 log
![](https://i.imgur.com/tWKz8YG.png){: width="400" }

接著回到前面提到的 F ，也是用 random 值開始，慢慢訓練，梯度更新
![](https://i.imgur.com/dLym5oo.png){: width="400" }

**結論** : BigCLAM 可以對指定 Graph 訓練，變成可生成此 Graph 模型，並且可以用這模型拿到 community 包含具有重疊的情況
