---
title: LeetCode之合并二叉树
date: '2020/7/23 9:42:11'
updated: '2025-07-07 15:59:00'
categories: 算法
tags:
  - 算法
  - LeetCode
---
# 合并二叉树


## 1. 题目


![](/images/034c87a4d5a194023798f5c9848a3a93.png)



## 2. 解题思路


采用递归和后序遍历的方式来同时遍历两棵树。遍历的同时，一定要判断两颗树的当前节点是否为空。然后创建一个新的节点，节点值为两个节点之和。遍历完成后即可得到合并后的树。



## 3. 解题步骤


1.  判断当前两颗树的节点全为空，如果为空则返回空。 
2.  反之有以下三种 
    - 可能两个节点都不为空
    - 左节点为空，右节点不为空
    - 右节点为空，左节点不为空
3.  采取以下措施 
    - 当为第一种情况时 ,创建一个新的节点，节点值为左右节点值的和,之后遍历左右节点
    - 当为第二种情况时 ,创建一个新的节点，节点值为右节点值,之后遍历右节点.
    - 当为第三种情况时 ,创建一个新的节点，节点值为左节点值,之后遍历左节点。
4.  返回创建的新的节点。 



## 4. 代码


```java
class Solution {
    public TreeNode mergeTrees(TreeNode t1, TreeNode t2) {
        if(t1==null&&t2==null){
            return null;
        }else{
            TreeNode t3;
            if(t1!=null&&t2!=null){
                t3=new TreeNode(t1.val+t2.val);
                t3.right=mergeTrees(t1.right,t2.right);
                t3.left=mergeTrees(t1.left,t2.left);
            }else if(t1!=null){
                t3=new TreeNode(t1.val);
                t3.right=mergeTrees(t1.right,null);
                t3.left=mergeTrees(t1.left,null);
            }else{
                t3=new TreeNode(t2.val);
                t3.right=mergeTrees(null,t2.right);
                t3.left=mergeTrees(null,t2.left);
            }
            return t3;
        }
    }
}
```



## 5. 总结


以目前我做的与树相关的题目来说。所需要做的事基本上就是一件事：**遍历**。而树的遍历一般可以通过递归的方式来进行一些简单的遍历。遍历一般分为三种：前序遍历、中序遍历、后序遍历。前序、中序、后序的名字是按照遍历中父节点，相对于左右子节点中的顺序。如果父节点，在其子父节点中最后一个遍历，称之为后序遍历。其他的遍历顺序可由以上类推。











