---
title: LeetCode 1. 两数之和
date: 2026-09-15T22:53:09+08:00
lastmod: 2026-09-15T22:53:09+08:00
---

# LeetCode 1. 两数之和

> 难度：Easy　标签：数组、哈希表　链接：https://leetcode.cn/problems/two-sum/

## 题目

给定一个整数数组 ​`nums`​ 和一个整数目标值 ​`target`​，请你在该数组中找出**和为目标值** ​**​`target`​** 的那两个整数，并返回它们的数组下标。

- 可以假设每种输入**只会对应一个答案**，且**同一个元素不能用两次**；
- 返回答案的顺序任意。

示例：

- 输入：​`nums = [2,7,11,15], target = 9`​ → 输出：​`[0,1]`（因为 nums[0]+nums[1]==9）
- 输入：​`nums = [3,2,4], target = 6`​ → 输出：​`[1,2]`
- 输入：​`nums = [3,3], target = 6`​ → 输出：​`[0,1]`

数据范围：​`2 <= nums.length <= 10^4`​，​`-10^9 <= nums[i], target <= 10^9`。

## 思路

第一反应肯定是暴力：两层循环挨个试，看哪两个数加起来等于 target。这题数据量才一万，暴力是能过的。但题目末尾那个 Follow up 明晃晃写着"能不能做到比 O(n²) 更快"，那肯定得再想一版。

我一开始想：能不能开个数组当哈希表？一看范围就熄火了——​`nums[i]`​ 能到 ​`-10^9`​，C 语言的数组下标又不能是负数，想开个 20 亿的数组也不现实。所以这里得**自己动手写一个哈希表**（反正我也正好想练练手搓数据结构）。

关键点就一句话：**边遍历边把"已经见过的数"记进哈希表**。对当前这个数 ​`nums[i]`​，我先算一下我"需要"的那个数 ​`need = target - nums[i]`，然后去哈希表里翻一翻它之前出现过没有：

- 翻到了 → 大功告成，直接返回 ​`[那个数的下标, i]`；
- 没翻到 → 把 ​`nums[i]`​ 和它的下标 ​`i` 塞进哈希表，继续往后走。

因为我是"先查再插"，所以天然就不会用到同一个元素两次，连重复元素的坑都顺手填了。

- 关键点：① 用哈希表把"回头找另一半"从 O(n) 降到 O(1)；② 先查后插，保证不重复用同一个元素。
- 复杂度：暴力 O(n²)；哈希表 O(n) 时间、O(n) 空间。

## 代码（C）

### 解法一：暴力枚举（好想，但慢）

```c
// LeetCode 1 两数之和
// 最简单的一版，先求做出来，能过就行
// 思路：两层循环，把所有"两个数的组合"都试一遍

/**
 * Note: The returned array must be malloced, assume caller calls free().
 * 上面这句是力扣模板自带的提醒：返回的数组得自己 malloc，力扣会自动帮你 free。
 */
int* twoSum(int* nums, int numsSize, int target, int* returnSize) {
    int* result = (int*)malloc(2 * sizeof(int)); // 题目保证只有一个答案，开两个 int 就够
    *returnSize = 0;                             // 先当作"没找到"

    for (int i = 0; i < numsSize; i++) {         // 第一个数
        for (int j = i + 1; j < numsSize; j++) { // 第二个数，从 i 后面开始，避免自己和自配、也避免重复
            if (nums[i] + nums[j] == target) {
                result[0] = i;
                result[1] = j;
                *returnSize = 2;                 // 找到了，长度改成 2
                return result;
            }
        }
    }
    return result; // 题目说有解，理论上走不到这里，但函数得有个兜底 return
}
```

### 解法二：手搓哈希表（链地址法，O(n)）

```c
// LeetCode 1 两数之和 —— 手搓哈希表版本
// 不想再用两层循环了，这题就该用哈希表。
// 但 C 语言没有现成的哈希表，只能自己写一个。
// 为什么不能开数组当下标？因为 nums[i] 小到 -1e9，下标不能为负，空间也开不下。
// 所以用最经典的"数组 + 链表"拉链法：数组是桶，冲突的元素挂在桶后面的链上。

#define HASHSIZE 10007 // 桶的个数，取个质数，冲突少一点

typedef struct HashNode {
    int key;                // 存的数值，也就是 nums[i]
    int idx;                // 这个数值在原数组里的下标
    struct HashNode* next;  // 拉链：同一个桶里的下一个节点
} HashNode;

// 哈希函数：把任意 key（可能是负数）映射到 [0, HASHSIZE)
// 注意：C 语言里负数取模结果还是负数，所以要先取绝对值
// （这题 nums[i] 最大到 1e9，取绝对值不会溢出 int；要是真遇到 INT_MIN 就得小心了）
int hashFn(int key) {
    int x = key < 0 ? -key : key;
    return x % HASHSIZE;
}

// 在哈希表里找 key，找到返回它的下标，找不到返回 -1
int hashFind(HashNode** buckets, int key) {
    HashNode* p = buckets[hashFn(key)];
    while (p) {
        if (p->key == key) return p->idx;
        p = p->next;
    }
    return -1;
}

// 头插法把 (key, idx) 塞进哈希表，头插最省事，O(1)
void hashInsert(HashNode** buckets, int key, int idx) {
    int h = hashFn(key);
    HashNode* node = (HashNode*)malloc(sizeof(HashNode));
    node->key = key;
    node->idx = idx;
    node->next = buckets[h]; // 新节点的 next 指向原来桶里的第一个节点
    buckets[h] = node;       // 桶头换成新节点
}

int* twoSum(int* nums, int numsSize, int target, int* returnSize) {
    *returnSize = 2; // 题目保证有解，最后一定是两个下标
    int* result = (int*)malloc(2 * sizeof(int));

    // calloc 会把所有桶初始化为 NULL，用 malloc 的话还得自己 memset，calloc 省事
    HashNode** buckets = (HashNode**)calloc(HASHSIZE, sizeof(HashNode*));

    for (int i = 0; i < numsSize; i++) {
        int need = target - nums[i]; // 我需要的另一半是谁

        int j = hashFind(buckets, need); // 它之前出现过吗？
        if (j != -1) {
            // 出现过！那么 (j, i) 就是答案。
            // 因为我是先查后插，所以 j 一定是"更早"的那个下标，不会撞上 i 自己
            result[0] = j;
            result[1] = i;
            return result;
        }

        // 没找到，就把当前这个数记下来，方便后面的元素来查
        hashInsert(buckets, nums[i], i);
    }
    return result;
}
```

## 本地想自己跑一下

力扣是"核心代码模式"，只交那个函数；但我想自己在本地用样例跑通，就补一个 ​`main`（把上面手搓哈希表的部分整段复制过来即可编译）：

```c
// 本地调试用的 main，力扣提交时不需要这段
#include <stdio.h>
#include <stdlib.h>

// ...（把上面解法二的 hashtable 三个函数 + twoSum 贴过来）...

int main() {
    int nums[] = {2, 7, 11, 15};
    int target = 9;
    int returnSize = 0;
    int* res = twoSum(nums, 4, target, &returnSize);
    printf("返回长度: %d\n", returnSize);
    printf("[%d, %d]\n", res[0], res[1]); // 期望输出 [0, 1]
    free(res);
    return 0;
}
```

## 踩坑与收获

- 头一回认真用 ​`calloc`​：它和 ​`malloc`​ 的区别就是**会把内存清零**。这里哈希桶数组必须初始化为 ​`NULL`​，不然 ​`hashFind`​ 里 ​`while (p)`​ 会去读野指针，直接崩。用 ​`calloc`​ 省掉一次 ​`memset`。
- **负数取模**这个坑：C 语言里 ​`-5 % 3 == -2`（不是 1），所以我先把 key 取了绝对值再取模。以前写代码从没注意过这个，这次被哈希函数逼着想明白了。
- "先查后插"的顺序很关键：如果我先把 ​`nums[i]`​ 插进去再查 ​`target - nums[i]`​，当 ​`target = 2 * nums[i]`​ 时就会查到自己，错用同一个元素。顺序一换，重复元素的问题自动解决（比如 ​`[3,3]` 那个样例）。
- 哦买噶，写完才发现：C 语言里想用哈希表，**要么手搓，要么用第三方库（如 uthash）** ，不像 C++ 有现成的 ​`unordered_map`。这题我算是把拉链法的手感找回来了一点点。
