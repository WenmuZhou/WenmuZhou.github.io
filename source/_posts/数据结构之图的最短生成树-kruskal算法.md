---
title: 数据结构之图的最短生成树-kruskal算法
date: 2017/03/15 21:03:00
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
>1. 每次从所有边中选取权值最小的一条边，将首尾节点加入集合T,剩余节点集合为G
>2. 从剩余节点组成的所有边中再选择一条权值最小的边，将首尾节点加入集合T,剩余节点集合为G
>3. 重复2并且保证已选出的边不构成回路，直到G中没有节点。

## 和prim的区别
kruskal的计算是基于边的，在边比较少的情况下会比较快，边比较多时，prim会更好。

例子为
![最小生成树.jpg --图来自慕课网视频](http://upload-images.jianshu.io/upload_images/1575688-3141ecc2cc6b1af5.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 代码
```c++
void MyGraph::kruskalTree() {
    //1、将连接矩阵转换为边集合
    int n,m;
    vector<MyEdge> edge_vector = transformMatrixToEdge();
    //用于判断便于边是否形成环路，下标为边的起始点，值为边的结束点
    int parent[this->num];

    for(int i = 0;i<this->num;i++){
        parent[i] = 0;
    }

    for (int j = 0; j < edge_vector.size(); ++j) {
        n = this->Find(parent,edge_vector[j].start);
        m = this->Find(parent,edge_vector[j].end);
        if(n!=m){
            parent[n] = m;
            cout<<this->node_array[edge_vector[j].start].data<<"--->"<<this->node_array[edge_vector[j].end].data<<" value: "<<edge_vector[j].value<<endl;
        }
    }
}

vector<MyEdge> MyGraph::transformMatrixToEdge() {
    vector<MyEdge> edges;
    for (int i = 0; i < this->num; ++i) {
        for (int j = i+1; j < this->num; ++j) {
            if(this->array[i*this->num+j]!=this->max_value) {
                MyEdge edge = MyEdge(i,j,this->array[i*this->num+j]);
                edges.push_back(edge);
            }
        }
    }
    for (int k = 0; k < edges.size(); ++k) {
        for (int i = k+1; i < edges.size(); ++i) {
            if(edges[k].value>edges[i].value){
                MyEdge edge = MyEdge(edges[k].start,edges[k].end,edges[k].value);
                edges[k] = edges[i];
                edges[i] = edge;
            }
        }
    }
    return edges;
}

int MyGraph::Find(int *parent, int f) {
    while(parent[f]>0){
        f = parent[f];
    }
    return f;
}
```

## 运行结果

![kruskal结果.png](http://upload-images.jianshu.io/upload_images/1575688-e9e644e262f533f1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
