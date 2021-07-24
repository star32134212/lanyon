---
layout: post
title: Graph 14 - GNN Adversarial Attack paper
tags: 
  - "graph" 
  - "tech"
---

列一些 GNN Adversarial Attack paper的懶人包，大致上用 Gray / White / Black Box 分類，之後有空再整理，以下隨手記的一些 GNN Adversarial Attack paper 讀到的名詞 : 
- SSL 半監督式學習
    - Pure Semi-supervised Learning : 把Label與Unlabel data都拿去訓練，最後用一開始就分割出得testing data測試模型結果
    - Transductive Learning : 把Label與Unlabel data都拿去訓練，最後預測unlabel data的label (GRAPH 較常用)

![](https://i.imgur.com/8jjsdcp.png)

- 雙重優化 : Poisoning attack會是雙重優化問題，內層是Max，外層是Min，內層Loss要maximize是因為要讓他對某類的正確率差距越大越好(越難分到正確的類上)，外層Loss要minimize則是因為改了圖後要再train一次，要讓他在這張錯誤的圖上學得好一點(往錯誤的結果學習)

# Gray Box
## **Adversarial Attacks on Neural Networks for Graph Data**
作者：Daniel Zügner, Amir Akbarnejad, Stephan Günnemann  
來源：KDD2018 [paper link](https://arxiv.org/abs/1805.07984)  
code : [github](https://github.com/danielzuegner/nettack)  

這篇有寫過blog介紹過了，參考[這裡](https://star32134212.github.io/OrangeBlog/2021/07/19/Nettack/)  

## **Adversarial Attacks on Graph Neural Networks: Perturbations and their Patterns**
作者：DANIEL ZÜGNER, OLIVER BORCHERT, AMIR AKBARNEJAD, and STEPHAN GÜNNEMANN  
發表：June, 2020 [paper link](https://dl.acm.org/doi/pdf/10.1145/3394520)  

跟 Nettack 那篇同作者，設定了幾個指標探討加邊減邊可以讓攻擊成功，從中找出原則並分析防禦方法，並藉由這些指標知道攻擊時哪種指標的影響最大。  

![](https://i.imgur.com/EQF3W83.png)  

分成絕對指標與相對指標：  
- 絕對指標 ex : degree
- 相對指標 ex : 兩個 node 的 degree 差



## **Adversarial Attacks on Graph Neural Networks via Meta Learning**
作者：Daniel Zügner, Stephan Günnemann  
發表：Feb 2019 [paper link](https://arxiv.org/abs/1902.08412)  

- untargeted attack ，降低整體 model 的正確率
- 使用 meta gradient 來決定要改哪條邊
- [meta learning](https://biic.ee.nthu.edu.tw/blog-post/learning-to-learn-meta-learning)是把超參數也拿來學習，找出一個比較好的超參數組合，所以meta gradient是某個超參數的組合的gradient
- 把graph的結構矩陣也當成超參數，所以任務目標變成在怎樣的結構下GCN的acc最低

## **Adversarial Attacks on Graph Neural Networks via Node Injections: A Hierarchical Reinforcement Learning Approach**
作者：Yiwei Sun, Suhang Wang, Xianfeng Tang, Tsung-Yu Hsieh, Vasant Honavar  
發表：April, 2020 [paper link](https://dl.acm.org/doi/pdf/10.1145/3366423.3380149)  

- untargeted attack
- 利用增加node來攻擊而不是加邊減邊

# White Box
## **Adversarial Examples on Graph Data: Deep Insights into Attack and Defense**
作者：Huijun Wu, Chen Wang, Yuriy Tyshetskiy, Andrew Docherty, Kai Lu, Liming Zhu  
發表：IJCAI'19 March, 2019 [paper link](https://arxiv.org/abs/1903.01610)  

- 基本資訊 : White box, Direct attack, Targeted, Evasion attack, Add/Delete edges and modify features, Victim Model : GCN
- 因為Graph Data通常是 0,1，一般的gradient沒辦法正確地表示這種值變化，例如f (x) = ReLU (x − 0.5)，x從0變1(add edge)，function value會增加0.5，但是 x=0 的gradient會是 0，沒辦法得知這個操作讓gradient變了多少去做下一步
- 用了另一種gradient : `integrated gradient`，可以更好的找到該攻擊的位置
- Given: original graph G, target node v0, GCN model trained on G, perturbations budget ∆
- Goal: maximize the misclassification rate (e.g. log-probabilities/logits) on target node v0 of the GCN model 增加誤判率

連續型：  
xi是終點、xi'是起點，相減後乘以總面積  
![](https://i.imgur.com/sWXFiUb.png){: width="450" }  

離散型：  
原本有連，Aij = 1，對應到是否要remove，反之原本沒連，Aij = 0對應到是否要add  
![](https://i.imgur.com/dH22jbs.png){: width="450" }  

結論  
- 攻擊者喜歡add edge勝過remove edge或改feature
- 攻擊者喜歡連接兩個不相似的點
- 防禦方法：事先去看整個Graph，若兩個node相似度太低的話就先砍掉再去訓練，論文用的相似度指標是**Jaccard Similarity**
### Integrated gradient 積分梯度
假設今天有個大象辨識模型，鼻子長度越長越有可能是大象，但當長度到了某個程度，斜率變化就幾乎為0了，離如圖中的長度 1 m，之後斜率就幾乎不會增加了(趨近0)，這塊區域是梯度飽和區，但長度為 2 m 應該要比 1 m 更像大象，所以積分梯度被提出，改以積分來表示機率(累積機率面積/總面積)  
![](https://i.imgur.com/wAW6r7M.png){: width="450" }  

## **Topology Attack and Defense for Graph Neural Networks: An Optimization Perspective**
作者：Kaidi Xu, Hongge Chen, Sijia Liu, Pin-Yu Chen, Tsui-Wei Weng, Mingyi Hong, Xue Lin  
發表：IJCAI 2019 Jun 2019 [paper link](https://arxiv.org/abs/1906.04214)  

- 基本資訊 : Untargeted, Add/Delete edges, Victim Model : GCN
- 跟前面那篇一樣，認為Graph的特徵多是0,1，一般的gradient沒辦法很好地表示，這篇使用 projected gradient 來代替 vanilla gradient
- Given: original graph G, GCN model trained on G, perturbations budget ∆
- Goal: Minimize the predefined attack loss (negative cross entropy or Carlini/Wagner loss)  

### Projected gradient
因為會有預算限制，假設限制要在綠色區域內，從x_k算Gradient得到最好的結果更新到綠色區域外，那就找Project讓更新的點最靠近最佳點且在綠色區域內  
![](https://i.imgur.com/uFe4dJL.png){: width="450" }  

Adversarial是雙重優化問題：PGD用在外層的gradient，內層用gradient ascent  
![](https://i.imgur.com/92BtBI3.png){: width="450" }  

# Black Box
## **Adversarial Attacks on Node Embeddings via Graph Poisoning**
作者：A Bojchevski, S Günnemann  
發表：2019 [paper link](http://proceedings.mlr.press/v97/bojchevski19a.html)  

- 在不知道model的情況下，以unsupervised方式學習
- 探索矩陣分解以及Graph的頻譜去找到最適合攻擊的edge
- Poisoning，Random walk用在內層找最適合攻擊edge
![](https://i.imgur.com/5eQYlaE.png)
- 一樣有兩個挑戰：(1)Poisoning的雙重優化問題 (2)random work是不可微分的，要如何得到他的loss
- 為了要解決Random walk優化，可以計算generalized eigenvalues/vectors，並且用[deep walk](https://zhuanlan.zhihu.com/p/45167021)找到最優解 

### Random walk based embeddings
假設有一張圖，在上面多次random walks會sample好幾種node組合，去觀察這些node組合可以知道有哪些node常常一起出現，代表他們是相近的點。
![](https://i.imgur.com/ktLGbfB.png){: width="450" }  

### 雙層優化轉單層優化
![](https://i.imgur.com/Kn4wWnO.png){: width="450" }  
- Compute generalized eigenvalues/vectors (Λ/𝑈) of the graph 
- For all candidate edge flips (𝑖,𝑗) compute the change in Λ/𝑈 
- Greedily pick the top candidates leading to largest loss ℒ

## **Adversarial Attack on Graph Structured Data**
作者：Hanjun Dai, Hui Li, Tian Tian, Xin Huang, Lin Wang, Jun Zhu, Le Song  
發表：2019 [paper link](http://proceedings.mlr.press/v80/dai18b.html)  

- 動機 : 黑箱攻擊沒有label，那要怎麼讓攻擊者得知攻擊的edge是否正確
- 作法 : 用強化式學習訓練，假設改了一條edge，讓model回傳一個正回饋
- 因為RL很花時間，如果還要重train會沒效率，所以這篇是研究 **evasion attack**

## **Towards More Practical Adversarial Attacks on Graph Neural Networks**
作者：Jiaqi Ma, Shuangrui Ding, Qiaozhu Mei  
發表：NeurIPS 2020 Jun 2020 [paper link](https://arxiv.org/abs/2006.05057)  

- 動機 : 在某種條件下，GCN可以跟Random walk有連結
- 作法 : 用 random walk transition matrix 去加 edge 或改 node features
- 能操控的點應該也要有限制，因為在現實世界中越重要的人可能越難入侵，所以多了一個限制是只能選degree 在門檻以下的 node，去改他的 edge 或 feature
- 這裡的 Loss 跟之前的幾篇不一樣，這篇是錯誤的機率的機率減正確的機率，所以也是越大越好
- 攻擊的點數量不能超過 r，攻擊的點 degree 不能超過m
![](https://i.imgur.com/5aAGAK6.png){: width="450" }  
- 比黑箱更黑箱，假設連 training label 都沒有
- 因為 random walk 是不用 label 的，所以要靠 random walk
![](https://i.imgur.com/ZyYct20.png){: width="450" }  
- 紅色是 walk 的起點，顏色越接近紅色代表對起點的影響度越大，以此來連結 random walk 和 GCN
- ML 是走了 L 次後，random walk 的 transition matrix，這個式子代表走到 node I 的機率，我們無法知道 node 的 label，但可以知道某個點被走到的機率大不大，以此來預期攻擊這個點可以得到比較好的結果
![](https://i.imgur.com/5h6pJy4.png){: width="450" }  
- 論文中把某個點的重要性用 Random Walk Column Sum(RWCS 視為他的分數)表示，攻擊的點就選分數高的
