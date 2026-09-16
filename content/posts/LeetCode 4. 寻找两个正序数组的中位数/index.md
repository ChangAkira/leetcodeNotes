---
title: LeetCode 4. 寻找两个正序数组的中位数
date: 2026-09-16T17:39:06+08:00
lastmod: 2026-09-16T17:39:06+08:00
---

# LeetCode 4. 寻找两个正序数组的中位数

> 难度：Hard　标签：数组、二分查找、分治　链接：https://leetcode.cn/problems/median-of-two-sorted-arrays/

## 题目

给你两个已经**从小到大排好序**的数组 ​`nums1`​（长 ​`m`​）和 ​`nums2`​（长 ​`n`），求这两个数组合并以后的中位数。

- 奇数个取正中间那个，偶数个取中间两个求平均（所以返回值是 ​`double`）
- `0 <= m, n <= 1000`​，​`1 <= m + n <= 2000`​ —— 也就是说**允许其中一个是空数组**
- 元素范围 ​`-10^6 <= nums1[i], nums2[i] <= 10^6`
- 题面还额外加了一句：时间复杂度必须是 ​`O(log (m+n))`，这句才是本题被评为 Hard 的原因

示例 1：​`nums1 = [1,3]`​，​`nums2 = [2]`​ → ​`2.00000`​（合并 ​`[1,2,3]`​，中间是 2）  
示例 2：​`nums1 = [1,2]`​，​`nums2 = [3,4]`​ → ​`2.50000`​（合并 ​`[1,2,3,4]`​，​`(2+3)/2 = 2.5`）

## 思路

第一反应：这题有什么难的？两个有序数组，归并排序我最熟了，直接合并成一个大数组，再按下标取中间不就完了？——这就是**解法一**，五分钟写完，样例全过，一提交也 AC 了。

可是抬头一看题目要求 ​`O(log(m+n))`​……我这个是 ​`O(m+n)`​，还多开了 ​`O(m+n)`​ 的空间，属于「能过但没做题」。这题之所以挂 Hard，考的根本不是「怎么求中位数」，而是「**不合并**怎么把中位数找出来」。

第二反应：不开数组了，搞两个指针在各自的数组上滑，谁小谁往前走一步，数到第 ​`(m+n)/2`​ 个就停。——**解法二**，空间压到了 ​`O(1)`​，可时间还是 ​`O(m+n)`，复杂度仍然没达标。

真正的正解（**解法三**）是「中位数」换个说法：如果我能把两个数组切一刀，让**左半部分一共正好** ​**​`half = (m+n+1)/2`​**​ **个数**，并且**左边的最大值 ≤ 右边的最小值**，那这一刀就是正确的划分——奇数个时左边最大那个就是中位数，偶数个时 ​`(左边最大 + 右边最小) / 2` 就是了。

那这一刀怎么找？设从 ​`nums1`​ 里切 ​`i`​ 个放左边，​`nums2`​ 里就得切 ​`j = half - i`​ 个。​`i`​ 越大 ​`j`​ 就越小，而两个数组本身是有序的，所以「找合适的 ​`i`​」这件事可以**直接二分**！而且只要在**短的那个**数组上二分，复杂度就是题目要的 ​`O(log(min(m,n)))`。

- 关键点 1：进函数先把 ​`nums1`​、​`nums2`​ 换个位置，保证 ​`nums1`​ 是短的。这既是复杂度的保证，也是 ​`j = half - i` 不会越界的前提。
- 关键点 2：四个边界值（​`i==0`​、​`i==m`​、​`j==0`​、​`j==n`​）全用 ​`±∞`​ 兜住，空数组和切到头这些极端情况就不用写一堆 ​`if` 特判了。
- 关键点 3：判断标准就两条：​`nums1[i-1] <= nums2[j]`​ 且 ​`nums2[j-1] <= nums1[i]`​；不满足就看是哪边太大，​`i` 往左收还是往右扩。
- 复杂度：解法一 ​`O(m+n)`​ 时间 + ​`O(m+n)`​ 空间；解法二 ​`O(m+n)`​ 时间 + ​`O(1)`​ 空间；解法三 ​`O(log(min(m,n)))`​ 时间 + ​`O(1)` 空间。

## 代码（C）

### 解法一：归并成一个大数组再取中间（好想，但不符合要求）

```c
// LeetCode 4 寻找两个正序数组的中位数
// 一开始就是这么写的：两个有序数组，归并一下不就完事了，爽！
// 缺点：O(m+n) 时间、O(m+n) 空间。题目要 O(log(m+n))，这就被打脸了。
// 部分正确（能 AC 但复杂度不达标），问过 AI：我干的是「合并」，不是「划分」。
#include <stdlib.h>

double findMedianSortedArrays(int* nums1, int nums1Size, int* nums2, int nums2Size) {
    int total = nums1Size + nums2Size;
    // 开一个装得下所有人的桶
    int *merged = (int *)malloc(total * sizeof(int));
    int i = 0, j = 0, k = 0;

    // 归并的老套路：谁小谁先出队
    while (i < nums1Size && j < nums2Size) {
        if (nums1[i] <= nums2[j]) merged[k++] = nums1[i++];
        else                      merged[k++] = nums2[j++];
    }
    // 剩下的直接搬过来
    while (i < nums1Size) merged[k++] = nums1[i++];
    while (j < nums2Size) merged[k++] = nums2[j++];

    double ans;
    if (total % 2 == 1) {
        ans = merged[total / 2];                 // 奇数：正中间那个
    } else {
        // 偶数：中间两个求平均。一定要写 2.0！写成 /2 就是整数除法，
        // 2.5 会被截成 2.0，这个坑以前在 PAT 里踩过
        ans = (merged[total / 2 - 1] + merged[total / 2]) / 2.0;
    }
    free(merged);                                // 自己 malloc 的，自己还回去
    return ans;
}
```

### 解法二：双指针数到中间（不开数组，O(m+n) 时间 / O(1) 空间）

```c
// 不想开数组了，那就两个指针各自滑：谁小谁前进一步，
// 数到第 midL、midR 个就记下来——中位数只跟这一两个位置有关
double findMedianSortedArrays(int* nums1, int nums1Size, int* nums2, int nums2Size) {
    int total = nums1Size + nums2Size;
    // 这样取下标有个好处：奇数时 midL == midR，就不用分奇偶写两套了
    int midL = (total - 1) / 2;
    int midR = total / 2;
    int i = 0, j = 0, cnt = 0;
    int left = 0, right = 0;

    while (cnt <= midR) {
        int cur;
        // 越界判断必须写在前面！短路求值是从左往右算的，
        // 要是把 nums1[i] <= nums2[j] 提到前面，某个数组走空后就先越界读了
        if (i < nums1Size && (j >= nums2Size || nums1[i] <= nums2[j])) cur = nums1[i++];
        else                                                           cur = nums2[j++];

        if (cnt == midL) left  = cur;
        if (cnt == midR) right = cur;
        cnt++;
    }
    return (left + right) / 2.0;   // 又是 2.0，别再写成 2 了
}
```

### 解法三：二分划分（O(log(min(m,n)))，正解）

```c
// 正解：不合并，而是「切一刀」。
// 目标：左边一共 half = (m+n+1)/2 个数，且 左边最大 <= 右边最小。
// 从 nums1 取 i 个到左边，nums2 就得取 j = half - i 个，于是「找 i」可以二分。
// 先让 nums1 是短的那个数组：i 的搜索范围小，而且 j 绝不会越界。
#define INF  (1 << 30)      // 当 +∞ 用的哨兵，比元素上限 1e6 大得多就够
#define NEG  (-(1 << 30))   // 故意不用 INT_MIN/INT_MAX：不用管 limits.h，也躲开字面量的坑

double findMedianSortedArrays(int* nums1, int nums1Size, int* nums2, int nums2Size) {
    // 保证 nums1 更短：二分用在短数组上，复杂度才是 log(min(m,n))。
    // C 里没有现成的 swap，但交换「地址 + 长度」最省事的办法就是递归调用自己
    if (nums1Size > nums2Size)
        return findMedianSortedArrays(nums2, nums2Size, nums1, nums1Size);

    int m = nums1Size, n = nums2Size;
    int total = m + n;
    int half = (total + 1) / 2;   // 左半边要装的个数：+1 是为了奇数时左边多一个，直接取左边最大就完事

    int lo = 0, hi = m;           // i 的范围：从 nums1 里取几个放左边
    while (lo <= hi) {
        int i = (lo + hi) / 2;    // nums1 贡献 i 个
        int j = half - i;         // nums2 就得贡献 j 个（因为 m <= n，0 <= j <= n 恒成立）

        // 四个边界值：切到头了就用 ±∞ 顶上，省掉特判空数组/切在边上的 if
        int inLeft  = (i > 0) ? nums1[i - 1] : NEG;   // nums1 左半的最大
        int inRight = (i < m) ? nums1[i]     : INF;   // nums1 右半的最小
        int jnLeft  = (j > 0) ? nums2[j - 1] : NEG;   // nums2 左半的最大
        int jnRight = (j < n) ? nums2[j]     : INF;   // nums2 右半的最小

        if (inLeft <= jnRight && jnLeft <= inRight) {
            // 这一刀切对了！
            if (total % 2 == 1) {                     // 奇数：左边最大的那个就是中位数
                return inLeft > jnLeft ? inLeft : jnLeft;
            }
            int leftMax  = inLeft > jnLeft ? inLeft : jnLeft;
            int rightMin = inRight < jnRight ? inRight : jnRight;
            return (leftMax + rightMin) / 2.0;        // 偶数：中间两个求平均
        } else if (inLeft > jnRight) {
            hi = i - 1;   // nums1 左边给的太大了，i 往左收
        } else {
            lo = i + 1;   // nums2 左边给的太大了（等价于 nums1 给少了），i 往右扩
        }
    }
    return 0.0;   // 题目保证输入合法，理论上走不到这
}
```

## 本地想自己跑一下

```c
// 本地调试用的 main，力扣提交时不需要这段
// 下面贴的是解法三；想试解法二就把函数替换掉、名字保持一样即可
#include <stdio.h>

#define INF  (1 << 30)
#define NEG  (-(1 << 30))

/* ==================== 解答函数（和解法三一样） ==================== */
double findMedianSortedArrays(int* nums1, int nums1Size, int* nums2, int nums2Size) {
    if (nums1Size > nums2Size)
        return findMedianSortedArrays(nums2, nums2Size, nums1, nums1Size);

    int m = nums1Size, n = nums2Size;
    int total = m + n;
    int half = (total + 1) / 2;

    int lo = 0, hi = m;
    while (lo <= hi) {
        int i = (lo + hi) / 2;
        int j = half - i;

        int inLeft  = (i > 0) ? nums1[i - 1] : NEG;
        int inRight = (i < m) ? nums1[i]     : INF;
        int jnLeft  = (j > 0) ? nums2[j - 1] : NEG;
        int jnRight = (j < n) ? nums2[j]     : INF;

        if (inLeft <= jnRight && jnLeft <= inRight) {
            if (total % 2 == 1)
                return inLeft > jnLeft ? inLeft : jnLeft;
            int leftMax  = inLeft > jnLeft ? inLeft : jnLeft;
            int rightMin = inRight < jnRight ? inRight : jnRight;
            return (leftMax + rightMin) / 2.0;
        } else if (inLeft > jnRight) {
            hi = i - 1;
        } else {
            lo = i + 1;
        }
    }
    return 0.0;
}

/* ==================== 测试 ==================== */
int main(void) {
    int a1[] = {1, 3},          b1[] = {2};
    printf("%.5f  期望 2.00000\n", findMedianSortedArrays(a1, 2, b1, 1));

    int a2[] = {1, 2},          b2[] = {3, 4};
    printf("%.5f  期望 2.50000\n", findMedianSortedArrays(a2, 2, b2, 2));

    int a3[] = {0, 0},          b3[] = {0, 0};
    printf("%.5f  期望 0.00000\n", findMedianSortedArrays(a3, 2, b3, 2));

    int a4[] = {2};             /* 另一边是空数组：测 m / n 为 0 的边界 */
    printf("%.5f  期望 2.00000\n", findMedianSortedArrays(a4, 1, NULL, 0));

    int b5[] = {1};
    printf("%.5f  期望 1.00000\n", findMedianSortedArrays(NULL, 0, b5, 1));

    int a6[] = {1, 3, 5, 7},    b6[] = {2, 4, 6, 8};
    printf("%.5f  期望 4.50000\n", findMedianSortedArrays(a6, 4, b6, 4));

    int a7[] = {1, 2, 3, 4, 5}, b7[] = {6, 7, 8, 9};
    printf("%.5f  期望 5.00000\n", findMedianSortedArrays(a7, 5, b7, 4));

    int a8[] = {1},             b8[] = {2, 3, 4, 5, 6};
    printf("%.5f  期望 3.50000\n", findMedianSortedArrays(a8, 1, b8, 5));

    return 0;
}
```

## 踩坑与收获

- **这题最大的收获：「中位数」可以换个说法**。以前我理解的求中位数就是「排好序取中间」，AI 告诉我还能理解成「找到一个划分，让左边个数固定、左边最大 ≤ 右边最小」——把「取值」问题变成「找位置」问题，这才有了二分的份。原来是这样！
-  **​`/ 2.0`​**​ **和** ​ **​`/ 2`​**​ **的区别**：这题返回 ​`double`​，偶数个时中间两个求平均必须写 ​`2.0`，不然整数除法把 2.5 直接截成 2.0。这个坑我以前在 PAT 里栽过，这次一眼就看出来了，有进步。
- **短路求值的顺序**：解法二里 ​`i < nums1Size && (j >= nums2Size || nums1[i] <= nums2[j])`​，越界判断一定写在读取 ​`nums1[i]` 前面。以前写双指针老把条件写反，然后喜提 RE。
-  **±∞ 哨兵**：四个边界全用 ​`±(1<<30)`​ 顶掉，代码短了一截，也不会漏掉「某个数组为空」「i 切到 0 或 m」这些情况。一开始我用的是 ​`INT_MIN`​，得 include ​`limits.h`​，还老担心 ​`-2147483648` 这种字面量的坑；现在干脆拿一个偏大的数当无穷，反正元素最大只有 1e6。
- **​`j = half - i`​**​ **会不会越界？** ——这里就体现出「先保证 nums1 是短数组」的重要性了：因为 ​`m <= n`​，所以 ​`half = (m+n+1)/2`​ 一定满足 ​`half - m >= 0`​ 且 ​`half <= n`​，于是 ​`0 <= j <= n`​ 恒成立。要是没交换、让长数组去二分，​`j`​ 就可能变成负数或者超过 ​`n`，越界读马上就来了。这个推理是我自己顺出来的，写出来特别爽。
- **递归把自己当成 swap 用**：C 里没有现成的 ​`swap`，交换「地址 + 长度」四个东西最省事的就是递归调用自己一次。有点 hack，但比手写中间变量干净。
- 三种解法对比下来：解法一最直白但多开数组、复杂度不达标；解法二空间压到 ​`O(1)`​ 但时间还是 ​`O(m+n)`；解法三是正解，思路一通代码不到 20 行。从「合并」到「划分」，这题是真学到东西了，我牛大了。
