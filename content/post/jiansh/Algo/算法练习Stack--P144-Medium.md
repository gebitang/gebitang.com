+++
title = "算法练习Stack--P144-Medium"
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



[link on JianShu](https://www.jianshu.com/p/5b21c3d1dd4c)

[栈stack](https://leetcode.com/tag/stack/)    
[144. Binary Tree Preorder Traversal](https://leetcode.com/problems/binary-tree-preorder-traversal/) Medium

Given a binary tree, return the preorder traversal of its nodes' values.

Example:

>Input: [1,null,2,3]
```
   1
    \
     2
    /
   3
```
>Output: [1,2,3]

Follow up: Recursive solution is trivial, could you do it iteratively?

- *前序*就是根节点在最前`根->左->右`，

出栈顺序和入栈顺序相反，每次添加节点的时候先添加右节点，再添加左节点。这样在下一轮访问子树的时候，就会先访问左子树，再访问右子树。

Runtime: 1 ms, faster than 47.44% of Java online submissions for Binary Tree Preorder Traversal.
Memory Usage: 34.9 MB, less than 100.00% of Java online submissions for Binary Tree Preorder Traversal.
```
public List<Integer> preorderTraversal(TreeNode root) {
        List<Integer> result = new LinkedList<>();
        if (root == null) return result;

        Stack<TreeNode> toVisit = new Stack<>();
        toVisit.push(root);
        TreeNode cur;

        while (!toVisit.isEmpty()) {
            cur = toVisit.pop();
            result.add(cur.val); // 访问根节点
            if (cur.right != null) toVisit.push(cur.right); // 右节点入栈
            if (cur.left != null) toVisit.push(cur.left); // 左节点入栈
        }
        return result;
    }
```
