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

## Prim Algorithm

```
Prim(G):
1 A = ∅
2 pick any u ∈ V:
3    MAKE-SET U (u)
4 while U != V		
5		for each (u, v) in G.E ordered by weight(u, v), where u ∈ U and v ∈ V/U increasing:    
6       A = A ∪ {(u_min, v_min)}
7       add v into U
8 return A
```

### Code
```c++
struct Edge{
    int src, dst, weight;
    Edge(int _src, int _dst, int _weight) : src(_src), dst(_dst), weight(_weight) {}
};

vector<Edge> Prim(vector<vector<int>>& map, vector<vector<int>>& paths){
    unordered_set<int> visited{3};
    vector<Edge> res;
    int n = map.size();
    
    auto com = [](Edge a, Edge b){return a.weight > b.weight;};

    while(visited.size() != n){

        priority_queue<Edge, vector<Edge>, decltype(com)> pq(com);

        for(auto it = visited.begin(); it != visited.end(); ++it){
            for(int i = 0; i < map.size(); ++i)
                if( !visited.count(i) && map[*it][i] != INT_MAX)
                    pq.push(Edge(*it, i, map[*it][i]));
        }
        
        res.push_back(pq.top());
        visited.insert(pq.top().dst);
    }
    return res;
}
```

# Topological Sort

## DFS Method 

### Example 

[LeetCode 207](https://leetcode.com/problems/course-schedule/description/)

[LeetCode 210](https://leetcode.com/problems/course-schedule-ii/description/)

For DFS, it will first visit a node, then one neighbor of it, then one neighbor of this neighbor... and so on.

If it meets a node which was visited in the current process of DFS visit, a cycle is detected and we will return false. 
Otherwise it will start from another unvisited node and repeat this process till all the nodes have been visited. 

Note that you should make two records: one is to record all the visited nodes and the other is to record the visited nodes in the current DFS visit.

The code is as follows. We use a vector<bool> visited to record all the visited nodes and another vector<bool> onpath 
to record the visited nodes of the current DFS visit. Once the current visit is finished, we reset the onpath value of the starting node to false.


### Code
```c++
class Solution {
public:
    vector<int> findOrder(int numCourses, vector<pair<int, int>>& prerequisites) {
        vector<unordered_set<int>> graph = make_graph(numCourses, prerequisites);
        vector<int> toposort;
        vector<bool> onpath(numCourses, false), visited(numCourses, false);
        for (int i = 0; i < numCourses; i++)
            if (!visited[i] && dfs(graph, i, onpath, visited, toposort))
                return {};
        reverse(toposort.begin(), toposort.end());
        return toposort;
    }
private:
    vector<unordered_set<int>> make_graph(int numCourses, vector<pair<int, int>>& prerequisites) {
        vector<unordered_set<int>> graph(numCourses);
        for (auto pre : prerequisites)
            graph[pre.second].insert(pre.first);
        return graph;
    }
    bool dfs(vector<unordered_set<int>>& graph, int node, vector<bool>& onpath, vector<bool>& visited, vector<int>& toposort) { 
        if (visited[node]) return false;
        onpath[node] = visited[node] = true; 
        for (int neigh : graph[node])
            if (onpath[neigh] || dfs(graph, neigh, onpath, visited, toposort))
                return true;
        toposort.push_back(node);
        return onpath[node] = false;
    }
};

```