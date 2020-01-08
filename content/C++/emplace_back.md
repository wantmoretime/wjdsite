---
title: "C++11 emplace相关函数"
date: 2020-01-05T17:32:29+08:00
draft: false
description: "C++11新特性"
tags: 
 - C++
 - leetcode
categories: 
 - C++
---





C++11中，顺序容器(如vector、deque、list)，新标准引入了三个新成员：emplace_front、emplace和emplace_back代替push_front、insert和push_back，可提高效率。

 <!--more-->



在leetcode上刷题，层次遍历二叉树，需要向vector<vector<int>>数组添加vector<int>元素，使用push_back和emplace_back效率相差4、5倍。

[107. Binary Tree Level Order Traversal II](https://leetcode-cn.com/problems/binary-tree-level-order-traversal-ii/)

```c++
/**
 * Definition for a binary tree node.
 * struct TreeNode {
 *     int val;
 *     TreeNode *left;
 *     TreeNode *right;
 *     TreeNode(int x) : val(x), left(NULL), right(NULL) {}
 * };
 */
class Solution {
public:
    vector<vector<int>> levelOrderBottom(TreeNode* root) {
        vector<vector<int>> res;
        if(!root)
            return res;
        queue<TreeNode*> q;

        q.push(root);
        while(!q.empty())
        {
        	int size=q.size();
        	vector<int> lvn;
        	for(int i=0;i<size; ++i)
        	{
        		TreeNode* tmp = q.front();
        		q.pop();
        		lvn.push_back(tmp->val);
        		if(tmp->left)
        			q.push(tmp->left);
        		if(tmp->right)
        			q.push(tmp->right);
    		}
            //res.push_back(lvn);
    		res.emplace_back(lvn);
        }
        reverse(res.begin(),res.end());
        return res;
    }
};
```



使用push_back()的时候，会首先构造一个元素，然后拷贝复制传递给容器。

使用emplace_back()时，会在容器所在的内存空间直接构造一个元素，避免了额外的移动或赋值操作。

emplace()  函数的功能是可以直接将参数传给对象的构造函数，在容器中构造出一个对象，从而实现 0 拷贝。(底层机制是调用placement new即布局new运算符来实现的)。

总而言之：emplace相关函数可以减少内存拷贝和移动。