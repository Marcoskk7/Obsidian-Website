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


## P8 前后指针

- [x] 237. 删除链表中的节点：[题解](https://leetcode.cn/problems/delete-node-in-a-linked-list/solution/ru-he-shan-chu-jie-dian-liu-fen-zhong-ga-x3kn/)
- [x] 19. 删除链表的倒数第 N 个结点：[题解](https://leetcode.cn/problems/remove-nth-node-from-end-of-list/solution/ru-he-shan-chu-jie-dian-liu-fen-zhong-ga-xpfs/)
- 讲解 dummynode 的使用情形，如果需要删除头结点，那么最好使用 dummynode

- [x] 83. 删除排序链表中的重复元素：[题解](https://leetcode.cn/problems/remove-duplicates-from-sorted-list/solution/ru-he-qu-zhong-yi-ge-shi-pin-jiang-tou-p-98g7/)
- [x] 82. 删除排序链表中的重复元素 II：[题解](https://leetcode.cn/problems/remove-duplicates-from-sorted-list-ii/solution/ru-he-qu-zhong-yi-ge-shi-pin-jiang-tou-p-2ddn/)

### 课后作业

- [x] 203. 移除链表元素：[题目](https://leetcode.cn/problems/remove-linked-list-elements/)
- [x] 3217. 从链表中移除在数组中存在的节点：[题目](https://leetcode.cn/problems/delete-nodes-from-linked-list-present-in-array/)
- 如果在 List 用查找那么时间复杂度是 O(n)。如果把 nums 变为一个 hash 集合，那么它的查找时间复杂度会降到 O(1)。 

- [x] 2487. 从链表中移除节点：[题目](https://leetcode.cn/problems/remove-nodes-from-linked-list/)
- 没思路的时候可以试试反转
- [x] 1669. 合并两个链表：[题目](https://leetcode.cn/problems/merge-in-between-linked-lists/)