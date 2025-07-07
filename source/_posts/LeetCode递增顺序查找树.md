---
title: LeetCode递增顺序查找树
date: '2020/7/23 9:42:11'
updated: '2025-07-07 15:59:09'
categories: 算法
tags:
  - 算法
  - LeetCode
---
# 题目详情


![](/images/f8dd164777ed276e7944d2fc0c091e0b.png)  
[题目详情链接](https://leetcode-cn.com/problems/increasing-order-search-tree/)



# 一、解题思路


将原树进行中序遍历将树中的节点的非空值放入到一个list集合中，创建一棵新树然后通过递归的方式将不断生成的新的右子树直到集合遍历完。



# 二、使用步骤


```plain
1.对原树进行中序遍历。将非空树的值一次放入到List集合中。
2.创建一个函数用于对集合进行遍历，将每次遍历得到的值用来创建当前树的值。
  在集合遍历完之前，继续递归该函数，将传递的实参改为当前树的右子树。
```



# 三、代码


```java
class Solution {
    public TreeNode increasingBST(TreeNode root) {
        if(root!=null){
           ArrayList array=new ArrayList<>();
            Solution s=new Solution();
            s.inOrderTraversal(root,array);
           root=s.toTree(null,array,0);
            return root;
        }else{
            return null;
        }
    }
	//通过递归的方式不断将集合中的值有做新的右子树的值
    public TreeNode toTree(TreeNode root,List<Integer> array,int length){
        root=new TreeNode(array.get(length));
        root.left=null;
        if(length<array.size()-1){
            length++;
            root.right=toTree(root.right,array,length);
        }
        return root;
    }

	//中序遍历
    public void inOrderTraversal(TreeNode root,List<Integer> array){
        if(root==null){
           return ;
        }else{
           inOrderTraversal(root.left,array);
           array.add(root.val);
           inOrderTraversal(root.right,array);
        }
    }
}
```



# 总结


在我看来树的重点需要掌握的就是树的遍历方式。前序、中序、后序、层序。基本上许多和树有关的题目都会涉及到他的遍历方式。这些方式都可以通过递归的方式来实现。

