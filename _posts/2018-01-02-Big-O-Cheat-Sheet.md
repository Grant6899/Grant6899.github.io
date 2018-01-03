---
layout:		post
title:		Big-O Cheat Sheet
subtitle:
date:		2018-01-02
author: 	Grant6899
header-img: img/post-bg-c++.jpg
catalog: true
tags:
    - Algorithm
    - Interview
---

# Data Structures

<table align="middle">
	<tr>
    <th rowspan="2">Data Structure</th>
    <th colspan="3">Average cases</th>
    <th colspan="3">Worst cases</th>
    </tr>
	<tr>
    	<th>Insert</th>
    	<th>Delete</th>
        <th>Search</th>
        <th>Insert</th>
        <th>Delete</th>
        <th>Search</th>
  	</tr>
  	<tr>
    	<th>Array</th>
        <td>O(n)</td>
        <td>N/A</td>
        <td>O(1)</td>
        <td>N/A</td>
        <td>N/A</td>
        <td>O(1)</td>
  	</tr>
  	<tr>
    	<th>Sorted array</th>
        <td>O(n)</td>
        <td>O(n)</td>
        <td>O(log(n))</td>
        <td>O(n)</td>
        <td>O(n)</td>
        <td>O(log(n))</td>
  	</tr>    
  	<tr>
    	<th>Stack/Queue</th>
        <td>O(1)</td>
        <td>O(1)</td>
        <td>O(n)</td>
        <td>O(1)</td>
        <td>O(1)</td>
        <td>O(n)</td>
  	</tr>
  	<tr>
    	<th>Linked list</th>
        <td>O(1)</td>
        <td>O(1)</td>
        <td>O(n)</td>
        <td>O(1)</td>
        <td>O(1)</td>
        <td>O(n)</td>
  	</tr>
  	<tr>
    	<th>Doubly linked list</th>
        <td>O(1)</td>
        <td>O(1)</td>
        <td>O(n)</td>
        <td>O(1)</td>
        <td>O(1)</td>
        <td>O(n)</td>
  	</tr>
  	<tr>
    	<th>Hash table</th>
        <td>O(1)</td>
        <td>O(1)</td>
        <td>O(1)</td>
        <td>O(n)</td>
        <td>O(n)</td>
        <td>O(n)</td>
  	</tr>    
  	<tr>
    	<th>Binary search tree</th>
        <td>O(log(n))</td>
        <td>O(log(n))</td>
        <td>O(log(n))</td>
        <td>O(n)</td>
        <td>O(n)</td>
        <td>O(n)</td>
  	</tr>    
  	<tr>
    	<th>AVL tree</th>
        <td>O(log(n))</td>
        <td>O(log(n))</td>
        <td>O(log(n))</td>
        <td>O(log(n))</td>
        <td>O(log(n))</td>
        <td>O(log(n))</td>
  	</tr>
  	<tr>
    	<th>Red-Black tree</th>
        <td>O(log(n))</td>
        <td>O(log(n))</td>
        <td>O(log(n))</td>
        <td>O(log(n))</td>
        <td>O(log(n))</td>
        <td>O(log(n))</td>
  	</tr>    
</table>


# Graphs

<table align="middle">
	<tr>
    <th>Node/edge management</th>
    <th>Storage size</th>
    <th>Add vertex</th>
    <th>Add edge</th>
    <th>Remove vertex</th>
    <th>Remove edge</th>
    <th>Query</th>
    </tr>
	<tr>
    	<th>Adjacency list</th>
        <td>O(|V| + |E|)</td>
        <td>O(1)</td>
        <td>O(1)</td>
        <td>O(|V| + |E|)</td>
        <td>O(|E|)</td>
        <td>O(|V|)</td>
  	</tr>
  	<tr>
    	<th>Adjacency matrix</th>
        <td>O(|V|&sup2)</td>
        <td>O(|V|&sup2)</td>
        <td>O(1)</td>
        <td>O(|V|&sup2)</td>
        <td>O(1)</td>
        <td>O(1)</td>
  	</tr>
</table>

# Sorting algorithms

<table align="middle">
	<tr>
    <th rowspan="2">Algorithm(applied to array)</th>
    <th colspan="3">Time complexity</th>
    <th rowspan="2">Stability</th>
    </tr>
	<tr>
    	<th>Best cases</th>
    	<th>Average cases</th>
        <th>Worst cases</th>
  	</tr>
  	<tr>
    	<th>Bubble sort</th>
        <td>O(n)</td>
        <td>O(n&sup2)</td>
        <td>O(n&sup2)</td>
        <td>Yes</td>
  	</tr>
  	<tr>
    	<th>Selection sort</th>
        <td>O(n&sup2)</td>
        <td>O(n&sup2)</td>
        <td>O(n&sup2)</td>
        <td>No</td>
  	</tr>
  	<tr>
    	<th>Insertion sort</th>
        <td>O(n)</td>
        <td>O(n&sup2)</td>
        <td>O(n&sup2)</td>
        <td>Yes</td>
  	</tr>
  	<tr>
    	<th>Merge sort</th>
        <td>O(n log(n))</td>
        <td>O(n log(n))</td>
        <td>O(n log(n))</td>
        <td>Yes</td>
  	</tr>  	
    <tr>
    	<th>Quick sort</th>
        <td>O(n log(n))</td>
        <td>O(n log(n))</td>
        <td>O(n&sup2)</td>
        <td>Typical in-place sort is not stable; stable versions exist</td>
  	</tr>    
</table>

# Searching algorithms

<table align="middle">
	<tr>
    <th>Algorithm</th>
    <th>Data structure</th>
    <th>Worst case</th>
    </tr>
	<tr>
    	<th>Sequential  search</th>
        <td>Array and linked list</td>
        <td>O(n)</td>
  	</tr>
  	<tr>
    	<th>Binary search</th>
        <td>Sorted array and binary search tree</td>
        <td>O(log(n))</td>
  	</tr>
  	<tr>
    	<th>Depth-first search(DFS)</th>
        <td>Graph of |V| vertices and |E| edges</td>
        <td>O(|V| + |E|)</td>
  	</tr>    
  	<tr>
    	<th>Breadth-first search(BFS)</th>
        <td>Graph of |V| vertices and |E| edges</td>
        <td>O(|V| + |E|)</td>
  	</tr></table>