---
title: LeetCode 2. 两数相加
date: 2026-09-15T23:01:18+08:00
lastmod: 2026-09-15T23:01:18+08:00
---

# LeetCode 2. 两数相加

> 难度：Medium　标签：递归、链表、数学　链接：https://leetcode.cn/problems/add-two-numbers/

## 题目

给你两个**非空**的链表，表示两个非负整数：它们是**逆序**存的，也就是表头是个位，往后依次是十位、百位……每个结点只存一位数字。

请你把这两个数加起来，用同样「逆序、一位一结点」的形式返回一个新的链表。

- 每个链表结点数在 ​`[1, 100]` 内
- `0 <= Node.val <= 9`
- 数据保证没有前导零（除了数字 0 本身）

示例 1：​`l1 = [2,4,3]`​，​`l2 = [5,6,4]`​ → ​`[7,0,8]`​（即 342 + 465 = 807）  
示例 2：​`l1 = [0]`​，​`l2 = [0]`​ → ​`[0]`​  
示例 3：​`l1 = [9,9,9,9,9,9,9]`​，​`l2 = [9,9,9,9]`​ → ​`[8,9,9,9,0,0,0,1]`（这个 9999999 + 9999 最后还多进了一位，别漏）

## 思路

第一反应：这还不简单？把链表读成一个整数，加起来，再拆回链表不就完了。

写完才发现，题目说每个链表最多 **100 个结点**，也就是那个数能有 100 位！​`long long`​ 也就 19 位，直接原地爆炸。所以这题根本不是在考"怎么把数读出来"，而是在考**手写竖式加法**。

好在它已经是逆序存的，跟竖式从个位开始算的方向**完全一致**，所以可以一边遍历一边加，根本不用反转链表。

- 关键点 1：进位 ​`carry`​。每一位 ​`sum = a + b + carry`​，本位留 ​`sum % 10`​，进位更新成 ​`sum / 10`。
- 关键点 2：两个链表不一定一样长，短的那个"缺位"就当 0。
- 关键点 3：循环条件一定要带 ​`|| carry != 0`，否则最后那一位进位会被丢掉（示例 3 就是靠这个才出结果的）。
- 关键点 4：用**哨兵头结点**（dummy head），省掉"第一个结点要特殊处理"那堆 ​`if`​，最后 ​`return dummy->next` 就行。
- 复杂度：时间 O(max(m, n))，空间 O(max(m, n))（结果链表本身）。

## 代码（C）

### 解法一：读成整数再加（好想，但会溢出）

```c
// LeetCode 2 两数相加
// 这个思路最顺，可惜只能过小数据：100 位数字 long long 装不下
// 部分正确，问过 AI：题目最多 100 个结点，得用字符串/竖式思想
struct ListNode* addTwoNumbers(struct ListNode* l1, struct ListNode* l2) {
    long long a = 0, b = 0, unit = 1;
    for (struct ListNode *p = l1; p; p = p->next) {   // 逆序存，正好从低位开始乘 1、10、100
        a += (long long)p->val * unit;
        unit *= 10;
    }
    unit = 1;
    for (struct ListNode *p = l2; p; p = p->next) {
        b += (long long)p->val * unit;
        unit *= 10;
    }
    long long sum = a + b;
    // 把 sum 拆回逆序链表；注意 sum == 0 时要单独处理，不然 head 是空的
    struct ListNode dummy, *tail = &dummy;
    dummy.next = NULL;
    if (sum == 0) {
        tail->next = malloc(sizeof(struct ListNode));
        tail->next->val = 0;
        tail->next->next = NULL;
        return dummy.next;
    }
    while (sum > 0) {
        struct ListNode *node = malloc(sizeof(struct ListNode));
        node->val = sum % 10;
        node->next = NULL;
        tail->next = node;
        tail = node;
        sum /= 10;
    }
    return dummy.next;
}
// 结论：思路对，但 100 位直接溢出，力扣上一堆测试点过不去。老老实实写竖式吧。
```

### 解法二：模拟竖式，边遍历边加（哨兵 + 尾插，O(max(m,n))）

```c
// 哨兵头结点真是个好东西：不用再纠结"结果的第一个结点谁来做"，
// 全程只管往 tail 屁股后面挂新结点就行了
struct ListNode* addTwoNumbers(struct ListNode* l1, struct ListNode* l2) {
    struct ListNode dummy;              // 栈上的哨兵，不会跟着返回，只借用它的 next 指针
    struct ListNode *tail = &dummy;     // tail 永远指向结果链表的最后一个结点
    dummy.next = NULL;
    int carry = 0;                      // 进位，一开始当然是 0

    // 注意第三个条件：两个链表都空了但还有进位，也得再补一个结点（示例 3 的那种情况）
    while (l1 != NULL || l2 != NULL || carry != 0) {
        int a = (l1 != NULL) ? l1->val : 0;   // 短的链表缺位就当 0，不用写两套分支
        int b = (l2 != NULL) ? l2->val : 0;
        int sum = a + b + carry;

        carry = sum / 10;                     // 该进多少位
        struct ListNode *node = malloc(sizeof(struct ListNode));
        node->val = sum % 10;                 // 本位留在链表上的数字
        node->next = NULL;
        tail->next = node;                    // 尾插
        tail = node;

        if (l1 != NULL) l1 = l1->next;        // 判空再走，不然空指针直接寄
        if (l2 != NULL) l2 = l2->next;
    }
    return dummy.next;   // 千万别 return &dummy，那是个栈变量，函数一退出就废了
}
```

### 解法三：递归版（链表题的通解套路）

```c
// 链表题的递归写法永远是最短的，思路就是：
// "当前位算好，剩下的交给下一层"，进位当成参数往下传
static struct ListNode *helper(struct ListNode *l1, struct ListNode *l2, int carry) {
    // 两个都空、且没有进位了，才真正结束
    if (l1 == NULL && l2 == NULL && carry == 0) return NULL;

    int a = (l1 != NULL) ? l1->val : 0;
    int b = (l2 != NULL) ? l2->val : 0;
    int sum = a + b + carry;

    struct ListNode *node = malloc(sizeof(struct ListNode));
    node->val = sum % 10;
    // 下一层接着算：谁空了就传 NULL，进位接着传
    node->next = helper(l1 ? l1->next : NULL,
                        l2 ? l2->next : NULL,
                        sum / 10);
    return node;
}

struct ListNode* addTwoNumbers(struct ListNode* l1, struct ListNode* l2) {
    return helper(l1, l2, 0);
}
// 递归和迭代复杂度一样，但递归有栈深度（最多 100 层，没事）；
// 迭代更省心，比赛时我还是更爱写迭代那一版。
```

## 本地想自己跑一下

```c
// 本地调试用的 main，力扣提交时不需要这段
#include <stdio.h>
#include <stdlib.h>

struct ListNode {
    int val;
    struct ListNode *next;
};

// ===== 解答函数（和解法二一样）=====
struct ListNode* addTwoNumbers(struct ListNode* l1, struct ListNode* l2) {
    struct ListNode dummy;
    struct ListNode *tail = &dummy;
    dummy.next = NULL;
    int carry = 0;

    while (l1 != NULL || l2 != NULL || carry != 0) {
        int a = (l1 != NULL) ? l1->val : 0;
        int b = (l2 != NULL) ? l2->val : 0;
        int sum = a + b + carry;

        carry = sum / 10;
        struct ListNode *node = malloc(sizeof(struct ListNode));
        node->val = sum % 10;
        node->next = NULL;
        tail->next = node;
        tail = node;

        if (l1 != NULL) l1 = l1->next;
        if (l2 != NULL) l2 = l2->next;
    }
    return dummy.next;
}

// ===== 用数组建链表的辅助函数（逆序，跟题目输入一致）=====
struct ListNode *build(int *arr, int n) {
    struct ListNode dummy, *tail = &dummy;
    dummy.next = NULL;
    for (int i = 0; i < n; i++) {
        struct ListNode *node = malloc(sizeof(struct ListNode));
        node->val = arr[i];
        node->next = NULL;
        tail->next = node;
        tail = node;
    }
    return dummy.next;
}

void print(struct ListNode *head) {
    printf("[");
    for (struct ListNode *p = head; p; p = p->next) {
        printf("%d%s", p->val, p->next ? "," : "");
    }
    printf("]\n");
}

int main(void) {
    int a[] = {2, 4, 3}, b[] = {5, 6, 4};
    print(addTwoNumbers(build(a, 3), build(b, 3)));    // 期望 [7,0,8]

    int c[] = {0}, d[] = {0};
    print(addTwoNumbers(build(c, 1), build(d, 1)));    // 期望 [0]

    int e[] = {9, 9, 9, 9, 9, 9, 9}, f[] = {9, 9, 9, 9};
    print(addTwoNumbers(build(e, 7), build(f, 4)));    // 期望 [8,9,9,9,0,0,0,1]
    return 0;
}
```

## 踩坑与收获

- **第一个坑就是"想太美"** ：一上来就想把链表读成整数，结果题目 100 个结点，直接超出 ​`long long`。以前做 PAT 那种题目输入都是普通整数，没在意过"位数"这件事，这次算是被教做人。
- **哨兵头结点（dummy head）** ：以前建链表老是要写 ​`if (head == NULL) head = node; else tail->next = node;`​，又臭又长。现在挂个哨兵，最后 ​`return dummy.next`，代码短一大截。原来是这样！
- **​`return &dummy`​**​ **是个要命的错误**：dummy 建在栈上，函数一返回就失效了。虽然运行时可能"看着没错"，但这是典型的悬空指针。问了下 AI，说要么返回 ​`dummy.next`​，要么 dummy 就用 ​`malloc` 出来。
- **循环条件的第三个判断** ​ **​`|| carry != 0`​**​：这个真的容易漏。示例 3 最后多出来一个 ​`1` 就是它兜住的。以前从没注意过这种"最后一位还要进位"的边界。
- **递归写法**：AI 还告诉我链表题基本都能写成"处理当前结点 + 递归剩余部分"的形式，进位这种"要往下传的状态"就挂在参数上，想通了之后发现确实好简单！
- 三种解法对比下来：解法一思路最直觉但会溢出；解法二最稳、比赛首选；解法三最短最优雅。比起刚学链表时在那儿手忙脚乱地 reverse 来 reverse 去，现在竟然这么顺畅，我牛大了。
