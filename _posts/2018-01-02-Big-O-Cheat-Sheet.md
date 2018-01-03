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

<table align="left">
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
    	<th>Array/Stack/Queue</th>
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
</table>