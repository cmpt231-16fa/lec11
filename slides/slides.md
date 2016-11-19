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
+ Output: find a tree *T* &sube; E **connecting** all vertices
  + Minimising **total weight** \`w(T) = sum\_T w(u,v)\`
+ **Why** must *T* be a tree?  Num **edges** in *T*? **Unique**?
+ **Applications**:
  + Utilities: gas/elec/water (Borůvka, Moravia), internet
  + Image analysis: registration, OCR

---
## Outline of generic solution
```
def MST( V, E ):
  init A = {}
  until A spans V:
    find a "safe edge" to add
    add it to A
```
+ Build up a solution *A*, one **edge** at a time
  + Loop iterates exactly *|V| - 1* times
+ What is a "**safe edge**" to add?
  + Adding it to *A* doesn't **prevent** us from finding a MST
  + *A* starts as a **subset** of some MST
  + *A* plus the edge is **still** a subset of some MST
+ **How** do we find "safe edges"?

---
## Safe edge theorem
+ Let *A* &sube; E be a **subset** of some MST
  + Let *(S, V-S)* be a **cut**: partition the vertices
  + We say that an edge *(u, v)* **crosses the cut** iff \`u in S and v in V-S\`
  + A cut **respects** *A* iff no edge in *A* crosses the cut
  + A **light edge** crossing the cut has *min weight*
    over all edges crossing the cut
+ Theorem: any **light edge** *(u, v)* crossing a cut *(S, V-S)* that
  **respects** *A* is a **safe edge** for *A*

![Safe edge theorem](static/img/safe-edge.png)

---
## Proof of safe edge theorem
+ Let *T* be a **MST**, and *A* &sube; T
  + Let *(S, V-S)* be a **cut** respecting *A*
  + Let *(u, v)* be a **light edge** crossing that cut
+ Since *T* is a **tree**, &exist; a **unique** path *u* &rarr; *v* in *T*
  + That path must **cross** the cut *(S, V-S)*:
  + Let *(x, y)* be an **edge** in the path that crosses the cut
  + Cut **respects** *A*, so *(x, y)* &notin; A
+ Since *(u, v)* is a **light edge**, *w(u,v)* &le; *w(x,y)*
+ So **swap** out the edge: *T'* = *T* - {*(x,y)*} &cup; {*(u,v)*}
  + Then *w(T')* &le; *w(T)*, so *T'* is also a **MST**
  + Also, *A* &cup; {*(u,v)*} &sube; *T'*, so *(u,v)* is **safe** for *A*

---
## Outline

---
## Kruskal's algorithm for MST
+ Initialise each **vertex** as its own **component**
+ **Merge** components by choosing **light edges**
  + Scan edge list in **increasing** order of weight
+ Use **disjoint-set** ADT to find crossing edges *(ch21)*

<div class="imgbox"><div><pre><code data-trim>
def KruskalMST( V, E, w ):
  A = empty
  for v in V:
    MakeSet( v )
  sort E by weight w
  for ( u, v ) in E:
    if FindSet( u ) != FindSet( v ):
      A = A + { ( u, v ) }
      Union( u, v )
  return A
</code></pre></div><div>
![Fig 23-1: Kruskal](static/img/Fig-23-1.svg)
</div></div>
