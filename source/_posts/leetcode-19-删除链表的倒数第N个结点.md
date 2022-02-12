---
title: leetcode-19.删除链表的倒数第N个结点
date: 2022-02-12 23:44:48
update: 2022-02-12 23:44:48
tags: 链表
categories: Algorithm
---
[来源] 🔗[19. 删除链表的倒数第 N 个结点](https://leetcode-cn.com/problems/remove-nth-node-from-end-of-list/)

# 题目

给你一个链表，删除链表的倒数第 `n` 个结点，并且返回链表的头结点。

**示例 1:**

![image](image.png)

```JavaScript
输入：head = [1,2,3,4,5], n = 2
输出：[1,2,3,5]
```

**示例 2:**

```JavaScript
输入：head = [1], n = 1
输出：[]
```

**示例 2:**

```JavaScript
输入：head = [1,2], n = 1
输出：[1]
```

# 思路一

1. 我们可以先把链表的总长度len求出来
2. len - N 就是我们正序遍历需要删除的倒数第N个节点
3. 那么我们就可以定位到len - N - 1的节点，进行一个删除的动作

# 代码一

```JavaScript
 * class ListNode {
 *     val: number
 *     next: ListNode | null
 *     constructor(val?: number, next?: ListNode | null) {
 *         this.val = (val===undefined ? 0 : val)
 *         this.next = (next===undefined ? null : next)
 *     }
 * }
 */

function removeNthFromEnd(head: ListNode | null, n: number): ListNode | null {
    let pNode = new ListNode(0);
    pNode.next = head;

    let
        len = 0, // 统计链表长度
        temp = head;
    while (temp) {
        temp = temp.next;
        len++;
    }

    temp = pNode;
    for (let i = 0; i < len - n; i++) {
        temp = temp.next;
    }

    temp.next = temp.next.next;
    return pNode.next;

```

# 思路二

1. 定义双指针p，q
2. 让q前移N个节点，p此时仍处于原位，那么q和p的距离就是N
3. 此时同时移动p,q，那么到q指向null的时候，p刚好就指向要被删除的节点的前一个节点

# 代码二

```JavaScript
function removeNthFromEnd(head: ListNode | null, n: number): ListNode | null {
    let dummyHead = new ListNode(0);
    dummyHead.next = head;

    let p = dummyHead, q = dummyHead;

    for (let i = 0; i <= n; i++) {
        q = q.next;
    }

    while (q) {
        p = p.next;
        q = q.next;
    }

    p.next = p.next.next;
    return dummyHead.next;

};
```