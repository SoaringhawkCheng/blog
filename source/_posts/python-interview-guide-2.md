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
### 4.1 排序
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

### 4.2 链表
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

### 4.3 栈和队列

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


### 4.4 二叉树算法

* 前序遍历

```
class Solution {
public:
    vector<int> preorderTraversal(TreeNode *root) {
        vector<int> result;

        stack<TreeNode *> stk;
        stk.push(root);

        while (!stk.empty()) {
            TreeNode *top = stk.top();
            stk.pop();
            if (top==NULL) continue;

            result.push_back(top->val);
            stk.push(top->right);
            stk.push(top->left);
        }

        return result;
    }
};
```

* 中序遍历

```
class Solution {
public:
    vector<int> inorderTraversal(TreeNode *root) {
        vector<int> result;
        stack<TreeNode *> stk;
        TreeNode *curr = root;
        while (!stk.empty() || curr != NULL) {
            while (curr != NULL) {
                stk.push(curr);
                curr = curr->left;
            }

            curr = stk.top();
            stk.pop();
            result.push_back(curr->val);
            curr = curr->right;
        }
        return result;
    }
};
```

* 后序遍历

```
class Solution {
public:
    vector<int> postorderTraversal(TreeNode *root) {
        vector<int> result;

        stack<TreeNode *> stk;
        stk.push(root);

        while (!stk.empty()) {
            TreeNode *top = stk.top();
            stk.pop();
            if (top == NULL) continue;

            result.push_back(top->val);
            stk.push(top->left);
            stk.push(top->right);
        }

        reverse(result.begin(), result.end());
        return result;
    }
};
```

* 层序遍历

```
class Solution {
public:
    vector<vector<int>> levelOrder(TreeNode* root) {
        queue<TreeNode *> q;
        q.push(root);
        vector<vector<int>> result;

        while(!q.empty()) {
            int size = q.size();
            vector<int> level;
            for (int i=0;i<size;i++) {
                TreeNode *front = q.front();
                q.pop();
                if (front==NULL) continue;
                level.push_back(front->val);
                q.push(front->left);
                q.push(front->right);
            }

            if (!level.empty()) {
                result.push_back(level);
            }
        }

        return result;
    }
};
```

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
class Solution {
public:
    vector<double> averageOfLevels(TreeNode *root) {
        queue<TreeNode *> q;
        vector<double> result;

        if (root == NULL) {
            return result;
        }

        q.push(root);
        while (!q.empty()) {
            int size = q.size();
            double avg = 0.0;
            for (int i = 0; i < size; i++) {
                TreeNode *front = q.front();
                avg += front->val;
                if (front->left != NULL) q.push(front->left);
                if (front->right != NULL) q.push(front->right);
                q.pop();
            }

            if (size>0) {
                result.push_back(avg/=size);
            }
        }

        return result;
    }
};

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

### 4.5 字符串算法

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

### 4.6 哈希算法

* 实现哈希表

```
class MyHashMap {
private:
    vector<list<pair<int, int>>> table;
    int bucketSize;
public:
    MyHashMap(int _bucketSize) {
        bucketSize = _bucketSize;
        table = vector<list<pair<int, int>>>(bucketSize, list<pair<int, int>>());
    }

    void put(int key, int value) {
        int hash = key % bucketSize;
        list<pair<int, int>> &bucketList = table[hash];
        for (auto iter = bucketList.begin(); iter != bucketList.end(); iter++) {
            if (iter->first==key) {
                iter->second=value;
                return;
            }
        }
        bucketList.push_back(make_pair(key, value));
    }

    int get(int key) {
        int hash = key % bucketSize;
        list<pair<int, int>> &bucketList = table[hash];
        for (auto iter = bucketList.begin(); iter != bucketList.end(); iter++) {
            if (iter->first==key) {
                return iter->second;
            }
        }
        return -1;
    }

    void remove(int key) {
        int hash = key % bucketSize;
        list<pair<int, int>> &bucketList = table[hash];
        for (auto iter = bucketList.begin(); iter != bucketList.end(); iter++) {
            if (iter->first==key) {
                bucketList.erase(iter);
                return;
            }
        }
    }
};
```

* 两数之和

```
class Solution {
public:
    vector<int> twoSum(vector<int> &nums, int target) {
        unordered_map<int, int> numIndex;
        for (int i = 0; i < nums.size(); i++) {
            if (numIndex.find(target - nums[i]) != numIndex.end()) {
                return {numIndex[target - nums[i]], i};
            } else {
                numIndex[nums[i]] = i;
            }
        }
        return vector<int>();
    }
};
```

* 三数之和

```
class Solution {
public:
    vector<vector<int>> threeSum(vector<int> &nums) {
        sort(nums.begin(), nums.end());
        vector<vector<int>> result;
        for (int i = 0; i < nums.size() - 2; i++) {
            int target = nums[i];
            int left = i + 1;
            int right = nums.size() - 1;

            while (left < right) {
                int sum = nums[left] + nums[right] + target;
                if (sum == 0) {
                    result.push_back({target, nums[left], nums[right]});
                    break;
                } else if (sum < 0) {
                    left++;
                } else {
                    right--;
                }
            }
        }
        return result;
    }
};

```

* 重复的DNA序列

```
class Solution {
public:
    vector<string> findRepeatedDnaSequences(string s) {
        unordered_map<string, int> subStrMap;
        vector<string> result;
        if (s.size() < 10) {
            return result;
        }

        for (int i = 0; i <= s.size() - 10; i++) {
            string subStr = s.substr(i, 10);
            if (subStrMap.find(subStr) == subStrMap.end()) {
                subStrMap[subStr] = 1;
            } else if (subStrMap[subStr] == 1) {
                result.push_back(subStr);
                subStrMap[subStr]++;
            }
        }

        return result;
    }
};
```

* 两个数组的交集

```
class Solution {
public:
    vector<int> intersect(vector<int> &nums1, vector<int> &nums2) {
        if (nums1.size() > nums1.size()) {
            return intersect(nums2, nums1);
        }

        unordered_map<int, int> numMap;
        for (auto num: nums1) {
            if (numMap.find(num) == numMap.end()) {
                numMap[num] = 0;
            }
            numMap[num]++;
        }

        vector<int> result;
        for (auto num: nums2) {
            if (numMap.find(num) != numMap.end() && numMap[num] > 0) {
                numMap[num]--;
                result.push_back(num);
            }
        }

        return result;
    }
};
```

### 4.7 回溯

* 括号生成

```
class Solution {
private:
    int countDown;
    string brackets;
    int limit;
    string currStr;
    vector<string> result;
public:
    Solution() : countDown(0), brackets("()") {}

    vector<string> generateParenthesis(int n) { // DFS
        limit = n;
        dfs();
        return result;
    }

    void dfs() {
        if (currStr.size() == 2 * limit) {
            if (countDown == 0) {
                result.push_back(currStr);
            }
            return;
        }

        if (countDown < limit) {
            currStr.push_back(brackets[0]);
            countDown++;
            dfs();
            countDown--;
            currStr.pop_back();
        }

        if (countDown > 0) {
            currStr.push_back(brackets[1]);
            countDown--;
            dfs();
            countDown++;
            currStr.pop_back();
        }
    }
};
```

* 组合总和

```
class Solution {
private:
    int currSum;
    vector<int> currList;
    vector<vector<int>> result;
public:
    Solution() : currSum(0) {}

    vector<vector<int>> combinationSum(vector<int> &candidates, int target) {
        sort(candidates.begin(), candidates.end());
        dfs(candidates, target, 0);
        return result;
    }

private:
    void dfs(const vector<int> &candidates, int target, int currIndex) {
        if (currSum == target && !currList.empty()) {
            result.push_back(currList);
        }

        if (currSum >= target) {
            return;
        }

        for (int i = currIndex; i < candidates.size(); i++) {
            currList.push_back(candidates[i]);
            currSum += candidates[i];
            dfs(candidates, target, i);
            currSum -= candidates[i];
            currList.pop_back();
        }
    }
};
```

* 组合总和2

```
class Solution {
private:
    vector<pair<int, int>> freq;
    int currSum;
    vector<int> currList;
    vector<vector<int>> result;
public:
    Solution() : currSum(0) {}

    vector<vector<int>> combinationSum2(vector<int> &candidates, int target) {
        sort(candidates.begin(), candidates.end());
        for (auto candidate: candidates) {
            if (freq.empty() || freq.back().first != candidate) {
                freq.push_back(make_pair(candidate, 1));
            } else {
                freq.back().second++;
            }
        }

        dfs(freq, target, 0);
        return result;
    }

private:
    void dfs(vector<pair<int, int>> freq, int target, int currIndex) {
        if (currSum == target && !currList.empty()) {
            result.push_back(currList);
        }

        if (currSum >= target) {
            return;
        }

        for (int i = currIndex; i < freq.size(); i++) {
            pair<int,int> &numFreq = freq[i];
            if (numFreq.second==0) continue;

            currList.push_back(freq[i].first);
            currSum += freq[i].first;
            numFreq.second--;
            dfs(freq, target, i);
            numFreq.second++;
            currSum -= freq[i].first;
            currList.pop_back();
        }
    }
};
```

* 全排列

```
class Solution {
private:
    unordered_map<int, bool> visited;
    vector<int> path;
    vector<vector<int>> results;
public:
    vector<vector<int>> permute(vector<int> &nums) {
        for (int i = 0; i < nums.size(); i++) {
            visited[i] = false;
        }

        dfs(nums);
        return results;
    }

    void dfs(const vector<int> &nums) {
        if (path.size() == nums.size()) {
            results.push_back(path);
            return;
        }

        for (int i = 0; i < nums.size(); i++) {
            if (!visited[i]) {
                path.push_back(nums[i]);
                visited[i] = true;
                dfs(nums);
                visited[i] = false;
                path.pop_back();

            }
        }
    }
};
```

* 单词搜索

```
class Solution {
private:
    bool wordExists;
    vector<pair<int, int>> directs;
    vector<vector<bool>> visited;
public:
    Solution() {
        wordExists = false;
        directs = {{-1, 0},
                   {1,  0},
                   {0,  -1},
                   {0,  1}};
    };

    bool exist(vector<vector<char>> &board, string word) {// 多起点dfs
        if (board.empty() || board[0].empty() || word.empty()) {
            return false;
        }

        visited = vector<vector<bool>>(board.size(), vector<bool>(board[0].size(), false));

        for (int i = 0; i < board.size(); i++) {
            for (int j = 0; j < board[i].size(); j++) {
                if (board[i][j] == word[0]) {
                    visited[i][j]=true;
                    dfs(board, word, 0, make_pair(i, j));
                    visited[i][j]=false;
                    if (wordExists) {
                        return true;
                    }
                }
            }
        }

        return false;
    }

private:
    void dfs(const vector<vector<char>> &board, const string &word, int index, pair<int, int> pos) {
        if (index == word.size() - 1) {
            wordExists = true;
            return;
        }

        for (auto direct: directs) {
            pair<int, int> newPos = calcPos(board, pos, direct);
            if (checkPos(board, word, index + 1, newPos)) {
                visited[newPos.first][newPos.second] = true;
                dfs(board, word, index + 1, newPos);
                visited[newPos.first][newPos.second] = false;
                if (wordExists) {
                    return;
                }
            }
        }
    }

    pair<int, int> calcPos(const vector<vector<char>> &board, pair<int, int> pos, pair<int, int> direct) {
        return make_pair(pos.first + direct.first, pos.second + direct.second);
    }

    bool checkPos(const vector<vector<char>> &board, const string &word, int index, pair<int, int> &pos) {
        if (pos.first < 0 || pos.first >= board.size()) {
            return false;
        }
        if (pos.second < 0 || pos.second >= board[0].size()) {
            return false;
        }
        if (visited[pos.first][pos.second]) {
            return false;
        }

        return board[pos.first][pos.second] == word[index];
    }
};
```

### 4.8 分治

* 寻找两个正序数组的中位数

```
class Solution {
private:
    typedef vector<int>::iterator Iterator;
public:
    double findMedianSortedArrays(vector<int> &nums1, vector<int> &nums2) {
        int total = nums1.size() + nums2.size();
        if (total == 0) {
            return 0.0;
        }

        if (total % 2) {
            return findMedianSortedArrays(nums1.begin(), nums1.end(), nums2.begin(), nums2.end(), total / 2 + 1);
        } else {
            return (findMedianSortedArrays(nums1.begin(), nums1.end(), nums2.begin(), nums2.end(), total / 2) +
                    findMedianSortedArrays(nums1.begin(), nums1.end(), nums2.begin(), nums2.end(), total / 2 + 1)
                   ) / 2.0;
        }
    }

private:
    double findMedianSortedArrays(Iterator begin1, Iterator end1, Iterator begin2, Iterator end2, int target) {
        if (end1 - begin1 > end2 - begin2) return findMedianSortedArrays(begin2, end2, begin1, end1, target);
        if (begin1 >= end1) return *(begin2 + target - 1);
        if (target == 1) return min(*begin1, *begin2);

        int leftNum = min((target) / 2, int(end1 - begin1));
        int rightNum = target - leftNum;

        if (*(begin1 + leftNum - 1) == *(begin2 + rightNum - 1)) {
            return *(begin1 + leftNum - 1);
        } else if (*(begin1 + leftNum - 1) < *(begin2 + rightNum - 1)) {
            return findMedianSortedArrays(begin1 + leftNum, end1, begin2, end2, target - leftNum);
        } else {
            return findMedianSortedArrays(begin1, end1, begin2 + rightNum, end2, target - rightNum);
        }
    }
};
```

* 数组中的第k个最大元素

```
class Solution {
    typedef vector<int>::iterator Iterator;
public:
    int findKthLargest(vector<int> &nums, int k) {
        auto begin = nums.begin(), end = nums.end();
        while (begin < end) {
            auto pivot = partition(begin, end);
            int diff = pivot - nums.begin() + 1 - k;
            if (diff == 0) return *pivot;
            else if (diff > 0) {
                end = pivot;
            } else {
                begin = next(pivot);
            }
        }
        return -1;
    }

private:
    Iterator partition(Iterator begin, Iterator end) {
        auto pivot = begin;
        auto iter = next(begin), index = next(begin);
        for (; iter < end; iter++) {
            if (*iter > *pivot) {
                iter_swap(iter, index);
                index++;
            }
        }
        iter_swap(pivot, prev(index));
        return prev(index);
    }
};
```

* 为运算表达式设计优先级

```
class Solution {
    typedef string::iterator Iter;
public:
    vector<int> diffWaysToCompute(string expression) {
        return diffWaysToCompute(expression, expression.begin(), expression.end());
    }

private:
    vector<int> diffWaysToCompute(const string &exp, Iter begin, Iter end) {
        vector<int> result;

        int num = convertToNumber(begin, end);
        if (num >= 0) {
            result.push_back(num);
            return result;
        }

        for (auto iter = begin; iter != end; iter++) {
            if (isOperation(*iter)) {
                vector<int> lhsRes = diffWaysToCompute(exp, begin, iter);
                vector<int> rhsRes = diffWaysToCompute(exp, next(iter), end);
                for (int j = 0; j < lhsRes.size(); j++) {
                    for (int k = 0; k < rhsRes.size(); k++) {
                        result.push_back(calculate(lhsRes[j], rhsRes[k], *iter));
                    }
                }
            }
        }

        return result;
    }

    int calculate(int lhs, int rhs, char c) {
        switch (c) {
            case '+':
                return lhs + rhs;
            case '-':
                return lhs - rhs;
            case '*':
                return lhs * rhs;
        }
        return -1;
    }

    bool isOperation(char c) {
        return c == '+' || c == '-' || c == '*';
    }

    int convertToNumber(Iter begin, Iter end) {
        int num = 0;
        for (auto iter = begin; iter != end; iter++) {
            if (*iter < '0' || *iter > '9') {
                return -1;
            }
            num = num * 10 + (*iter - '0');
        }
        return num;
    }
};
```


### 4.9 动态规划

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



