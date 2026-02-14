---
title: 排序算法
copyright: true
tags:
  - 数据结构与算法
  - 排序算法
categories:
  - 数据结构与算法
excerpt: 介绍数据结构与算法中的排序算法, 包括多种内部排序与外部排序
date: 2024-08-15 14:17:49
updated: 2024-08-15 14:17:49
---


## 插入排序

### 直接插入排序

```c++
void sort(std::vector<int>& v)
{
    for (int i = 0;i < v.size();i++)
    {
        for (int j = i - 1;j >= 0;j--)
        {
            if (v[j] > v[j + 1]) std::swap(v[j], v[j + 1]);
            else break;
        }
    }
}
```

- 最好情况: 数组有序, 外层循环遍历一遍即可, 时间复杂度为 $O(n)$
- 最差情况: 数组倒序, `break` 将不会执行, 时间复杂度为 $O(n^2)$
- 平均 **时间复杂度** 为 $O(n^2)$
- 只需要常数个额外空间, 所以 **空间复杂度** 为 $O(1)$
- **稳定排序**

### 折半插入排序

对直接插入排序的小优化, 在直接插入排序中, 需要从后向前比较, 找到第一个小于等于待插入元素的位置. 将这一查找改为二分(折半)查找, 可以提升一部分性能. 但是算法时间复杂度仍为 $O(n^2)$, 这是因为需要将数组元素整体向后移动导致的.

```c++
void sort(std::vector<int>& v)
{
    for (int i = 0;i < v.size();i++)
    {
        int l = 0;
        int r = i - 1;
        while (l <= r)
        {
            int m = (l + r) / 2;
            if (v[m] <= v[i])
            {
                l = m + 1;
            }
            else
            {
                r = m - 1;
            }
        }
        for (int j = i - 1;j >= l;j--)
        {
            std::swap(v[j], v[j + 1]);
        }
    }
}
```

### 希尔排序

希尔排序也称为 **缩小增量排序**. 具体过程如下

1. 取序列中以 `gap` 为间隔的子序列
2. 对每一个子序列使用插入排序
3. 减少 `gap` 后重复上述过程, 直到 `gap` 为 1(`gap` 为 1 时也要进行一次插入排序)

注: 对于 `gap` 的减少方式, 一般直接减半就可以. 额外的讨论可见 [oi-wiki](https://oi-wiki.org/basic/shell-sort/#%E8%BF%87%E7%A8%8B) 和 [维基百科](https://zh.wikipedia.org/wiki/%E5%B8%8C%E5%B0%94%E6%8E%92%E5%BA%8F#C++)

```c++
void sort(std::vector<int>& v)
{
    int gap = v.size();
    while (gap > 1)
    {
        gap /= 2;
        for (int i = gap;i < v.size();i++)
        {
            for (int j = i; j >= gap && v[j] < v[j - gap]; j -= gap)
            {
                std::swap(v[j], v[j - gap]);
            }
        }
    }
}
```

上面这段代码中, 两个 `for` 循环不太好理解. 因为他把对一个子序列的插入排序分散到外层循环里去了. 在外层循环中遍历每一个将要插入的元素 `v[i]`, 内层循环完成一个子序列 (`... v[i-gap] v[i] v[i+gap] ...`) 中 `v[i]` 的插入排序.

- 最坏情况下, **时间复杂度** 为 $O(n^2)$, 在步长序列为 $2^k-1$ 时, 时间复杂度为 $O(n^{\frac{3}{2}})$; 在步长序列为 $2^i3^j$ 时, 时间复杂度为 $O(n\log_2^2n)$
- 希尔排序需要常数的额外空间, 所以 **空间复杂度** 为 $O(1)$
- **不稳定排序**

## 交换排序

### 冒泡排序

```c++
void sort(std::vector<int>& v)
{
    bool flag = true;
    while (flag)
    {
        flag = false;
        for (int i = 1; i < v.size(); ++i)
        {
            if (v[i] < v[i - 1])
            {
                flag = true;
                std::swap(v[i], v[i - 1]);
            }
        }
    }
}
```

代码中使用使用一个 `flag` 变量来记录数组是否存在无序的两个元素, 如果存在无序就继续循环, 否则可以停止. 最差情况下需要 `while` 循环 $n-1$ 次.

- 最好情况下, 数组有序, 冒泡排序只需遍历一遍数组即可, 时间复杂度为 $O(1)$
- 最差情况下, 数组逆序, 时间复杂度为 $O(n^2)$
- 平均情况下, **时间复杂度** 为 $O(n^2)$
- **空间复杂度** 为 $O(1)$
- **稳定排序**

### 快速排序

快速排序有很多种写法, 这里是一个比较容易理解的(对我自己来说)挖坑法. 更多写法可以看 [六大排序算法](https://blog.csdn.net/weixin_50886514/article/details/119045154)

快速排序是一个递归分治的过程, 所以需要一个递归函数 `quick_sort`. 需要排序的元素是数组 `v` 的 `low_boundary` 到 `high_boundary` 全闭区间内的元素. 所以在 `sort` 函数中, 边界为 $[0, v.size() - 1]$ 即数组的全部元素.

在 `quick_sort` 中, `povit` 默认选择 `left` 也就是 `low_boundary` 处的元素, 并以该位置为第一个坑. 然后在循环体中我们就需要先从 `right` 处开始找第一个比 `povit` 小的元素. 同理我们可以知道, 如果我们以 `high_boundary` 处为第一个坑, 那么循环里我们就要先从 `left` 处开始找第一个比 `povit` 大的元素了.

在这种做法中, 可以预见有三种情况: 
1. 坑位在 `left` 处
2. 坑位在 `right` 处
3. 坑位在 `left` 和 `right` 处(此时 `left` 与 `right` 相同)

该算法中一直满足的约束是 `left` 以左都是比 `povit` 小的数, `right` 以右都是比 `povit` 大的数. 一开始 `left` 左侧没有数, `right` 右侧也没有数, 自然满足该约束.
 
在算法的结束时, `left` 和 `right` 相等, 坑位在 `left`(`right`) 处, 且其满足上面的约束, 那么我们就可以知道此时数组被分为了小于 `povit` (左侧) 和大于 `povit` (右侧) 两部分. 然后把 `povit` 填入坑位即可递归的排序左侧和右侧的数据.

```c++
void quick_sort(std::vector<int>& v, int low_boundary, int high_boundary)
{
    if (low_boundary >= high_boundary) return;
    int left = low_boundary;
    int right = high_boundary;
    int povit = v[left];
    while (left < right)
    {
        while (left < right && v[right] >= povit) right--;
        v[left] = v[right];
        while (left < right && v[left] <= povit) left++;
        v[right] = v[left];
    }
    v[right] = povit;
    quick_sort(v, low_boundary, left - 1);
    quick_sort(v, right + 1, high_boundary);
}

void sort(std::vector<int>& v)
{
    quick_sort(v, 0, v.size() - 1);
}
```

- 最好情况下, 时间复杂度 $O(n\log_2n)$, 空间复杂度 $O(\log_2n)$
- 最坏情况下, 时间复杂度 $O(n^2)$, 空间复杂度 $O(n)$
- 平均情况下, **时间复杂度** $O(n\log_2n)$, **空间复杂度** $O(\log_2n)$
- **不稳定排序**

## 选择排序

### 简单选择排序

每次选择数组中最小(大)的元素排序.

```c++
void sort(std::vector<int>& v)
{
    for (int i = 0; i < v.size(); i++)
    {
        int min_item = v[i];
        int min_idx = i;
        for (int j = i + 1;j < v.size();j++)
        {
            if (min_item > v[j])
            {
                min_idx = j;
                min_item = v[j];
            }
        }
        std::swap(v[i], v[min_idx]);
    }
}

```

- 最坏时间复杂度 $O(n^2)$
- 最优时间复杂度 $O(n^2)$
- 平均 **时间复杂度** $O(n^2)$
- **空间复杂度** $O(1)$
- **不稳定排序**

### 堆排序

利用二叉堆这种数据结构所设计的一种排序算法. 首先建立大顶堆, 然后将堆顶的元素取出, 作为最大值, 与数组尾部的元素交换, 并维持残余堆的性质(对堆进行调整); 之后将堆顶的元素取出, 作为次大值, 与数组倒数第二位元素交换, 并维持残余堆的性质; 以此类推, 在第 n-1 次操作后, 整个数组就完成了排序.

原地建堆有两种方式, 一种是从上向下建堆, 另一种是从下向上建堆.

从下向上建堆, 保证数组后面一部分满足二叉堆(但是把后面一部分单独拿出来并不是二叉堆).

```c++
void buildMaxHeap(std::vector<int>& v)
{
    // 从最后一个中间节点开始
    for (int i = (v.size() - 2) / 2;i >= 0;i--)
    {
        int this_idx = i;
        int child_idx = this_idx * 2 + 1;
        while (child_idx < v.size())
        {
            // 选择两个儿子中较大的那一个
            if (child_idx + 1 < v.size() && v[child_idx] < v[child_idx + 1]) child_idx += 1;
            if (v[this_idx] >= v[child_idx]) break;
            std::swap(v[this_idx], v[child_idx]);
            this_idx = child_idx;
            child_idx = this_idx * 2 + 1;
        }
    }
}
```

从上向下建堆, 保证数组前面一部分满足二叉堆(把数组前面一部分单独拿出来是一个完整的二叉堆).

```c++
void buildMaxHeap(std::vector<int>& v)
{
    for (int i = 0;i < v.size();i++)
    {
        int parent_idx = (i - 1) / 2;
        int this_idx = i;
        while (parent_idx >= 0)
        {
            // 比父亲大就向上调整
            if (v[this_idx] > v[parent_idx]) std::swap(v[this_idx], v[parent_idx]);
            else break;
            this_idx = parent_idx;
            parent_idx = (parent_idx - 1) / 2;
        }
    }
}
```

完整堆排序

```c++
void max_heapify(std::vector<int>& v, int heap_start, int heap_end)
{
    int this_idx = heap_start;
    int child_idx = this_idx * 2 + 1;
    while (child_idx <= heap_end)
    {
        if (child_idx + 1 <= heap_end && v[child_idx] < v[child_idx + 1])
            child_idx += 1;
        if (v[this_idx] >= v[child_idx]) break;
        std::swap(v[this_idx], v[child_idx]);
        this_idx = child_idx;
        child_idx = this_idx * 2 + 1;
    }
}

void sort(std::vector<int>& v)
{
    for (int i = (v.size() - 2) / 2;i >= 0;i--) 
        max_heapify(v, i, v.size() - 1);
    for (int i = v.size() - 1;i > 0;i--)
    {
        std::swap(v[0], v[i]);
        max_heapify(v, 0, i - 1);
    }
}
```

- 最坏, 最优, 平均 **时间复杂度** 为 $O(n\log_2n)$
- 在数组上原地建堆 **空间复杂度** 为 $O(1)$
- **不稳定排序**

## 归并排序

归并排序基于分治思想将数组分段排序后合并, 

```c++

void merge_sort(std::vector<int>& v, int start, int end)
{
    if (start >= end) return;
    int mid = (start + end) / 2;
    merge_sort(v, start, mid);
    merge_sort(v, mid + 1, end);
    std::vector<int> temp;
    int v1_idx = start, v2_idx = mid + 1;
    // 选择两个子数组中较小的一个加入临时数组
    while (v1_idx <= mid && v2_idx <= end)
    {
        if (v[v1_idx] <= v[v2_idx]) // 决定升序还是降序, 等于号保证稳定排序
        {
            temp.push_back(v[v1_idx]);
            v1_idx += 1;
        }
        else
        {
            temp.push_back(v[v2_idx]);
            v2_idx += 1;
        }
    }
    // 将两个子数组中的剩余的元素归并
    for (; v1_idx <= mid; v1_idx += 1) temp.push_back(v[v1_idx]);
    for (; v2_idx <= end; v2_idx += 1) temp.push_back(v[v2_idx]);
    // 拷贝到原数组
    for (int i = start;i <= end;i++) v[i] = temp[i - start];
}

void sort(std::vector<int>& v)
{
    merge_sort(v, 0, v.size() - 1);
}
```

- 最优, 最坏与平均情况下 **时间复杂度** 均为 $O(n\log_2n)$
- **空间复杂度** 为 $O(n)$。
- **稳定排序**

## 基数排序

将待排序的元素分为多个关键字, 比如排序 0~999 范围内的整数. 我们令元素(也就是一个整数)的第 1 关键字为百位上的数字, 第 2 关键字为十位上的数字, 第 3 关键字为个位上的数字. 可以知道, 第 1 关键字的权重最大, 第 3 关键字的权重最小.

我们将元素首先根据第 1 关键字排序, 然后第 2 关键字, 最后第 3 关键字. 这种从权重大的关键字排序到权重小的关键字的排序方式称为 **MSD(Most Significant Digit first) 基数排序**. 相反, 我们从权重小的关键字排序到权重大的关键字时, 称为 **LSD(Least Significant Digit first) 基数排序**.

对每个关键字的排序我们可以使用插入排序等稳定排序, 这保证了基数排序的稳定性.

- **时间复杂度** 为 $O(d(n+r))$, $d$ 为关键字数量, $n$ 为元素数量, $r$ 为关键字值域
- **空间复杂度** 为 $O(r)$
- 如果对于每个关键字的排序是稳定排序, 那么基数排序就是 **稳定排序**

## 外部排序

待补充