---
layout: post
title: Graph 9 - Graph Generation
tags: 
  - "graph" 
  - "tech"
---

在真實世界中，Graph Data的取得可能還不那麼容易，因此有人提出了Graph Generative Model，既然Graph不好取得，那就提出一個能生成合理Graph的模型。在驗證自己的某個 Graph model 是否可行時，很常會用以下方法大量產生 Graph 去測試。

要產生一個符合real world的Graph，需要先了解Graph有哪些特性。以下介紹一些名詞定義 : 
- Degree distribution P(k) : 定義P(k)表示隨機取一個node，得到degree為k的機率，算法就統計各degree的node數N_k，然後都除以總node數N
![](https://i.imgur.com/q25bLCT.png){: width="500" }
- Clustering coefficient C : node i 的鄰居之間的關係程度
**e_i 是 node i 的鄰居之間的edge數 (非node i 的 edge)**
![](https://i.imgur.com/weLJBPS.png){: width="200" }
![](https://i.imgur.com/cuate5a.png){: width="300" }
`2 * 6 / ( 4 * 3 ) = 1`
- Connectivity : 某個Graph中最大的子集，一般會用BFS找
- Path Length : 
	- Diameter : Graph中最長的path
	- Average path length : 所有path的平均長度
- Components : 內部互相可到達，外部無法到達 (一種subgraph)
- expansion α : 代表要分離出component的程度，公式如下，其中V是整張圖，S是一個subgraph
![](https://i.imgur.com/8Vkgtjh.png){: width="300" }
![](https://i.imgur.com/5wBOBtr.png){: width="300" }

### 從真實Graph歸納出的一些特性 
- degree 越高 Cluster越低
- degree 滿足 power laws
- 不會有很大的Component

## Erdös-Renyi (ER) Random Graphs
直接隨機生成，有兩種變體 : 
1. G_np : 定義n個node和edge出現的機率p
![](https://i.imgur.com/S6aIKqo.png){: width="500" }
2. G_nm : 隨機選n個nodes跟m條edges

以下以G_np為例 :  
- **Degree distribution** 
G_np 的 distribution 是 binomial，所以mean跟var也能用binomial 公式算
![](https://i.imgur.com/sREQIEQ.png){: width="400" }
![](https://i.imgur.com/hVMo502.png){: width="300" }

- **Clustering Coefficient**
首先E[ei]這個期望值可以看成是多少個pair乘以出現機率p，得到的e再帶回C公式中的e，最終可以讀到E[C]的期望值，化簡後為p，根據binomial的mean = p(n-1)，p = average degree 除以 n，又因為通常node都很大(Graph的稀疏性)，所以這個值很小。
![](https://i.imgur.com/YAyWPlI.png){: width="550" }

- **Connected Components** 
也就是Graph中最大subgraph，下圖是p和components的關係，研究發現當p = 1/(n-1)時avg degree = 1，此時就很有可能出現一個超大的Giant component。
![](https://i.imgur.com/0NI6Sjt.png){: width="550" }
左圖是模擬結果，右圖是有Giant component的範例 :
![](https://i.imgur.com/jNSndJf.png){: width="500" }

- **Shortest Path**
當n設很大時，會發現他的Shortest Path收斂。
![](https://i.imgur.com/pFs2yIp.png){: width="500" }

### Conclusion
**研究發現，ER Model 產生的real graph效果並不好，跟真實世界的graph比起來不像。**

## Small-World Model
研究發現 Real Graph 有 high clustering 的特性，因此要建立一個可以符合 high clustering 且 short path 也要符合 real world length 的圖生成模型

### step 1
Start with a low-dimensional regular lattice : 
![](https://i.imgur.com/AUMMdxn.png){: width="300" }
這裡用一個還表示，每個node與他第一層鄰居的鄰居也都是好友，這種情況下，紅色四個點是朋友的機率很高，所以有high clustering coefficient

### step 2
Rewire: Introduce randomness (“shortcuts”) : 
![](https://i.imgur.com/2rQGCTJ.png){: width="300" }
把外圍連接隔壁點的edge隨機拔掉一些去連到隨機的其他點，定義一個p代表外圍的每個edge要重新連線的機率，p越大代表diameter越小。

### Conclusion 
隨著p的變化會有以下的特性，p越接近1就會有 Low clustering 和 Low diameter 的特性
![](https://i.imgur.com/djcB1SO.png){: width="500" }

然而 clustering 和 diameter 的下降速度不一樣，會有一個範圍內的p所生成的 Graph 可以同時符合 Real Graph 的 **high clustering**, **low diameter** 的特性。
![](https://i.imgur.com/BRC11PA.png){: width="500" }

從結果來看 Small-World Model 生成的圖與 Real Graph 的相似度提升很多。

## Kronecker Graph Model
這個model的想法是根據 Real Graph 的 **Self-similarity** 特性來生成，研究認為 Graph 的某部分與他的整體很相似，所以透過複製一小部分的 Graph 來生成整張 Graph。
![](https://i.imgur.com/a84VZJ1.png){: width="500" }

使用**Kronecker product**來完成這件事，Kronecker product是就是線代的 ⊗
![](https://i.imgur.com/TOCpVLO.png){: width="500" }

做法有兩種，一種是從一小部分的 Adjency Matrix 推廣 : 
![](https://i.imgur.com/Ujd8ff5.png){: width="500" }

另一種是從一小部分的生成edge機率矩陣推廣，再用最終的機率搭配 ER Model 去生成Graph

![](https://i.imgur.com/UKWnMcT.png){: width="500" }

### Conclusion 
效果非常好，也是常用的方法。