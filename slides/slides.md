<!-- .slide: data-background-image="https://sermons.seanho.com/img/bg/unsplash-1-29wyvvLJA-maps.jpg" -->
# CMPT231
## Lecture 11: ch23-24
### Minimum Spanning Tree and Shortest Path

---
<!-- .slide: data-background-image="https://sermons.seanho.com/img/bg/unsplash-1-29wyvvLJA-maps.jpg" -->
## Isaiah 40:12-13 <span class="ref">(NASB)</span>
Who has *measured* the **waters** <br/>
in the hollow of His hand, <br/>
And *marked off* the **heavens** by the span,

And *calculated* the **dust** of the earth by the measure, <br/>
And *weighed* the **mountains** in a balance <br/>
And the **hills** in a pair of scales?

Who has *directed* the **Spirit** of the Lord, <br/>
Or as His counselor has *informed* Him?

---
<!-- .slide: data-background-image="https://sermons.seanho.com/img/bg/unsplash-1-29wyvvLJA-maps.jpg" -->
## Outline for today
+ **Minimum spanning tree** (*MST*)
  + Outline of **greedy** solutions
  + **Kruskal**'s algorithm (*disjoint-set forest*)
  + **Prim**'s algorithm (*priority queue*)
  + **Summary** of MST
+ **Single-source** shortest paths
  + Optimal **substructure**
  + **Bellman-Ford** algorithm (*allowing weight < 0*)
  + Special case for **DAG** (*no cycles*)
  + **Dijkstra**'s algorithm (*weights &ge; 0*)

---
## Minimum spanning tree
+ Input: connected, **undirected** graph *G = (V, E)*
  + Each edge has a **weight** *w(u, v)* &ge; 0
+ Output: a tree *T* &sube; E, **connecting** all vertices
  + Minimise **total weight** \`w(T) = sum\_((u,v) in T) w(u,v)\`

<div class="imgbox"><div><ul>
<li> Complexity of <strong>brute-force</strong> exhaustive search? </li>
<li> <strong>Why</strong> must <em>T</em> be a tree? </li>
<li> Num <strong>edges</strong> in <em>T</em>? <strong>Unique</strong>? </li>
</ul></div><div>
![Fig 23-1: MST](static/img/Fig-23-1.svg)
</div></div>

>>>
+ brute force: 2^|E|
+ tree = no cycles: if cycle, then can delete an edge
+ MST must have exactly |V|-1 edges
+ MST not necessarily unique

---
## Applications: power grid
[Otakar Borůvka](http://www-groups.dcs.st-and.ac.uk/history/Biographies/Boruvka.html),
Czech mathematician, <br/>
designing **electrical grid** in Moravia, 1926
<div class="imgbox"><div>
![Otakar Borůvka](static/img/boruvka.jpg)
</div><div>
![BC Hydro Transmission lines](static/img/BC-Hydro-Generating-Facilities-and-Major-Transmission-Line.gif)
</div></div>

---
## Applications: networking
[Spanning tree protocol](http://www.cisco.com/c/en/us/support/docs/lan-switching/spanning-tree-protocol/24062-146.html)
![Wireless mesh network](static/img/Wireless_mesh_network_diagram.jpg)

---
## Applications: image analysis
Image **segmentation** / registration using Renyi entropy

![MRI segmentation of meningioma](static/img/MeningiomaMRISegmentation.png)

---
## Application: dithering
Rasterise image as 3000 dots,
generate [Voronoi diagram](https://en.wikipedia.org/wiki/Voronoi_diagram)
to find nearest neighbours,
use Prim's algorithm for MST.

<div class="caption">
([Mario Klingemann](http://mario-klingemann.tumblr.com/),
[Algorithmic Art](https://www.flickr.com/photos/quasimondo/2695373627/))
</div>
![MST dithering](static/img/Klingemann-MST-dither.jpg)
<!-- .element: style="width: 80%" -->

---
## Application: genomics / proteomics
Compression of [DNA sequence DBs](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC2426707/)

<div class="imgbox"><div>
![CBP chemist reading DNA profile of imported goods](static/img/CBP-DNA_profiling.jpg)
</div><div>
[![Hemagglutinin alignments](static/img/Hemagglutinin-alignments.png)](https://commons.wikimedia.org/wiki/File:Hemagglutinin-alignments.png)
</div></div>

---
## Outline of greedy solution
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
    + To satisfy **greedy choice** property
  + *A* starts as a **subset** of some MST
  + *A* plus the edge is **still** a subset of some MST
+ **How** do we find "safe edges"?

---
## Safe edge theorem
+ Let *A* &sube; E be a **subset** of some MST
  + Let *(S, V-S)* be a **cut**: partition the vertices
  + We say that an edge *(u, v)* **crosses the cut** iff \`u in S and v in V-S\`
  + A cut **respects** *A* iff no edge in *A* crosses the cut
  + A **light edge** has *min weight* over all edges crossing the cut

<div class="imgbox"><div style="flex:2">
Theorem: <br/>
any <strong>light edge</strong> <em>(u, v)</em> <br/>
crossing a <strong>cut</strong> <em>(S, V-S)</em> <br/>
that <strong>respects</strong> <em>A</em> <br/>
is a <strong>safe edge</strong> for <em>A</em>
</div><div style="flex:3">
![Light edges: Fig 23-2](static/img/Fig-23-2.svg)
</div></div>

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
## Greedy solutions to MST
+ **Kruskal**: merge *components*: O( *|E| lg |E|* )
+ **Prim**: add *edges*: O( *|V| lg |V|* + *|E|* )
+ Simplifying **assumptions**:
  + Edge weights **distinct**:
    + Greedy algorithm still *works*
      with equal weights, but need to tweak proof
  + **Connected** graph:
    + If not, Kruksal will still produce
      minimum spanning *forest*:
    + A MST on each *component*

---
<!-- .slide: data-background-image="https://sermons.seanho.com/img/bg/unsplash-1-29wyvvLJA-maps.jpg" -->
## Outline for today
+ Minimum spanning tree
  + Outline of greedy solutions
  + **Kruskal's algorithm (disjoint-set forest)**
  + Prim's algorithm (priority queue)
  + Summary of MST
+ Single-source shortest paths
  + Optimal substructure
  + Bellman-Ford algorithm (allowing weight < 0)
  + Special case for DAG (no cycles)
  + Dijkstra's algorithm (weights &ge; 0)

---
## Kruskal's algorithm for MST
+ Initialise each **vertex** as its own **component**
+ **Merge** components by choosing **light edges**
  + Scan edge list in **increasing** order of weight
  + Ensure adding edge won't create a **cycle**
+ Use **disjoint-set** ADT (*ch21*) to track *components*
  + Operations: *MakeSet*(), *FindSet*(), *Union*()

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

---
## Kruskal: complexity
+ **Initialise** components: *|V|* calls to *MakeSet*
+ **Sort** edge list by weight: *|E| lg |E|*
+ Main **for** loop: *|E|* calls to *FindSet* + *|V|* calls to *Union*
+ **Disjoint-set** forest w/ union by rank + path compress:
  + *FindSet* and *Union* are both O( *&alpha;(|V|)* )
  + *&alpha;()*: inverse [Ackermann](https://en.wikipedia.org/wiki/Ackermann_function)
    function: <br/> very slow growth,
    &le; *4* for reasonable *n* (< \`2^(2^(2^(2^16)))\`)
+ So Kruskal is O( *&alpha;(|V|)|E|* + *|E| lg |E|* ) = O( *|E| lg |E|* )
  + Note that \`|V|-1 <= |E| <= |V|^2\`
+ If edges are **pre-sorted**, this is just O( *|E| &alpha;(|V|)* ),
  + or basically **linear** in *|E|*

---
<!-- .slide: data-background-image="https://sermons.seanho.com/img/bg/unsplash-1-29wyvvLJA-maps.jpg" -->
## Outline for today
+ Minimum spanning tree
  + Outline of greedy solutions
  + Kruskal's algorithm (disjoint-set forest)
  + **Prim's algorithm (priority queue)**
  + Summary of MST
+ Single-source shortest paths
  + Optimal substructure
  + Bellman-Ford algorithm (allowing weight < 0)
  + Special case for DAG (no cycles)
  + Dijkstra's algorithm (weights &ge; 0)

---
## Prim's algorithm for MST
+ Start from arbitrary **root** *r*
+ Build tree by adding **light edges** crossing \`(V\_A, V-V\_A)\`
  + \`V\_A\` = vertices **incident** on *A*
+ Use **priority queue** *Q* to store vertices in \`V-V\_A\`:
  + **Key** of vertex *v* is \`min\_(u in V\_A) w(u,v)\`
    + Min distance from *v* to *A*
  + *Q.popMin()* returns the destination of a **light edge**
+ At each **iteration**: *A* is always a **tree**, and
  + *A* = { (*v*, *v.par*): *v* &in; *V* - {*r*} - *Q* }
  + Encode MST in the **parent** links *v.par*

---
## Prim: example
<div class="imgbox"><div style="flex:3"><pre><code data-trim>
def PrimMST( V, E, w, r ):
  Q = new PriorityQueue( V )
  Q.setPriority( r, 0 )

  while Q.notEmpty():
    u = Q.popMin()
    for v in E.adj[ u ]:
      if Q.exists( v ) and w( u, v ) < v.key:
        v.parent = u
        Q.setPriority( v, w( u, v ) )
</code></pre>
<strong>Complexity?</strong> <br/> (# calls to queue)
</div><div style="flex:2">
![Fig 23-1: Kruskal](static/img/Fig-23-1.svg)
</div></div>

---
## Prim: complexity
+ Main **while** loop: *|V|* calls to *Q.popMin*
  + and O(*|E|*) calls to *Q.setPriority*
+ Using **binary min-heap** implementation:
  + All operations are O( *lg |V|* )
  + **Total**: O( *|V| lg |V|* + *|E| lg |V|* ) = O( *|E| lg |V|* )
+ Using **Fibonacci heaps** *(ch19)* instead:
  + *Q.setPriority* takes only O(*1*) **amortised** time
  + **Total**: O( *|V| lg |V|* + *|E|* )
+ Using an unordered **array** of vertices:
  + *setPriority* takes O(*1*), but *popMin* takes O(*|V|*)
  + **Total**: \`O(|V|^2)\` (best for *dense* graphs)

---
<!-- .slide: data-background-image="https://sermons.seanho.com/img/bg/unsplash-1-29wyvvLJA-maps.jpg" -->
## Outline for today
+ Minimum spanning tree
  + Outline of greedy solutions
  + Kruskal's algorithm (disjoint-set forest)
  + Prim's algorithm (priority queue)
  + **Summary of MST**
+ Single-source shortest paths
  + Optimal substructure
  + Bellman-Ford algorithm (allowing weight < 0)
  + Special case for DAG (no cycles)
  + Dijkstra's algorithm (weights &ge; 0)

---
## Summary of MST algos
+ All following generic **greedy** outline:
  + Add one *light edge* at a time
  + *Greedy property*: doing this doesn't lock us out of finding MST
+ **Kruskal**:
  + Merges *components*
  + Uses *disjoint-set forest* ADT
  + O( *|E| lg |E|* ), or if pre-sorted edges: O( *|E|* )
+ **Prim**:
  + *BFS* while updating shortest distance to each vertex
  + Uses *Fibonacci heap* ADT for *priority queue*
    + Or *d*-way heaps, or unordered *edge list*, or ...
  + O( *|V| lg |V|* + *|E|* )

---
## Uniqueness of MST
+ In general, may be **multiple** MSTs
+ *(#23.1-6)*: if every **cut** has a **unique** light edge crossing it,
  then MST is **unique**
+ **Proof**: Let *T* and *T'* be two **MST**s of a graph
  + Let *(u,v)* &in; *T*.  Want to show *(u,v)* &in; *T'*:
+ *T* is a **tree**, so *T* - {*(u,v)*} produces a **cut**: call it *(S, V-S)*
  + Then *(u,v)* is a **light edge** crossing *(S, V-S)* (*#23.1-3*)
+ But *T'* must also **cross** the cut: call its edge *(x,y)*
  + *(x,y)* is also a **light edge** crossing *(S, V-S)*
+ By assumption, the light edge is **unique**
  + So *(u,v)* = *(x,y)*, so *(u,v)* &in; *T'*

---
<!-- .slide: data-background-image="https://sermons.seanho.com/img/bg/unsplash-1-29wyvvLJA-maps.jpg" -->
## Outline for today
+ Minimum spanning tree
  + Outline of greedy solutions
  + Kruskal's algorithm (disjoint-set forest)
  + Prim's algorithm (priority queue)
  + Summary of MST
+ **Single-source shortest paths**
  + Optimal substructure
  + Bellman-Ford algorithm (allowing weight < 0)
  + Special case for DAG (no cycles)
  + Dijkstra's algorithm (weights &ge; 0)

---
## Shortest-path
+ **Input**: directed **graph** *(V, E)* and edge **weights** *w*
+ **Output**: find **shortest paths** between all vertices
  + For any **path** \`p = {v\_i}\_0^k\`,
    its **weight** is \`w(p) = sum w(v\_(i-1), v\_i)\`
+ The **shortest-path weight** is *&delta;(u,v)* = *min( w(p) )*
  + (or *&infin;* if *v* is not **reachable** from *u*)
  + Shortest path not always **unique**

![Fig 24-2: shortest paths](static/img/Fig-24-2.svg)

---
## Applications of shortest-path
+ **GPS**/maps: turn-by-turn *directions*
  + **All-pairs**: optimise over entire *fleet* of trucks
+ **Networking**: optimal *routing*
+ **Robotics**, self-driving: *path* planning
+ *Layout* in **factories**, FPGA / **chip** design
+ Solving **puzzles**, e.g., *Rubik's Cube*: *E* = moves

<div class="imgbox"><div>
![Google self-driving car](static/img/Google-LexusRX450h-self-drive.jpg)
<!-- .element: style="width: 65%" -->
<div class="caption">
Google self-driving Lexus RX450h
</div>
</div><div>
![CPU chip design](static/img/intel-haswell-die.jpg)
<!-- .element: style="width: 65%" -->
<div class="caption">
Intel Haswell die
</div>
</div></div>

---
## Variants of shortest-path
+ Single **source**: for a given source *s* &in; V,
  + Find shortest paths to **all** other vertices in V
+ Single **destination**: fix **sink** instead
+ Single **pair**: given *u, v* &in; V
  + No better known way than to use **single-source**
+ **All-pairs**: simultaneously find paths for **all**
  possible sources and destinations *(ch25)*

<hr/>

We'll focus on *single-source* today

*All-pairs* next week

---
## Negative-weight edges
+ We've assumed all edge weights are **positive**: *w(u,v) > 0*
+ Actually, **negative** weights are not a problem
  + Just can't allow **net-negative** cycles!
+ Net-**positive** cycles are allowable
  + *Shortest* paths will never take such a cycle

![cycles in shortest-path](static/img/Fig-24-1.svg)
<!-- .element: style="width: 80%" -->

---
<!-- .slide: data-background-image="https://sermons.seanho.com/img/bg/unsplash-1-29wyvvLJA-maps.jpg" -->
## Outline for today
+ Minimum spanning tree
  + Outline of greedy solutions
  + Kruskal's algorithm (disjoint-set forest)
  + Prim's algorithm (priority queue)
  + Summary of MST
+ Single-source shortest paths
  + **Optimal substructure**
  + Bellman-Ford algorithm (allowing weight < 0)
  + Special case for DAG (no cycles)
  + Dijkstra's algorithm (weights &ge; 0)

---
## Single-source shortest paths
+ **Output**: for each vertex *v* &in; V, store:
  + *v.parent*: links form a **tree** rooted at source
  + *v.d*: shortest-path **weight** from source
    + All initially *&infin;*, except *src.d* = *0*
+ Method: **edge relaxation**
  + Will **using** the edge *(u,v)* give us a
    **shorter** path to *v*?
+ Algorithms differ in **sequence** of relaxing edges

```
def relaxEdge( u, v, w ):
  if v.d > u.d + w( u, v ):
    v.d = u.d + w( u, v )
    v.parent = u
```

---
## Shortest-path: optim substruct
+ Any **subpath** of a shortest path is itself a shortest path:
+ Let \`p = p\_(ux) + p\_(xy) + p\_(yv)\` be a **shortest path**
  from *u* &rarr; *v*:
  + So \`delta(u, v) = w(p) = w(p\_(ux)) + w(p\_(xy)) + w(p\_(yv))\`
+ Let \`p'\_(xy)\` be a **shorter** path from *x* &rarr; *y*:
  + So \`w(p'\_(xy)) < w(p\_(xy))\`
+ Then we can **swap** out \`p'\_(xy)\` for \`p\_(xy)\`:
  + Let \`p' = p\_(ux) + p'\_(xy) + p\_(yv)\`
  + So *w(p')* = \`w(p\_(ux)) + w(p'\_(xy)) + w(p\_(yv))\`
    <br/> < \`w(p\_(ux)) + w(p\_(xy)) + w(p\_(yv))\` = *w(p)*
+ This **contradicts** the assumption <br/>
  that *p* was the shortest path from *u* &rarr; *v*

---
## Properties / lemmas
+ **Triangle inequality**: *&delta;(s,v)* &le; *&delta;(s,u)* + *w(u,v)*
+ **Upper-bound**: *v.d* is monotone **non-increasing** with edge relaxations,
  and *v.d* &ge; *&delta;(s,v)* always
+ **No-path** property: if *&delta;(s,v)* = *&infin;*, then *v.d* = *&infin;* always
+ **Convergence** property:
  + If *s* &#x21DD; *u* &rarr; *v* is a **shortest path**,
    and *u.d* = *&delta;(s,u)*,
  + then after **relaxing** *(u,v)*, we have *v.d* = *&delta;(s,v)*
  + (due to **optimal substructure**: *&delta;(s,u)* + *w(u,v)* = *&delta;(s,v)*)
+ **Path relaxation** property:
  + If \`p = (v\_0=s, v\_1, ..., v\_k=v)\` is a **shortest path** to *v*,
  + Then after **relaxing** its edges in **order**, *v.d* = *&delta;(s,v)*

---
<!-- .slide: data-background-image="https://sermons.seanho.com/img/bg/unsplash-1-29wyvvLJA-maps.jpg" -->
## Outline for today
+ Minimum spanning tree
  + Outline of greedy solutions
  + Kruskal's algorithm (disjoint-set forest)
  + Prim's algorithm (priority queue)
  + Summary of MST
+ Single-source shortest paths
  + Optimal substructure
  + **Bellman-Ford algorithm (allowing weight < 0)**
  + **Special case for DAG (no cycles)**
  + Dijkstra's algorithm (weights &ge; 0)

---
## Bellman-Ford algo for SSSP
+ Allows **negative-weight** edges
  + If any **net-negative** cycle is reachable, returns *FALSE*
+ **Relax** every edge, *|V|-1* times (**complexity**?)
+ Guaranteed to **converge**: shortest paths &le; *|V|-1* edges
  + Each **iteration** relaxes one edge along shortest path

<div class="imgbox"><div><pre><code data-trim>
def initSingleSource( V, E, src ):
  for v in V:
    v.d = &infin;
  src.d = 0

def ssspBellmanFord( V, E, w, src ):
  initSingleSource( V, E, src )
  for i in 1 .. |V| - 1:
    for (u,v) in E:
      relaxEdge( u, v, w )
  for (u,v) in E:
    if v.d > u.d + w( u, v ):
      return FALSE
</code></pre></div><div>
![Bellman-Ford: Fig 24-4(b)](static/img/Fig-24-4b.png)
</div></div>

---
## Single-source in DAG
+ Directed **acyclic** graph: no worries about cycles
+ Pre-sort vertices by **topological sort**:
  + Edges of **all** paths are relaxed **in order**
  + Don't need to **iterate** *|V|-1* times over all edges

<div class="imgbox"><div style="flex:2"><pre><code data-trim>
def ssspDAG( V, E, w, src ):
  initSingleSource( V, E, src )
  topologicalSort( V, E )
  for u in V:
    for v in E.adj[ u ]:
      relaxEdge( u, v, w )
</code></pre></div><div style="flex:3">
![topological sort: Fig 24-5(a)](static/img/Fig-24-5a.png)
</div></div>

---
<!-- .slide: data-background-image="https://sermons.seanho.com/img/bg/unsplash-1-29wyvvLJA-maps.jpg" -->
## Outline for today
+ **Minimum spanning tree** (*MST*)
  + Outline of **greedy** solutions
  + **Kruskal**'s algorithm (*disjoint-set forest*)
  + **Prim**'s algorithm (*priority queue*)
  + **Summary** of MST
+ **Single-source** shortest paths
  + Optimal **substructure**
  + **Bellman-Ford** algorithm (*allowing weight < 0*)
  + Special case for **DAG** (*no cycles*)
  + **Dijkstra**'s algorithm (*weights &ge; 0*)

---
<!-- .slide: data-background-image="https://sermons.seanho.com/img/bg/unsplash-1-29wyvvLJA-maps.jpg" class="empty" -->
