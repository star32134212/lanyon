---
layout: post
title: Basic Graph
tags: 
  - "graph" 
  - "tech"
---
這篇整理了在讀paper跟看教學影片時遇到的所有專有名詞，有看到新的會持續更新~

### 基本名詞
- graph : G = (V,E)
- order (階) : `|G|` = 圖的node數
- degree (度): 一個node的edge數，以deg(v)表示
![](https://i.imgur.com/s6SmGWF.png)
    - deg(A) = 4
    - avg degree 是總邊數的兩倍除以node數，因為無向會算兩次。
![](https://i.imgur.com/RPgxkfX.png)
    - 如果是有向圖就會分in,out
- density : 存在的edge / 所有可能的edge，density一般很小，avg degree較常用
![](https://i.imgur.com/aU9nxfP.png){: width="400" }
- degree centrality : degree 除以 network中的node數-1 (D/(N-1))
![](https://i.imgur.com/V9QW61P.png){: width="400" } 
- walk (路徑) : 一個點到另一個點的路徑，walk長度為經過的edge數
- closed walk (閉環路徑) : 起點跟終點一樣的路徑
- cycle : 某張圖有一個closed walk使圖的edge都最多只出現一次
- diameter : 圖中最長的walk長度，兩node的最遠距離
- distance : 兩個node的最短路徑
- component : 內部互相連接，與外部沒有連接的一個群體
- bridge : 如果某一條邊被移除後可以分出兩個component，稱這條邊為bridge
- local bridge : 如果A,B之間的邊移除後，將使A到B的距離至少增加兩個單位則稱這個邊為local bridge
- neighborhoodoverlap : A,B共同鄰居數量除以A和B的鄰居總數(若A,B是鄰居則不包含A,B)
- betweenness : 以k表示，先定義兩點i,j，i到j的所有最短路經中會經過k點的比例，假設有五條最短路經，其中2條有經過k點則為0.4，i,j為所有node組合，算出所有kij再算平均即為k
- clustering coefficient (集聚係數) : 以C(i)表示node i的任兩個朋友也是朋友的機率
    - 3/C5取2 = 3/10 = 0.
![](https://i.imgur.com/PLhb2h7.png){: width="400" }

### 一些理論
- Triadic Closure Triangle : 如果兩人有共同好友，則兩人之間也很有可能有edge
- Strong Triadic Closure Property : edge也有分強弱，有非常好的朋友(可能weight較重)也有普通朋友，若A跟B,C之間有strong edge，則B,C間應該有edge相連
- Densification Power Law (DPL) : 隨著時間的推移，邊的數量以冪律的形式隨著節點的數量增長

### Group cohesiveness 凝聚力
- clique : 任兩個node都有link的subgraph
![](https://i.imgur.com/4wywqSl.png){: width="400" } 
- N-clique : 任兩個node之間距離不超過N
**Small-world experiment**
與一個完全不認識的人之間大約6條link可以建立連線，所以對一堆node，大概設到6~7 clique就可以包含整張圖了
![](https://i.imgur.com/zWAeyPi.png){: width="400" }

- N-Club : 要考慮最短路徑小於N，走不到就不算
![](https://i.imgur.com/ErgcVUI.png){: width="200" }
3-club and 3-clique: {1,2,3,4,5,6}
3-clique only: {1,2,3,6}
**差在能不能走到**
- quasi-clique : 考慮密度而不是路徑
![](https://i.imgur.com/Snn2PtV.png){: width="400" } 
G1是一個**0.5-quasi-clique**
G2是一個**0.6-quasi-clique**

- k-plex : 每個node至少要連到n-k個node
![](https://i.imgur.com/Qx45jO6.png){: width="250" }
這是一個**3-plex**，因為5個node，每個node有5-3個相鄰node  

**在這以上都是NP complex問題，以下是polynomial time問題**

- k-core : 每個node有至少k個node相連

**k-plex 跟 k-core 的差別**
> k-plex (1) |V(gk)| = n , (2)deg(v) >= n-k-1 any v is gk
> k-core (1) deg(v) >= k any v

- k-truss : Each edge within a k-truss is contained in at least (k-2) triangles
![](https://i.imgur.com/b51BtbS.png){: width="400" }
從圖可以看到x2與之相鄰的node無法組出任一個三角形，所以不在3-truss的範圍內，x1可以組成的三角形只有一個，所以不在4-truss的範圍內，所有的node都可以在2-truss的範圍內。

### 圖的結構
- Path : 一條鏈
- Cycle : 環
- Complete Graph : 每個node跟其他node都有edge
- Bipartite Graph : 二分圖，類似NN那樣，可以找到兩個子集合使集合內沒有edge，集合之間有edge

### 矩陣
- Adjacency Matrix (相鄰矩陣) : 用矩陣表示點之間是否有連接，1是有，0是沒有，以A表示
![](https://i.imgur.com/YQHSRz5.png){: width="400" }
一些比較特別的例子
![](https://i.imgur.com/S2bvHXx.png){: width="400" } 
實際上資料匯存成的樣子是下面這種**list of edges**
![](https://i.imgur.com/34MjBna.png){: width="400" } 
- A^k : k個A相乘，會得到一個node到走k步到另一個node有幾種走法的矩陣
- Degree Matrix (度矩陣) : 以D表示 是一個diagonal matrix對角矩陣，記載每個node的degree在對角線上
- Laplacian Matrix (拉普拉斯矩陣) : 以L表示 L = D - A，也稱調和矩陣
- Spectrum : 就是一張圖的pectrum為它的Laplacian的eigenvalues
(找eigenvalues的線上計算機 ：https://matrixcalc.org/en/vectors.html)
- 歸一化矩陣 : L' = D^1/2 * L * D^1/2 = D ^1/2 * (D - A) * D ^1/2 = I - S
