---
title: LeetCode将有序数组转化为二叉搜索树
date: '2020/5/23 9:42:11'
updated: '2025-07-07 15:59:05'
categories: 算法
tags:
  - 算法
  - LeetCode
---
# 一、将有序数组转化为二叉搜索树


## 题目


![](/images/07b9bd3cf5f4afa1093787db538b5fc7.png)  
**详情链接：**[有序数组转二叉树](https://leetcode-cn.com/problems/convert-sorted-array-to-binary-search-tree/)



# 二、使用步骤


## 1.解题思路


将当前数组的中间值用于创建当前的节点，然后中间值左边的数，分为一个新的子数组，这里我们暂且叫左子数组，将右边的分为右子数组。将左子数组放入当前节点的左子树。右子树组，放入当前节点的右子树。重复上述步骤即可。



## 2.解题步骤


1. 创建一个新的节点节点值为数组的中位值 。
2. 判断当前子数组大小是否为1。  
* 如果为1，则直接返回该节点。  
* 反之执行第三步。
3. 判断当前数组的 **(中间值得索引-1)>=0**， **就是判断当前节点是否存在左子树**。  
* 	如果大于0，创建一个新的数组命名为左子数组，值为当前数组的中间值左边的所有数值。然后将将左子数组传入当前函数执行。  
*  反之，则表明数组越界。直接执行第四步
4. 判断当前数组的**(中间值的索引是+1)<=当前数组的长度** ，就是判断当前节点是否存在右子树。  
* 如果小于的话，创建一个新的数组命名为右子数组，值为当前数组的中间值右边的所有数值。然后将将右子数组传入当前函数执行。  
* 反之，执行第五步
5. 返回当前节点。



## 代码


```java
class Solution {
    public TreeNode sortedArrayToBST(int[] nums) {
        if(nums==null||nums.length==0){
            return null;
        }{
            return arrrayToBST(nums,0,nums.length-1);
        }
        
    }


    public TreeNode arrrayToBST(int[] nums,int start ,int end){
       int length=end-start+1;
        TreeNode root=new TreeNode(nums[start+length/2]);
        if(start==end){
            return root;
        }else {
           
            if(length/2-1>=0){
              
                root.left=arrrayToBST(nums,start,start+length/2-1);
            }
            if(length/2+1<=length-1){
             
                root.right=arrrayToBST(nums,start+length/2+1,end);
            }
            return root;
        }
    }
}
```



# 三、总结


树的问题一般是通过递归和三种遍历方式来解决。目前我所写的很多树的比较简单的题目都是通过递归和树的三种遍历方式来实现的。



**各位大佬们看完后觉得我写得很差的的话，可以在评论去疯狂踩踏我蹂躏我。但是最最为重要的事就是不要白嫖！！！！虽然我知道在看的各位都是白嫖党！！！！**



