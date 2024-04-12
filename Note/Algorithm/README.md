---
description: 排序算法
---

# SortAlgorithm

> 稳定的排序
>
> * 冒泡排序（Bubble Sort）&#x20;
> * 插入排序（Insertion Sort）
> * 归并排序（Merge Sort）&#x20;
> * 计数排序（Counting Sort）&#x20;
> * 桶排序（Bucket Sort）
> * 基数排序（Radix Sort）

## 冒泡排序算法实现

```java
public void bubbleSort(int[] nums) {
    if (nums == null || nums.length <= 1) {
        return;
    }

    // 每一趟冒泡可使待排序序列中最大的元素上浮至正确位置，故需进行 n-1 趟
    for (int i = 0; i < nums.length - 1; i++) { 
        // 用于判断是否已经是有序序列
        boolean isSorted = true;

        // 在每一趟冒泡中，通过逐一比较相邻记录的关键字，可使关键字最大的记录上浮至正确位置（这里为 nums.length-1-i）
        for (int j = 0; j < nums.length - 1 - i; j++) { 
            // 当前一位的数字大于后一位的数字
            if (nums[j] > nums[j + 1]) { 
                // 交换这两位数字
                int temp = nums[j];
                nums[j] = nums[j + 1];
                nums[j + 1] = temp;
                // 出现元素交换，说明序列中仍存在逆序元素，故将标志位设为 false
                isSorted = false;
            }
        }

        // 若在一趟冒泡中未发生元素交换，则说明序列已变为有序，故可直接返回
        if (isSorted) {
            return;
        }
    }
}
```

## 选择排序算法实现

```java
public void selectionSort(int[] nums) {
    if (nums == null || nums.length <= 1) {
        return;
    }

    // 遍历整个数组
    for (int i = 0; i < nums.length - 1; i++) { 
        // 记录最小元素的索引
        int minIndex = i; 

        // 从待排序序列（即，位置 i 后的序列）中找出最小元素
        for (int j = i + 1; j < nums.length; j++) { 
            // 记录最小元素的位置
            if (nums[j] < nums[minIndex]) { 
                minIndex = j;
            }
        }

        // 如果最小的元素不在前端（位置 i），则将最小元素换到前端
        if (minIndex != i) { 
            int temp = nums[i];
            nums[i] = nums[minIndex];
            nums[minIndex] = temp;
        }
    }
}
```

## 插入排序算法实现

```java
public int[] sortArray(int[] nums) {
    // 从数组的第二个元素开始循环向后遍历
    for (int i = 1; i < nums.length; i++) {
        // 初始化已排序部分的最后一个位置
        int j = i;
        // 如果当前元素小于前一个元素
        while (j > 0 && nums[j] < nums[j - 1]) {
            // 调用swap函数来交换这两个元素的位置
            swap(nums, j, j - 1);
            // 继续向前查看，保证插入后的数组仍然有序
            j--;
        }
    }
    // 返回排序后的数组
    return nums;
}

// 交换数组中的两个元素
private void swap(int[] nums, int i, int j) {
    // 临时变量用于存储nums[i]的值
    int temp = nums[i];
    // 将nums[j]的值赋值给nums[i]
    nums[i] = nums[j];
    // 将临时变量的值赋值给nums[j]
    nums[j] = temp;
}
```

## 希尔排序算法实现

```java
public void shellSort(int[] nums) {
    if (nums == null || nums.length <= 1) {
        return;
    }

    int n = nums.length;
    // 设定初始增量
    int gap = n / 2;

    while (gap > 0) {
        for (int i = gap; i < n; i++) {
            // 此处相当于一个插入排序
            int j = i;
            while (j >= gap && nums[j - gap] > nums[j]) {
                int temp = nums[j - gap];
                nums[j - gap] = nums[j];
                nums[j] = temp;
                j -= gap;
            }
        }
        // 缩减增量
        gap = gap / 2;
    }
}
```

## 归并排序算法实现

```java
//分治算法-归并排序,带注释
public int[] sortArray(int[] nums) {
    //创建一个临时数组，用于存放排序后的结果
    int[] temp = new int[nums.length];
    //调用归并排序方法
    mergeSort(nums, 0, nums.length - 1, temp);
    return nums;
}

//归并排序
private void mergeSort(int[] nums, int left, int right, int[] temp) {
    //递归结束条件
    if (left >= right) {
        return;
    }
    //计算中间位置
    int mid = left + (right - left) / 2;
    //递归左边
    mergeSort(nums, left, mid, temp);
    //递归右边
    mergeSort(nums, mid + 1, right, temp);
    //合并左右两边
    merge(nums, left, mid, right, temp);
}

//合并左右两边
private void merge(int[] nums, int left, int mid, int right, int[] temp) {
    //左边数组的起始位置
    int i = left;
    //右边数组的起始位置
    int j = mid + 1;
    //临时数组的起始位置
    int t = 0;
    //当左右两边都有元素时
    while (i <= mid && j <= right) {
        //比较左右两边的元素，将较小的元素放入临时数组
        if (nums[i] <= nums[j]) {
            temp[t++] = nums[i++];
        } else {
            temp[t++] = nums[j++];
        }
    }
    //将左边剩余的元素放入临时数组
    while (i <= mid) {
        temp[t++] = nums[i++];
    }
    //将右边剩余的元素放入临时数组
    while (j <= right) {
        temp[t++] = nums[j++];
    }
    //将临时数组的元素复制到原数组
    t = 0;
    while (left <= right) {
        nums[left++] = temp[t++];
    }
}
```

## 快速排序算法实现

```java
public void quickSort(int[] nums) {
    if (nums == null || nums.length == 0) {
        return;
    }
    quickSort(nums, 0, nums.length - 1);
}

private void quickSort(int[] nums, int left, int right) {
    if (left < right) {
        // 划分数组，返回分界点的索引
        int pivotIndex = partition(nums, left, right);
        // 对分界点左边的子数组进行快速排序
        quickSort(nums, left, pivotIndex - 1);
        // 对分界点右边的子数组进行快速排序
        quickSort(nums, pivotIndex + 1, right);
    }
}

private int partition(int[] nums, int left, int right) {
    // 选择最后一个元素作为基准元素
    int pivot = nums[right];
    int i = left - 1; // i 是小于等于基准元素的区域的右边界
    // 遍历区间 [left, right-1]
    for (int j = left; j < right; j++) {
        // 如果当前元素小于等于基准元素，则将其交换到小于等于基准元素的区域
        if (nums[j] <= pivot) {
            i++;
            swap(nums, i, j);
        }
    }
    // 将基准元素交换到正确的位置
    swap(nums, i + 1, right);
    // 返回基准元素的最终位置
    return i + 1;
}

private void swap(int[] nums, int i, int j) {
    int temp = nums[i];
    nums[i] = nums[j];
    nums[j] = temp;
}

public static void main(String[] args) {
    int[] nums = {5, 3, 8, 6, 2, 7, 1, 4};
    Solution sorter = new Solution();
    sorter.quickSort(nums);
    System.out.println("Sorted array: " + Arrays.toString(nums));
}
```

## 堆排序算法实现

```java
public int[] sortArray(int[] nums) {
    //堆排序，不使用PriorityQueue
    //1.建立大顶堆
    for (int i = nums.length / 2 - 1; i >= 0; i--) {
        adjustHeap(nums, i, nums.length);
    }
    //2.交换堆顶元素和末尾元素，重新调整大顶堆
    for (int i = nums.length - 1; i > 0; i--) {
        swap(nums, 0, i);
        adjustHeap(nums, 0, i);
    }
    return nums;
}

private void adjustHeap(int[] nums, int i, int length) {
    int temp = nums[i];
    //从i结点的左子结点开始，也就是2i+1处开始
    for (int k = 2 * i + 1; k < length; k = 2 * k + 1) {
        //如果左子结点小于右子结点，k指向右子结点
        if (k + 1 < length && nums[k] < nums[k + 1]) {
            k++;
        }
        //如果子节点大于父节点，将子节点值赋给父节点（不用进行交换）
        if (nums[k] > temp) {
            nums[i] = nums[k];
            i = k;
        } else {
            break;
        }
    }
    //将temp值放到最终的位置
    nums[i] = temp;
}

private void swap(int[] nums, int i, int j) {
    int temp = nums[j];
    nums[j] = nums[i];
    nums[i] = temp;

}
```

