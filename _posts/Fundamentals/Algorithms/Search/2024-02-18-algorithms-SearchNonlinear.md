---
title: "Algorithms : Search Algorithms for Non-linear Data Structures"

categories:
    - algorithms

tags:
    - [Algorithms, C++, Search, Non-linear Data Structures, Binary Search Tree, Graph, Breadth First Search, Depth First Search]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2024-02-18
---

# Search Algorithms So Far…
Search algorithms for the data structures which use the index for their elements have been covered so far
- in this post, the search algorithms for non-linear data structures which do **not use the index** would be covered

> 이 포스트는 C로 배우는 쉬운 자료구조를 바탕으로 작성되었습니다.

# Binary Search Tree

## Concept
The search algorithm for the [**binary search tree**](https://sadoe3.github.io/data-structures/structures-BinarySearchTree/) is to use [**`.get()` method**](https://sadoe3.github.io/data-structures/structures-BinarySearchTree/#get)
- because `.get()` method is based on the traversal function from the [binary tree](https://sadoe3.github.io/data-structures/structures-Tree/#binary-tree-traversal), there are 3 possible ways to search for the target node
    * preoder
	* inorder
	* postorder


## Time Complexity
`O(h)`
- where `h` is the height of the binary search tree


## Implementation

### Binary Search Tree
[**Binary search tree**](https://sadoe3.github.io/data-structures/structures-BinarySearchTree/#node)
- `.get()` method is the core implementation to search for the target node 



# Breadth First Search

## Concept
This search algorithm is for the [**graph**](https://sadoe3.github.io/data-structures/structures-Graph/)
- you can see the detailed explanation regarding its concept through [**here**](https://sadoe3.github.io/data-structures/structures-Graph/#breadth-first-search) 


## Time Complexity
`O(v+e)`
- where `v` is the number of vertices
- and `e` is the number of edges


## Implementation

### Breadth First Search
[**Breadth first search**](https://sadoe3.github.io/data-structures/structures-Graph/#bfs)



# Depth First Search

## Concept
This search algorithm is for the [**graph**](https://sadoe3.github.io/data-structures/structures-Graph)
- you can see the detailed explanation regarding its concept through [**here**](https://sadoe3.github.io/data-structures/structures-Graph/#depth-first-search) 


## Time Complexity
`O(v+e)`
- where `v` is the number of vertices
- and `e` is the number of edges


## Implementation

### Depth First Search
[**Depth first search**](https://sadoe3.github.io/data-structures/structures-Graph/#dfs)



[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}