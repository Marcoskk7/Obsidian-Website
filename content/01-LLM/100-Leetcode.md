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

## 二叉树层序遍历 bfs
~~~python
def levelOrder(self, root: Optional[TreeNode]) -> List[List[int]]:  
    if root is None:  
        return []  
    ans = []  
    q = deque([root])  
    while q:  
        vals = []  
        for _ in range(len(q)):  
            node = q.popleft()  
            vals.append(node.val)  
            if node.left: q.append(node.left)  
            if node.right: q.append(node.right)  
        ans.append(vals)  
  
    return ans
~~~


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

## P9 二叉树 递归 数学归纳法

核心思想，写递归的时候不用考虑每一层子递归是怎么操作的，只要想明白单层是怎么做的，剩下的会自动正确

递归的核心：
首先要确定什么时候"归"
其次确定每一层循环要执行的函数

- [x] 104. 二叉树的最大深度：[题解](https://leetcode.cn/problems/maximum-depth-of-binary-tree/solution/kan-wan-zhe-ge-shi-pin-rang-ni-dui-di-gu-44uz/)

### 课后作业

- [x] 111. 二叉树的最小深度：[题目](https://leetcode.cn/problems/minimum-depth-of-binary-tree/)
- [x] 404. 左叶子之和：[题目](https://leetcode.cn/problems/sum-of-left-leaves/)
- [x] 112. 路径总和：[题目](https://leetcode.cn/problems/path-sum/)
- [ ] 129. 求根节点到叶节点数字之和：[题目](https://leetcode.cn/problems/sum-root-to-leaf-numbers/)
- [ ] 1448. 统计二叉树中好节点的数目：[题目](https://leetcode.cn/problems/count-good-nodes-in-binary-tree/)
- [ ] 987. 二叉树的垂序遍历：[题目](https://leetcode.cn/problems/vertical-order-traversal-of-a-binary-tree/)
## P10 二叉树 相同 对称 平衡

- [x] 100. 相同的树：[题解](https://leetcode.cn/problems/same-tree/solution/ru-he-ling-huo-yun-yong-di-gui-lai-kan-s-empk/)
- [x] 101. 对称二叉树：[题解](https://leetcode.cn/problems/symmetric-tree/solution/ru-he-ling-huo-yun-yong-di-gui-lai-kan-s-6dq5/)
- [x] 110. 平衡二叉树：[题解](https://leetcode.cn/problems/balanced-binary-tree/solution/ru-he-ling-huo-yun-yong-di-gui-lai-kan-s-c3wj/)
- [x] 199. 二叉树的右视图：[题解](https://leetcode.cn/problems/binary-tree-right-side-view/solution/ru-he-ling-huo-yun-yong-di-gui-lai-kan-s-r1nc/)

### 课后作业

- [x] 965. 单值二叉树：[题目](https://leetcode.cn/problems/univalued-binary-tree/)
- [ ] 951. 翻转等价二叉树：[题目](https://leetcode.cn/problems/flip-equivalent-binary-trees/)
- [x] 226. 翻转二叉树：[题目](https://leetcode.cn/problems/invert-binary-tree/)
- [x] 617. 合并二叉树：[题目](https://leetcode.cn/problems/merge-two-binary-trees/)
- [ ] 2331. 计算布尔二叉树的值：[题目](https://leetcode.cn/problems/evaluate-boolean-binary-tree/)
- [x] 508. 出现次数最多的子树元素和：[题目](https://leetcode.cn/problems/most-frequent-subtree-sum/)
- [ ] 1026. 节点与其祖先之间的最大差值：[题目](https://leetcode.cn/problems/maximum-difference-between-node-and-ancestor/)
- [ ] 1372. 二叉树中的最长交错路径：[题目](https://leetcode.cn/problems/longest-zigzag-path-in-a-binary-tree/)
- [ ] 1080. 根到叶路径上的不足节点：[题目](https://leetcode.cn/problems/insufficient-nodes-in-root-to-leaf-paths/)

## P11 二叉搜索树 前序 中序 后序

做这类题可以先根据样例，心中模拟一遍流程，然后依旧是指盯着当前的循环即可
前序，中左右，把范围传下去
中序，左中右，一定严格递增
后序，左右中，把范围向上报

- [x] 98. 验证二叉搜索树：[题解](https://leetcode.cn/problems/validate-binary-search-tree/solution/qian-xu-zhong-xu-hou-xu-san-chong-fang-f-yxvh/)
- [x] 前序方法
- [x] 中序方法
- [x] 后序方法
### 课后作业

- [x] 700. 二叉搜索树中的搜索：[题目](https://leetcode.cn/problems/search-in-a-binary-search-tree/)
- [x] 938. 二叉搜索树的范围和：[题目](https://leetcode.cn/problems/range-sum-of-bst/)
- [ ] 530. 二叉搜索树的最小绝对差：[题目](https://leetcode.cn/problems/minimum-absolute-difference-in-bst/)
- [ ] 2476. 二叉搜索树最近节点查询：[题目](https://leetcode.cn/problems/closest-nodes-queries-in-a-binary-search-tree/)
- [ ] 501. 二叉搜索树中的众数：[题目](https://leetcode.cn/problems/find-mode-in-binary-search-tree/)
- [x] 230. 二叉搜索树中第 K 小的元素：[题目](https://leetcode.cn/problems/kth-smallest-element-in-a-bst/)
- [ ] 1373. 二叉搜索子树的最大键值和：[题目](https://leetcode.cn/problems/maximum-sum-bst-in-binary-tree/)
- [ ] 105. 从前序与中序遍历序列构造二叉树：[题目](https://leetcode.cn/problems/construct-binary-tree-from-preorder-and-inorder-traversal/)
- [ ] 106. 从中序与后序遍历序列构造二叉树：[题目](https://leetcode.cn/problems/construct-binary-tree-from-inorder-and-postorder-traversal/)
- [ ] 889. 根据前序和后序遍历构造二叉树：[题目](https://leetcode.cn/problems/construct-binary-tree-from-preorder-and-postorder-traversal/)
- [ ] 1110. 删点成林：[题目](https://leetcode.cn/problems/delete-nodes-and-return-forest/)

## P12 二叉树 最近公共祖先

> 此类问题的重点是想明白每一种情况，怎么进行分类讨论
> 只要画好分类图，解法就迎刃而解了

> 这类递归算法，在考虑空间复杂度的时候，要考虑调用栈的使用，
> 函数调用栈，一层层调用，也会使用 O（N）的位置
- [x] 236. 二叉树的最近公共祖先：[题解](https://leetcode.cn/problems/lowest-common-ancestor-of-a-binary-tree/solutions/2023872/fen-lei-tao-lun-luan-ru-ma-yi-ge-shi-pin-2r95/)
- [x] 235. 二叉搜索树的最近公共祖先：[题解](https://leetcode.cn/problems/lowest-common-ancestor-of-a-binary-search-tree/solutions/2023873/zui-jin-gong-gong-zu-xian-yi-ge-shi-pin-8h2zc/)

### 课后作业

- [x] 1123. 最深叶节点的最近公共祖先：[题目](https://leetcode.cn/problems/lowest-common-ancestor-of-deepest-leaves/)

## P13 二叉树 层序遍历 BFS 队列

- [x] 102. 二叉树的层序遍历：[题解](https://leetcode.cn/problems/binary-tree-level-order-traversal/solution/bfs-wei-shi-yao-yao-yong-dui-lie-yi-ge-s-xlpz/)
- [x] 103. 二叉树的锯齿形层序遍历：[题解](https://leetcode.cn/problems/binary-tree-zigzag-level-order-traversal/solution/bfs-wei-shi-yao-yao-yong-dui-lie-yi-ge-s-xlv3/)
- [x] 513. 找树左下角的值：[题解](https://leetcode.cn/problems/find-bottom-left-tree-value/solution/bfs-wei-shi-yao-yao-yong-dui-lie-yi-ge-s-f34y/)

### 课后作业

- [x] 107. 二叉树的层序遍历 II：[题目](https://leetcode.cn/problems/binary-tree-level-order-traversal-ii/)
- [x] 104. 二叉树的最大深度：[题目](https://leetcode.cn/problems/maximum-depth-of-binary-tree/)
- [ ] 111. 二叉树的最小深度：[题目](https://leetcode.cn/problems/minimum-depth-of-binary-tree/)
- [x] 2583. 二叉树中的第 K 大层和：[题目](https://leetcode.cn/problems/kth-largest-sum-in-a-binary-tree/)
- [ ] 199. 二叉树的右视图：[题目](https://leetcode.cn/problems/binary-tree-right-side-view/)
- [ ] 116. 填充每个节点的下一个右侧节点指针：[题目](https://leetcode.cn/problems/populating-next-right-pointers-in-each-node/)
- [ ] 117. 填充每个节点的下一个右侧节点指针 II：[题目](https://leetcode.cn/problems/populating-next-right-pointers-in-each-node-ii/)
- [ ] 1302. 层数最深叶子节点的和：[题目](https://leetcode.cn/problems/deepest-leaves-sum/)
- [x] 1609. 奇偶树：[题目](https://leetcode.cn/problems/even-odd-tree/)
- [ ] 2415. 反转二叉树的奇数层：[题目](https://leetcode.cn/problems/reverse-odd-levels-of-binary-tree/)
- [ ] 2641. 二叉树的堂兄弟节点 II：[题目](https://leetcode.cn/problems/cousins-in-binary-tree-ii/)

## P14 子集型回溯 分割回文串

增量问题，适合使用回溯实现，回溯通过递归实现

> 回溯三问
> - 当前操作？枚举 path[i]要填入的字母
> - 子问题？构造字符串>=i 的部分
> - 下一个子问题？构造字符串>=i+1 的部分

- [x] 17. 电话号码的字母组合：[题解](https://leetcode.cn/problems/letter-combinations-of-a-phone-number/solutions/2059416/hui-su-bu-hui-xie-tao-lu-zai-ci-pythonja-3orv/)
- [x] 78. 子集：[题解](https://leetcode.cn/problems/subsets/solutions/2059409/hui-su-bu-hui-xie-tao-lu-zai-ci-pythonja-8tkl/)
- [ ] 131. 分割回文串：[题解](https://leetcode.cn/problems/palindrome-partitioning/solutions/2059414/hui-su-bu-hui-xie-tao-lu-zai-ci-pythonja-fues/)

### 课后作业

- [ ] 257. 二叉树的所有路径：[题目](https://leetcode.cn/problems/binary-tree-paths/) 
- [ ] 113. 路径总和 II：[题目](https://leetcode.cn/problems/path-sum-ii/) 
- [ ] 784. 字母大小写全排列：[题目](https://leetcode.cn/problems/letter-case-permutation/)
- [ ] LCP 51. 烹饪料理：[题目](https://leetcode.cn/problems/UEcfPD/)
- [ ] 2397. 被列覆盖的最多行数：[题目](https://leetcode.cn/problems/maximum-rows-covered-by-columns/)
- [ ] 1239. 串联字符串的最大长度：[题目](https://leetcode.cn/problems/maximum-length-of-a-concatenated-string-with-unique-characters/)
- [ ] 2212. 射箭比赛中的最大得分：[题目](https://leetcode.cn/problems/maximum-points-in-an-archery-competition/)
- [ ] 2698. 求一个整数的惩罚数：[题目](https://leetcode.cn/problems/find-the-punishment-number-of-an-integer/)
- [ ] 93. 复原 IP 地址：[题目](https://leetcode.cn/problems/restore-ip-addresses/)