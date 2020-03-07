+++
title = "算法练习Stack--P94-Medium"
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



[link on JianShu](https://www.jianshu.com/p/07edbaa4194d)

[栈stack](https://leetcode.com/tag/stack/)    
[94. Binary Tree Inorder Traversal](https://leetcode.com/problems/binary-tree-inorder-traversal/) Medium


Given a binary tree, return the *inorder* traversal of its nodes' values.

Example:

>Input: [1,null,2,3]
```
   1
    \
     2
    /
   3
```
>Output: [1,3,2]

Follow up: Recursive solution is trivial, could you do it iteratively?

先了解一下 `preorder (前序)`，`inorder(中序)`，`postorder(后序) `的中英文对照。[二叉树的前序，中序，后序遍历方法总结](https://segmentfault.com/a/1190000016674584)

**前 中 后** 这三个词是针对根节点的访问顺序而言的：
- *前序*就是根节点在最前`根->左->右`，
- *中序*是根节点在中间`左->根->右`，
- *后序*是根节点在最后`左->右->根`。


Runtime: 1 ms, faster than 46.02% of Java online submissions for Binary Tree Inorder Traversal.
Memory Usage: 34.7 MB, less than 100.00% of Java online submissions for Binary Tree Inorder Traversal.
```
public List<Integer> inorderTraversal(TreeNode root) {
        List<Integer> result = new LinkedList<>();
        Stack<TreeNode> toVisit = new Stack<>();
        TreeNode cur = root;

        while (cur != null || !toVisit.isEmpty()) {
            while (cur != null) {
                toVisit.push(cur); // 添加根节点
                cur = cur.left; // 循环添加左节点
            }
            cur = toVisit.pop(); // 当前栈顶已经是最底层的左节点了，取出栈顶元素，访问该节点
            result.add(cur.val);
            cur = cur.right; // 添加右节点
        }
        return result;
    }
```
