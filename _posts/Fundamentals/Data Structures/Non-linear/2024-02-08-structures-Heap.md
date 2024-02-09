---
title: "Data Structures : Heap"

categories:
    - data-structures

tags:
    - [Data Structures, C++, Non-linear Data Structures, Heap]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2024-02-08
---

# Heap

> 이 포스트는 C로 배우는 쉬운 자료구조를 바탕으로 작성되었습니다.

## Concept
Heaps are typical examples of **non-linear** data structures

### Heap
**Heap**s are implemented based on the [**binary trees**](https://sadoe3.github.io/data-structures/structures-Tree/#binary-tree)
- the difference is that
    * in heaps
        + whenever new node is **added** or existing node is **removed**
		+ the **root** node should be 
	        - the **maximum** or the **minimum** node
- if it has the maximum value
	* then that heap is called as **max heap**
- otherwise it’s called as **min heap**

### Max Heap and Min Heap
- for **max** heap
    * the value of the parent node `>=` the values of its child nodes
- for **min** heap
	* the value of the parent node `<=` the values of its child nodes


## Core Operations

### add()
This method follows the steps listed below 
1.	add the new node at the position of the last index
2.	compare it with its direct parent node
    * if the relationship is not proper
        + swap the parent and the new node
    * otherwise
        + the process is done
3.	repeat step 2 until the process is done

### remove()
The important point of remove() method is that only root node can be deleted
- this method follows the steps listed below
1.	swap the root node and the last element
2.	remove the root node at the position of the last index
3.	compare the last element at the position of the first index with its direct child node
    * if the relationship is not proper,
        + swap them
    * otherwise
        + the process is ended
4.	repeat step 3 until the process is done

### getRootNode()
- returns the pointer to the root node


## Implementation
Heaps can be implemented with the data structure that supports the **random access with the given index**
- the point is that
    * the **index of the root** node should be `1`
        + so that the index of the parent node `=` index / 2
        + and the indices of its child nodes `=` index * 2, (index * 2) + 1

### Min Heap
```c++

```

### Max Heap
```c++

```



[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}