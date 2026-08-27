---
tags:
  - 面试
  - 嵌入式面试
source: "Interview/Algorithm/linked_list.md"
created: 2026-08-27
---

> ## 🚀 在 EmbeddedInterviewLab 上练习与深入（Practice & deep-dive）
>
> 使用内置的 C/C++ 编辑器和 AI 评分评估，在浏览器中在线解答这些问题，还可浏览按难度排序的题库。
>
> 👉 **[使用 AI 反馈练习编码问题 →](https://embeddedinterviewlab.com/coding?utm_source=github&utm_medium=referral&utm_campaign=kb_cta&utm_content=interview_algorithm)** &nbsp;·&nbsp; **[打开面试题库 →](https://embeddedinterviewlab.com/questions?utm_source=github&utm_medium=referral&utm_campaign=kb_cta&utm_content=interview_algorithm)**

---

## 题目（Problems）

索引 # | 标题 | 难度 | 重要性/频率
----|----|----|---
1 | 反转链表（Reverse Linked List） | 简单 | *****
2 | 反转链表 II（Reverse Linked List II） | 中等 | ***
3 | 链表环（Linked List Cycle） | 简单 | *****
4 | 回文链表（Palindrome Linked List） | 简单 | ****
5 | 移除链表元素（Remove Linked List Elements） | 简单 | ****
6 | 删除链表节点（Delete Node in a Linked List） | 简单 | *****
7 | 删除排序链表中的重复元素（Remove Duplicates from Sorted List） | 简单 | *****
8 | 合并两个有序链表（Merge Two Sorted Lists） | 简单 | *****
9 | 二叉树展开为链表（Flatten Binary Tree to Linked List） | 中等 | **
10 | 相交链表（Intersection of Two Linked Lists） | 简单 | ****
11 | LRU 缓存（LRU cache） | 困难 | ****
12 | 链表的中间节点（Middle of linked list） | 简单 | ****
13 | 用链表实现队列（Implement queue by linked list） | 简单 | ****
14 | 重排链表（Reorder List） | 中等 | ****

## 实现（Implementation）

### **重排链表（Reorder List）**

***Big O：*** O(n) 时间，O(1) 空间
```
提示：

1. 找到中间节点。
2. 反转链表的后半部分。
3. 合并两个链表。
```
```c++
class Solution {
public:
	void reorderList(ListNode* head) {
		if ( ! head ) return;
		ListNode *slow = head, *fast = head;
		while ( fast->next && fast->next->next )
		{
			slow = slow->next;
			fast = fast->next->next;
		}
			
		
		ListNode *prev = NULL, *cur = slow->next, *save;
		while ( cur )
		{
			save = cur->next;
			cur->next = prev;
			prev = cur;
			cur = save;
		}
			
		slow->next = NULL;
		
		ListNode *head2 = prev;
		while ( head2 )
		{
			save = head->next;
			head->next = head2;
			head = head2;
			head2 = save;
		}      
	}
};
```

## 相关页面
- [[Array]] —— 数组相关算法题
- [[dataStructure]] —— 数据结构题（LRU 缓存等）
- [[string]] —— 字符串相关算法题
- [[matrix]] —— 矩阵相关算法题
- [[math]] —— 数学类算法题

返回索引 [[00-索引]]
