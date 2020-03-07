+++
title = "算法练习Stack--P145-Hard"
tags = [
    "jiansh"
]
date = "2020-03-07"
topics = [
    "jiansh",
    "Algo"
]
toc = true
+++



[link on JianShu](https://www.jianshu.com/p/87161c06ebb2)

[栈stack](https://leetcode.com/tag/stack/)    
[145. Binary Tree Postorder Traversal](https://leetcode.com/problems/binary-tree-postorder-traversal/) Hard

Given a binary tree, return the postorder traversal of its nodes' values.

Example:

>Input: [1,null,2,3]
```
   1
    \
     2
    /
   3
```
>Output: [3,2,1]

Follow up: Recursive solution is trivial, could you do it iteratively?

后序遍历在访问完左子树向上回退到根节点的时候不是立马访问根节点的，而是得先去访问右子树，访问完右子树后在回退到根节点。

Runtime: 1 ms, faster than 57.09% of Java online submissions for Binary Tree Postorder Traversal.
Memory Usage: 35 MB, less than 100.00% of Java online submissions for Binary Tree Postorder Traversal.
```
 public List<Integer> postorderTraversal(TreeNode root) {
        List<Integer> result = new LinkedList<>();
        Stack<TreeNode> toVisit = new Stack<>();
        TreeNode cur = root;
        TreeNode pre = null;

        while (cur != null || !toVisit.isEmpty()) {
            while (cur != null) {
                toVisit.push(cur); // 添加根节点
                cur = cur.left; // 递归添加左节点
            }
            cur = toVisit.peek(); // 已经访问到最左的节点了
            //在不存在右节点或者右节点已经访问过的情况下，访问根节点
            if (cur.right == null || cur.right == pre) { 
                toVisit.pop();
                result.add(cur.val);
                pre = cur;
                cur = null;
            } else {
                cur = cur.right; // 右节点还没有访问过就先访问右节点
            }
        }
        return result;
    }
```
