+++
title = "算法练习Stack(六)--P173"
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



[link on JianShu](https://www.jianshu.com/p/2622d5de1452)

[栈stack](https://leetcode.com/tag/stack/)    
[173. Binary Search Tree Iterator](https://leetcode.com/problems/binary-search-tree-iterator/) Medium

Implement an iterator over a binary search tree (BST). Your iterator will be initialized with the root node of a BST.  
Calling `next()` will return the next smallest number in the BST.

![](https://upload-images.jianshu.io/upload_images/3296949-1b89de105739cf41.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

Example: 
```
BSTIterator iterator = new BSTIterator(root);
iterator.next();    // return 3
iterator.next();    // return 7
iterator.hasNext(); // return true
iterator.next();    // return 9
iterator.hasNext(); // return true
iterator.next();    // return 15
iterator.hasNext(); // return true
iterator.next();    // return 20
iterator.hasNext(); // return false
```

Note:
- `next()` and `hasNext()` should run in average O(1) time and uses O(h) memory, where h is the height of the tree.
- You may assume that `next()` call will always be valid, that is, there will be at least a next smallest number in the BST when `next()` is called.

这块的思路还没理清楚。跟 二叉树 的属性有关系。

```
private final Stack<TreeNode> stack;

    public BSTIterator(TreeNode root) {
        stack = new Stack<>();
        TreeNode cur = root;
        while (cur != null) {
            stack.push(cur);
            cur = cur.left;
        }
    }

    public boolean hasNext() {
        return !stack.isEmpty();
    }

    public int next() {
        TreeNode node = stack.pop();

        // Traversal cur node's right branch
        TreeNode cur = node.right;
        while (cur != null){
            stack.push(cur);
            cur = cur.left;
        }

        return node.val;
    }
```
