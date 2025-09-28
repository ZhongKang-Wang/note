咱现在还没到打周赛的程度，运气好的时候磨蹭一个多小时勉强能做出来第一道简单题，运气不好的时候第一题还没读完咱就感觉已经被秒杀了。所以咱现在就是静下心来学习算法。

这一周呢，我们主要学习的**滑动窗口**。
# 20250828
----

## 滑动窗口

### 定长滑窗套路

其中最经典的一道题：1456. 定长子串中元音的最大数目。

窗口右端点在i时，窗口长度为k，所以有i - k + 1 = L，也即左端点在i - k + 1

三步：

1. 入

下标为i的元素进入窗口，更新相关统计量，如果L < 0，即i < k - 1，则没有行程第1个窗口，重复第一步；

2. 更新

更新答案

3. 出

下标为i - k + 1的元素离开窗口，更新相关统计量



### 不定长滑窗

----

#### 1. 求最长子数组

以 3.无重复字符的最长子串 为例。

首先，看到无重复字符，立刻想到用哈希表去重，哈希表可以是`vec`、`set`、`map`，到底用哪一种，具体情况具体分析。不过这不是重点。

其次，看到求最长子串，其实不一定就能联想到不定长滑窗，只是这道题恰好是不定长滑窗。

最后，具体怎么做呢？

​	1.1. 入

​		右边界每来一个元素，就用`map`记录

​	1.2. 出

​		判断当前元素是否重复，一旦重复，左边界向右移动（去重）

​	1.3. 更新

​		当窗口合法，即可更新相关统计量。

大概是这样。

#### 2. 求最短子数组



- 定义窗口范围：用左右两个指针\[left、right\]表示当前窗口的边界，初始时通常都指向数组起点。
- 扩张右边界：右指针主动向右移动，将元素逐步纳入窗口，直到窗口内的元素满足题目条件（如和 / 乘积达到目标、包含特定元素等。
- 收缩左边界：当窗口满足条件后，尝试右移左指针缩小窗口，同时检查是否仍满足条件。每次收缩都可能得到一个更短的有效窗口，记录此时的最短长度。
- 更新结果：在窗口扩张和收缩的过程中，始终跟踪满足条件的最短子数组长度，最终返回这个最小值。



#### 3. 求子数组个数

- 越短越合法
  - 固定右边界，找合法左边界范围：右指针从左到右遍历，每固定一个右边界 `right`，寻找所有可能的左边界 `left`（`left ≤ right`），使得子数组 `[left, right]` 满足条件。
  - 累计计数：每移动一次右指针，就将当前右边界对应的有效子数组数量累加到结果中。







# 20250917

## 二分查找

34. 在排序数组元素中查找元素的第1个和最后一个位置

```python
class Solution:
    # 左闭右闭区间
    def lower_bound(self, nums: List[int], target: int) -> int:
        left = 0
        right = len(nums) - 1
        while left <= right: # [left, right]
            mid = (left + right) // 2
            if nums[mid] < target:
                left = mid + 1 # [mid + 1, right]
            else:
                right = mid - 1 # [left, mid - 1]
        return left
    # 左闭右开区间
    def lower_bound2(self, nums: List[int], target: int) -> int:
        left = 0
        right = len(nums)
        while left < right: # [left, right)
            mid = (left + right) // 2
            if nums[mid] < target:
                left = mid + 1 # [mid + 1, right)
            else:
                right = mid # [left, mid)
        return left
    # 左开右开区间
    def lower_bound3(self, nums: List[int], target: int) -> int:
        left = -1
        right = len(nums)
        while left + 1 < right: # (left, right)
            mid = (left + right) // 2
            if nums[mid] < target: 
                left = mid # (mid, right)
            else:
                right = mid # (left, mid)
        return left + 1 # right
    def searchRange(self, nums: List[int], target: int) -> List[int]:
        start = self.lower_bound(nums, target)
        if start == len(nums) or nums[start] != target:
            return [-1, -1]
        end = self.lower_bound(nums, target + 1) - 1
        return [start, end]
```

注意：如果题目不是要求>=x，我们可以对其进一步转化

| >= x |            |
| ---- | ---------- |
| > x  | >= x + 1   |
| < x  | (>= x) - 1 |
| <= x | (> x) - 1  |
|      |            |
|      |            |

