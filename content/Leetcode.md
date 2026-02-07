---
title: LeetCode
tags:
  - leetcode
  - 算法
---

# LeetCode

## 模板
### 链表
```python
def middleNode(head):
    slow = head
    fast = head
    while fast and fast.next:
        slow = slow.next
        fast = fast.next.next
    return slow


def reverseList(head):
    # dummy = ListNode(next=head)  # 可选：占位节点（这里不需要也能写）
    cur = head
    pre = None
    # cur 最后走到 None；pre 是反转后的头结点
    while cur:
        nxt = cur.next
        cur.next = pre
        pre = cur
        cur = nxt
    return pre
```

## P6 反转链表
- [x] 206. 反转链表：[题解](https://leetcode.cn/problems/reverse-linked-list/solution/you-xie-cuo-liao-yi-ge-shi-pin-jiang-tou-o5zy/)
- [x] 92. 反转链表 II：[题解](https://leetcode.cn/problems/reverse-linked-list-ii/solution/you-xie-cuo-liao-yi-ge-shi-pin-jiang-tou-teqq/)
![image.png](https://raw.githubusercontent.com/Marcoskk7/TyporaImageHosting/main/20260205131542150.png)
- [x] 25. K 个一组翻转链表：[题解](https://leetcode.cn/problems/reverse-nodes-in-k-group/solution/you-xie-cuo-liao-yi-ge-shi-pin-jiang-tou-plfs/)

### 课后作业

- [x] 24. 两两交换链表中的节点：[题目](https://leetcode.cn/problems/swap-nodes-in-pairs/)

- [x] 445. 两数相加 II：[题目](https://leetcode.cn/problems/add-two-numbers-ii/)

- [x] 2816. 翻倍以链表形式表示的数字：[题目](https://leetcode.cn/problems/double-a-number-represented-as-a-linked-list/)


## P7 快慢指针
- [x] 876. 链表的中间结点：[题解](https://leetcode.cn/problems/middle-of-the-linked-list/solution/mei-xiang-ming-bai-yi-ge-shi-pin-jiang-t-wzwm/)

- [x] 141. 环形链表：[题解](https://leetcode.cn/problems/linked-list-cycle/solution/mei-xiang-ming-bai-yi-ge-shi-pin-jiang-t-c4sw/)

- [x] 142. 环形链表 II：[题解](https://leetcode.cn/problems/linked-list-cycle-ii/solution/mei-xiang-ming-bai-yi-ge-shi-pin-jiang-t-nvsq/)

- [x] 143. 重排链表：[题解](https://leetcode.cn/problems/reorder-list/solution/mei-xiang-ming-bai-yi-ge-shi-pin-jiang-t-u66q/)

### 课后作业

- [x] 234. 回文链表：[题目](https://leetcode.cn/problems/palindrome-linked-list/)

- [x] 2130. 链表最大孪生和：[题目](https://leetcode.cn/problems/maximum-twin-sum-of-a-linked-list/)