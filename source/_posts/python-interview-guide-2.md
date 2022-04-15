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

* 从尾到头打印链表

```
class Solution {
    vector<int> reversePrintListNode(ListNode *head) {
        vector<int> reverseList;
        while(head!=NULL) {
            reverseList.push_back(head->val);
            head=head->next;
        }
        
        reverse(reverseList.begin(), reverseList.end());
        return reverseList;
    }
};
```

* LRU缓存机制

```
struct CacheNode {
    string key;
    int value;
    CacheNode *prev;
    CacheNode *next;
    CacheNode(string _key, int _val) : key(_key), value(_val), prev(NULL), next(NULL) {}
};

class LRUCache {
private:
    unordered_map<string, CacheNode *> nodeTable;
    CacheNode *head;
    CacheNode *tail;
    int size;
    int capacity;

public:
    LRUCache(int _capacity) : size(0), capacity(_capacity) {
        head = new CacheNode("", -1);
        tail = new CacheNode("", -1);
        head->next = tail;
        tail->prev = head;
    }

    int get(string key) {
        if (nodeTable.find(key) == nodeTable.end()) {
            return -1;
        }

        CacheNode *node = nodeTable[key];
        removeNode(node);
        addToHead(node);
        return node->value;
    }

    void put(string key, int value) {
        if (nodeTable.find(key) == nodeTable.end()) {
            CacheNode *node = new CacheNode(key, value);
            nodeTable[key] = node;
            addToHead(node);
            ++size;

            if (size > capacity) {
                removeTail();
            }

        } else {
            CacheNode *node = nodeTable[key];
            node->value = value;
            removeNode(node);
            addToHead(node);
        }
    }

    void print() {
        CacheNode *node = head->next;
        while (node != tail) {
            cout << node->key << ":" << node->value << ",";
            node = node->next;
        }
        cout << endl;
    }

private:
    void addToHead(CacheNode *node) {
        node->next = head->next;
        node->prev = head;

        head->next->prev = node;
        head->next = node;
    }

    void removeNode(CacheNode *node) {
        node->prev->next = node->next;
        node->next->prev = node->prev;
    }

    CacheNode *removeTail() {
        if (tail->prev == head) {
            return NULL;
        }

        CacheNode *node = tail->prev;
        removeNode(node);
        nodeTable.erase(node->key);
        --size;
        return node;
    }
};
```

* 合并k个排序链表

```
class Solution {
public:
    ListNode *mergeKLists(vector<ListNode *> &lists) {
        return merge(lists, 0, lists.size() - 1);
    }

private:
    ListNode *merge(vector<ListNode *> &lists, int left, int right) {
        if (left == right) return lists[left];
        if (left > right) return NULL;
        int mid = left + (right - left) / 2;
        return mergeTwoLists(merge(lists, left, mid), merge(lists, mid + 1, right));
    }

    ListNode *mergeTwoLists(ListNode *head1, ListNode *head2) {
        if (head1 == NULL || head2 == NULL) {
            return head1 == NULL ? head2 : head1;
        }

        ListNode dummy(-1), *curr = &dummy;
        while (head1 != NULL && head2 != NULL) {
            if (head1->val < head2->val) {
                curr->next = head1;
                head1 = head1->next;
            } else {
                curr->next = head2;
                head2 = head2->next;
            }

            curr = curr->next;
        }

        curr->next = (head1 == NULL) ? head2 : head1;
        return dummy.next;
    }
};
```

### 5.3 栈和队列

* 两个栈实现队列

```
class MyQueue {
private:
    stack<int> in;
    stack<int> out;
public:
    MyQueue() {

    }

    void push(int x) {
        in.push(x);
    }

    int pop() {
        if (out.empty()) in2out();
        int res = out.top();
        out.pop();
        return res;
    }

    int peek() {
        if (out.empty()) in2out();
        return out.top();
    }

    bool empty() {
        return in.empty() && out.empty();
    }

private:
    void in2out() {
        while (!in.empty()) {
            int top = in.top();
            out.push(top);
            in.pop();
        }
    }
};
```

* 栈的压入、弹出序列

```
class Solution {
public:
    bool validateStackSequences(vector<int> &pushed, vector<int> &popped) {
        stack<int> stk;
        for (int i = 0, j = 0; i < pushed.size(); i++) {
            stk.push(pushed[i]);
            while (!stk.empty() && j < popped.size() && popped[j] == stk.top()) {
                stk.pop();
                j++;
            }
        }

        return stk.empty();
    }
};
```

* 包含min函数的栈

```
class MinStack {
private:
    stack<int> stk;
    stack<int> min_stk;
public:
    MinStack() {
        min_stk.push(INT_MAX);
    }

    void push(int val) {
        stk.push(val);
        min_stk.push(min(min_stk.top(), val));
    }

    void pop() {
        stk.pop();
        min_stk.pop();
    }

    int top() {
        return stk.top();
    }

    int getMin() {
        return min_stk.top();
    }
};
```

* 验证栈序列

```
class Solution {
public:
    bool validateStackSequences(vector<int>& pushed, vector<int>& popped) {
                stack<int> stk;

        if (pushed.size() != popped.size()) {
            return false;
        }

        for (int i = 0, j = 0; i < pushed.size(); i++) {
            stk.push(pushed[i]);
            while (!stk.empty() && stk.top() == popped[j]) {
                stk.pop();
                j++;
            }
        }
        
        return stk.empty();
    }
};
```

* 约瑟夫环

```
class Solution {
public:
    int lastRemaining(int n, int m) {
        if (m <= 0 || n <= 1) {
            return -1;
        }

        deque<int> queue;
        for (int i = 0; i < n; i++) {
            queue.push_back(i);
        }

        while (queue.size() > 1) {
            for (int count = 0; count < m - 1; count++) {
                int num = queue.front();
                queue.pop_front();
                queue.push_back(num);
            }
            queue.pop_front();
        }

        return queue.front();
    }
};
```

* 合并K个排序链表

```
struct PriorityListNode {
    int val;
    ListNode *ptr;

    bool operator<(const PriorityListNode &rhs) const {
        return val > rhs.val;
    }
};

class Solution {
private:
    priority_queue<PriorityListNode> q;
public:
    ListNode *mergeKLists(vector<ListNode *> &lists) {
        for (auto node: lists) {
            if (node != NULL) {
                q.push({node->val, node});
            }
        }
        ListNode dummy(-1);
        ListNode *curr = &dummy;
        while (!q.empty()) {
            auto top = q.top();
            q.pop();
            curr->next = top.ptr;
            curr = curr->next;
            if (top.ptr->next != NULL)
                q.push({top.ptr->next->val, top.ptr->next});
        }
        return dummy.next;
    }
};
```


### 5.4 二叉树算法

* 前序遍历

* 中序遍历

* 后序遍历

* 层序遍历

* 最小深度

```
class Solution {
public:
    int minDepth(TreeNode *root) {
        int depth = 0;
        if (root == NULL) return depth;

        queue<TreeNode *> prevQ, nextQ;
        prevQ.push(root);
        
        while (!prevQ.empty()) {
            while (!prevQ.empty()) {
                TreeNode *curr = prevQ.front();
                if (curr->left == NULL && curr->right == NULL) return depth + 1;
                
                if (curr->left!=NULL) {
                    nextQ.push(curr->left);
                }
                if (curr->right!=NULL) {
                    nextQ.push(curr->right);
                }
                prevQ.pop();
            }

            depth = depth + 1;
            queue<TreeNode *> tmp = prevQ;
            prevQ = nextQ;
            nextQ = tmp;
        }
        
        return depth;
    }
};
```

* 层平均值

```


```



* 直径

```
class Solution {
private:
    int diameter;
public:
    Solution() : diameter(0) {}

    // 一个节点的最大直径，是左右深度组成的路径，与左右子树的直径三者的最大值
    int diameterOfBinaryTree(TreeNode *root) {
        dfs(root);
        return diameter;
    }

    int dfs(TreeNode *root) {
        if (root == NULL) {
            return 0;
        }

        int left = dfs(root->left);
        int right = dfs(root->right);
        diameter = max(diameter, left + right);
        return max(left, right) + 1;
    }
};
```

* 最低公共祖先节点

```
class Solution {
private:
    TreeNode *pNode, *qNode;
    TreeNode *ancestor;
public:
    // 后序遍历，返回第一个节点
    TreeNode *lowestCommonAncestor(TreeNode *root, TreeNode *p, TreeNode *q) {
        pNode = p, qNode = q, ancestor = NULL;
        postTraverse(root);
        return ancestor;
    }

private:
    pair<bool, bool> postTraverse(TreeNode *root) {
        if (root == NULL) {
            return make_pair(false, false);
        }

        pair<bool, bool> leftStatus = postTraverse(root->left);
        pair<bool, bool> rightStatus = postTraverse(root->right);

        pair<bool, bool> result;
        if (leftStatus.first || rightStatus.first || root == pNode) {
            result.first = true;
        }
        if (leftStatus.second || rightStatus.second || root == qNode) {
            result.second = true;
        }

        if (result.first && result.second && ancestor == NULL) {
            ancestor = root;
        }

        return result;
    }
};
```

### 5.5 字符串算法

* 大数相乘

```
class Solution {
public:
    string multiply(string s1, string s2) {
        if (s1.empty() || s2.empty()) {
            return "0";
        }

        int size1 = s1.size();
        int size2 = s2.size();
        vector<int> resultList(size1 + size2, 0);
        for (int i = size1 - 1; i >= 0; i--) {
            int num1 = s1[i] - '0';
            for (int j = size2 - 1; j >= 0; j--) {
                int index = size1 + size2 - i - j - 2;
                int num2 = s2[j] - '0';
                int sum = resultList[index] + num1 * num2;
                resultList[index] = sum % 10;
                resultList[index + 1] += sum / 10;
            }
        }

        string result;
        int index = resultList.size() - 1;
        while (index > 0 && resultList[index] == 0) index--;
        while (index >= 0) {
            result.push_back('0' + resultList[index]);
            index--;
        }

        return result;
    }
};
```

* 最长公共前缀

```
class Solution {
public:
    string longestCommonPrefix(vector<string> &strs) {
        if (strs.empty()) {
            return "";
        } else {
            return longestCommonPrefix(strs, 0, strs.size() - 1);
        }
    }

    string longestCommonPrefix(const vector<string> &strs, int start, int end) {
        if (start == end) {
            return strs[start];
        } else {
            int mid = start + (end - start) / 2;
            string leftStr = longestCommonPrefix(strs, start, mid);
            string rightStr = longestCommonPrefix(strs, mid + 1, end);
            return commonPrefix(leftStr, rightStr);
        }
    }

    string commonPrefix(const string &leftStr, const string &rightStr) {
        int minLength = min(leftStr.size(), rightStr.size());
        int index = 0;
        string commonStr = "";
        while (index < minLength && leftStr[index] == rightStr[index]) {
            commonStr.push_back(leftStr[index++]);
        }

        return commonStr;
    }
};
```

* 最长回文子串

```
class Solution {
public:
    string longestPalindrome(string s) {
        int size = s.size();
        if (size < 2) return s;

        vector<vector<bool>> dp(size, vector<bool>(size, 0));
        int maxLen = 1;
        int begin = 0;

        for (int i = 0; i < size; i++) {
            dp[i][i] = true;
        }

        for (int len = 2; len <= size; len++) {
            for (int i = 0; i < size; i++) {
                int j = len + i - 1;
                if (j >= size) break;
                if (s[i] != s[j]) dp[i][j] = false;
                else if (j - i < 3) dp[i][j] = true;
                else {
                    dp[i][j] = dp[i + 1][j - 1];
                }

                if (dp[i][j] && j - i + 1 > maxLen) {
                    maxLen = j - i + 1;
                    begin = i;
                }
            }
        }

        return s.substr(begin, maxLen);
    }
};
```

* 无重复最长子串

```
class Solution {
public:
    int lengthOfLongestSubstring(string s) {
        int size = s.size();
        int maxLen = 0, left = 0;
        unordered_set<char> lookup;

        for (int i = 0; i < size; i++) {
            while (lookup.find(s[i]) != lookup.end()) {
                lookup.erase(s[left]);
                left++;
            }

            maxLen = max(maxLen, i - left + 1);
            lookup.insert(s[i]);
        }

        return maxLen;
    }
};
```

### 5.6 哈希算法

* 实现哈希表

* 两数之和

* 三数之和

* 重复的DNA序列

* 两个数组的交集

### 5.7 回溯

### 5.8 分治

### 5.9 动态规划

```
class Solution {
public:
    int longestValidParentheses(string s) {
        if (s.size() <= 1) {
            return 0;
        }

        vector<int> dp(s.size(), 0);
        int maxLen = 0;
        for (int i = 1; i < s.size(); i++) {
            if (s[i] == ')') {
                if (s[i - 1] == '(') {
                    dp[i] = ((i >= 2) ? dp[i - 2] : 0) + 2;
                } else if (i - 1 - dp[i - 1] >= 0 && s[i - 1 - dp[i - 1]] == '(') { // ')'
                    dp[i] = dp[i - 1] + ((i - 2 - dp[i - 1] >= 0) ? dp[i - 2 - dp[i - 1]] : 0) + 2;
                }

                maxLen = max(maxLen, dp[i]);
            }
        }

        return maxLen;
    }
};
```



