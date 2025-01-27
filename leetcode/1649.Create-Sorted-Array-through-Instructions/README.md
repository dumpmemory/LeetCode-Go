# [1649. Create Sorted Array through Instructions](https://leetcode.com/problems/create-sorted-array-through-instructions/)

## 题目

Given an integer array `instructions`, you are asked to create a sorted array from the elements in `instructions`. You start with an empty container `nums`. For each element from **left to right** in `instructions`, insert it into `nums`. The **cost** of each insertion is the **minimum** of the following:

- The number of elements currently in `nums` that are **strictly less than** `instructions[i]`.
- The number of elements currently in `nums` that are **strictly greater than** `instructions[i]`.

For example, if inserting element `3` into `nums = [1,2,3,5]`, the **cost** of insertion is `min(2, 1)` (elements `1` and `2` are less than `3`, element `5` is greater than `3`) and `nums` will become `[1,2,3,3,5]`.

Return *the **total cost** to insert all elements from* `instructions` *into* `nums`. Since the answer may be large, return it **modulo** `10^9 + 7`

**Example 1:**

```
Input: instructions = [1,5,6,2]
Output: 1
Explanation: Begin with nums = [].
Insert 1 with cost min(0, 0) = 0, now nums = [1].
Insert 5 with cost min(1, 0) = 0, now nums = [1,5].
Insert 6 with cost min(2, 0) = 0, now nums = [1,5,6].
Insert 2 with cost min(1, 2) = 1, now nums = [1,2,5,6].
The total cost is 0 + 0 + 0 + 1 = 1.
```

**Example 2:**

```
Input: instructions = [1,2,3,6,5,4]
Output: 3
Explanation: Begin with nums = [].
Insert 1 with cost min(0, 0) = 0, now nums = [1].
Insert 2 with cost min(1, 0) = 0, now nums = [1,2].
Insert 3 with cost min(2, 0) = 0, now nums = [1,2,3].
Insert 6 with cost min(3, 0) = 0, now nums = [1,2,3,6].
Insert 5 with cost min(3, 1) = 1, now nums = [1,2,3,5,6].
Insert 4 with cost min(3, 2) = 2, now nums = [1,2,3,4,5,6].
The total cost is 0 + 0 + 0 + 0 + 1 + 2 = 3.
```

**Example 3:**

```
Input: instructions = [1,3,3,3,2,4,2,1,2]
Output: 4
Explanation: Begin with nums = [].
Insert 1 with cost min(0, 0) = 0, now nums = [1].
Insert 3 with cost min(1, 0) = 0, now nums = [1,3].
Insert 3 with cost min(1, 0) = 0, now nums = [1,3,3].
Insert 3 with cost min(1, 0) = 0, now nums = [1,3,3,3].
Insert 2 with cost min(1, 3) = 1, now nums = [1,2,3,3,3].
Insert 4 with cost min(5, 0) = 0, now nums = [1,2,3,3,3,4].
Insert 2 with cost min(1, 4) = 1, now nums = [1,2,2,3,3,3,4].
Insert 1 with cost min(0, 6) = 0, now nums = [1,1,2,2,3,3,3,4].
Insert 2 with cost min(2, 4) = 2, now nums = [1,1,2,2,2,3,3,3,4].
The total cost is 0 + 0 + 0 + 0 + 1 + 0 + 1 + 0 + 2 = 4.
```

**Constraints:**

- `1 <= instructions.length <= 105`
- `1 <= instructions[i] <= 105`

## 题目大意

给你一个整数数组 instructions ，你需要根据 instructions 中的元素创建一个有序数组。一开始你有一个空的数组 nums ，你需要 从左到右 遍历 instructions 中的元素，将它们依次插入 nums 数组中。每一次插入操作的 代价 是以下两者的 较小值 ：

- nums 中 严格小于 instructions[i] 的数字数目。
- nums 中 严格大于 instructions[i] 的数字数目。

比方说，如果要将 3 插入到 nums = [1,2,3,5] ，那么插入操作的 代价 为 min(2, 1) (元素 1 和 2 小于 3 ，元素 5 大于 3 ），插入后 nums 变成 [1,2,3,3,5] 。请你返回将 instructions 中所有元素依次插入 nums 后的 总最小代价 。由于答案会很大，请将它对 10^9 + 7 取余 后返回。

## 解题思路

- 给出一个数组，要求将其中的元素从头开始往另外一个空数组中插入，每次插入前，累加代价值 cost = min(**strictly less than**, **strictly greater than**)。最后输出累加值。
- 这一题虽然是 Hard 题，但是读完题以后就可以判定这是模板题了。可以用线段树和树状数组来解决。这里简单说说线段树的思路吧，先将待插入的数组排序，获得总的区间。每次循环做 4 步：2 次 `query` 分别得到 `strictlyLessThan` 和 `strictlyGreaterThan` ，再比较出两者中的最小值累加，最后一步就是 `update`。
- 由于题目给的数据比较大，所以建立线段树之前记得要先离散化。这一题核心代码不超过 10 行，其他的都是模板代码。具体实现见代码。

## 代码

```go
package leetcode

import (
	"sort"

	"github.com/halfrost/leetcode-go/template"
)

// 解法一 树状数组 Binary Indexed Tree
func createSortedArray(instructions []int) int {
	bit, res := template.BinaryIndexedTree{}, 0
	bit.Init(100001)
	for i, v := range instructions {
		less := bit.Query(v - 1)
		greater := i - bit.Query(v)
		res = (res + min(less, greater)) % (1e9 + 7)
		bit.Add(v, 1)
	}
	return res
}

// 解法二 线段树 SegmentTree
func createSortedArray1(instructions []int) int {
	if len(instructions) == 0 {
		return 0
	}
	st, res, mod := template.SegmentCountTree{}, 0, 1000000007
	numsMap, numsArray, tmpArray := discretization1649(instructions)
	// 初始化线段树，节点内的值都赋值为 0，即计数为 0
	st.Init(tmpArray, func(i, j int) int {
		return 0
	})
	for i := 0; i < len(instructions); i++ {
		strictlyLessThan := st.Query(0, numsMap[instructions[i]]-1)
		strictlyGreaterThan := st.Query(numsMap[instructions[i]]+1, numsArray[len(numsArray)-1])
		res = (res + min(strictlyLessThan, strictlyGreaterThan)) % mod
		st.UpdateCount(numsMap[instructions[i]])
	}
	return res
}

func discretization1649(instructions []int) (map[int]int, []int, []int) {
	tmpArray, numsArray, numsMap := []int{}, []int{}, map[int]int{}
	for i := 0; i < len(instructions); i++ {
		numsMap[instructions[i]] = instructions[i]
	}
	for _, v := range numsMap {
		numsArray = append(numsArray, v)
	}
	sort.Ints(numsArray)
	for i, num := range numsArray {
		numsMap[num] = i
	}
	for i := range numsArray {
		tmpArray = append(tmpArray, i)
	}
	return numsMap, numsArray, tmpArray
}

func min(a int, b int) int {
	if a > b {
		return b
	}
	return a
}

```