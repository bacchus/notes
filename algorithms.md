<a id="home"></a>
# Algorithms

* [Intro](#intro)
* [Sort](#sort)
* [Structs](#structs)
* [Graphs](#graphs)


***

<a id="intro"></a>
## Intro [home](#home)
algorithms strategies:
- incremental (a[i] + a[i+1])
- divide-n-conquer (a[0,i/2] + a[i/2,i])
- dynamic programing (cache, downres with mem, upres)
- greedy (first choose best)

#### big O notation
X(g(n)) = f(n): exist c1,c2,n0>0: for all n>=n0:

    O(g(n)): 0 <           f(n) < c2*g(n) - upper bound
    Q(g(n)): 0 < c1*g(n) < f(n) < c2*g(n) - exact bound
    W(g(n)): 0 < c1*g(n) < f(n)           - lower bound


#### divide-n-conquer: master method
T(n) = a*T(n/b) + f(n)
    
    if f(n)=O(n^(logb(a-e)))      - T(n) = Q(n^logb(a))
    if f(n)=Q(n^(logb(a)))        - T(n) = Q(n^logb(a)*log(n))
    if f(n)=W(n^(logb(a+e)))      - T(n) = Q(f(n))
        , a*f(n/b) < c*f(n), c<1    


#### dynamic programing
cache results, 2 types:
- downres with mem: recursive, remember sub-problems results
- upres: start from minimal sub-problems (i = 1,2,3...), until solve (n)

can be used if:
- exist optimal sub-structure:
    optimal solution contains optimal sub-solutions
- overlaping sub-problems:
    same sub-problems appear again and again
- sub-problems are independent, no new sub-problems appear (so-so)


#### greedy
- no need to solve all sub-problems as in dynamic programing
- each time make optimal choice

can be used if:
- exist optimal sub-structure
- can get optimal solution making local optimal(greedy) choice 


***

<a id="sort"></a>
## Sort [home](#home)
* [insert sort ](#insert)
* [bubble sort ](#bubble)
* [merge sort ](#merge)
* [heap sort ](#heap  )
* [quick sort](#quick )
* [count sort ](#count)
* [radix sort ](#radix)
* [bucket sort ](#bucket)
* [median    ](#median)


<a id="insert"></a>
#### insert sort [up](#sort)
uses incremental strategy, good for small n
- O(n^2)
- sort inplace
- no additional memory

pseudocode:

    insert_sort(A) {
        for (int j = 1; j < A.size; ++j) {
            for (i = j-1; i >= 0 && a[i] > a[j]; --i) {
                a[i+1] = a[i];
            }
            a[i+1] = a[j];
        }
    }

<a id="bubble"></a>
#### bubble sort [up](#sort)
- O(n^2)
- sort inplace
- no additional memory

pseudocode:

    bubble_sort(A) {
        for (int j = 1; j < A.size; ++j) {
            for (i = j-1; i >= 0; --i) {
                if (a[i] > a[i+1])          // compare-exchange
                    swap(a[i], a[i+1]);
            }
        }
    }


<a id="merge"></a>
#### merge sort [up](#sort)
uses divide-n-conquer strategy
- O(n*log(n))

pseudocode:

    merge(A,l,r,q) {
        L = A[l,q];
        R = A[q+1,r];
        int i = 1, j = 1;
        for (int k = l; k < r; ++k) {
            if (L[i] < R[j])            // A[k] = min(L[i], R[j])
                A[k] = L[i++];          // if L: i++
            else
                A[k] = R[j++];          // if R: j++
        }
    }
    
    merge_sort(A,l,r) {
        if (l < r) {                    // if num > 1
            q = (l+r)/2;                // pivot - middle
            merge_sort(A,l,q);          // sort left
            merge_sort(A,q+1,r);        // sort right
            merge(A,l,r,q);             // merge
        }
    }


<a id="heap"></a>
#### heap sort [up](#sort)
- O(n*log(n))

pseudocode:

    parent(i) { return i/2; }
    left(i) { return 2*i; }
    right(i) { return 2*i + 1; }
    
    max_heapify(A, i) {
        l = left(i);
        r = right(i);
        if (l <= A.heap_size && A[l] > A[i])        // largest = max(A[i], A[l], A[r])
            largest = l;
        else
            largest = i;
        if (r <= A.heap_size && A[r] > A[largest])
            largest = r;
        
        if (largest != i) {                         // if A[l] or A[r] > A[i]
            swap(A[i], A[largest]);                 // swap A[i] with largest
            max_heapify(A[largest]);                // repeat
        }
    }
    
    build_max_heap(A) {
        A.heap_size = A.length;
        for (i = A.heap_size/2; i >= 0; ++i)        // A[n/2, n] - are leaves
            max_heapify(A,i);                       // from root to nodes with leaves
    }
    
    heap_sort(A) {
        build_max_heap(A);                          // build heap
        for (i = A.length; i > 0; ++i) {            // for all backwards
            swap(A[i],A[0]);                        // A[0] - max, now last
            --A.heap_size;                          // decrease heap_size by 1
            max_heapify(A,0);                       // rebuild heap from root
        }
    }

priority queue: push, pop_max, increase_key

    pop_max(A) {
        assert(A.heap_size > 0);                    // heap not empty
        max = A[0];                                 // root is max
        A[0] = A[A.heap_size - 1];                  // swap root with last
        --A.heap_size;                              // decrease heap_size
        max_heapify(A,0);                           // rebuild heap from root
        return max;
    }
    
    increase_key(A,i,key) {
        assert(key > A[i]);                         // new key larger than old
        A[i] = key;
        while (i > 0                                // until reach root
          && A[i] > A[parent(i)]) {                 // if new key larger than parent
            swap(A[i], A[parent(i)]);               // swap i with parent
            i = parent(i);                          // repeat
        }
    }
    
    push(A, key) {
        ++A.heap_size;                              // increase heap_size
        A[A.heap_size - 1] = -inf;                  // push to last node
        increase_key(A, A.heap_size - 1, key);      // set key by increase
    }


<a id="quick"></a>
#### quick sort [up](#sort)
- O(n*log(n)) - avg
- O(n^2) - worst

pseudocode:

    partition(A,l,r) {
        swap(A[random(l,r)], A[r]);     // pivot - random node, swap with last
        x = A[r];                       // pivot - last node
        i = l - 1;
        for (j = l; j < r; ++j) {       // for all nodes
            if (A[j] <= x) {            // if node less than pivot
                ++i;                    // curent insert point
                swap(A[j], A[i]);       // place node to insert point
            }
        }
        swap(A[i + 1], A[r]);           // place pivot to insert point
        return i + 1;                   // return insert point index
    }
    
    quick_sort(A,l,r) {
        if (l < r) {                    // A.size > 1
            p = partition(A,l,r);       // partition by pivot p
            quick_sort(A,l,p);          // sort left
            quick_sort(A,p+1,r);        // sort right
        }
    }

<a id="count"></a>
#### count sort [up](#sort)
- O(n + k)
- all elements int: [0,k]
- stable sort: same elements stay in same order as before sort 

pseudocode:

    count_sort(A) {
        Res[A.size];                    // result
        C[k];                           // counter arr
        for (i = 0; i < k; ++i)
            C[k] = 0;                   // zero counters
        for (i = 0; i < A.size; ++i)
            ++C[A[i]];                  // C[i] = num of A[i]
        for (i = 0; i < k; ++i)
            C[i] += C[i - 1];           // C[i] = num of less than A[i]
        for (i = A.size; i >= 0; --i)   // backwards (for stable sort)
            Res[ C[A[j]] ] = A[j];      // place A[j] to sorted position
            --C[A[j]];                  // decrease A[j] counter
        return Res;
    }


<a id="radix"></a>
#### radix sort [up](#sort)
- O(d*(n + k))
- d-bit int numbers
- each time stable sort by bit: from lower to higher

pseudocode:

    radix_sort(A) {
        for (i = 0; i < d; ++i) {
            stable sort A by i-th bit;
        }
    }


<a id="bucket"></a>
#### bucket sort [up](#sort)
- O(n) - avg; O(n^2) - worst
- all elements - real numbers [0.0, 1.0]

pseudocode:

    bucket_sort(A) {
        n = A.size;
        Res[n];                         // n buckets - lists
        for (i = 0; i < n; ++i)         
            Res[i] = new list           // init empty lists
        for (i = 0; i < n; ++i)
            Res[n*A[i]].insert(A[i]);   // put in its bucket
        for (i = 0; i < n; ++i)
            insert_sort(Res[i]);        // sort each bucket
        join all lists Res[i] to Res    // join buckets
        return Res;
    }

<a id="median"></a>
#### median [up](#sort)
minimum O(n):

    minimum(A) {
        min = A[0];
        for (i = 1; i < A.size; ++i)
            if (A[i] < min)
                min = A[i];
    }

i-th element O(n^2) - worst, O(n) - avg:

    select(A,l,r,i) {
        if (l == r)                         // found
            return A[l];
        p = partition(A,l,r);               // random partition, p - pivot
        k = p - l + 1;                      // k-th = i-th in A[l,r]
        if (i == k)
            return A[p];                    // its pivot
        else if (i < k)                     // if i < k
            return select(A,l,p,i);         // then search in left part
        else
            return select(A,p+1,r,i-k);     // else in right, with index i-k
    }

***

<a id="structs"></a>
## Structs [home](#home)
* [bin-tree](#binary)
* [rb-tree ](#rb    )
* [fib-heap](#fib   )

<a id="binary"></a>
#### bin-tree [up](#structs)
- node: key, left, right, parent
- all nodes in left sub-tree - less than root
- all nodes in right sub-tree - more than root

print nodes in sorted order:

    inorder_tree_walk(x) {
        if (x != null) {
            inorder_tree_walk(x.left);
            print(x.key);
            inorder_tree_walk(x.right);
        }
    }

    tree_search(x,k) {
        if (x == null || k == x.key)
            return x;
        if (k < x.key)
            return tree_search(x.left,k);
        else
            return tree_search(x.right,k);
    }
    
    tree_search_iterative(x,k) {
        while (x != null && k != x.key)
            if (k < x.key)
                x = x.left;
            else
                x = x.right;
        return x;
    }
    
    tree_min(x) {
        while (x.left != null)
            x = x.left;
        return x;
    }
    
    tree_successor(x) {
        if (x.right != null)                    // if x.right not empty
            return tree_min(x.right);           // res = smallest of greater elements
        y = x.parent;                           // go up
        while (y != null && x == y.right) {     // res - most parent
            x = y;                              // , with x - its left sub-tree
            y = y.parent;
        }
        return y;
    }
    
    tree_insert(T,z) {
        y = null;
        x = T.root;
        while (x != null) {
            y = x;
            if (z.key < x.key)
                x = x.left;
            else
                x = x.right;
        }
        z.parent = y;
        if (y == null)
            T.root = z;
        else if (x.key < y.key)
            y.left = z;
        else
            y.right = z;
    }
    
    tree_delete(T, z) {
        if (z.left == null) {                   // z have no left
            transplant(T,z,z.right);
        } else if (z.right == null) {           // z have no right, but have left
            transplant(T,z,z.left);
        } else {                                // z have right and left
            y = tree_min(z.right);              // y have no left
            if (y.parent != z) {
                transplant(T,y,y.right);
                y.right = z.right;
                y.right.parent = y;
            }
            transplant(T,z,y);
            y.left = z.left;
            y.left.parent = y;
        }
    }
    
    transplant(T,x,y) {                         // replaces x by y
        if (x.parent == null)
            T.root = y;
        else if (x.parent.left == x)
            x.parent.left = y;
        else
            x.parent.right = y;
        
        if (y != null)
            y.parent = x.parent;
    }


<a id="rb"></a>
#### rb-tree [up](#structs)
- balanced bin-tree
- node: bin-tree node, color
- heigth <= 2*log(n+1)

properties:
- each node is red or black
- root is black
- each leaf(null) is black
- if node red: both child black
- for each node all paths to leaves has same black heigth
- new inserted node is red (not in main properties)

rotations:

      |                             |       
      x      --left_rot(x)-->       y       
     / \                           / \      
    a   y    <--right_rot(y)--    x   c     
       / \                       / \        
      b   c                     a   b       

pseudocode:

    left_rot(T,x) {
        y = x.right;                    // set y
        
        x.right = y.left;               // move b from y to x
        if (y.left != null)
            y.left.parent = x;          // set b parent
        
        y.parent = x.parent;            // reparent y
        if (x.parent == null)
            T.root = y;                 // - as root, if x was root
        else if (x.parent.left == x)
            x.parent.left = y;          // - as left, if x was left
        else
            x.parent.right = y;         // - as right, if x was right

        x.parent = y;                   // reparent x
        y.left = x;                     // as left of y
    }
    
    right_rot(T,y) {...}                // symmetrical

    // same as in bin-tree, fix RB properties by rotations and color change
    rb_insert() {} 
    rb_delete() {}


<a id="fib"></a>
#### fib-heap [up](#structs)
ops: insert, pop_min, union, decrease_key, remove
- node: bin-tree node, mark, degree
- root_list: ring db-linked list of root trees
- in tree: node.key >= node.parent.key
- in tree: node has ring db-linked list of children
- node has pointer to first child
- node.degree: children num
- node.mark: if node lost children since it become child of other node
- H.min - points to min node in root_list
- potential: F(H) = root_list.size + 2*marked_num

pseudocode:

    insert(H,x) {                               // O(1)
        if (H.min == null) {                    // insert in empty root_list
            new H.root_list {x};        
            H.min = x;
        } else {
            H.root_list.insert(x);              // insert new node to root_list
            if (x.key < H.min.key)              // if smaller than min
                H.min = x;                      // it become min
        }
        ++H.size;
    }
    
    union(H1,H2) {                              // O(1)
        H = new fib_heap;
        H.root_list = union(H1.root_list, H2.root_list);
        H.min = min(H1.min, H2.min);
        H.size = H1.size + H2.size;
        return H;
    }

    pop_min(H) {                                // O(degree_max(H)) = O(log(n))
        z = H.min;
        if (z != null) {
            for (x: z children) {
                H.root_list.insert(x);          // move all children to root_list
                x.parent = null;
            }
            H.root_list.remove(z);
            if (z == z.right) {                 // root_list size was 1
                H.min = null;
            } else {
                H.min = z.right;                // set min to next node after z
                consolidate(H);                 // fix fib_heap properties
            }
            --H.size;
        }
        return z;
    }
    
    consolidate(H) {
        A[degree_max(H)];                       // arr for degree
        for (w: H.root_list) {                  // for all roots
            x = w;
            d = x.degree;
            while (A[d] != null) {              // find 2 roots with same degree
                y = A[d];                       // other node with same degree as x
                if (x.key > y.key)  
                    swap(x,y);                  // make x.key < y.key
                
                H.root_list.remove(y);          // remove y from root_list
                x.children.insert(y);           // add y to x children
                y.parent = x;                   // set y parent to x
                ++x.degree;                     // increase x degree
                y.mark = false;

                A[d] = null;
                ++d;                            // check other degree > d
            }
            A[d] = x;                           // only x left with degree d
        }
        
        H.min = null;                           // rebuild root_list
        for (int i = 0; i < A.size; ++i) {
            if (A[i] != null) {                 // same as insert, without size increase
                if (H.min == null) {            
                    new H.root_list {A[i]};     // insert in empty root_list
                    H.min = A[i];  
                } else {    
                    H.root_list.insert(A[i]);   // insert new node to root_list
                    if (A[i].key < H.min.key)   // if smaller than min
                        H.min = A[i];           // it become min
                }
            }
        }
    }
    
    decrease_key(H,x,k) {                       // O(1)
        assert(k < x.key);
        x.key = k;
        y = x.parent;
        if (y != null && x.key < y.key) {       // if properties violated
            cut(H,x,y);
            cascading_cut(H,y);
        }
        if (x.key < H.min.key)
            H.min = x;
    }
    
    cut(H,x,y) {
        y.children.remove(x);                   // remove x from y children
        --y.degree;
        H.root_list.insert(x);                  // add x to root_list
        x.parent = null;
        x.mark = false;                         // x mark clear
    }
    
    cascading_cut(H,y) {
        z = y.parent;
        if (z != null) {
            if (y.mark == false) {
                y.mark = true;
            } else {                            // repeat up to root
                cut(H,y,z);
                cascading_cut(H,z);
            }
        }
    }
    
    remove(H,x) {                               // O(log(n))
        decrease_key(H,x,-inf);                 // make x - min node
        pop_min(H);                             // extract it
    }
    
    degree_max(H) {
        return log(H.size, 0.5*(1+sqrt(5)));    // golden ratio log
    }

***

<a id="graphs"></a>
## Graphs [home](#home)
* [Graph representation             ](#representation  )
* [Simple algs                      ](#simple_algs     )
    - [breadth first search (BFS)       ](#bfs             )
    - [depth first search (DFS)         ](#dfs             )
    - [topological sort                 ](#topological     )
    - [strongly connected components    ](#components      )
* [Minimal spaning tree (MST)       ](#mst             )
    - [Kruskal                          ](#kruskal         )
    - [Prim                             ](#prim            )
* [Shortest paths from 1 vert       ](#paths_vert      )
    - [Bellman-Ford                     ](#bellman_ford    )
    - [Dag                              ](#dag             )
    - [Dijkstra                         ](#dijkstra        )
* [Shortest paths between all verts ](#paths_all       )
    - [Floyd-Warshall                   ](#floyd_warshall  )
    - [Johnson                          ](#johnson         )
* [Max flow                         ](#max_flow        )
    - [Ford-Fulkerson                   ](#ford_fulkerson  )
    - [max pairs                        ](#max_pairs       )


<a id="representation"></a>
### Graph representation
graph: vertexes (V), edges(E)
- adjacency-list (for sparse: E << V^2)
- matrix (for dense: E ~ V^2)

types:
- oriented
- weighted


<a id="simple_algs"></a>
### Simple algs [up](#graphs)
- recursive print path, by parent vert
- backwards: from dest(v) to start(s)

pseudocode:

    print_path(G,s,v) {
        if (v == s) {                       // reach start vert
            print(s);
        } else if (v.parent == null) {      // reach root
            print("no way");                // but not start
        } else {
            print_path(G,s,v.parent);       // recursive go parent
            print(v);                       // print curent
        }
    }


<a id="bfs"></a>
#### breadth first search (BFS)
- O(V + E)
- bfs tree (min distances) is build

alg desc:
- add initial vert to queue
- while exist verts in queue
-   add adj verts of curent to queue if not visited
-   mark it as visited

pseudocode:

    bfs(G,s) {
        for (u: G.V - s) {                  // init all V
            u.color = WHITE;                // not visited
            u.dist = inf;                   // min dist - inf
            u.parent = null;                // no parent(in bfs tree)
        }
        
        s.color = GRAY;                     // to be visited
        s.dist = 0;                         // initial vert
        s.parent = null;                    // root
        
        queue.push(s);                      // add initial vert to queue
        while (!queue.empty()) {            // while exist not visited verts
            u = queue.pop();                // get next vert
            for (v: G.agj(u)) {             // visit all adj verts
                if (v.color == WHITE) {     // if not visited
                    v.color = GRAY;         // mark to be visited(not necessary)
                    v.dist = u.dist + 1;    // increase dist
                    v.parent = u;           // remember parent
                    queue.push(v);          // add to queue
                }
            }
            u.color = BLACK;                // mark as visited
        }
    }


<a id="dfs"></a>
#### depth first search (DFS)
- O(V + E)
- dfs forest is build
- open/close time create correct parenthesis struct: ((())())
- if u and v open/close not cross: non of them is parent of other
- if u open/close is inside v open/close: v is parent(distant) of u

pseudocode:

    dfs(G) {
        for (u: G.V) {                      // init all V
            u.color = WHITE;                // not visited
            u.parent = null;                // no parent(in bfs tree)
        }
        time = 0;                           // open/close timastamp
        for (u: G.V) {                      // for all V
            if (u.color == WHITE)           // if not visited(next root in forest)
                dfs_visit(G,u);             // recursive visit
        }
    }
    
    dfs_visit(G,u) {
        ++time;                             // increase time
        u.open = time;                      // save open time
        u.color = GRAY;                     // mark to be visited
        
        for (v: G.agj(u)) {                 // visit all adj verts
            if (v.color == WHITE) {         // if not visited
                v.parent = u;               // remember parent
                dfs_visit(G,v);             // recursive visit child -> child -> ...
            }   
        }   
        u.color = BLACK;                    // mark as visited
        u.close = ++time;                   // save close time
    }


<a id="topological"></a>
#### topological sort
- for: oriented acyclic graph
- for edge (u->v) in sorted list: u located before v

pseudocode:
    
    topological_sort(G) {
        dfs(G) for calc close time;
        when v mark as visited: result_list.push_front(v);
    }

<a id="components"></a>
#### strongly connected components
- max vert: for any u,v: u -> v && v -> u
- any vert in SCC is reachable from any other
- T = transpose(G): for all (u,v) in G.E -> (v,u) in T.E

pseudocode:

    strongly_connected_components(G) {
        dfs(G) for calc close time;
        T = transpose(G);
        dfs(T) in decrease order of close time;
        each SCC is a tree in dfs forest of T;
    }


***

<a id="mst"></a>
### Minimal spaning tree (MST)[up](#graphs)
- for: weighted nonoriented graph
- sub-edges of G.E of minimal sum weight, connecting all vertexes

generic pseudocode:

    generic_mst(G) {
        A = new tree;
        while (A not MST) {
            find edge (u,v), safe for A;
            A += (u,v);
        }
    }


<a id="kruskal"></a>
#### Kruskal
- E < V^2: O(log(E)) == O(log(V))
- O(E*log(V)): using bin-tree
- each step add edge (u,v) with min weight: u,v in diff sets

pseudocode:

    kruskal(G) {
        A = new tree;
        for (v: G.V)                                // O(V)
            make_set(v);
        sorted_list = G.E sorted by increase w;     // O(E*log(E))
        for ({u,v}: sorted_list) {                  // O(E)
            if (find_set(u) != find_set(v)) {
                A += (u,v);
                union(u,v);                         // O(log(V))
            }
        }
    }


<a id="prim"></a>
#### Prim
- O(E*log(V)): using bin-tree
- O(E + V*log(V)): using fib-heap
- each step add edge (u,v) with min weight: u not in A, v in A
- priority queue of verts that not in A
- v.key is min weight edge connecting it to A

pseudocode:

    prim(G,r) {
        for (u: G.V) {
            u.key = inf;
            u.parent = null;
        }
        r.key = 0;
        queue = new priority queue (G.V);               // O(V)
        while (!queue.empty()) {                        // O(V)
            u = queue.pop_min();                        // O(log(V))
            for (v: G.adj(u)) {                         // O(E) - total
                if (v in queue && G.W(u,v) < v.key) {
                    v.parent = u;
                    v.key = G.W(u,v);
                    queue.decrease_key(v, G.W(u,v));    // O(1)
                }
            }
        }
    }


***

<a id="paths_vert"></a>
### Shortest paths from 1 vert [up](#graphs)
- v.dist: shortest path estimate
- relaxation

pseudocode:

    init_single_source(G,s) {
        for (v: G.V) {
            v.dist = inf;
            v.parent = null;
        }
        s.dist = 0;
    }
    
    relax(u,v,w) {
        if (v.dist > u.dist + w) {
            v.dist = u.dist + w
            v.parent = u;
        }
    }


<a id="bellman_ford"></a>
#### Bellman-Ford
- O(V*E)
- for: oriented weighted graph, W can be negative

pseudocode:

    bellman_ford(G,s) {
        init_single_source(G,s);
        for (i = 0; i < G.V.size - 1; ++i) {        // V - 1 times: O(V)
            for ({u,v}: G.E) {                      // for all edges: O(E)
                relax(u,v,G.W(u,v));                // relax
            }
        }
        for ({u,v}: G.E) {                          // for all edges
            if (v.dist > u.dist + G.W(u,v))         // check shortest path estimate
                return false;                       // if fails: exist negative cycles
        }
        return true;                                // if ok: no negative cycles
    }


<a id="dag"></a>
#### Dag
- O(V + E)
- for: weighted oriented noncyclic graph
- relax in topological sort order

pseudocode:

    dag(G,s) {
        sorted_list = topological_sort(G);      // O(V + E)
        init_single_source(G,s);                // O(V)
        for (u: sorted_list) {                  // O(V)
            for (v: G.adj(u)) {                 // O(E) - total
                relax(u,v,G.W(u,v));
            }
        }
    }


<a id="dijkstra"></a>
#### Dijkstra
- O(E*log(V)) - bin-tree; O(V*log(V) + E) - fib-heap
- for: weighted oriented graph, non negative weight
- finished vertexes S with dist is ready
- on each step choose u from V - S, with min dist estimate
- then add u to S and relax all edges (u,v): v: G.adj(u)

pseudocode:

    dijkstra(G,s) {
        init_single_source(G,s);
        S = new set;
        queue = new priority queue(G.V);
        while (!queue.empty()) {
            u = queue.pop_min();
            S += u;
            for (v: G.adj(u)) {
                relax(u,v,G.W(u,v));
                queue.decrease_key(v);
            }
        }
    }


***

<a id="paths_all"></a>
### Shortest paths between all verts [up](#graphs)
- predcessor matrix P: P(i,j) - parent of j on path i->j

pseudocode:

    print_path(P,i,j) {
        if (i==j) {
            print(i);
        } else if (P(i,j) == null) {
            print("no way");
        } else {
            print_path(P,i,P(i,j));
            print(j);
        }
    }


<a id="floyd_warshall"></a>
#### Floyd-Warshall
- O(V^3)
- for weighted oriented graph, no negative cycles
- graph representation by matrix
- return matrix with shortest paths weight
- used property: if path i->j is shortest, and vert k is on this path
- than i->k and k->j also shortest

pseudocode:

    floyd_warshall(GM) {
        n = GM.rows;
        D = GM;
        for (k = 0; k < n; ++k) {                               // all intermediate verts
            for (i = 0; i < n; ++i) {                           // all start verts
                for (j = 0; j < n; ++j) {                       // all dest verts
                    D(i,j) = min(D(i,j), D(i,k) + D(k,j));      // min of i->j and i->k + k->j
                }
            }
        }
        return D;
    }


<a id="johnson"></a>
#### Johnson
- O(V^2*log(V) + V*E)
- w can be nagative, can have negative cycles(checks this)
- if non negative w: we can use dijkstra for all vertexes
- for negative weight we recalc it to positive weight W2
- so path u->v is shortest for G.W, also shortest for W2
- W2(u,v) = G.W(u,v) + H(u) - H(v); H - transit func
- add new vert s and edges (s,v): v:G.V, with weight 0

pseudocode:

    johnson(G) {
        s = new vert;
        G2 = new graph;                             // init G2
        G2.V = G.V + s;                             // with new vert s
        G2.E = G.E + {(s,v): v in G.V};             // edges form s to all G.V
        G2.W(s,v) = 0: v in G.V;                    // with weight 0
        
        if (bellman_ford(G2,s) == false) {          // check for negative cycles
            print("negative cycles exist here");
        } else {                                    // no negative cycles
            for (v: G2.V)
                H(v) = v.dist;                      // v.dist can be < 0, calc by bellman_ford
            for ({u,v}: G2.E)                       // recalc weight, to be > 0
                G.W(u,v) = G.W(u,v) + H(u) - H(v);
            D = new matrix(n,n);                    // result paths weight
            for (u: G.V)
                dijkstra(G,u);                      // dijkstra path, with new weight
            for (v: G.V)                            // recalc weight back
                D(u,v) = v.dist + H(v) - H(u);      // v.dist - min path weight u->v
            return D;
        }
    }


***

<a id="max_flow"></a>
### Max flow [up](#graphs)
- flow network

<a id="ford_fulkerson"></a>
#### Ford-Fulkerson
- O(V*E^2)
- find max flow s -> t
- F - flow estimate
- sum of F(s,u): u in G.adj(s) = max flow
- while exist additional path in residual network(GF)
- increase flow on this path
- Edmonds-Karp: search path using BFS

pseudocode:

    ford_fulkerson(G,s,t) {
        for ({u,v}: G.E) {                              // for all edges
            F(u,v) = 0;                                 // init flow with 0
            F(v,u) = 0;                                 // and back flow with 0
        }                                       
        while (exist path P s->t in GF) {               // use BFS to find min path s->t
                                                        // (u,v) visited if F(u,v) < G(u,v)
                                                        // so exist additional flow
            minw = min(G(u,v) - F(u,v): (u,v) in P);    // min weight edge on path P
            for ({u,v}: P) {                            // for all edges on path P
                F(u,v) += minw;                         // increase by additional min flow
                F(v,u) = -F(u,v);                       // recalc back flow
            }
        }
    }


<a id="max_pairs"></a>
#### max pairs
- in bipartite nonweighted nonoriented graph; G.V = L + R
- create flow network G2 oriented weighted
- G2.V = G.V + s + t;
- G2.E = {(u,v): u in L, v in R} + {(s,u): u in L} + {(v,t): v in R};
- G2.W = 1: all weight 1
- use ford_fulkerson to find max flow = max pairs

