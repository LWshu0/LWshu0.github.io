---
title: 查找算法
copyright: true
date: 2024-08-16 23:42:27
updated: 2024-08-16 23:42:27
tags:
    - 数据结构与算法
    - 查找算法
categories:
    - 数据结构与算法
excerpt: 介绍数据结构与算法中的查找算法, 包括线性查找, 树形查找, 散列查找
---


## 线性结构查找

### 顺序查找

**基本顺序查找**

```c++
bool find(std::vector<int>& v, int k)
{
    for (int i = 0; i < v.size(); i++)
    {
        if (v[i] == k) return true;
    }
    return false;
}
```

注: 这里的 `v.size()` 是 `constexpr` 即编译时计算, 在运行时并不会产生函数调用, 所以如下两种写法在效率上是一样的:
```c++
for (int i = 0; i < v.size(); i++){}    // 方式一
for (int i = size() - 1; i >= 0; i--){} // 方式二
```

**哨兵**

在上面的基本顺序查找中, 每次循环需要两次比较 `i < v.size()` 和 `v[i] == k`. 通过引入哨兵可以减少一次比较(但是要消耗额外的一个空间):

```c++
// 此时数组中有效数据是 [1, v.size() - 1], v[0] 将用来存放哨兵
bool find(std::vector<int>& v, int k)
{
    v[0] = k;   // 哨兵
    int i;
    for (i = size() - 1; v[i] != k; i--);
    return i != 0;
}
```

上面的例子中把 `v[0]` 作为了哨兵, 这样子在 `for` 循环遍历中, 一定可以 **避免访问越界** (因为 `v[0]` 就是我们要找的元素, 此时一定结束循环), 同时 **减少了一次比较运算**. 此外, 我们也可以把 `v[v.size() - 1]` 作为哨兵, 那么就需要正向遍历数组了, 代码实现非常相似, 此处不再赘述.

**有序表的顺序查找**

不妨设此时数组是升序的, 当我们从 0 开始遍历数组查找 `k` 时, 当我们找到一个大于 `k` 的值时, 说明其后也都是大于 `k` 的值(一定没有 `k` 了), 此时就可以提前结束以提高效率.

```c++
bool find(std::vector<int>& v, int k)
{
    int i = 0;
    while (i < v.size())
    {
        if (v[i] == k) return true;
        else if(v[i] > k) return false; // 此时再往后查找没有意义
        else i++;
    }
    return false;
}
```

尽管上面提到了一些优化, 顺序查找的 **时间复杂度** 都为 $O(n)$. 但是它具有十分的通用性, 不论是连续存储还是链式存储, 该方法都可以使用.

### 折半查找

折半查找也叫二分查找, 仅用于连续有序数组上(保证可以随机访问).

```c++
bool myfind(std::vector<int>& v, int k)
{
    int l = 0;
    int r = v.size() - 1;
    while (l < r)
    {
        int mid = (l + r) / 2;
        if (v[mid] < k) l = mid + 1;
        else if (v[mid] > k) r = mid - 1;
        else return true;
    }
    return v[l] == k;
}
```

时间复杂度为 $O(log_2n)$

### 分块查找

将数据(下图中的查找表)分为多个块, **块内无序**, **块间有序**. 额外维护一个索引表. 在查找元素时, 首先通过二分(或者顺序)查找到对应的块, 然后在块内进行顺序查找.

![分块查找](/img/DataStruct-Find/1723890464172.png)

## 树形结构查找

### 二叉排序树(BST)

[代码实现-非递归版本](https://github.com/LWshu0/SmallStep/blob/main/DataStructure%26Algorithm/Tree/BST.hpp)

#### 二叉排序树定义

1. 如果根节点左子树不为空, 则左子树中所有节点的值都小于根节点
2. 如果根节点右子树不为空, 则右子树中所有节点的值都大于根节点
3. 左右子树也是二叉排序树

由定义可知, 对于二叉排序树的中序遍历将得到一个非递减的序列.

>注: 因为二叉排序树中可能存在重复的元素, 对于可能有重复元素的树, 我们在节点中额外添加一个计数字段, 用来记录该元素出现的次数. 同时因为重复元素的存在, 导致中序遍历将是一个非递减序列.

代码实现树节点的定义, 这里给出了 `TreeNode` 两个构造函数, 后续代码中将使用第二个构造.

```c++
struct TreeNode {
    TreeNode* left;     // 指向左子树的指针
    TreeNode* right;    // 指向右子树的指针
    int count;          // 节点元素出现的次数
    int value;          // 节点元素值
    TreeNode() :left(nullptr), right(nullptr), count(0), value(0) {}
    TreeNode(int v) :left(nullptr), right(nullptr), count(1), value(v) {}
};
```

二叉排序树类 `BST` 的成员包括一个 `TreeNode* root;` 即树的根节点, 初始情况下 `root` 的值为 `nullptr`. 后续代码都作为 `BST` 类的成员函数.

#### 二叉排序树查找

从根节点开始, 沿着某一个分支逐层向下查找. 定义待查找的元素为 `value`, 树的根节点为 `root`, 我们以根节点为当前节点 `search_ptr` 开始查找, 具体过程:
1. 如果 `value` 的值小于 `search_ptr` , 向左子树移动
2. 如果 `value` 的值大于 `search_ptr` , 向右子树移动
3. 如果 `value` 的值等于 `search_ptr` , 返回查找成功

```c++
TreeNode* findNode(int value)
{
    TreeNode* search_ptr = root;
    while (search_ptr)
    {
        if (value < search_ptr->value) search_ptr = search_ptr->left;
        else if (value > search_ptr->value) search_ptr = search_ptr->right;
        else return search_ptr; // 查找成功
    }
    return nullptr; // 查找失败
}
```

二叉排序树平均查找时间为 $O(\log_2n)$, 最坏情况下, 退化为一个单链表, 此时查找时间为 $O(n)$

#### 二叉排序树的插入

二叉排序树的插入过程(下面的代码是 **非递归** 的写法, 如果使用 **递归** 代码会更简洁, 但是会产生递归的内存开销):
1. 如果二叉排序树为空, 新建根节点.
2. 当二叉排序树不空时, 从根节点开始向下找待插入元素的插入位置. 即代码中令 `search_ptr` 等于 `root`.
    - 当 `search_ptr` 的值小于待插入的 `value` 时, 需要向右子树移动, 此时应该先检查右子树是否存在. 如果右子树不存在说明待插入的 `value` 是一个不存在在二叉排序树中的新的值, 新建一个节点然后跳出循环即可. 否则就向右子树移动.
    - 当 `search_ptr` 的值大于待插入的 `value` 时, 需要向左子树移动, 操作与上述小于的情况类似.
    - 当 `search_ptr` 的值大于待插入的 `value` 时, 说明插入了一个重复元素, 令其计数加一即可.

```c++
void insertNode(int value)
{
    if (nullptr == root)
    {
        root = new TreeNode(value);
    }
    else
    {
        TreeNode* search_ptr = root;
        while (search_ptr)
        {
            if (search_ptr->value < value)
            {
                if (nullptr == search_ptr->right)
                {
                    search_ptr->right = new TreeNode(value);
                    break;
                }
                else search_ptr = search_ptr->right;
            }
            else if (search_ptr->value > value)
            {
                if (nullptr == search_ptr->left)
                {
                    search_ptr->left = new TreeNode(value);
                    break;
                }
                else search_ptr = search_ptr->left;
            }
            else
            {
                search_ptr->count += 1;
                break;
            }
        }
    }
}
```

#### 二叉排序树的删除

> 二叉排序树删除然后插入同一个元素, 可能会改变树的结构

二叉排序树的节点删除, 如果计数大于 1, 只要减少计数即可, 此时删除操作不会改变二叉排序树的结构, 当待删除的节点计数为 1 时, 需要把节点中书上移除, 此时一共有三种情况:
1. 待删除的节点为叶子节点(没有左右子树), 此时直接删除
2. 待删除的节点有一个子树(左子树或者右子树), 删除节点后让子树代替删除节点的位置
3. 待删除的节点有两个子树, 选择右子树最小的节点(或者左子树最大的节点)移动到删除节点的位置代替删除节点

为了解决上述第三种情况, 我们需要先写一个拿到右子树中最小节点的方法 `cutMinimum(TreeNode** subtree)`, 该方法将从子树中剪下最小的节点返回. 这里之所以传入一个 `TreeNode**` 是因为当子树的根节点只有右子树时, 子树中最小节点就是根节点本身, 当剪下最小节点(即根节点)后, 需要修改指向子树的指针使其指向原来根节点的右子树. 这样做保证了整个二叉排序树的结构完整性, 避免了野指针造成的内存泄漏.

下面的代码中使用了一个 `TreeNode**` 类型的 `pre_search_ptr`. 该变量的意义是用来找到 `search_ptr` 指向的节点的父节点中指向 `search_ptr` 指向的节点的指针(在 `cutMinimum` 函数中为 `left` 指针), 作用是维护树结构, 使在剪切节点时树不断裂. 如果这个描述有些绕, 可以看后面的图示帮助理解.

```c++
TreeNode* cutMinimum(TreeNode** subtree)
{
    // 传入子树为空, 返回 nullptr
    if (nullptr == *subtree) return nullptr;
    TreeNode** pre_search_ptr = subtree;
    TreeNode* search_ptr = *subtree;
    // 找到最小元素节点
    while (search_ptr->left)
    {
        pre_search_ptr = &(search_ptr->left);
        search_ptr = search_ptr->left;
    }
    // 将最小节点的右子树接到其父节点的 left 处
    *pre_search_ptr = search_ptr->right;
    search_ptr->right = nullptr;
    return search_ptr;
}
```

下面的图可以形象地表达 `pre_search_ptr` 与 `search_ptr` 的关系. 在查找节点的过程中, `search_ptr` 指向当前正在查看的节点(下图中的时刻, 正在查看值为 3 的节点, 我们后面称值为 n 的节点为 n 号节点). `pre_search_ptr` 指向 3 号节点的父节点(14 号节点)中指向 3 号节点的指针 `left`. 

![pre_search_ptr 与 search_ptr](/img/DataStruct-Find/1724079577803.png)

此时由于 3 号节点的左子树为空, 循环结束. 执行 `*pre_search_ptr = search_ptr->right;` 将把 14 号节点的 `left` 指针指向 3 号节点的右子树(不过, 此时 3 号节点的右子树为空), 这一操作保证在 3 号节点右子树不空时, 树不会断裂. 然后执行 `search_ptr->right = nullptr;` 彻底将 3 号节点从树上剥离下来.

在理解了上面的操作后, 接下来是正式的删除节点代码. 删除代码中也使用了 `pre_search_ptr` 和 `search_ptr`, 所以先理解上面的代码后再看.

```c++
void removeNode(int value)
{
    TreeNode** pre_search_ptr = &root;
    TreeNode* search_ptr = root;
    // 尝试找到待删除元素
    while (search_ptr)
    {
        if (value < search_ptr->value)
        {
            pre_search_ptr = &(search_ptr->left);
            search_ptr = search_ptr->left;
        }
        else if (value > search_ptr->value)
        {
            pre_search_ptr = &(search_ptr->right);
            search_ptr = search_ptr->right;
        }
        else break;
    }

    if (nullptr == search_ptr) return;  // 树为空或树中不存在待删除的元素

    // 如果节点计数大于 1, 只需要更改计数, 不做节点删除
    if (search_ptr->count > 1)
    {
        search_ptr->count--;
        return;
    }

    if (nullptr == search_ptr->left)    // 待删除节点左子树为空(右子树可能为空, 也可能不空)
    {
        *pre_search_ptr = search_ptr->right;
        delete search_ptr;
    }
    else if (nullptr == search_ptr->right)  // 待删除节点右子树为空(左子树不空)
    {
        *pre_search_ptr = search_ptr->left;
        delete search_ptr;
    }
    else  // 待删除节点左右子树都不空
    {
        // 得到右子树的最小节点
        TreeNode* min_node = cutMinimum(&search_ptr->right);
        min_node->left = search_ptr->left;
        min_node->right = search_ptr->right;
        *pre_search_ptr = min_node;
        delete search_ptr;
    }
}
```

### 二叉平衡树

**二叉平衡树**(BBT, Balanced Binary Tree), 也称为 **AVL 树**(大学教授 G.M. Adelson-Velsky 和 E.M. Landis 名称的缩写). [代码实现-非递归版本](https://github.com/LWshu0/SmallStep/blob/main/DataStructure%26Algorithm/Tree/AVL.hpp)

#### 基础定义

- **AVL 树**: AVL 树是一个满足平衡因子约束的 **二叉排序树**.
- **树高**: 定义叶子节点的树高为 1, 向上依次递增. 当一个节点的左右子树高度不一致时, 取子树高度最大值加 1 作为当前节点树高.
- **平衡因子**: 一个节点的平衡因子为其左子树高度 - 右子树高度. 在 AVL 树中要求每个节点的平衡因子取值范围为 $[-1, 1]$, 当一个节点的平衡因子小于 -1 或者 大于 1 时, 需要对树结构进行调整(即旋转)

#### AVL 查找

由于 AVL 树是一个二叉平衡树, 所以其查找方式与二叉平衡树完全一致.

#### AVL 的旋转

一个不满足平衡因子约束的 AVL 树可以通过旋转操作使其满足约束, 旋转操作分为左旋和右旋. 

> 注: 该小节中的示例 *AVL 左旋* 和 *AVL 右旋* 不满足 AVL 的定义

**左旋**

下图例为以 3 号节点为根节点的子树左旋. 左旋操作更改了 2 号节点的 `right` 指针, 3 号节点的 `right` 指针, 4 号节点的 `left` 指针. (把 4 号节点原来的左子树接到 3 号节点的右子树上)

![AVL 左旋](/img/DataStruct-Find/1724240692673.png)

要实现上图中的左旋, 需要传入指向 3 号节点的指针(2 号节点的 `right` 指针). 为了保证树的结构, 需要调用后赋值 `node2->right = rotateLeft(node2->right);`. 由于 3 号和 4 号树节点的高度变化, 所以还要更新这两个树节点的高度.

```c++
NodeType* rotateLeft(NodeType* subtree)
{
    NodeType* new_root = subtree->right();
    subtree->right() = new_root->left();
    new_root->left() = subtree;
    updateHeight(subtree);
    updateHeight(new_root);
    return new_root;
}
```

**右旋**

下图例为以 3 号节点为根节点的子树右旋. 右旋操作更改了 4 号节点的 `left` 指针, 3 号节点的 `left` 指针, 2 号节点的 `right` 指针. (把 2 号节点原来的右子树接到 3 号节点的左子树上)

![AVL 右旋](/img/DataStruct-Find/1724240660885.png)

右旋代码与左旋类似, 不再赘述.

```c++
NodeType* rotateRight(NodeType* subtree)
{
    NodeType* new_root = subtree->left();
    subtree->left() = new_root->right();
    new_root->right() = subtree;
    updateHeight(subtree);
    updateHeight(new_root);
    return new_root;
}
```

#### AVL 插入

插入已存在节点只需计数加一即可, 下面讨论需要新建节点的情况. 新建节点一定是叶子节点(参考二叉排序树的插入操作), 然后根据不同的情况进行旋转使其满足 AVL 树的定义. 总共四种情况: LL, LR, RR, RL.

**LL**, 即插入位置为左(Left)子树的左侧(Left). 下图新插入的节点是 1 号节点, 插入后 3 号节点的平衡因子为 2, 不满足 AVL 树的定义, 且以 3 号节点为根节点的子树是最小的不平衡子树. 由于插入位置是 3 号节点(最小不平衡子树的根节点)的左子树的左侧, 所以此时执行 LL 旋转, 即以 3 号节点为根节点右旋一次.

![LL旋转](/img/DataStruct-Find/1724241757371.png)

**LR**, 即插入位置为左(Left)子树的右侧(Right). 下图新插入的节点是 2 号节点, 插入后 3 号节点的平衡因子为 2, 不满足 AVL 树的定义, 且以 3 号节点为根节点的子树是最小的不平衡子树. 由于插入位置是 3 号节点(最小不平衡子树的根节点)的左子树的右侧. 所以执行 LR 旋转, 即首先以 1 号节点为根节点左旋一次, 转为上面图 *LL旋转* 的情况. 然后以 3 号节点为根节点右旋一次.

![LR旋转](/img/DataStruct-Find/1724242284680.png)

**RR**, 即插入位置为右(Right)子树的右侧(Right). 下图新插入节点为 5 号节点, 插入后 3 号节点的平衡因子为 -2, 不满足 AVL 树的定义, 且以 3 号节点为根节点的子树是最小的不平衡子树. 由于插入位置是 3 号节点(最小不平衡子树的根节点)的右子树的右侧. 所以执行 RR 旋转, 即以 3 号节点为根节点左旋一次.

![RR旋转](/img/DataStruct-Find/1724242696304.png)

**RL**, 即插入位置为右(Right)子树的左侧(Left). 下图新插入的节点为 4 号节点, 插入后 3 号节点的平衡因子为 -2, 不满足 AVL 树的定义, 且以 3 号节点为根节点的子树是最小的不平衡子树. 由于插入位置是 3 号节点(最小不平衡子树的根节点)的右子树的左侧. 所以执行 RL 旋转, 即首先以 5 号节点为根节点右旋一次, 转为上图 *RR旋转* 的情况, 然后以 3 号节点为根节点左旋一次.

![RL旋转](/img/DataStruct-Find/1724242408379.png)

#### AVL 删除

按照二叉排序树的删除方式删除节点. 然后再调整不平衡的子树.

### 红黑树

#### 红黑树定义

0. 红黑树是一棵二叉排序树
1. 红黑树的每个节点要么是红色, 要么是黑色
2. 红黑树的根节点是黑色
3. 红黑树的叶子节点(指虚构的 NULL 节点)是黑色的
4. 不存在相邻的红色节点(即一个红色节点的儿子, 父亲都为黑色节点)
5. 对每一个节点, 从该节点到任一叶子节点的简单路径上, 所含的黑色节点数量相同(这个黑色节点数量包括 NULL 节点)

推论:
- **从根节点到叶子节点最长的路径不大于最短路径的两倍.** 因为最长路径是黑红交替的路径, 最短路径是全黑的路径.
- **有 $n$ 个内部节点的红黑树的高度 $h \leq 2\log_2(n+1)$ .** 对于一个高度为 $h$ 的红黑树, 可以确定其黑高至少为 $h/2$. 那么根据红黑树全为黑色节点来计算得到的内部节点数量一定小于实际的内部节点数量, 因为红色节点可能在红黑树的某些位置出现. 所以可以得到 $n \geq 2^{h/2}-1$, 即 $$h \leq 2\log_2(n+1)$

#### 红黑树插入

红黑树的插入与二叉查找树的插入方式类似, 只是还要进行额外的旋转与变色操作使插入后满足红黑树的定义(新插入的节点默认为红色). 所以红黑树的插入可以分为两大步, 第一步是插入, 第二步是调整. 红黑树插入新节点的位置必然是某个叶子节点, 插入后再向上调整知道整个树满足红黑树定义. 由于需要向上调整, 所以我们令当前需要调整的节点为 **当前节点**, 当前节点最开始是新插入的红色节点, 调整过程中可能是某个内部红色节点. 下面根据可能出现的调整情况从简单到复杂依次说明.

**当前节点为根节点(没有父节点)**

将其置为黑色, 结束调整.

**当前节点的父节点为黑色**

此时满足红黑树的定义, 结束调整.

**当前节点的父节点为红色**

此时出现了相邻的红色, 违反红黑树的定义. 因为原来是一颗红黑树, 所以父节点必有其父节点, 且为黑色, 此时需要根据叔节点的颜色进行判断(叔节点可能为 NULL 黑色节点).

如果叔节点为黑色, 需要进行旋转调整.

如果叔节点(下图节点 5)为红色, 将父节点与叔节点置为黑色, 祖父节点(节点 4)置为红色. 然后令祖父节点为当前节点进入下一次循环进行调整, 这里的例子中, 在下一次循环中因节点 4 为根节点, 所以置为黑色并结束调整.

![1725200502724](/img/DataStruct-Find/1725200502724.png)

#### 红黑树删除



### B树

### B+树

## 散列结构查找

### 散列表
