---
title: MinHeap-Optimized Dijkstra
date: 2015-04-05 11:22:33
categories:
  - practice
tags:
  - algorithm
---

In Microsoft campus recuitment contest, there is a problem which could use [Dijkstra shortest path algorithm](http://en.wikipedia.org/wiki/Dijkstra's_algorithm) to solve. And I write a version whose time complexity is **O(n2)** in the contest, however, that makes me feel not very good. So I decide to have a good review of some basic algorithms.

## The simple version of Dijkstra

Dijkstra shortest path algorithms is the fastest single point shortest pathfinding algorithms so far. And it is very easy to program. The basic pseudo code is as follows.

```python
Node S = the Start node
Set M = the set contains those nodes have already find the shortest path to S
Set N = the set contains those nodes have not find the shortest path to S yet
Array d = the current shortest path to S
Function w(x, y) = return the distance between x and y

for each i in d:
    d(i) = infinite
d(S) = 0
for N is not empty:
    v = Extract_min(N)
    for each node i in N:
        if d(i) > d(v) + w(v, i):
            d(i) = d(v) + w(v, i)
    add v to M
```

So basic idea of Dijsktra is that we must find the current min value to the origin **S** and gradually update the value of distance from other nodes to the origin **S**. Actually, we find the only point that could be optimized is the function `Extract_min`. So the problem comes to how to extract a min value from an array.

In a very common style of thinking, we will consider the algorithm like this.

```python
function Extract_min(d):
    v, min = infinite
    for each i in d:
        if d(i) < min:
            min = d(i)
            v = i
    return i
```

This algorithm use the time complexity of **O(n)** to find a min value, is not so acceptable if the number of nodes is large. In fact, we really know a way to optimize this operation, that is a **heap**.

Heap is a kind of data structure that can get the min/max element of a set of things. The time complexity of pop/push an element is **O(logn)** is a good optimization compare to the sequential compare. In **C++**, there are also some useful functions to implement the operations.

In **C++**, there is no a practical data structure named **heap**, like `vector`, `set` or `unordered_map`. However, something related to the max heap named priority queue is an **STL** container. So how are the heap implemented? See the following functions.

```cpp
make_heap(vector<T>::iterator begin, vector<T>::iterator end);
push_heap(vector<T>::iterator begin, vector<T>::iterator end);
pop_heap(vector<T>::iterator begin, vector<T>::iterator end);
```

These three functions is all we need to implement a heap. The three functions also can accept another parameter named **Compare**, which determines two problems, max or min and which attributes are to compare. And this function is very important for us to customized the STL function so that we could use heap to optimize Dijkstra algorithm.

## Minheap-Optimized Dijkstra
Ok, let's just start to optimized the `Extract_min` function.

First, we also need to initialize a **vector**, which contains the distance of every nodes from the origin S.

```cpp
vector<pair<int, int>> d;
d.resize(node_size + 1, make_pair(0, INT_MAX));
int cnt = 0;
for(auto &kv : d) {
    k.first = ++cnt;
}
d[begin].second = 0;
```

We use the index to represent the nodes index in the graph which begins from 1. And the first node extracted from the node set must be the origin **S** itself. So we change the value of **S** in **d** to **ZERO**.

Second, we need a loop to find if there are still things in the **Set N**, and also to `Extract_min` from the set of number.

```cpp
while(!d.empty()) {
    make_heap(d.begin(), d.end(), dijkstra_min_heap);
    pop_heap(d.begin(), d.end(), dijkstra_min_heap);
    ...
}
```

These two key statements are to optimized the `Extract_min` function, the `pop_heap` function will put the min or max element to the end of the vector, so we just need to get the element from the vector's back. And `dijkstra_min_heap` is just as follow.

```cpp
bool dijkstra_min_heap(const pii &a, const pii &b) {
    return a.second > b.second;
}
```

So, when we use the **STL**, it is easy for us to implement some algorithms fast and accurate.
