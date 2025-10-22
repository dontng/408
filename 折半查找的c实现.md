408 考研对于**折半查找（Binary Search）的考察，重点在于前提条件**、**边界处理**和**效率分析**。

折半查找**必须**应用于**有序的顺序表**。

我们仍然沿用上一题封装的`SqList`结构体（假设数据从`data[1]`开始存储，`data[0]`保留）。

对于折半查找，有两种C语言实现：**非递归（迭代）和递归**。

**重点：非递归版本是考察的重中之重**，因为它效率更高（没有函数调用的额外开销），也是算法题中默认要求的写法。

------



### 1. 非递归实现 (Iterative Version) - 408重点



这是最标准、最高效的实现。你需要牢牢掌握它的三个边界细节。

C

```
#include <stdio.h>

// 假设 SqList 结构体已定义
// typedef struct {
//     int data[MAX_SIZE + 1]; // data[0] 不用或作哨兵
//     int length;
// } SqList;

/*
 * 折半查找 - 非递归版本 (408标准写法)
 * L: 指向有序顺序表的指针 (假设元素从 data[1] 开始存储)
 * key: 待查找的关键字
 * 返回: 找到则返回其在数组中的“下标”(1...length)，找不到返回 0
 */
int Binary_Search(SqList *L, int key) {
    int low, high, mid;
    
    // 细节1: 初始化边界
    low = 1;          // 最小下标
    high = L->length; // 最大下标

    // 细节2: 循环条件 (low <= high)
    // 必须带等号，否则当 low == high (剩最后一个元素)时会退出，导致漏判
    while (low <= high) {
        
        // 细节3: 计算 mid (防溢出写法)
        // (low + high) / 2 在 low 和 high 极大时有整数溢出风险
        mid = low + (high - low) / 2; 

        if (L->data[mid] == key) {
            return mid; // 查找成功
        } 
        else if (key < L->data[mid]) {
            // key 在左半区，[low ... mid-1]
            high = mid - 1; // 关键：mid 已经被排除了
        } 
        else { // key > L->data[mid]
            // key 在右半区，[mid+1 ... high]
            low = mid + 1;  // 关键：mid 已经被排除了
        }
    }

    return 0; // 查找失败 (循环结束 low > high)
}
```

**408 考点（非递归版）：**

1. **为什么 `while (low <= high)`？**
   - 答：折半查找的循环不变式是“在 `[low, high]` 闭区间内查找 `key`”。如果 `low == high`，这个区间 `[low, low]` 仍然有一个元素，需要检查。如果此时退出，就漏掉了这最后一次比较。
2. **为什么是 `high = mid - 1` 和 `low = mid + 1`？**
   - 答：因为 `L->data[mid] == key` 已经在循环中判断过了。既然不相等，那么 `mid` 这个位置的元素一定不是 `key`，必须将其**排除**在下一次的查找区间之外。如果写成 `high = mid`，可能会导致死循环（例如当 `low` 和 `high` 相邻时）。
3. **`mid = low + (high - low) / 2`**
   - 这是408考察C语言基础时的一个加分项，它等价于 `(low + high) / 2`，但能有效防止 `low` 和 `high` 相加时超过 `int` 的最大值而溢出。

------



### 2. 递归实现 (Recursive Version)



递归版本的逻辑更清晰，易于理解，但实际运行效率较低（有函数调用开销）。408主要考察你是否能写出递归的终止条件和递归体。

C

```
/*
 * 递归查找的辅助函数
 */
int Binary_Search_Recursive(SqList *L, int key, int low, int high) {
    // 递归终止条件1: 查找失败
    if (low > high) {
        return 0; 
    }

    int mid = low + (high - low) / 2;

    // 递归终止条件2: 查找成功
    if (key == L->data[mid]) {
        return mid;
    }
    // 递归体1: 向左半区查找
    else if (key < L->data[mid]) {
        return Binary_Search_Recursive(L, key, low, mid - 1);
    }
    // 递归体2: 向右半区查找
    else {
        return Binary_Search_Recursive(L, key, mid + 1, high);
    }
}

/*
 * 折半查找 - 递归版本 (入口函数)
 * (这个函数只是一个“壳”，方便调用)
 */
int Search_Bin_Rec_Wrapper(SqList *L, int key) {
    return Binary_Search_Recursive(L, key, 1, L->length);
}
```

------



### 总结：408 核心考察点



1. **前提条件（选择题）**：折半查找的前提是 **顺序存储** 且 **元素有序**。
2. **适用性（选择题）**：它不适用于链表（因为链表无法 $O(1)$ 随机访问`mid`元素）或无序表。
3. **时间复杂度（大题、选择题）**：$O(\log_2 n)$。
4. **空间复杂度（选择题）**：非递归版 $O(1)$；递归版 $O(\log_2 n)$（因为递归栈的深度）。
5. **判定树（大题）**：折半查找的过程对应一棵“判定树”（平衡二叉排序树）。查找失败时的比较次数，就是判定树中对应空指针（失败节点）的层数。查找成功时的比较次数，就是对应节点的层数。