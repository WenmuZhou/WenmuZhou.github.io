---
title: 数据结构之图的最短路径-Floyd算法
date: 2017/03/20 17:19:00
comments: true
categories: 
- 数据结构
- 图
tags: 
- 数据结构
- 图
- C++
---

# 问题描述
>在带有权值的图中，我们需要找到一点到另外一点所经过的边的权值之和最小，这样的一条边就是最短路径。

# 基本思想
>变量：起始点v0,终点vn,中间点vk
如果dis[v0][vn] > dis[v0][vk] + dis[vk][vn],那么久将vk作为从点v0到vn的转折点。

# 分析
>Floyd算法的本质是二重循环初始化最短路径矩阵dis，三重循环修正dis，时间复杂度为o(nxnxn)。
上一章说Dijkstra算法智能计算出特定起始点到其他点的最短路径，时间复杂度为o(n*n)，要求出所有点之间的最短路径，可以在外面套一个循环，这样时间复杂度为o(nxnxn)，就和Floyd算法一样。但是Floyd算法的实现更为巧妙。


# 代码
~~~c++
void MyGraph::shortestPathFloyd() {
    int pathMatrix[this->num][this->num];
    int shortPath[this->num][this->num];

    //初始化矩阵
    for (int i = 0; i < this->num; ++i) {
        for (int j = 0; j < this->num; ++j) {
            shortPath[i][j] = this->array[i * this->num + j];
            pathMatrix[i][j] = j;
        }
    }
    //算法主体
    for (int k = 0; k < this->num; ++k) {
        for (int i = 0; i < this->num; ++i) {
            for (int j = 0; j < this->num; ++j) {
                if (shortPath[i][k] + shortPath[k][j] < shortPath[i][j]) {
                    shortPath[i][j] = shortPath[i][k] + shortPath[k][j];
                    pathMatrix[i][j] = pathMatrix[i][k];
                }
            }
        }
    }
    //打印i到j的最短路径值和路径
    for (int i = 0; i < this->num; ++i) {
        for (int j = i + 1; j < this->num; ++j) {
            cout << this->node_array[i].data << "-->" << this->node_array[j].data << " value:" << shortPath[i][j];

            int key = pathMatrix[i][j];
            cout << " path:" << this->node_array[i].data;
            while (key != j) {
                cout<<"-->"<<this->node_array[key].data;
                key = pathMatrix[key][j];
            }
            cout<<"-->"<<this->node_array[j].data<<endl;
        }
    }
~~~
# 所用测试图结构
![最短路径.jpg](http://upload-images.jianshu.io/upload_images/1575688-4171c84728a8e325.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
# 输出结果
![Floyd算法结果.png](http://upload-images.jianshu.io/upload_images/1575688-17653cfeec16edff.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
