# 回溯

1. 数据集里的元素有没有重复的，对应的措施就是排序；
2. 数据集里的元素允不允许重复使用，对应的措施就是递归的时候index加不加1
3. for循环的选择，如果i从index开始，一般不需要辅助标记；如果i从0开始（意味着全排列），可能需要辅助标记。

## 求所有子集

```python
```



## 无重复元素的列表中选择一些数，其和为target

```python
from typing import List

class Solution:
    def combination_sum(self, nums: List[int], target: int) -> List[List[int]]:
        result = []
        self.back_track(0, nums, target, result)
        return result

    def back_track(self, idx: int, nums: List[int], target: int, result: List[int]):
        if target == 0:
            return True
        if idx >= len(nums):
            return False

        for index in range(idx, len(nums)):
            result.append(nums[index])
            if self.back_track(index + 1, nums, target - nums[index], result):
                return True
            result.pop(-1)
        return False
```



```py
from typing import List

class Solution:
    def combination_sum(self, nums: List[int], target: int) -> List[List[int]]:
        result = []
        self.back_track(nums, target, result)
        return result

    def back_track(self, nums: List[int], target: int, result: List[int]):
        if target == 0:
            return True
        if len(nums) <= 0:
            return False
        result.append(nums[0])
        if self.back_track(nums[1:], target - nums[0], result):
            return True
        result.pop(-1)
        if self.back_track(nums[1:], target, result):
            return True
        return False
```



## 有重复元素的列表中选择一些数，其和为target

```python
from typing import List

class Solution:
    def combination_sum(self, nums: List[int], target: int) -> List[List[int]]:
        result = []
        nums_temp = sorted(nums)
        self.back_track(0, nums_temp, target, result)
        return result

    def back_track(self, idx: int, nums: List[int], target: int, result: List[int]):
        if target == 0:
            return True
        if idx >= len(nums):
            return False

        for index in range(idx, len(nums)):
            if idx > 0 and nums[index] == nums[index - 1]:
                continue	# 剪枝，避免重复元素
            result.append(nums[index])
            if self.back_track(index + 1, nums, target - nums[index], result):
                return True
            result.pop(-1)
        return False
```



## 无重复元素的列表全排列

```python
from typing import List

class Solution:
    visit = []

    def permute(self, nums: List[int]) -> List[List[int]]:
        result = []
        self.visit = [False] * len(nums)
        self.back_track(0, nums, result, [])
        return result

    def back_track(self, idx: int, nums: List[int], result: List[List[int]], item_result: List[int]):
        if idx >= len(nums):
            result.append(List.copy(item_result))
            return

        for index, item in enumerate(nums):
            if self.visit[index]:
                continue
            item_result.append(item)
            self.visit[index] = True
            self.back_track(idx + 1, nums, result, item_result)
            item_result.pop(-1)
            self.visit[index] = False
```



## 有重复元素的列表全排列

```python
from typing import List

class Solution:
    visit = []

    def permuteUnique(self, nums: List[int]) -> List[List[int]]:
        nums_temp = sorted(nums)
        result = []
        self.visit = [False] * len(nums)
        self.back_track(0, nums_temp, result, [])
        return result

    def back_track(self, idx: int, nums: List[int], result: List[List[int]], item_result: List[int]):
        if idx >= len(nums):
            result.append(List.copy(item_result))
            return

        for index, item in enumerate(nums):
            if self.visit[index] or (index > 0 and nums[index] == nums[index - 1] and not self.visit[index - 1]):
                continue	# 剪枝，避免重复元素
            item_result.append(item)
            self.visit[index] = True
            self.back_track(idx + 1, nums, result, item_result)
            item_result.pop(-1)
            self.visit[index] = False
```



