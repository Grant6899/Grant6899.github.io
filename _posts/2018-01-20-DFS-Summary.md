---
layout:		post
title:		DFS Summary
subtitle:
date:		2018-01-20
author: 	Grant6899
header-img: img/post-bg-c++.jpg
catalog: true
tags:
    - algorithm
    - Search
---

## What is DFS?

Depth-first search (DFS) is an algorithm for traversing or searching tree or graph data structures. One starts at the root (selecting some arbitrary node as the root in the case of a graph) and explores as far as possible along each branch before backtracking.


## Scenario 


### 1. Find all possible solution

Example question: [Leetcode#131](https://leetcode.com/problems/palindrome-partitioning/description/)

Given a string s, partition s such that every substring of the partition is a palindrome.

Return all possible palindrome partitioning of s.

For example, given s = "aab",
Return
```
[
  ["aa","b"],
  ["a","a","b"]
]
```

```c++
class Solution {
public:
    vector<vector<string>> partition(string s) {
        find(s, 0);
        return result;
    }

private:
    vector<vector<string>> result;
    vector<string> current;
    
    // void recursive function
    void find(string s, int st){
        // check if reach target
        if(st == s.size()){
            result.push_back(current);
            return;
        }
        
        // iterate each possible next step
        for(int i = st; i < s.size(); ++i){
            
            // check if current operation is a valid next step
            if(isPalindrome(s, st, i)){
            	// modify current stage
                current.push_back(s.substr(st, i - st + 1));
                // go find next stage
                find(s, i + 1);
                // revert modification to previous stage
                current.pop_back();
            }
        }
    }

    bool isPalindrome(string s, int l, int r){
        bool res = true;
        while(l<r){
            if(s[l] != s[r])
                return false;
            l++;
            r--;
        }
        return res;
    }

};
```

### 2. Find one possible target stage

Same question as above, but only one solution is needed.

```c++
class Solution {
public:
    vector<vector<string>> partition(string s) {
        bool temp = find(s, 0);
        return result;
    }

private:
    vector<vector<string>> result;
    vector<string> current;
    
    // recursive function returns bool value to indicate if a solution is already found
    bool find(string s, int st){
    
    	// check if reach target
        if(st == s.size()){
            result.push_back(current);
            // return true to tell following steps a solution is found
            return true;
        }
        
        // iterate each possible next step
        for(int i = st; i < s.size(); ++i){
            
            // check if current operation is a valid next step
            if(isPalindrome(s, st, i)){
            	// modify current stage
                current.push_back(s.substr(st, i - st + 1));
                // put recursive function as a condition, if ture, then return true diretly to previous level without reverting current stage
                if(find(s, i + 1))
                    return true;
				// revert modification to previous stage                    
                current.pop_back();
            }
        }
		
		// if cannot find, return false. Important!!!
        return false;
    }

    bool isPalindrome(string s, int l, int r){
        bool res = true;
        while(l<r){
            if(s[l] != s[r])
                return false;
            l++;
            r--;
        }
        return res;
    }
};
```

### Transverse a graph to finish certain operation

Example question: [LeetCode#733](https://leetcode.com/problems/flood-fill/description/)

An image is represented by a 2-D array of integers, each integer representing the pixel value of the image (from 0 to 65535).

Given a coordinate (sr, sc) representing the starting pixel (row and column) of the flood fill, and a pixel value newColor, "flood fill" the image.

To perform a "flood fill", consider the starting pixel, plus any pixels connected 4-directionally to the starting pixel of the same color as the starting pixel, plus any pixels connected 4-directionally to those pixels (also with the same color as the starting pixel), and so on. Replace the color of all of the aforementioned pixels with the newColor.

At the end, return the modified image.

Example 1:
```
Input: 
image = [[1,1,1],[1,1,0],[1,0,1]]
sr = 1, sc = 1, newColor = 2
Output: [[2,2,2],[2,2,0],[2,0,1]]
```
Explanation: 
From the center of the image (with position (sr, sc) = (1, 1)), all pixels connected 
by a path of the same color as the starting pixel are colored with the new color.
Note the bottom corner is not colored 2, because it is not 4-directionally connected
to the starting pixel.

From 

| 1 | 1 | 1 |
|:-:|:-:|:-:|  
| 1 | 1 | 0 |   
| 1 | 0 | 1 |

To:

| 2 | 2 | 2 |
|:-:|:-:|:-:|  
| 2 | 2 | 0 |   
| 2 | 0 | 1 |

```c++
class Solution {
public:
    vector<vector<int>> floodFill(vector<vector<int>>& image, int sr, int sc, int newColor) {
        if (image.empty() || image[0].empty())
            return image;
        
        fill(image, sr, sc, newColor, image[sr][sc]);
        return image;
    }

private:
    void fill(vector<vector<int>>& map, int i, int j, int newColor, int oldColor){
    	// check if current position needs modifiying
        if(oldColor != newColor && i >=0 && i < map.size() && j >= 0 && j < map[0].size() && map[i][j] == oldColor){
        	// certain operation
            map[i][j] = newColor;
            // apply the operation to surroundings
            fill(map, i-1, j, newColor, oldColor);
            fill(map, i+1, j, newColor, oldColor); 
            fill(map, i, j-1, newColor, oldColor); 
            fill(map, i, j+1, newColor, oldColor);
        }
        return;
    }
};
```


## Optimization

### Memorized DFS

Sometimes we need to sacrifice space to get better performance on time complexity.

Example question: [LeetCode#140](https://leetcode.com/problems/word-break-ii/description/)

Given a non-empty string s and a dictionary wordDict containing a list of non-empty words, add spaces in s to construct a sentence where each word is a valid dictionary word. You may assume the dictionary does not contain duplicate words.

Return all such possible sentences.

For example, given
```
s = "catsanddog",
dict = ["cat", "cats", "and", "sand", "dog"].

A solution is ["cats and dog", "cat sand dog"].
```


```c++
class Solution {
public:
    vector<string> wordBreak(string s, vector<string>& wordDict) {
        unordered_set<string> dic;
        for(auto ele : wordDict)
            dic.insert(ele);
        unordered_map<string, vector<string>> map;
        vector<string> res = find(s, dic, map);
        
        return res;
    }

	// map stores mappings between string and vector<string>
    // for any key - str, we save all possible word break plans in value map[str]
    
    vector<string> find(string s, const unordered_set<string>& dic, unordered_map<string, vector<string>>& map){
    	
        // if the branch is visited, we don't need to go any deeper but return existing solutions for s's break down
        if(map.find(s) != map.end()){
            return map[s];
        }

		// res is the result for string s
        vector<string> res;
        
        // check if we come to the end of string
        if(s.size() == 0){
            res.push_back("");
            return res;
        }
		
        // iterate each possible next word
        for(string word : dic){
            if(s.substr(0, word.size()) == word){
            	// subvec stores all possible word break down plan for s.substr(word.size())
                vector<string> subvec = find(s.substr(word.size()), dic, map);
                
                // concatenate current word with plans in subvec 
                for(string sub : subvec) 
                    res.push_back(word + (sub.empty() ? "" : " ") + sub);
            }
        }
		
        // save res to map as map[s]
        map[s] = res;
        
        return res;
    }
};
```

## Dealing with duplicates

Example question: [LeetCode#47](https://leetcode.com/problems/permutations-ii/description/)

Given a collection of numbers that might contain duplicates, return all possible unique permutations.

For example,
[1,1,2] have the following unique permutations:
```
[
  [1,1,2],
  [1,2,1],
  [2,1,1]
]
```

```c++
class Solution {
public:
	vector<vector<int>> permuteUnique(vector<int>& nums) {
		this->used = vector<bool>(nums.size(), false);
		this->target = nums.size();
		this->candidates = nums;
		std::sort(this->candidates.begin(), this->candidates.end());
		find(0);
		return result;
	}

	void find(int next) {
		if (current.size() == target) {
			result.push_back(current);
            for(int ele : current)
                cout << ele << " ";
            cout << endl;
			return;
		}

		for (int i = 0; i< candidates.size(); i++)
			if (used[i] == false) {
            
            	// candidates    1    2    2    3 
            	// used          1    1    0    0
                //                         i
                // when we trying to add the second 2 to the current stage, it gets returned then the next stage is:
                // candidates    1    2    2    3 
            	// used          1    0    1    0
                //                    i     
                // then we push the first 2 to current stage as the second 2, by which we avoid having 1 2 2 3 twice.
				if (i > 0 && candidates[i] == candidates[i - 1] && used[i - 1])
					return;
                    
				current.push_back(candidates[i]);
				used[i] = true;
				find(next + 1);
				used[i] = false;
				current.pop_back();
			}
	}

private:
	vector<int> current{};
	vector<bool> used;
	vector<int> candidates{};
	vector<vector<int>> result{};
	int target = 0;

};
```




