---
title: 数据结构之图的最短生成树-prim算法
date: 2017/03/15 11:19:00
comments: true
categories: 
- 数据结构
- 图
tags: 
- 数据结构
- 图
- C++
---

# 基本思想
>## 最小生成树(Minimum cost Spanning Tree)
构造连通图的最小代价生成树称为最小生成树---《大话数据结构》
通俗来说就是寻找权值最小的路径集合来连接图中所有的节点。

## prim算法
1. 将起始点(可以是图中的任意节点)加入集合G.
2. 从图中寻找到G最短的路径，加入路径集合T. 并把路径的终点加入集合G.
3. 重复步骤2，知道G中包含所有的点 or T中边数量=点的数量-1.

为了实现此算法，我们需要定义几个变量
+ 保存剩余节点到G的最短距离的集合lowcost
```C++
 int lowcost[num]  //下标为外部的点，值为到G的距离
```
+ 保存lowcost中每个边连接到G中的哪一个节点的节点集合
```C++
 int adjvex[num]  //下标为外部的点，值为G中的点
```

## 代码是在大话数据结构的基础上修改的--现在能从任意节点开始
图的初始化为
```C++
for (int i = 0; i < this->num; i++) {
        for (int j = 0; j < this->num; j++) {
            if (i == j) {
                this->array[i * num + j] = 0;
            } else {
                this->array[i * num + j] = 65535;
            };
        }
    }
```

## 算法实现

```C++
void MyGraph::primTree(int node_index) {
    int min, i, j, k, MAXVELUE = 65535;
    //保存最短边的起始点，下标为外部的点，值为当前生成树中的点
    int adjvex[this->num];
    //保存各顶点到当前最小生成树的距离(权值)，下标为外部的点，值为到当前生成树的距离
    int lowcost[this->num];

    //将和起始点相关的点之间的权值加入,并设置起始点为node_index处的点
    for (i = 0; i < this->num; i++) {
        lowcost[i] = this->array[node_index * num + i];
        adjvex[i] = node_index;
    }
    //循环剩下的点
    for (i = 0; i < this->num; i++) {
        if (i != node_index) {
            min = MAXVELUE;
            j = 0;
            k = 0;

            /* 循环全部顶点 选择最小值*/
            while (j < this->num) {
                if (lowcost[j] != 0 && lowcost[j] < min) {
                    /* 当前权值成为最小值 */
                    min = lowcost[j];
                    /* 将当前最小值的下标存入k */
                    k = j;
                }
                j++;
            }
            cout << "bian: (" << this->node_array[adjvex[k]].data << "," << this->node_array[k].data << ")" << endl;
            /* 将当前顶点的权值设置为0,表示此顶点已经加入生成树豪华套餐 */
            lowcost[k] = 0;
            //重新计算剩余点到生成树的距离
            for (j = 0; j < this->num; j++) {
                /* 如果下标为k顶点各边权值小于此前这些顶点到达生成树的最短距离 */
                if (lowcost[j] != 0 && this->array[k * this->num + j] < lowcost[j]) {
                    /* 将较小的权值存入lowcost相应位置 */
                    lowcost[j] = this->array[k * this->num + j];
                    /* 将当前最短边的起始点置为k */
                    adjvex[j] = k;
                }
            }
        }
    }

}
```

## 运行例子中的图为

![最小生成树.jpg --图来自慕课网视频](http://upload-images.jianshu.io/upload_images/1575688-3141ecc2cc6b1af5.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


以节点B开始，最终输出为
![prim结果.png](http://upload-images.jianshu.io/upload_images/1575688-7f098fa1dde5a9d4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
