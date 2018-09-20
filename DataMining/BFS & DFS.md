![BFS&DFS](https://wx3.sinaimg.cn/mw690/af9e9c30ly1fveqcshj2uj20vg0mzq40.jpg)

`BFS` 核心在于 队列，取出队列中第一个元素。

`DFS` 核心在于 堆栈，取出栈中最后被压入的元素。

```python
# _*_coding:utf-8_*_

gragh = {
    'A':['B', 'C'],
    'B':['A', 'C', 'D'],
    'C':['A', 'B', 'D', 'E'],
    'D':['B', 'C', 'E', 'F'],
    'E':['C', 'D'],
    'F':['D']
}

def bfs(gragh, node):
    queue = []
    queue.append(node)
    seen = set()
    seen.add(node)
    while len(queue)>0:
        vertex = queue.pop(0)
        nodes = gragh[vertex]
        for node in nodes:
            if node not in seen:
                queue.append(node)
                seen.add(node)
        print(vertex)

def dfs(gragh, node):
    stack = []
    stack.append(node)
    seen = set()
    seen.add(node)
    while len(stack)>0:
        vertex = stack.pop()
        nodes = gragh[vertex]
        for node in nodes:
            if node not in seen:
                stack.append(node)
                seen.add(node)
        print(vertex)

if __name__ == '__main__':
    bfs(gragh, 'A')
    dfs(gragh, 'A')
```

`BFS` 和 `DFS` 二者是队列和堆栈的体现形式。在代码实现中，唯一的区别就是：

```python
vertex = queue.pop(0)  # BFS
vertex = queue.pop()   # DFS
```

***

基于 `BFS` 的最短路径实现：

![BFS最短路径](https://wx3.sinaimg.cn/mw690/af9e9c30ly1fvg10nanyxj21480azwfo.jpg)

```python
# _*_coding:utf-8_*_

gragh = {
    'A':['B', 'C'],
    'B':['A', 'C', 'D'],
    'C':['A', 'B', 'D', 'E'],
    'D':['B', 'C', 'E', 'F'],
    'E':['C', 'D'],
    'F':['D']
}

def pathBfs(gragh, node):
    queue = []
    queue.append(node)
    seen = set()
    seen.add(node)
    parent = {node : None}
    while len(queue)>0:
        vertex = queue.pop(0)
        nodes = gragh[vertex]
        for node in nodes:
            if node not in seen:
                queue.append(node)
                seen.add(node)
                parent[node] = vertex
        # print(vertex)
    return parent

if __name__ == '__main__':
    parent = pathBfs(gragh, 'A')
    node = 'E'
    while node != None:
        print(node)
        node = parent[node]
```

