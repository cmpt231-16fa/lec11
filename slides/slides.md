# CMPT231
## Lecture 11: ch23-24
### Minimum Spanning Tree and Shortest Path

---
## Devotional

---
## Outline for today
+ Minimum spanning tree
  + Outline of generic solution
  + Kruskal's algorithm (builds a forest)
  + Prim's algorithm (builds a tree)
  + Uniqueness of MST
+ Single-source shortest paths
  + Properties / lemmas
  + Bellman-Ford algorithm (allowing weight < 0)
  + Special case for DAG (no cycles)
  + Dijkstra's algorithm (weights &ge; 0)

---
## Minimum spanning tree
+ Input: connected, **undirected** graph *G = (V, E)*,
  + with **weights** *w(u, v)* on each edge
+ Output: find a tree \`T sube E\` **connecting** all vertices
  + Minimising **total weight** \`w(T) = sum\_T w(u,v)\`
+ **Why** must *T* be a tree?  Num **edges** in *T*? **Unique**?
+ **Applications**:
  + Utilities: gas/elec/water (Borůvka, Moravia), internet
  + Image analysis: registration, OCR
