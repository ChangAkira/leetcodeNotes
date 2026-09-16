---
title: LeetCode 3. 无重复字符的最长子串
date: 2026-09-16T15:55:14+08:00
lastmod: 2026-09-16T15:55:14+08:00
---

# LeetCode 3. 无重复字符的最长子串

> 难度：Medium　标签：哈希表、字符串、滑动窗口　链接：https://leetcode.cn/problems/longest-substring-without-repeating-characters/

## 题目

给一个字符串 ​`s`​，找出里面**不含重复字符**的**最长子串**，返回它的长度。注意是"子串"（连续的一段），不是"子序列"（可以跳着挑），这个区别坑过我好多次。

- 约束：​`0 <= s.length <= 10^5`​，​`s` 由英文字母、数字、符号和空格组成。
- 示例 1：​`s = "abcabcbb"`​ → ​`3`​（​`"abc"`）
- 示例 2：​`s = "bbbbb"`​ → ​`1`​（​`"b"`）
- 示例 3：​`s = "pwwkew"`​ → ​`3`​（​`"wke"`​，注意 ​`"pwke"` 是子序列，不算）

## 思路

第一反应当然是暴力：以每个位置为起点，一直往后加字符，用一个标记数组记住哪些字符出现过了，一撞到重复就停，记下这轮长度，取最大。想是好想，但两重循环是 O(n²)，n 到 10 万的时候心里有点发毛。

然后我盯着"起点往后加"这句话发呆：每次换新起点都要重新数一遍，前面那一大段其实白扫了。比如 ​`"abcabcbb"`​，从下标 0 数到 ​`"abcab"`​，发现第二个 ​`b`​ 重复了；那下一轮从下标 1 开始时，我为什么还要一个个重新标记？直接把左边界挪到重复字符 ​`b`​ 的**上次出现位置 + 1** 不就完了。这不就是**滑动窗口**嘛：右指针负责往右扩张，左指针只在必要时往右跳，被跳过的字符统统踢出窗口。每个字符最多进窗口一次、出窗口一次，整体就是 O(n)。

- 关键点：用 ​`last[c]`​ 记字符 ​`c`​ **上一次出现的下标**；当 ​`last[c] >= left`​（说明它还在当前窗口里）时，令 ​`left = last[c] + 1`，否则左指针不动。
- 复杂度：时间 O(n)，空间 O(1)（字符集固定，最多 128 个 ASCII 字符）。

## 代码（C）

### 解法一：暴力枚举（好想，但慢）

```c
// LeetCode 3 无重复字符的最长子串
// 这题看着像道数学题，其实就是"从每个位置往后数"的老套路
// 思路：双层循环 + 一个标记数组，看看从下标 i 出发最多能延伸多长

#include <string.h>
#include <stdbool.h>

int lengthOfLongestSubstring(char* s) {
    int n = strlen(s);
    int best = 0;

    // 枚举每一个起点 i
    for (int i = 0; i < n; i++) {
        // seen[c]：字符 c 在"这一轮以 i 为起点"的子串里出现过没有
        // 注意它每换一个起点就要清空一次，这就是暴力慢的原因
        bool seen[128] = {false};
        int j = i;

        // 从 i 往后一直加，直到撞见重复字符为止
        for (; j < n; j++) {
            unsigned char c = (unsigned char)s[j]; // 关键！转成 unsigned char 再当下标，防止出现负下标
            if (seen[c]) break;   // 这个字符在本轮已经出现过了，本轮到此结束
            seen[c] = true;       // 标记为已出现
        }

        int len = j - i;          // 本轮最长无重复子串长度
        if (len > best) best = len;
    }
    return best;
}
```

时间复杂度 O(n²)（最坏），空间 O(128)。

### 解法二：滑动窗口 + 手搓数组哈希表（O(n)）

```c
// 解法二：滑动窗口，一次遍历 O(n)
// 关键洞察：右指针右移时，如果新字符 c 之前出现过，而且它上次的位置
// 还在窗口里（last[c] >= left），那左指针直接跳到 "last[c] + 1"，
// 中间那段字符全部扔掉；要是上次位置已经在窗口外了，左指针不用动。
// 千万别无脑写 left = last[c] + 1，那样左指针可能会往回退！

#include <string.h>

int lengthOfLongestSubstring(char* s) {
    int last[128];                          // last[c] = 字符 c 上次出现的下标
    for (int i = 0; i < 128; i++) last[i] = -1; // -1 表示这个字符还没出现过

    int n = strlen(s);
    int left = 0;                           // 窗口左边界
    int best = 0;

    for (int right = 0; right < n; right++) {
        unsigned char c = (unsigned char)s[right];

        if (last[c] >= left) {              // 它还在窗口里，冲突了
            left = last[c] + 1;             // 左边界跳到它的下一位
        }
        last[c] = right;                    // 更新它的最新位置

        int len = right - left + 1;         // 当前窗口长度
        if (len > best) best = len;
    }
    return best;
}
```

时间复杂度 O(n)，空间 O(128)。每个字符只被访问常数次，比暴力快了一个数量级。

## 本地想自己跑一下

```c
// 本地调试用的 main，力扣提交时不需要这段
// 把解答函数和 main 合起来就是一个能直接编译运行的完整程序

#include <stdio.h>
#include <string.h>

int lengthOfLongestSubstring(char* s) {
    int last[128];
    for (int i = 0; i < 128; i++) last[i] = -1;

    int n = strlen(s);
    int left = 0;
    int best = 0;

    for (int right = 0; right < n; right++) {
        unsigned char c = (unsigned char)s[right];
        if (last[c] >= left) {
            left = last[c] + 1;
        }
        last[c] = right;
        int len = right - left + 1;
        if (len > best) best = len;
    }
    return best;
}

int main(void) {
    // 注意：字符串字面量要放到可写的数组里，别写 char* p = "abc";
    char a[] = "abcabcbb";
    char b[] = "bbbbb";
    char c[] = "pwwkew";
    char d[] = "";   // 空串的边界情况

    printf("abcabcbb -> %d\n", lengthOfLongestSubstring(a)); // 期望 3
    printf("bbbbb    -> %d\n", lengthOfLongestSubstring(b)); // 期望 1
    printf("pwwkew   -> %d\n", lengthOfLongestSubstring(c)); // 期望 3
    printf("(空串)   -> %d\n", lengthOfLongestSubstring(d)); // 期望 0

    return 0;
}
```

## 踩坑与收获

- **空串的边界**：​`s.length`​ 允许为 0，一开始根本没管。好在滑动窗口的循环天然不执行、返回 0，刚好对；但要是代码里手贱写了 ​`last[s[0]]`​ 这种就当场崩了，所以还是老实把 ​`last` 全部初始化成 -1 稳妥。
- **​`char`​**​ **到底有没有符号**：​`char`​ 的符号性是"实现定义的"，在 x86 的 gcc/clang 上默认是 **signed char**（-128~127）。拿它当下标，遇到字节值 >127 的字符就会变成负数、直接越界。本题说 ​`s`​ 由英文字母、数字、符号、空格组成，全是 ASCII，其实不转也没事；但 ​`(unsigned char)s[j]` 这个习惯我打算一直留着，万一哪天遇到中文/UTF-8 字节就安全了。以前从没注意过！原来是这样！
- **左指针别乱退**：我第一版差点写成 ​`left = last[c] + 1;`​ 无条件执行，问了下 AI 才反应过来——如果 ​`last[c]`​ 在窗口外（​`last[c] < left`​），这行会把左指针**往回拽**，答案就错了。必须加 ​`if (last[c] >= left)`。这个条件才是整个滑窗的灵魂。
- **子串 vs 子序列**：题目专门拿示例 3 提醒你 ​`"pwke"` 不算——它是子序列不是子串。这种"题意陷阱"以后看到要第一时间划重点。
- 哦买噶，从 O(n²) 挖到 O(n)，还顺手搞懂了一个负下标的隐患，这波不亏，我牛大了。
