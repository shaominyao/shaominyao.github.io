---
layout:     post
title:      一篇夯实一个知识点系列－－Python实现十大排序算法
subtitle:   
date:       2020-08-09
author:     Yao Shaomin
header-img: img/post-bg-article.jpg
catalog:    true
tags:
-   Python Algorithm
---

#### 写在前面

>   排序是查找是算法中最重要的两个概念，我们大多数情况下都在进行查找和排序。科学家们穷尽努力，想使得排序和查找能够更加快速。本篇文章用Python实现十大排序算法。

#### 干货儿

>   排序算法从不同维度可以分为好多类别，从其排序思想(排序思想一般决定了其时间复杂度的量级)来看，主要可以分为四类：
>
>   -   双层循环比较排序：平方级排序
>   -   分治策略比较排序：对数级排序
>   -   另辟蹊径的非比较方式排序：线性级排序
>   -   笑死人不偿命的其它排序：有着天马行空的时间复杂度，难以描述。

##### 平方级排序

-   冒泡排序

    >   1.  从数组的第一个元素开始，比较当前元素和下一个元素，如果当前元素大于下一个元素，交换两元素位置。
    >   2.  接着从第二个元素开始，重复第一步，直到当前元素为最后一个元素。此时最后一个元素为最大元素。未排序数组为除最后一个元素之外的其它元素。
    >   3.  对未排序数组不断重复以上步骤，直到未排序数组为空。

    ```python
    def bubble_sort(arr):
        length = len(arr)
        for i in range(length):
            for j in range(length-i-1):
                if arr[j] > arr[j+1]:
                    arr[j], arr[j+1] = arr[j+1], arr[j]
        return arr
    ```

-   选择排序

    >   1.  选取数组中的最小元素，和数组中的第一个元素交换位置
    >   2.  选取数组中除第一个元素外剩余元素的最小元素，和数组中的第二个元素交换位置。
    >   3.  不断重复以上步骤，直到当前选取的元素为数组中最后一个元素。

    ```python
    def select_sort(arr):
        length = len(arr)
        for i in range(length):
            min_ix = i
            for j in range(i, length):
                if arr[j] < arr[min_ix]:
                    min_ix = j
            arr[min_ix], arr[i] = arr[i], arr[min_ix]
        return arr
    ```

-   插入排序

    >   1.  从数组的第一个元素开始，不断比较当前元素和前一个元素。如果当前元素比前一个元素小，那么就将当前元素插入到前一个元素的前面(即两者交换位置)
    >   2.  从第二个元素开始，不断重复以上步骤，直到所有元素全部经历上述步骤。

    ```python
    def insert_sort(arr):
        length = len(arr)
        for i in range(length):
            for j in range(i, 0, -1):
                if arr[j] < arr[j-1]:
                    arr[j], arr[j-1] = arr[j-1], arr[j]
        return arr
    ```

##### 对数级排序

-   希尔排序

    >   1.  选择一个增量值k，分别将数组中索引以k为间隔的元素放在同一个数组中。
    >   2.  将增量值缩小为原增量值的1/2，然后重复步骤1。
    >   3.  直到增量值为1，使用插入排序对已经部分有序的数组进行排序。

    ```python
    def shell_sort(arr):
        n = len(arr)
        gap = int(n/2)
        while gap > 0: 
            for i in range(gap,n): 
                temp = arr[i] 
                j = i 
                while  j >= gap and arr[j-gap] >temp: 
                    arr[j] = arr[j-gap] 
                    j -= gap 
                arr[j] = temp 
            gap = int(gap/2)
        return arr
    ```

-   归并排序

    >   1.  以数组中间元素为界，将数组分为等长的两个数组(可能不等长，和数组长度的奇偶性有关)。
    >   2.  对所有数组执行步骤1
    >   3.  不断重复以上步骤，直到将数组分割为多个包含单个元素的数组。
    >   4.  将以上数组两两合并，并排序，此时为多个包含有序的两个元素的数组(可能包含单个元素，跟数组长度的奇偶性有关)。
    >   5.  重复步骤4，直到将所有数组合并为一个数组

    ```python
    def merge(left, right):
        i = j = 0
        res = []
        while i < len(left) and j < len(right):
            if left[i] < right[j]:
                res.append(left[i])
                i += 1
            else:
                res.append(right[j])
                j += 1
        if i == len(left):
            res.extend(right[j:])
        else:
            res.extend(left[i:])
        return res
    
    def merge_sort(arr):
        if len(arr) <= 1:
            return arr
        length = len(arr)
        i = int(length / 2)
        left = merge_sort(arr[:i])
        right = merge_sort(arr[i:])
        return merge(left, right)
    ```

-   快速排序

    >   1.  挑选一个元素为基准
    >   2.  比基准大的元素作为一个数组，比基准小或者等于基准的元素作为一个数组。
    >   3.  对新分割的数组，不断重复以上步骤，直到分割后的数组只含有1个或者0个元素
    >   4.  递归地合并以上数组为有序数组，合并方式为：[小于等于基准的元素]+[基准]+[大于基准的元素]

    ```python
    def fast_sort(arr):
        if len(arr) <= 1:
            return arr
        pivot = arr.pop()
        left = [i for i in arr if i <= pivot]
        right = [i for i in arr if i > pivot]
        return fast_sort(left) + [pivot] + fast_sort(right)
    ```

    以上算法需要额外的空间，如果我们将小于等于基准的元素不断置于基准元素之前，大于基准的元素置于基准元素之后，那么就可以实现不需要额外空间的就地排序。

    ```python
    def fast_sort_on_extra_spacing(arr):
        l = 0
        h = len(arr)-1
    
        def partition(arr, l, h):
            pivot = arr[h]
            for i in range(l, h):
                if arr[i] <= pivot:
                    arr[l], arr[i] = arr[i], arr[l]
                    l += 1
            arr[h], arr[l] = arr[l], arr[h]
            return l
    
        def fast_sort(arr, l, h):
            if l < h:
                pivot = partition(arr, l, h)
                fast_sort(arr, l, pivot-1)
                fast_sort(arr, pivot+1, h)
            return arr
        return fast_sort(arr, l, h)
    ```

-   堆排序

    >   1.  先对待排序数组构造大根堆
    >   2.  将大根堆第一个元素和最后一个元素交换位置。此时最后一个元素为最大元素，待排序数组为除最后一个元素之外的所有元素。
    >   3.  对待排序数组不断重复以上步骤，直到待排序数组中只有一个元素。

    ```python
    def heapify(arr, n, i):
        # build a max root heap
        max_ix = i
        left_i = 2 * i + 1
        right_i = 2 * i + 2
    
        if left_i < n and arr[max_ix] < arr[left_i]:
            max_ix = left_i
        if right_i < n and arr[max_ix] < arr[right_i]:
            max_ix = right_i
        if max_ix != i:
            arr[max_ix], arr[i] = arr[i], arr[max_ix] 
            heapify(arr, n, max_ix)
    
    def heap_sort(arr):
        for i in range(n-1, -1, -1):
            heapify(arr, n, i)
    
        for i in range(n-1, 0, -1):
            arr[i], arr[0] = arr[0], arr[i]
            heapify(arr, i, 0)
        return arr
    ```

##### 线性级排序

>   此排序方法只适用于数组元素全部为整数的情景。

-   计数排序

    >   1.  找出待排序数组中最大的元素，构造一个长度为此元素值的计数数组。
    >   2.  遍历待排序数组元素，以当前元素为索引，将计数数组中的对应值加1.
    >   3.  此时计数数组中的索引为待排序数组中的元素，值为出现的次数。将计数数组中所有值非0的元素索引根据其出现次数串联起来。

    ```python
    def count_sort(arr):
        min_ix, max_ix = min(arr), max(arr)
        bucket = [0 for _ in range(max_ix+1)]
        for i in arr:
            bucket[i] += 1
        return sum([[i] * bucket[i] for i in range(len(bucket)) if bucket[i] != 0], [])
    ```

-   桶排序

    >   1.  设置固定数量的桶(这是个技术活儿).
    >   2.  将待排序数组中的元素放入对应的桶中(对应关系也是个技术活儿，下面的例子中采用整除)
    >   3.  将非空桶中的元素串联起来。

    ```python
    def bucket_sort(arr):
        min_ix, max_ix = min(arr), max(arr)
        bucket_range = (max_ix - min_ix) / len(arr)
        # +1 avoid for that max_ix - min_ix will raise a IndexError
        temp_bucket = [[] for i in range(len(arr) + 1)]
        for i in arr:
            temp_bucket[int((i-min_ix)//bucket_range)].append(i)
        return sum(temp_bucket, [])
    ```

-   基数排序

    >   1.  找出待排序数组中最大元素的位数。将所有元素补足此位数，补足方式为前面补0。
    >   2.  从最低位到最高位，进行多轮数组排序。

    ```python
    def radix_sort(arr):
        max_value = max(arr)
        num_digits = len(str(max_value))
        for i in range(num_digits):
            bucket = [[] for _ in range(10)]
            for j in arr:
                bucket[j//(10**i)%10].append(j)
            arr = [j for i in bucket for j in i]
        return arr
    ```

##### 笑死人不偿命排序

-   睡排序

    >   让多个进程(线程)分别睡眠待排序数组中的元素时长，先睡醒的进程(线程)，对应元素追加到结果数组中。

-   猴子排序

    >   不停随机排序，然后检查是否元素全部有序。如果你是欧皇，那么你可以尝试用这个排序算法，很可能一次搞定。

##### 排序算法复杂度、稳定性及通用性总结

| 算法     | 平均时间复杂度   | 最优时间复杂度        | 最坏时间复杂度        | 空间复杂度 | 是否原地排序 | 是否稳定 | 是否通用 |
| -------- | ---------------- | --------------------- | --------------------- | ---------- | ------------ | -------- | -------- |
| 冒泡排序 | O(n<sup>2</sup>) | O(n)                  | O(n<sup>2</sup>)      | O(1)       | 是           | 是       | 是       |
| 选择排序 | O(n<sup>2</sup>) | O(n<sup>2</sup>)      | O(n<sup>2</sup>)      | O(1)       | 是           | 否       | 是       |
| 插入排序 | O(n<sup>2</sup>) | O(n)                  | O(n<sup>2</sup>)      | O(1)       | 是           | 是       | 是       |
| 希尔排序 | O(n logn)        | O(n log<sup>2</sup>n) | O(n log<sup>2</sup>n) | O(1)       | 是           | 否       | 是       |
| 归并排序 | O(n logn)        | O(n logn)             | O(n logn)             | O(n)       | 否           | 是       | 是       |
| 快速排序 | O(n logn)        | O(n logn)             | O(n<sup>2</sup>)      | O(n logn)  | 是           | 否       | 是       |
| 堆排序   | O(n logn)        | O(n logn)             | O(n logn)             | O(1)       | 是           | 否       | 是       |
| 计数排序 | O(n+k)           | O(n+k)                | O(n+k)                | O(k)       | 否           | 是       | 否       |
| 桶排序   | O(n+k)           | O(n+k)                | O(n<sup>2</sup>)      | O(n+k)     | 否           | 是       | 否       |
| 基数排序 | O(n*k)           | O(n*k)                | O(n*k)                | O(n+k)     | 否           | 是       | 否       |

##### 写在最后

排序算法是算法学习中的核心。掌握排序算法及其思想是学习其它算法的基础。希望大家可以熟练掌握。欢迎关注个人博客:[药少敏的博客](https://shaominyao.github.io)。

