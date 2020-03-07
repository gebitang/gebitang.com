+++
title = "算法练习Stack(四)--P155"
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



[link on JianShu](https://www.jianshu.com/p/220070c413c5)

[栈stack](https://leetcode.com/tag/stack/)    
[155. Min Stack](https://leetcode.com/problems/min-stack/) Easy

Design a stack that supports push, pop, top, and retrieving the minimum element in constant time.

>push(x) -- Push element x onto stack.
pop() -- Removes the element on top of the stack.
top() -- Get the top element.
getMin() -- Retrieve the minimum element in the stack.
 

Example:

>MinStack minStack = new MinStack();
minStack.push(-2);
minStack.push(0);
minStack.push(-3);
minStack.getMin();   --> Returns -3.
minStack.pop();
minStack.top();      --> Returns 0.
minStack.getMin();   --> Returns -2.



审题，这是Stack类型下的算法题，再设计一个Stack：需要保证`push, pop, top, getMin`在常数时间内返回值。

算法与数据结构，数据结构很关键。
- push、pop是stack类型本身自带的方法并且支持常数操作
- `top, getMin`需要在常数时间内返回值，需要确保方法调用时依然可以利用stack本身的属性。以为着可以利用pop、seek方法将top、getMin的值一直保留在栈顶。

这样需要设计一个数据结构完成**将top、getMin的值一直保留在栈顶**的任务。stack本身存储的是这个结构类型的数据。

这样就比较轻松实现了：

Runtime: 5 ms, faster than 89.54% of Java online submissions for Min Stack.
Memory Usage: 40.2 MB, less than 29.71% of Java online submissions for Min Stack.
```
static class MinNode{
        int value;
        int minValue;
        MinNode(int v, int mv){
            value = v;
            minValue = mv;
        }
    }

    Stack<MinNode> stack;
    /** initialize your data structure here. */
    public MinStack() {
        stack = new Stack<>();
    }

    public void push(int x) {
        int min = (stack.isEmpty()) ? x : Math.min(stack.peek().minValue, x);
        MinNode node = new MinNode(x, min);
        stack.push(node);
    }

    public void pop() {
        stack.pop();
    }

    public int top() {
        return stack.peek().value;
    }

    public int getMin() {
        return stack.peek().minValue;
    }
```

