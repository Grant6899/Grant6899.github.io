---
layout:		post
title:		Graph Algorithm Summary
subtitle:
date:		2018-02-24
author: 	Grant6899
header-img: img/post-bg-c++.jpg
catalog: true
tags:
    - Graph
    - Algorithm
---

# Pre-assumptions 

We assume graphs discussed below have E edges and V vertices.

# Single-source shortest-path algorithm


## Dijkstra's algorithm

```
 1  function Dijkstra(Graph, source):
 2
 3      create vertex set Q
 4
 5      for each vertex v in Graph:             // Initialization
 6          dist[v] ← INFINITY                  // Unknown distance from source to v
 7          prev[v] ← UNDEFINED                 // Previous node in optimal path from source
 8          add v to Q                          // All nodes initially in Q (unvisited nodes)
 9
10      dist[source] ← 0                        // Distance from source to source
11      
12      while Q is not empty:
13          u ← vertex in Q with min dist[u]    // Node with the least distance will be selected first
14                                                      
15          remove u from Q 
16          
17          for each neighbor v of u:           // where v is still in Q.
18              alt ← dist[u] + length(u, v)
19              if alt < dist[v]:               // A shorter path to v has been found
20                  dist[v] ← alt 
21                  prev[v] ← u 
22
23      return dist[], prev[]
```

### Code
```c++
    vector<int> dijkstra(vector<vector<int>>& map, int N, int src){
        // dist[i] to store distance between src and i
        // prev[i] to store previous node in optimal path from source
        vector<int> dist(N, INT_MAX), prev(N, 0);
        
        // S[i] indicates if a node is visited
        vector<bool> S(N, false);
		
        // Initialization 
        for(int i = 0; i < N; i++){
            dist[i] = map[src][i];
            if(dist[i] == INT_MAX)
                prev[i] = -1;
            else
                prev[i] = src;
        }
        
        
        for(int i = 0; i < N; i++){
            int mindist = INT_MAX;
            int u = src;
            
            // find the node with the least distance in unvisited set - Extract Min
            for(int j = 0; j < N; ++j)
                if((!S[j]) && dist[j] < mindist){
                    u = j;
                    mindist = dist[j];
                }
			
            // mark as visited
            S[u] = true;

			// go over unvisted nodes, with the node found above as bridge, shorter current paths in the map. - Relax the map
            for(int j = 0; j < N; ++j){
                if((!S[j]) && map[u][j] < INT_MAX){
                    if(dist[u] + map[u][j] < dist[j]){
                        dist[j] = dist[u] + map[u][j];
                        prev[j] = u;
                    }
                }
            }
        }
        
        return dist;
    }
```

### Complexity analysis

Code version above's runtime complexity is V*O(V). Notice the step to find the least distance in unvisited set can be implemented with a priority queue, which can lower the complexity to V*O(logV).

Accordingly, the complexity of Map Relaxing is E*O(1) and E*O(logV) without and with priority queue respectively.

So total complexity:

Using priority queue: O(V*logV + E*logV) -> O(ElogV)

Not using priority queue: O(V^2 + E) -> O(V^2)



# All-pairs shortest-path algorithm

## Floyd-Warshall Algorithm

```
1 let dist be a |V| × |V| array of minimum distances initialized to ∞ (infinity)
2 for each vertex v
3    dist[v][v] ← 0
4 for each edge (u,v)
5    dist[u][v] ← w(u,v)  // the weight of the edge (u,v)
6 for k from 1 to |V|
7    for i from 1 to |V|
8       for j from 1 to |V|
9          if dist[i][j] > dist[i][k] + dist[k][j] 
10             dist[i][j] ← dist[i][k] + dist[k][j]
11         end if
```
### Code

```c++
// Revursive printing function 
void  prn_pass(int j , int k, vector<vector<int>>& Path){  
    if (Path[j][k]!=-1){  
        prn_pass(j,Path[j][k], Path);  
        cout<<"-->"<<Path[j][k];  
        prn_pass(Path[j][k],k, Path);  
    }  
}  

vector<vector<int>> Floyd(vector<vector<int>>& map){
    int n = map.size();    
    vector<vector<int>> res(map);
    vector<vector<int>> Path(n, vector<int>(n, -1));
    
    for (int k = 0;k < n;k++)  
        for (int i = 0;i < n;i++)  
            for (int j = 0;j < n;j++){  
                if (res[i][k] + res[k][j] > 0 && res[i][k] + res[k][j] < res[i][j]){  
                    res[i][j] = res[i][k] + res[k][j];  
                    Path[i][j] = k;  
                }  
            }  

    //输出最短路径和权值  
    for (int i = 0;i < n;i++)  
        for (int j = 0;j < n;j++){  
            if (i!=j)  
            {  
                cout<<i<<"到"<<j<<"的最短路径为:";  
                cout<<i;  
                prn_pass(i,j, Path);  
                cout<<"-->"<<j<<endl;  
                cout<<"最短路径长度为:"<<res[i][j]<<endl;  
            }  
        }  
   return res;
}
```

### Complexity Analysis

O(V^3)


# Minimum spanning tree

A minimum spanning tree (MST) or minimum weight spanning tree is a subset of the edges of a connected, edge-weighted (un)directed graph that connects all the vertices together, without any cycles and with the minimum possible total edge weight.

## Kruskal Algorithm

```
KRUSKAL(G):
1 A = ∅
2 foreach v ∈ G.V:
3    MAKE-SET(v)
4 foreach (u, v) in G.E ordered by weight(u, v), increasing:
5    if FIND-SET(u) ≠ FIND-SET(v):
6       A = A ∪ {(u, v)}
7       UNION(u, v)
8 return A
```

### Code
```c++
struct Edge{
    int src, dst, weight;
    Edge(int _src, int _dst, int _weight) : src(_src), dst(_dst), weight(_weight) {}
};

vector<Edge> Kruskal(vector<vector<int>>& map, vector<vector<int>>& paths){

    vector<int> Set(map.size(), 0);
    for(int i = 0; i < map.size(); ++i)
        Set[i] = i;
    
    vector<Edge> edges, res;

    for(auto p : paths)
        edges.push_back(Edge(p[0], p[1], p[2]));
    
    sort(edges.begin(), edges.end(), [](Edge a, Edge b){return a.weight < b.weight;});

    for(Edge e : edges){
        if( Set[e.src] != Set[e.dst]){
            res.push_back(e);  
            // This union operation is same as the one in union find
            for(int i = 0; i < Set.size(); ++i)
                if(Set[i] == Set[e.dst])
                    Set[i] = Set[e.src];
        }
    }
    return res;
}
```

## Prime Algorithm



# Topological Sort
