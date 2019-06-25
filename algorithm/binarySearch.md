# 二分查找

## 时间复杂度

- 最快O(n)
- 最差O(log2 n）

## 算法描述

!> 二分查找需要一个有序的集合，以下为**递增集合**

```java
 Integer binarySearch(int[] array, int item) {
        int start = 0;
        int end = array.length;
        while (start <= end) {
            int mid = (start + end) / 2;
            int guess = array[mid];
            if (guess == item) {
                return mid;
            }

            if (guess > item) {
            		//猜大了
                end = mid - 1; 
            } else {
            		//猜小了
                start = mid + 1;
            }

        }
        return null;
    }
```