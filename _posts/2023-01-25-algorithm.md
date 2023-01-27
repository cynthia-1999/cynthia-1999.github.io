---
title: 二分法的优雅写法
date: 2022-05-02 22:22:22
categories:
- 算法
tags:
- 算法
---

### 思路

二分查找也称折半查找（Binary Search），它是一种效率较高的查找方法。但是，折半查找要求线性表必须采用顺序存储结构，而且表中元素按关键字有序排列。

就是这样一种非常常见的算法，最关键的结构就是循环和判断，但是如果书写不当，很容易导致死循环。

在二分法的循环结构中，需要给出一个边界判断条件，但是我们可以看到很多不同的写法，它可以是`while(start < end)`, `while(start <= end)`, `while(start + 1 <= end)`，如果我们不清楚不同边界判断条件对应的循环体的区别，我们将很容易出错，写出死循环。

### <和<=的区别

我们常常会写出这两种写法，这两种写法的循环体内逻辑一致，在循环体中，`mid = start + (end - start)/2`, 当`nums[mid] == target`时返回，否则，根据target和nums[mid]的大小关系，变化start和end。

在这两种写法中，我们可以把start和end看成一个区间，这两种写法对应的区间分别为[start, end)和[start, end]，在循环内部，我们根据区间的定义来调整start和end。

```C++
// < 的写法
class Solution {
public:
    int search(vector<int>& nums, int target) {
      // 定义target在左闭右开的区间里，[start, end)
      int start = 0, end = nums.size();
      while(start < end){	// 当start == end时，区间[start, end)无效，所以用 < 
        int mid = start + (end - start) / 2;
        if(nums[mid] > target){
          end = mid;	//target在左区间，因为end无效，所以end=mid
				}
        else if(nums[mid] < target){
          start = mid + 1;	//target在右区间，因为start有效，所以start=mid+1
        }
        else return mid;	//数组中找到目标值
      }
      return -1;	//未找到目标值
    }
};

```



```C++
// <= 的写法
class Solution {
public:
    int search(vector<int>& nums, int target) {
        int start = 0;
        int end = nums.size() - 1; // 定义target在左闭右闭的区间里，[start, end]
        while (start <= end) { // 当start==end，区间[start, end]依然有效，所以用 <=
            int mid = start + ((end - start) / 2);// 防止溢出 等同于(start + end)/2
            if (nums[mid] > target) {
                end = mid - 1; // target 在左区间，所以[start, mid - 1]
            } else if (nums[mid] < target) {
                start = mid + 1; // target 在右区间，所以[mid + 1, end]
            } else { // nums[mid] == target
                return mid; // 数组中找到目标值，直接返回下标
            }
        }
        // 未找到目标值
        return -1;
    }
};

```

在上述两种方法中，循环内部的判断逻辑基本一致，主要的差别在于end的变化规则，如果end总是有效的（闭区间），则end = mid - 1，否则end = mid，如果没有想清楚这一点，是很容易出错的。

此外，我们可以在循环之前再加一些边界判断条件，可以避免一些无效的操作。

```C++
if(nums[start] > target || nums[end] < target){
  return -1;
}
```

### 不会发生死循环的写法

在前面两种写法中，如果不能理清楚区间边界变化规则，很容易写出死循环的代码。

什么时候会发生死循环？

> 考虑一种极端情况：nums = {-1, -1}，target = -2
>
> 当start = 0，end = 1时，midß = 1，target < nums[mid]，这时end应该变化为mid-1，如果误写成mid，就会发生死循环，这样发生的前提条件是mid跟start或者end之一重合，如果条件写的不对，就一定会发生死循环。

如果能够防止mid与start或end之一重合，就可以避免发生死循环。

如果我们将判断条件修改为start + 1  < end，就可以满足mid永远不会与start和end之一重合。当start + 1 = end时，就会跳出循环，但是这样在循环内部可能取不到nums的某些值，所以我们要将返回的判断条件放在循环之外，并且**在循环内部将<,==,>的情况分开写，然后根据情况进行合并，可以最大限度减少bug的出现**。

```C++
// start + 1 < end的写法
class Solution {
public:
    int search(vector<int>& nums, int target) {
        int start = 0;
        int end = nums.size() - 1; // 定义target在左闭右闭的区间里，[start, end]
        while (start +1 <= end) { 
            int mid = start + ((end - start) / 2);
            if(nums[mid] >= target){
              start = mid;
            }
          	else{
              end = mid-1;
            }
        }
      	// start == end时跳出循环
      	if(nums[start] == target){
          return start;
        }
      	else if(nums[end] == target){
          return end;
        }
        // 未找到目标值
        return -1;
    }
};
```

