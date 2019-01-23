

Dijkstra 算法是 BFS 的升级版。当一个图中的每条边都加上权值后，BFS 就没办法求一个点到另一个点的最短路径了。这时候，需要用到 Dijkstra 算法。从最基本原理上讲，把 BFS 改成 Dijkstra 算法，只需要把“队列”改成“优先队列”就可以了。

![](https://wx2.sinaimg.cn/mw690/af9e9c30ly1fvg2bx1vo9j216728hwmx.jpg)

```python
# _*_coding:utf-8_*_
import heapq
import math

gragh = {
    'A':{'B':5, 'C':1},
    'B':{'A':5, 'C':2, 'D':1},
    'C':{'A':1, 'B':2, 'D':4, 'E':8},
    'D':{'B':1, 'C':4, 'E':3, 'F':6},
    'E':{'C':8, 'D':3},
    'F':{'D':6}
}

def initDistance(gragh, node):
    distance = {node: 0}
    for vertex in gragh:
        if vertex != node:
            distance[vertex] = math.inf
    return distance

def dijkstra(gragh, node):
    pqueue = []
    heapq.heappush(pqueue, (0, node))
    seen = set()
    parent = {node : None}
    distance = initDistance(gragh, node)
    while len(pqueue)>0:
        pair = heapq.heappop(pqueue)
        dist = pair[0]
        vertex = pair[1]
        nodes = gragh[vertex].keys()
        seen.add(vertex)
        for node in nodes:
            if node not in seen:
                if dist + gragh[vertex][node] < distance[node]:
                    heapq.heappush(pqueue, (dist + gragh[vertex][node], node))
                    parent[node] = vertex
                    distance[node] = dist + gragh[vertex][node]
    return parent, distance

if __name__ == '__main__':
    parent, distance = dijkstra(gragh, 'A')
    print(parent)
    print(distance)
```



***

参考：

[ 从BFS到Dijkstra算法](https://www.bilibili.com/video/av25829980)