+++
title = "算法练习LinkedList(十)--P138"
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



[link on JianShu](https://www.jianshu.com/p/26ab4d0b6eb5)

[138. Copy List with Random Pointer](https://leetcode.com/problems/copy-list-with-random-pointer/)

A linked list is given such that each node contains an additional random pointer which could point to any node in the list or null.

Return a [**deep copy**](https://en.wikipedia.org/wiki/Object_copying#Deep_copy) of the list.

[Java中的深拷贝](https://segmentfault.com/a/1190000010648514)：对象中的对象需要重写clone方法，将所有基本类型重新clone

Input:
>{"\$id":"1","next":{"\$id":"2","next":null,"random":{"\$ref":"2"},"val":2},"random":{"\$ref":"2"},"val":1}

Explanation:
>Node 1's value is 1, both of its next and random pointer points to Node 2.
Node 2's value is 2, its next pointer points to null and its random pointer points to itself.

一直陷入了思维定式：如何拷贝Node中的next和random呢？

看了这里的[实现](https://leetcode.com/problems/copy-list-with-random-pointer/discuss/43488/Java-O(n)-solution)，非常简洁。

利用Map，向map里添加元素时，确保每次添加的都是deep copy的对象(利用构造方法，只传递基本类型的值，引用对象默认先为null)。

巧妙的地方在对：Node里的next和random进行赋值的操作，`map.get(node).next = map.get(node.next);` 

- 添加时，每个Node都是deep copy之后的对象；
- 每一个节点都添加到了map中，所以原始链表中的任意一个Node的next或random都一定在map中，**并且**位于 map.get(node.next)的位置
- 仅通过赋值就完成了链表的“链接”过程  
- 最后返回的就是添加到map里的第一个head节点的value值

```
public Node copyRandomList2(Node head) {
    if(head == null) return null;
    Map<Node, Node> map = new HashMap<>();
    Node node = head;
    //copy当前链表：只copy基本类型
    while (node != null) {
        map.put(node, new Node(node.val, null, null));
        node = node.next;
    }
    // 分配random内容
    node = head;
    while (node != null) {
        //巧妙之处
        map.get(node).next = map.get(node.next);
        map.get(node).random = map.get(node.random);

        node = node.next;
    }
    return map.get(head);
}
```
