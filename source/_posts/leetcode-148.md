---
title: 二叉树最小深度
date: 2021-09-15 09:55:03
tags: [leetcode, 树]
categories: 算法
mathjax: true
---
# 题目描述
求给定二叉树的最小深度。最小深度是指树的根结点到最近叶子结点的最短路径上结点的数量。
Given a binary tree, find its minimum depth.The minimum depth is the number of nodes along the shortest path from the root node down to the nearest leaf node.
# 题目分析
首先要了解二叉树，是指树的每个节点最多只能有**两个**子节点，在二叉树中分别成为“左子节点”和“右子节点”。因此二叉树的每一层最多含有$2^{n-1}$个节点，$n$为节点所在层数。
本题为给定二叉树，求二叉树的最小深度，有两种解题思路：
1. **递归**，如下图所示二叉树，其最小深度可以理解为左子树和右子树中的最小的深度加1，而左右子树又可以分为新的左右子树从而计算左右子树的最小深度，直到节点为null。
![二叉树示例](binary_tree.png)
1. **遍历**，树的深度为根节点到叶子节点的距离，利用遍历的方法寻找叶子节点，从所有的叶子节点中找出距离根节点最近的那个，即找到最小深度。树的遍历一般分为前、中、后序遍历和层序遍历，在本题中最好选用层序遍历。


# 解题
### 递归解法
```java
public class Solution {
    public int run(TreeNode root) {
        if(root == null) {
            return 0;    //递归结束条件
        }
        int left = run(root.left);  //计算左子树深度
        int right = run(root.right); //计算右子树深度
        if(left*right != 0){  // 若左右子树均有节点
            return (left>right?right:left)+1; //取最小的深度加1
        }else { //若左子树或右子树没有节点
            return (left>right?left:right)+1; //该子树深度为存在节点的那一边的深度加1
        }
    }
}
```
递归解法代码比较简答，需要注意的是递归结束条件为节点为null，还需要判断左子树或右子树无节点的情况。时间复杂度为{% raw %}$O(n)$，$n${% endraw %}为节点数（还不太懂，还要继续学习）。
### 遍历解法
首先记录二叉树的各种遍历方法：
1. **前序遍历**
递归实现代码：
```java
public void preOrderTraverse(TreeNode root) {
	if(root != null) {
		System.out.println(root.val);
		preOrderTraverse(root.left);
		preOrderTraverse(root.right);
	}
}
```
非递归实现代码：
```java
public void preOrderTraverse(TreeNode root) {
	Stack<TreeNode> s = new Stack<>();
	s.push(root);
	while(!s.isEmpty()) {
		TreeNode t = s.pop();
		while(t != null) {
			System.out.println(t.val);
			if(t.right != null)	s.push(t.right);
			t = t.left;
		}
	}
}
```
2. **中序遍历**
递归实现代码：
```java
public void midOrderTraverse(TreeNode root) {
	if(root == null) {
		return;
	}else {
		midOrderTraverse(root.left);
		System.out.println(root.val);
		midOrderTraverse(root.right);
	}
}
```
非递归实现代码：
```java
public void midOrderTraverse(TreeNode root) {
	Stack<TreeNode> s = new Stack<>();
	while(root!=null || !s.isEmpty()) {
		if(root != null) {
			s.push(root);
			root = root.left;
		}else {
			root = s.pop();
			System.out.println(root.val);
			root = root.right;
		}
	}
}
```
3. **后序遍历**
递归实现代码：
```java
public void postOrderTraverse(TreeNode root) {
	if(root == null) {
		return;
	}else {
		postOrderTraverse(root.left);
		postOrderTraverse(root.right);
		System.out.println(root.val);
	}
}
```
非递归实现代码：
```java
public void postOrderTraverse(TreeNode root) {
	Stack<TreeNode> s = new Stack<>();
	Stack<Integer> tag = new Stack<>();
	while(root!=null || !s.isEmpty()) {
		while(root != null) {
			s.push(root);
			tag.push(0);
			root = root.left;
		}
		if(!s.isEmpty() && tag.peek()==1){
			tag.pop();
			System.out.println(s.pop().val);
		}
		if(!s.isEmpty() && tag.peek()==0){
			tag.pop();
			tag.push(1);
			root = s.peek().right;
		}
	}
}
```
4. **层序遍历**
非递归实现代码：
```java
public void levelOrderTraverse(TreeNode root) {
	Queue<TreeNode> q = new LinkedList<>();
	q.offer(root);
	while(!q.isEmpty()){
		TreeNode t = q.poll();
		System.out.println(t.val);
		if(t.left!=null) q.offer(t.left);
		if(t.right!=null) q.offer(t.right);
	}
}
```
利用层序遍历求二叉树的最小深度：
```java
import java.util.LinkedList;
import java.util.Queue;

public class Solution {
    public int run(TreeNode root) {
        if(root == null) {
            return 0;
        }
        if(root.left==null && root.right==null){
            return 1;
        }
        int depth = 0;
        Queue<TreeNode> q = new LinkedList<>();
        q.offer(root);
        while(!q.isEmpty()){
            int len = q.size();
            depth++;
            for(int i=0;i<len;i++){
                TreeNode t = q.poll();
                if(t.left==null && t.right==null){
                    return depth;
                }
                if(t.left != null){
                    q.offer(t.left);
                }
                if(t.right != null){
                    q.offer(t.right);
                }
            }
        }
        return 0;
    }
}
```
算法时间复杂度为{% raw %}$O(n)${% endraw %}.
# 总结
这道题涉及二叉树的相关知识，学习到了二叉树的遍历方法（递归与非递归），也了解了有关递归解题的方法，需要坚持把148题总结完！！！有关算法复杂度的计算也要继续学习。
