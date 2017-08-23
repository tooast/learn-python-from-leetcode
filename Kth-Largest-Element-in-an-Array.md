## [Kth Largest Element in an Array](https://leetcode.com/problems/kth-largest-element-in-an-array/description/)

> Find the **k**th largest element in an unsorted array. Note that it is the kth largest element in the sorted order, not the kth distinct element.
>
> For example,Given `[3,2,1,5,6,4]` and k = 2, return 5.

无序数组寻求第 K 大的数，很容易就想到先将数组排序，然后直接返回第 K 大的数。时间复杂度为 $O(n\log{}n)$ 。不过这种方法显然不是最优的。

### 1. 改造选择排序 $O(n * k)$ 

通过改造选择排序，外层循环 K 次，时间复杂度为 $O(nk)$ 

```python
class Solution(object):
    def findKthLargest(self, nums, k):
        """
        :type nums: List[int]
        :type k: int
        :rtype: int
        """
        last_index = len(nums) - 1
        for i in range(last_index, last_index - k, -1): #从头扫到last_index - i，最大值放尾部
            max_index = i
            for j in range(i):
                if nums[j] > nums[max_index]:
                    max_index = j
            nums[i], nums[max_index] = nums[max_index], nums[i]
        return nums[-k]
```

### 2.改造快速排序  $O(n)$ 

利用快排中的 Partition 函数进行处理，如果 Partition 函数返回的 index 刚好为 ```len(nums)-k```，即此位置右边有 K - 1 个比它大的数，左边全是比他小的数，这个数字就是我们想要的第 K 大的数字。时间复杂度为 $O(nk)$ 

```python
class Solution(object):
    def findKthLargest(self, nums, k):
        import random
        
        def partition(nums, l, r): 
            pivot = random.randrange(l, r+1)
            nums[pivot], nums[l] = nums[l], nums[pivot]
            l_edge = l 
            for i in range(l+1, r+1):
                if nums[i] <nums[l]:
                    l_edge += 1
                    if l_edge != l:
                        nums[i], nums[l_edge] = nums[l_edge], nums[i]
            nums[l_edge], nums[l] = nums[l], nums[l_edge]
            return l_edge
          
        index = partition(nums, 0, len(nums) - 1)
        start = 0
        end = len(nums) - 1
        while index != len(nums) - k:
            if index > len(nums) - k:
                end = index - 1
                index  = partition(nums, start, end)
            else:
                start = index + 1
                index = partition(nums, start, end)
        return nums[index]
```

### 3.利用堆结构

构造一个大小为 K 的最小堆，遍历数组，堆中始终是前 K 个最大数，最后堆的最小值就是所求结果。 

```python
class Solution(object):
    def findKthLargest(self, nums, k):
        """
        :type nums: List[int]
        :type k: int
        :rtype: int
        """
        import heapq
        heap = nums[:k]
        heapq.heapify(heap)
        for num in nums[k:]:
            if num > heap[0]:
                heapq.heapreplace(heap,num)
        return heap[0]
```

关于 python 中 ```heapq``` 模块

* ```heapq.heapify(nums)``` 会将 ```nums``` 转变成一个 heap，in-place，in linear time.

下面是一个关于利用```heapq``` 模块实现一个优先级队列

```python
import heapq

class PriorityQueue:
    def __init__(self):
        self._queue = []
        self._index = 0
    
    def push(self, item, priority): #item无法比较时，下面的_index起作用，确保不会出错
        heapq.heappush(self._queue, (-priority, self._index, item)) 
        self._index += 1

    def pop(self):
      return heapq.heappop(self._queue)[-1]
```

使用方法：

```python
>>> class Item:
... def __init__(self, name):
... self.name = name
... def __repr__(self):
... return 'Item({!r})'.format(self.name)
...
>>> q = PriorityQueue()
>>> q.push(Item('foo'), 1)
>>> q.push(Item('bar'), 5)
>>> q.push(Item('spam'), 4)
>>> q.push(Item('grok'), 1)
>>> q.pop()
Item('bar')
>>> q.pop()
Item('spam')
>>> q.pop()
Item('foo')
>>> q.pop()
Item('grok')
>>>
```

