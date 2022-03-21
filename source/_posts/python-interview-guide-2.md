---
title: 「MorsoLi/python-interview-guide」学习笔记(中)
catalog: true
date: 2022-03-20 17:00:11
subtitle:
header-img:
tags:
- interview
categories:
- 工程
---

> 参考资料链接 [Github链接](https://github.com/MorsoLi/python-interview-guide#wsgi-uwsgi-uwsgi-%E5%8C%BA%E5%88%AB)
> 
> 书籍豆瓣链接：[《流畅的Python》](https://book.douban.com/subject/27028517/)
> 
> 开始学习时间：
> 
> 预计完成时间：
> 
> 实际完成时间：


## 四、数据结构与算法
### 5.1 排序
* 快速排序

```
class Solution {
public:
    void quickSort(vector<int> &nums) {
        quickSort(nums.begin(), nums.end());
    }

private:
    void quickSort(vector<int>::iterator begin, vector<int>::iterator end) {
        if (begin >= end) {
            return;
        }

        auto pivot = partition(begin, end);
        quickSort(begin, pivot);
        quickSort(next(pivot), end);
    }

    vector<int>::iterator partition(vector<int>::iterator begin, vector<int>::iterator end) {
        // step1: pivot取第一个元素，index始终指向第一个不小于pivot的元素
        // step2: iter从next(pivot)开始迭代，当遇到比pivot小的数，将index和iter的值互换并自增
        // step3: iter迭代完成，将index和pivot互换
        int pivotNum = *begin;
        auto index = next(begin);
        for (auto iter = next(begin); iter != end; iter++) {
            if (*iter < pivotNum) {
                iter_swap(iter, index);
                index++;
            }
        }
        iter_swap(prev(index), begin);
        return prev(index);
    }
};
```

### 5.2 链表
* 倒数K节点

```
class Solution {
public:
    ListNode *dellLastKNode(ListNode *head, int k) {
        if (head == NULL || head->next == NULL || k < 1) {
            return head;
        }

        ListNode *fast = head, *slow = head;
        for (int i = 0; i < k && fast != NULL; i++) {
            fast = fast->next;
        }

        if (fast == NULL) {
            return head->next;
        }

        while (fast->next != NULL) {
            fast = fast->next;
            slow = slow->next;
        }

        slow->next = slow->next->next;
        return head;
    }
}
```

* 有序链表合并

```
class Solution {
public:
    ListNode *mergeTwoSortedList(ListNode *head1, ListNode *head2) {
        ListNode dummy(-1);
        ListNode *curr = &dummy;

        while (head1 != NULL && head2 != NULL) {
            if (head1->val <= head2->val) {
                curr->next = head1;
                head1 = head1->next;
            } else {
                curr->next = head2;
                head2 = head2->next;
            }
            curr = curr->next;
        }

        if (head1 != NULL) {
            curr->next = head1;
        }

        if (head2 != NULL) {
            curr->next = head2;
        }

        return dummy.next;
    }
};
```

* 环检测

```
class Solution {
public:
    bool detectCircle(ListNode *head) {
        if (head == NULL || head->next == NULL) {
            return 0;
        }

        ListNode *fast = head, *slow = head;
        int len = 0;
        while (fast->next != NULL && fast->next->next != NULL) {
            fast = fast->next->next;
            slow = slow->next;
            len++;
            // 相遇即有环，且步数是环长
            if (fast == slow) {
                return len;
            }
        }

        return 0;
    }
};
```

* 大数加法

```
class Solution {
    ListNode *bigNumberSum(ListNode *head1, ListNode *head2) {
        ListNode dummy(-1);
        ListNode *curr = &dummy;
        int carryBit = 0;
        while (head1 || head2 || carryBit) {
            int value1 = head1 == NULL ? 0 : head1->val;
            int value2 = head2 == NULL ? 0 : head2->val;
            
            int sumNum = value1 + value2 + carryBit;
            carryBit = carryBit/10;
            
            curr->next = new ListNode(sumNum % 10);
            curr = curr->next;
        }
        
        return dummy.next;
    }
};
```

* 删除链表节点

```
```

* 从尾到头打印链表

```
```

* 反转链表

```
```

* LRU缓存机制

```
```

### 5.3 栈和队列

* 有效的括号

* 两个栈实现队列

* 栈的压入、弹出序列

* 包含min函数的栈

* 验证栈序列

* 约瑟夫环

* 合并K个排序链表

### 5.4 二叉树算法

* 前序遍历

* 中序遍历

* 后序遍历

* 层序遍历

* 最小深度

* 层平均值

* 直径

* 最低公共祖先节点

### 5.5 字符串算法

* 最长公共前缀

* 最长回文子串

* 无重复最长子串

### 5.6 哈希算法

* 实现哈希表

* 两数之和

* 三数之和

* 重复的DNA序列

* 两个数组的交集

### 5.7 常见算法基本思想

* 回溯

* 分治

* 动态规划




