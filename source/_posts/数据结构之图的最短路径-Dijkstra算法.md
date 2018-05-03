---
title: 数据结构之图的最短路径-Dijkstra算法
date: 2017/03/20 15:14:00
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
>1. 从起始点v0出发，找到和v0相连的点，记录下他们之间的距离。选择距离最短的尾节点v1作为下一个起始点，并记录v1最短路径已找到
2. 从v1出发，找到和v1相连的点，记录下他们之间的距离。比较从v0直接到点vk的距离和v0->v1->vk的距离，选择较小的一个值记为v0到vk的距离，并记录vk最短路径已找到
3. 重复步骤2，直到所有的点的最短距离均找到

# 分析
>从过程中可以看到Dijkstra算法可以找到起始点到所有点的最短路径，但是如果我只需要起始点到终点Vn的最短距离，是不是可以减低一下时间复杂度呢？
事实上，这个问题在“’大话数据结构“中已经有了说明
>>Dijkstra算法的时间复杂度为o(nxn)，寻找起始点到特定点的最短距离时间复杂度也是o(nxn)。
这就好比你吃了七个包子吃饱了，但是你就开始想，我是不是可以找到一个就能吃饱的包子，很简单，把七个包子做成一个大包子就行了。


# 代码
~~~c++
void MyGraph::shortestPathDijkstra(int node_index) {
    //用于存储最短路径下标的数组 patharc[w] = 0,表示w的最短顶点为0
    int patharc[this->num];
    //用于存储到各点最短路径的权值和
    int shortPath[this->num];
    //final[w] = 1 表示求得顶点到w的最短路径，0表示还未求得
    int final[this->num];

    //初始化
    for (int i = 0; i < this->num; i++) {
        //全部顶点初始化为位置最短路径状态
        final[i] = 0;
        //和node_index相连的的顶点加上权值
        shortPath[i] = this->array[node_index * this->num + i];
        //初始化为0
        patharc[i] = 0;
    }

    shortPath[node_index] = 0;
    final[node_index] = 1;

    int k, min;
    //开始主循环，每次求得node_index到某个顶点的最短路径
    for (int j = 1; j < this->num; ++j) {//剩余未求得的顶点数，因为第一个点(起始点自身)已经确定，所以从1开始
        min = this->max_value;
        for (int i = 0; i < this->num; ++i) {
            //找到与node_index相距最近的点，下标为k
            if (!final[i] && shortPath[i] < min) {
                k = i;
                //顶点i距node_index更近
                min = shortPath[i];
            }
        }
        //设置顶点k已访问
        final[k] = 1;
        //修正当前最短路径及距离
        //如果点node_index通过点k到于k相连的点s比直接到l近，就更新点node_index到l是距离，并设置到l最近的点为k
        for (int l = 0; l < this->num; ++l) {
            if (!final[l] && (min + this->array[k * this->num + l] < shortPath[l])) {
                shortPath[l] = min + this->array[k * this->num + l];
                patharc[l] = k;
            }
        }
    }

    //输出最短路径和值
    for (int m = 1; m < this->num; ++m) {
        int key = m;
        do {
            cout << this->node_array[key].data << "<---";
            key = patharc[key];
        } while (key != node_index);
        cout << this->node_array[key].data << "  value:" << shortPath[m] << endl;
    }

}
~~~
# 所用测试图结构
![最短路径.jpg](http://upload-images.jianshu.io/upload_images/1575688-4171c84728a8e325.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
# 输出结果
![Dijkstra结果.png](http://upload-images.jianshu.io/upload_images/1575688-e8cebc9b8bdc802d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
