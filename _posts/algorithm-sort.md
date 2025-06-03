---
name: sort
title: 排序算法
date: 2025-05-13
tags:
categories: 算法
---

# Serial Sort Algorithm

## HEAP SORT

```java
/* 时间复杂度:
   构建堆：O(n)
   每次调整堆：O(log n)
   总时间复杂度：O(n log n)
*/

// 空间复杂度
// O(1)（原地排序）

// 稳定性
// 不稳定：交换操作可能破坏相同元素的顺序。
public class HeapSort {
    public static void heapSort(int[] arr) {
        int n = arr.length;

        // 1. 构建最大堆（从最后一个非叶子节点开始调整）
        for (int i = n / 2 - 1; i >= 0; i--) {
            heapify(arr, n, i); // 将无序数组调整为满足父节点 ≥ 子节点的堆结构。
        }

        // 2. 逐步将堆顶元素与末尾交换，并调整堆，重复此过程直到堆的大小为 1。
        for (int i = n - 1; i > 0; i--) {
            // 交换堆顶和当前未排序部分的末尾
            swap(arr, 0, i);
            // 调整堆顶元素，使其下沉到正确位置
            heapify(arr, i, 0);
        }
    }

    // 对以 i 为根的子树执行下沉操作（确保最大堆性质）
    private static void heapify(int[] arr, int n, int i) {
        int largest = i;       // 初始化最大值的索引为根节点
        int left = 2 * i + 1;  // 左子节点索引
        int right = 2 * i + 2; // 右子节点索引

        // 如果左子节点存在且大于当前最大值
        if (left < n && arr[left] > arr[largest]) {
            largest = left;
        }

        // 如果右子节点存在且大于当前最大值
        if (right < n && arr[right] > arr[largest]) {
            largest = right;
        }

        // 如果最大值不在根节点，则交换并递归调整子树
        if (largest != i) {
            swap(arr, i, largest);
            heapify(arr, n, largest);
        }
    }

    private static void swap(int[] arr, int i, int j) {
        int temp = arr[i];
        arr[i] = arr[j];
        arr[j] = temp;
    }

    public static void main(String[] args) {
        int[] arr = {5, 3, 8, 4, 2, 7, 1, 6};
        heapSort(arr);
        System.out.println(Arrays.toString(arr)); // [1, 2, 3, 4, 5, 6, 7, 8]
    }
}
```

## MERGE SORT(DIVISION)

```java
// 分治法：将数组递归分成两半，分别排序后合并。
// 时间复杂度 O(n log n) 空间复杂度 O(n)（合并时需要辅助数组）稳定 
public class MergeSort {
    public static void mergeSort(int[] arr) {
        if (arr.length <= 1) return;
        int mid = arr.length / 2;
        int[] left = Arrays.copyOfRange(arr, 0, mid);
        int[] right = Arrays.copyOfRange(arr, mid, arr.length);
        mergeSort(left);
        mergeSort(right);
        merge(arr, left, right);
    }

    private static void merge(int[] arr, int[] left, int[] right) {
        int i = 0, j = 0, k = 0;
        while (i < left.length && j < right.length) {
            if (left[i] <= right[j]) {
                arr[k++] = left[i++];
            } else {
                arr[k++] = right[j++];
            }
        }
        // 处理剩余元素
        while (i < left.length) arr[k++] = left[i++];
        while (j < right.length) arr[k++] = right[j++];
    }

    public static void main(String[] args) {
        int[] arr = {5, 3, 8, 4, 2, 7, 1, 6};
        mergeSort(arr);
        System.out.println(Arrays.toString(arr)); // [1, 2, 3, 4, 5, 6, 7, 8]
    }
}
```

## QUICK SORT(DIVISION)


```java
/** 分治法：选择一个基准元素（pivot），将数组分为两部分，左边元素 ≤ 基准，右边元素 ≥ 基准，递归处理子数组。
    关键步骤：分区（Partition） 操作。
    时间复杂度：平均：O(n log n)   最坏（如已排序数组）：O(n²)（可通过随机选择基准优化）
    空间复杂度：O(log n)（递归栈）
    稳定性：不稳定（交换可能破坏顺序）**/
public class QuickSort {
    public static void quickSort(int[] arr, int low, int high) {
        if (low < high) {
            int pivotIndex = partition(arr, low, high);
            quickSort(arr, low, pivotIndex - 1);  // 递归左半部分
            quickSort(arr, pivotIndex + 1, high); // 递归右半部分
        }
    }

    private static int partition(int[] arr, int low, int high) {
        int pivot = arr[(low + high) / 2]; // 选择中间元素为基准（可优化为随机选择）
        int i = low, j = high;
        while (i <= j) {
            while (arr[i] < pivot) i++; // 找左边大于基准的元素
            while (arr[j] > pivot) j--; // 找右边小于基准的元素
            if (i <= j) {
                swap(arr, i, j);
                i++;
                j--;
            }
        }
        return i; // 返回分区点
    }

    private static void swap(int[] arr, int i, int j) {
        int temp = arr[i];
        arr[i] = arr[j];
        arr[j] = temp;
    }

    public static void main(String[] args) {
        int[] arr = {5, 3, 8, 4, 2, 7, 1, 6};
        quickSort(arr, 0, arr.length - 1);
        System.out.println(Arrays.toString(arr)); // [1, 2, 3, 4, 5, 6, 7, 8]
    }
}
```

## RADIX SORT

```java
// 非比较排序：按数字的每一位（从低位到高位）进行排序，每次使用稳定的子排序（如计数排序）。
// 时间复杂度 O(d(n+k))  d 是数字的最大位数，k 是基数（例如，对于十进制数，k=10）
// 空间复杂度 O(n+k) 额外空间用于存储计数和输出数组
// 稳定 适用于整数或定长字符串排序，位数较少时高效

public class RadixSort {
    public static void radixSort(int[] arr) {
        int max = Arrays.stream(arr).max().getAsInt();
        int exp = 1; // 当前处理的位数（个位开始）

        // 从低位到高位依次排序
        while (max / exp > 0) {
            countingSortByDigit(arr, exp);
            exp *= 10;
        }
    }

    private static void countingSortByDigit(int[] arr, int exp) {
        int[] output = new int[arr.length];
        int[] count = new int[10]; // 0-9的计数

        // 统计每个数字出现的次数
        for (int num : arr) {
            int digit = (num / exp) % 10;
            count[digit]++;
        }

        // 计算累计位置
        for (int i = 1; i < 10; i++) {
            count[i] += count[i - 1];
        }

        // 从后向前填充output数组（保证稳定性）
        for (int i = arr.length - 1; i >= 0; i--) {
            int digit = (arr[i] / exp) % 10;
            output[count[digit] - 1] = arr[i];
            count[digit]--;
        }

        // 将排序结果复制回原数组
        System.arraycopy(output, 0, arr, 0, arr.length);
    }

    public static void main(String[] args) {
        int[] arr = {170, 45, 75, 90, 802, 24, 2, 66};
        radixSort(arr);
        System.out.println(Arrays.toString(arr)); // [2, 24, 45, 66, 75, 90, 170, 802]
    }
}
```

## SELECTION SORT


```java
// 时间复杂度 O(n^2) 空间复杂度 O(1) 不稳定
public class SelectionSort {
    public static void selectionSort(int[] arr) {
        for (int i = 0; i < arr.length - 1; i++) {
            int minIndex = i;
            // 找未排序部分的最小值索引
            for (int j = i + 1; j < arr.length; j++) {
                if (arr[j] < arr[minIndex]) {
                    minIndex = j;
                }
            }
            // 交换最小值到正确位置
            int temp = arr[i];
            arr[i] = arr[minIndex];
            arr[minIndex] = temp;
        }
    }

    public static void main(String[] args) {
        int[] arr = {5, 3, 8, 4, 2, 7, 1, 6};
        selectionSort(arr);
        System.out.println(Arrays.toString(arr)); // [1, 2, 3, 4, 5, 6, 7, 8]
    }
}
```

## INSERTION SORT

```java
// 时间复杂度 最好-有序 O(n) 平均 O(n^2) 最坏 O(n^2) 空间复杂度 O(1) 稳定
// 插入排序：逐步构建有序序列，将未排序元素逐个插入到已排序部分的正确位置。
public class InsertionSort {
    public static void insertionSort(int[] arr) {
        for (int i = 1; i < arr.length; i++) {
            int key = arr[i];
            int j = i - 1;
            // 将比key大的元素后移
            while (j >= 0 && arr[j] > key) {
                arr[j + 1] = arr[j];
                j--;
            }
            arr[j + 1] = key; // 插入到正确位置
        }
    }

    public static void main(String[] args) {
        int[] arr = {5, 3, 8, 4, 2, 7, 1, 6};
        insertionSort(arr);
        System.out.println(Arrays.toString(arr)); // [1, 2, 3, 4, 5, 6, 7, 8]
    }
}

```

## SHELL SORT(OPTIMIZED INSERTION)

```java
// 插入排序的变种：通过分组插入排序逐步减少间隔（gap），最终进行一次标准插入排序。
// 关键步骤：动态调整间隔序列（如 Knuth 序列）
// 时间复杂度取决于间隔序列，通常为 O(n log² n) ~ O(n²)。 最优可达到 O(n log n)。
// 空间复杂度 O(1) 
// 不稳定（分组交换可能破坏顺序）
public class ShellSort {
    public static void shellSort(int[] arr) {
        int n = arr.length;
        // 初始间隔使用 Knuth 序列（h = 3*h + 1）
        int gap = 1;
        while (gap < n / 3) gap = 3 * gap + 1;

        while (gap >= 1) {
            // 对每个间隔分组进行插入排序
            for (int i = gap; i < n; i++) {
                int temp = arr[i];
                int j = i;
                while (j >= gap && arr[j - gap] > temp) {
                    arr[j] = arr[j - gap];
                    j -= gap;
                }
                arr[j] = temp;
            }
            gap /= 3; // 缩小间隔
        }
    }

    public static void main(String[] args) {
        int[] arr = {5, 3, 8, 4, 2, 7, 1, 6};
        shellSort(arr);
        System.out.println(Arrays.toString(arr)); // [1, 2, 3, 4, 5, 6, 7, 8]
    }
}
```

# [Parallel Sort Algorithm](https://scatteredream.github.io/2025/04/14/cpp_cuda_parallel_programming/#%E5%B9%B6%E8%A1%8C%E7%AE%97%E6%B3%95)



