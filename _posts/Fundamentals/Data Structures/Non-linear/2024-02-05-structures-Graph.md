---
title: "Data Structures : Graph"

categories:
    - data-structures

tags:
    - [Data Structures, C++, Non-linear Data Structures, Graph]

toc: true
toc_label: "목차"
toc_sticky: false
classes: wide

date: 2024-02-05
---

# Graph

> 이 포스트는 C로 배우는 쉬운 자료구조를 바탕으로 작성되었습니다.

## Concept
Graphs are typical examples of **non-linear** data structures

### Network
You can think of a graph as a network which consists of various elements.
- each element is regarded as a **vertex**
- and a vertex is connected to another vertex through the **edge**

### Path
**Path** is a possible way from the existing edges
- the **length** of the path is the **number** of the edges of the path
- **simple** path is the path which does **not** have any **duplicate vertex**
    * **cyclic** graph
        + has the **simple path** and
        + the **first** vertex of its path is **equal** to the **last** vertex
    * **acyclic** graph doesn't satisfy the conditions above

### Degree
The **degree** of the vertex is different based on whether the graph has the **direction** or not
- if it's **directed** graph, then
    * the **degree** of the vertex = **out-degree** + **in-degree**
        + where out-degree is the **number** of edges which have the vertex as their **tail** and
        + in-degree is the **number** of edges which have the vertex as their **head**
- if it's **undirected** graph, then
    * the **degree** of the vertex = the **number** of edges of the vertex

### Cetegories of the Graph
- **Undirected** Graph
    * its edge does **not** have the **direction**
    * `(A, B)` is **equal** to `(B, A)`
- **Directed** Graph
    * its edge does **have** the **direction**
    * `<A, B>` is **not equal** to `<B, A>`
- **DAG**
    * it stands for directed acylic graph
- **Connected** Graph
    * its vertices are **all accessible**
- **Disconnected** Graph
    * it's **not** possible to **access every vertex**
- **Complete** Graph
    * it has the **maximum edges**
        + which means that each vertex is connected to every other vertex
    * if it's **undirected** graph, then
        + the maxium number of edges = `N(N-1)/2`
        + where `N` is the number of vertices
    * if it's **directed** graph, then
        + the maxium number of edges = `N(N-1)`
        + where `N` is the number of vertices
- **Weighted** Graph
    * its edge has a **weight** (it could be a value or a cost)
- **Subgraph**
    * it can be created by **removing** certain vertex(s) or edge(s)
- **Finite** Graph
    * it is a graph in which the vertex set and the edge set are **finite sets**


## Core Operations

### connect()
- connects to the given vertex

### disconnect()
- removes the connection to the given vertex


## Graph Traversal
Graph traversal(accessing to every element) is implemented through the **non-member function**.
- in order to perfrom the traversal, you have to arbitrarily pick the **starting point**
    * because graph doesn't have the root node
- moreover, there should be a `visit flag` in the data field to implement the traversal
- graph traversal is also called as **graph search**
    * there are 2 types of graph search
        + **Breadth First Search**
        + **Depth First Search**

### **Breadth First Search**
BFS is implemented through the [**queue**](https://sadoe3.github.io/data-structures/structures-Queue/)
1. pick an arbitrary starting point, call it `v`, visit `v`, and enqueue it in the queue
2. check the unvisited **adjacent** vertices of the vertex `v`
    * if there is,
        + then, visit it and enqueue it in the queue
    * if there isn't
        + dequeue current `v`, and set the first vertex of the queue to be `v`
3. repeat step 2 until the number of queue becomes `0`

### **Depth First Search**
DFS is implemented through the [**stack**](https://sadoe3.github.io/data-structures/structures-Stack/)
1. pick an arbitrary starting point, call it `v`
2. visit `v` and check unvisited adjacent vertices
    * if there is,
        + then, push current `v` at top of the stack, and set the unvisited vertex to be `v`
    * if there isn't
        + set the last vertex of the stack to be `v`, and remove it from the stack
3. repeat step 2 until the number of stack becomes `0`


## Implementation
Graph can be implemented in 2 ways
- `Array`
    * **Adjacency Matrix** can be used to represent a finit graph
    * suppose that you have a cyclic graph where
        + `A` is connected to `B`, `B` is connected to `C`, `C` is connected to `D`, and `D` is connected to `A`
    * then, we can create a table like this
        ||`A`|`B`|`C`|`D`|
        |:---:|:---:|:---:|:---:|:---:|
        |`A`|0|1|0|1|
        |`B`|1|0|1|0|
        |`C`|0|1|0|1|
        |`D`|1|0|1|0|
    * if there's an edge between two vertices
        + then it's `1`
        + otherwise, it's `0`
    * also it's `0` if same vertices are met
    * then, we can use that table to create 2-dimensional array like this
        ```c++
        int graph[4][4] = { 
            {0, 1, 0, 1}, 
            {1, 0, 1, 0},
            {0, 1, 0, 1},
            {1, 0, 1, 0}
        };
        ```
- `Linked List`
    * the **vertex** is stored as the **data** field
    * the **edges** are stored as the **link** fields 

### Graph
```c++
struct Data {
	int value;
	/*other data*/
};

class Vertex {
public:
	Vertex(const Data& inputData) : data(inputData), edges(nullptr), count(0), size(4) {
		edges = new Vertex*[size];

		for (unsigned currentIndex = 0; currentIndex < size; currentIndex++)
			edges[currentIndex] = nullptr;
	}
	~Vertex() {
		delete[] edges;
	}

	void connect(Vertex* targetVertex);
	void disconnect(Vertex* targetVertex);
private:
	bool isDuplicate(Vertex* targetVertex);

	Data data;

	Vertex** edges;
	unsigned count;
	unsigned size;
};
void Vertex::connect(Vertex* targetVertex) {
	if (isDuplicate(targetVertex) == false) {
		edges[count] = targetVertex;
		count++;

		if (count == size) {
			size *= 2;
			Vertex** newEdges = new Vertex * [size];

			for (unsigned currentIndex = count; currentIndex < size; currentIndex++)
				newEdges[currentIndex] = nullptr;

			for (unsigned currentIndex = 0; currentIndex < count; currentIndex++)
				newEdges[currentIndex] = edges[currentIndex];
			delete[] edges;
			edges = newEdges;
		}

		// add this vertex to the target vertex because it's undireccted graph;
		// if you want to implement the directed graph, then remove this code
		targetVertex->connect(this);
	}
}
void Vertex::disconnect(Vertex* targetVertex) {
	unsigned targetIndex = 0;
	for (; targetIndex < count; targetIndex++) {
		if (edges[targetIndex] == targetVertex)
			break;
	}

	if (targetIndex != count) {
		for (unsigned currentIndex = targetIndex; currentIndex < count; currentIndex++)
			edges[currentIndex] = edges[currentIndex + 1];

		count--;

		// remove this vertex from the target vertex because it's undireccted graph;
		// if you want to implement the directed graph, then remove this code
		targetVertex->disconnect(this);

		// this code exists for test purpose only
		std::cout << "removal succeeded" << std::endl;
	}
}
bool Vertex::isDuplicate(Vertex* targetVertex) {
	for (unsigned currentIndex = 0; currentIndex < count; currentIndex++) {
		if (edges[currentIndex] == targetVertex)
			return true;
	}
	return false;
}




// some codes
Vertex* a = new Vertex({ 3 });
Vertex* b = new Vertex({ 1 });

a->connect(b);
b->disconnect(a);


delete a;
delete b;
/*
print result
removal succeeded
removal succeeded
*/
```

### BFS
```c++
class Vertex {
	friend void printGraphBFS(Vertex*);
	friend void resetVisitFlagBFS(Vertex*);
public:
	Vertex(const Data& inputData) : data(inputData), edges(nullptr), count(0), size(4), isVisited(false) {
		edges = new Vertex*[size];

		for (unsigned currentIndex = 0; currentIndex < size; currentIndex++)
			edges[currentIndex] = nullptr;
	}
    // other definitions
private:
	bool isVisited;
    // other definitions
};
```
```c++
void printGraphBFS(Vertex* startPoint) {
	Vertex* v = startPoint;
	// Queue is implemented by using the previous post regarding queue
	Queue<Vertex*> verticesQueue;
	
	std::cout << v->data.value << " ";
	v->isVisited = true;
	verticesQueue.push(v);

	while (true) {
		for (unsigned currentIndex = 0; currentIndex < v->count; currentIndex++) {
			if (v->edges[currentIndex]->isVisited == false) {
				std::cout << v->edges[currentIndex]->data.value << " ";
				v->edges[currentIndex]->isVisited = true;
				verticesQueue.push(v->edges[currentIndex]);
			}
		}

		try {
			v = verticesQueue.pop();
		}
		catch (std::exception) {
			std::cout << "\n" << std::endl;
			break;
		}
	}
}

void resetVisitFlagBFS(Vertex* startPoint) {
	Vertex* v = startPoint;
	// Queue is implemented by using the previous post regarding queue
	Queue<Vertex*> verticesQueue;

	v->isVisited = false;
	verticesQueue.push(v);

	while (true) {
		for (unsigned currentIndex = 0; currentIndex < v->count; currentIndex++) {
			if (v->edges[currentIndex]->isVisited == true) {
				v->edges[currentIndex]->isVisited = false;
				verticesQueue.push(v->edges[currentIndex]);
			}
		}

		try {
			v = verticesQueue.pop();
		}
		catch (std::exception) {
			break;
		}
	}
}
```
```c++
Vertex* a = new Vertex({ 0 });
Vertex* b = new Vertex({ 1 });
Vertex* c = new Vertex({ 2 });
Vertex* d = new Vertex({ 3 });
Vertex* e = new Vertex({ 4 });
Vertex* f = new Vertex({ 5 });
Vertex* g = new Vertex({ 1 });
Vertex* h = new Vertex({ 2 });
Vertex* i = new Vertex({ 3 });
Vertex* j = new Vertex({ 4 });
Vertex* k = new Vertex({ 5 });
Vertex* l = new Vertex({ 6 });
Vertex* m = new Vertex({ 7 });
Vertex* n = new Vertex({ 8 });
Vertex* o = new Vertex({ 9 });
Vertex* p = new Vertex({ 10 });

a->connect(b);
a->connect(c);
a->connect(d);
a->connect(e);
a->connect(f);
b->connect(g);
b->connect(h);
c->connect(i);
c->connect(j);
d->connect(k);
d->connect(l);
e->connect(m);
e->connect(n);
f->connect(o);
f->connect(p);


printGraphBFS(a);
resetVisitFlagBFS(a);
printGraphBFS(a);


delete a;
delete b;
delete c;
delete d;
delete e;
delete f;
delete g;
delete h;
delete i;
delete j;
delete k;
delete l;
delete m;
delete n;
delete o;
delete p;
/*
print result
0 1 2 3 4 5 1 2 3 4 5 6 7 8 9 10

0 1 2 3 4 5 1 2 3 4 5 6 7 8 9 10
*/
```

### DFS
```c++
class Vertex {
	friend void printGraphDFS(Vertex*);
	friend void resetVisitFlagDFS(Vertex*);
    // other definitions
};
```
```c++
void printGraphDFS(Vertex* startPoint) {
	Vertex* v = startPoint;
	// Stack is implemented by using the previous post regarding stack
	Stack<Vertex*> verticesStack;
	bool isFoundUnvisited = false;

	while (true) {
		if (v->isVisited == false) {
			std::cout << v->data.value << " ";
			v->isVisited = true;
		}

		for (unsigned currentIndex = 0; currentIndex < v->count; currentIndex++) {
			if (v->edges[currentIndex]->isVisited == false) {
				verticesStack.push(v);

				v = v->edges[currentIndex];
				isFoundUnvisited = true;
				break;
			}
		}

		if (isFoundUnvisited == false) {
			try {
				v = verticesStack.pop();
			}
			catch (std::exception) {
				std::cout << "\n" << std::endl;
				break;
			}
		}
		isFoundUnvisited = false;
	}
}
void resetVisitFlagDFS(Vertex* startPoint) {
	Vertex* v = startPoint;
	// Stack is implemented by using the previous post regarding stack
	Stack<Vertex*> verticesStack;
	bool isFoundUnvisited = false;

	while (true) {
		v->isVisited = false;

		for (unsigned currentIndex = 0; currentIndex < v->count; currentIndex++) {
			if (v->edges[currentIndex]->isVisited == true) {
				verticesStack.push(v);

				v = v->edges[currentIndex];
				isFoundUnvisited = true;
				break;
			}
		}

		if (isFoundUnvisited == false) {
			try {
				v = verticesStack.pop();
			}
			catch (std::exception) {
				break;
			}
		}
		isFoundUnvisited = false;
	}
}
```
```c++
Vertex* a = new Vertex({ 0 });
Vertex* b = new Vertex({ 1 });
Vertex* c = new Vertex({ 2 });
Vertex* d = new Vertex({ 3 });
Vertex* e = new Vertex({ 4 });
Vertex* f = new Vertex({ 5 });
Vertex* g = new Vertex({ 1 });
Vertex* h = new Vertex({ 2 });
Vertex* i = new Vertex({ 3 });
Vertex* j = new Vertex({ 4 });
Vertex* k = new Vertex({ 5 });
Vertex* l = new Vertex({ 6 });
Vertex* m = new Vertex({ 7 });
Vertex* n = new Vertex({ 8 });
Vertex* o = new Vertex({ 9 });
Vertex* p = new Vertex({ 10 });

a->connect(b);
a->connect(c);
a->connect(d);
a->connect(e);
a->connect(f);
b->connect(g);
b->connect(h);
c->connect(i);
c->connect(j);
d->connect(k);
d->connect(l);
e->connect(m);
e->connect(n);
f->connect(o);
f->connect(p);


printGraphDFS(a);
resetVisitFlagDFS(a);
printGraphDFS(a);


delete a;
delete b;
delete c;
delete d;
delete e;
delete f;
delete g;
delete h;
delete i;
delete j;
delete k;
delete l;
delete m;
delete n;
delete o;
delete p;
/*
print result
0 1 1 2 2 3 4 3 5 6 4 7 8 5 9 10

0 1 1 2 2 3 4 3 5 6 4 7 8 5 9 10
*/
```


[맨 위로 이동하기](#){: .btn .btn--primary }{: .align-right}