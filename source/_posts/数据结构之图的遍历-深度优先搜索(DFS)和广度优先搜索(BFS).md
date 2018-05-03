---
title: 数据结构之图的遍历-深度优先搜索(DFS)和广度优先搜索(BFS)
date: 2017/03/15 11:53:00
comments: true
categories: 
- 数据结构
- 图
tags: 
- 数据结构
- 图
- C++
---

# 两种遍历
图的遍历分为深度优先搜索(Depth First Search)和广度优先搜索
## 深度优先搜索(DFS)
>顺着起始节点的一条边一直找下去，知道这条边上的节点全部被找完，然后再开始顺着另一条边寻找。

## 广度优先搜索(BFS)
>选起始节点连接的所有边，然后将这些边的尾节点中没有访问的加入待寻找节点结合T，起始点置为已访问，接着寻找T中每一个点连接的边，并将尾节点加入T，直到T中包含所有的节点并且都已访问。

# 例子说明
![深度和广度优先搜索.jpg](http://upload-images.jianshu.io/upload_images/1575688-c9ddbca5b9fafc76.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
深度优先搜索结果为：A B C E F D G H
广度优先搜索结果为：A B D C F G H E

可以看出深度优先搜索就是先一条路走到底，再去走另外的路
广度优先搜索就是一层一层的访问

# 上代码
## 深度优先搜索(DFS)
```c++
void MyGraph::depthFirstTraverse(int node_index) {
    cout << this->node_array[node_index].data << " ";
    this->node_array[node_index].is_visited = true;

    for (int i = 0; i < this->num; i++) {
        if (this->array[node_index * this->num + i] != 65535) {
            if (!this->node_array[i].is_visited) {
                depthFirstTraverse(i);
            }
        }
    }
}
```
## 广度优先搜索(BFS)
需要使用两个结合来存储上层访问的节点和下一层待访问的节点，然后递归调用  breadthFirstTraverseIndex()
```c++
void MyGraph::breadthFirstTraverse(int node_index) {
    cout << this->node_array[node_index].data << " ";
    //已经访问过的节点就不再访问
    this->node_array[node_index].is_visited = true;

    vector<int> cur;
    cur.push_back(node_index);
    this->breadthFirstTraverseIndex(cur);
}

void MyGraph::breadthFirstTraverseIndex(vector<int> previous) {
    if (previous.size() <= 0) {
        return;
    }

    vector<int> last;
    for (int i = 0; i < previous.size(); i++) {
        for (int j = 0; j < this->num; j++) {
            if (this->array[previous[i] * this->num + j] != 65535) {
                if (!this->node_array[j].is_visited) {
                    cout << this->node_array[j].data << " ";
                    //已经访问过的节点就不再访问
                    this->node_array[j].is_visited = true;
                    last.push_back(j);
                }
            }
        }
    }
    this->breadthFirstTraverseIndex(last);
}
```
