---
title: LeetCode-求二叉树的最大深度
date: '2021/7/23 9:42:11'
updated: '2025-07-07 15:58:55'
categories: 算法
tags:
  - 算法
  - LeetCode
---
# 题目详情


![](/images/841b196cb7a5e6e90bb0224cb0d960e2.png)  
**题目链接：**[二叉树的最大深度](https://leetcode-cn.com/problems/maximum-depth-of-binary-tree/)



# 一、解题思路


通过递归的方式，不断遍历其树的每一个节点。然后判断当前节点是否为空，不为空高度加1,同时遍历当前节点的子节点，然后比较左右两节点的高度，返回最大的节点高度。反之则直接返回上一节点的高度。



# 二、解题步骤


## 1.详细步骤


1. 判断当前节点是否为空 
    - 如果为空，直接返回上一节点的高度
    - 反之高度加1	，并执行下一步
2. 继续递归调用该函数，将其节点设置为当前节点，同时高度设置为当前节点的高度。并设置变量用于获取函数的返回值。
3. 比较左右两个节点函数返回值，取其最大值。



## 2. 代码


代码如下：



```java
class Solution {
    public int maxDepth(TreeNode root) {
        return maxHeight(root,0);
    }


    public int maxHeight(TreeNode root,int height){
        if(root!=null){
            height++;
            int leftHeight=0;
            int rightHeight=0;      
            leftHeight=maxHeight(root.left,height);
            rightHeight=maxHeight(root.right,height);
            return leftHeight>rightHeight?leftHeight:rightHeight;
        }else{
            return height;
        }
    }
}
```

---

# 三、总结


这里面主要用到了递归的思想。递归在与树有关的很多的题目都可以使用该思想。通过该思想可以很大程度上简化代码。但是也不是没有缺点，如果递归次数过多的话，可能会造成堆栈溢出。

