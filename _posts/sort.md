---
title: 一些常见的排序算法
date: 2018-09-23 21:24:26
tags: sort
---

### 冒泡排序

冒泡排序比较简单，就是如同水泡一样不断将最大或者最小的元素放置到前面。

冒泡排序算法的原理如下：

1. 比较相邻的元素。如果第一个比第二个大，就交换他们两个。
2. 对每一对相邻元素做同样的工作，从开始第一对到结尾的最后一对。在这一点，最后的元素应该会是最大的数。
3. 针对所有的元素重复以上的步骤，除了最后一个。
4. 持续每次对越来越少的元素重复上面的步骤，直到没有任何一对数字需要比较。

算法稳定性:

冒泡排序就是把小的元素往前调或者把大的元素往后调。比较是相邻的两个元素比较，交换也发生在这两个元素之间。所以，如果两个元素相等，是不会再交换的；如果两个相等的元素没有相邻，那么即使通过前面的两两交换把两个相邻起来，这时候也不会交换，所以相同元素的前后顺序并没有改变，所以冒泡排序是一种稳定排序算法。

平均时间复杂度：O(n^2)

冒泡排序代码示例：
```javascript
let arr = [10,2,5,8,3,2,88,99,101,-3,-2,1]
function BubleSort (arr) {
  let copyArr = arr.slice()
  let i = length = copyArr.length, j = 0
  while (j < length) {
    while (i > j) {
      if (copyArr[i] < copyArr[i - 1]) {
        let temp = copyArr[i]
        copyArr[i] = copyArr[i - 1]
        copyArr[i - 1] = temp
      }
      i--
    }
    j++
    i = length
  }
  return copyArr
}
console.log(BubbleSort(copyArr)) // [ -3, -2, 1, 2, 2, 3, 5, 8, 10, 88, 99, 101 ]
```

### 选择排序

选择排序的思想其实和冒泡排序有点类似，都是在一次排序后把最小或者最大的元素放到最前面。但是过程不同，冒泡排序是通过相邻的比较和交换。而选择排序是通过对整体的选择。

算法稳定性：

选择排序是给每个位置选择当前元素最小的，比如给第一个位置选择最小的，在剩余元素里面给第二个元素选择第二小的，依次类推，直到第n-1个元素，第n个元素不用选择了，因为只剩下它一个最大的元素了。那么，在一趟选择，如果一个元素比当前元素小，而该小的元素又出现在一个和当前元素相等的元素后面，那么交换后稳定性就被破坏了。比较拗口，举个例子，序列5 8 5 2 9，我们知道第一遍选择第1个元素5会和2交换，那么原序列中两个5的相对前后顺序就被破坏了，所以选择排序是一个不稳定的排序算法。

平均时间复杂度：O(n^2)

选择排序示例：

```javascript
let arr = [10,2,5,8,3,2,88,99,101,-3,-2,1]
function SelectSort (arr) {
  let copyArr = arr.slice()
  let i = 0, j = 1, length = copyArr.length, l
  for (; i < length - 1; i++, j = i + 1) {
    l = i
    for (;j < length; j++) {
      if (copyArr[j] < copyArr[l]) {
        l = j
      }
    }
    if (l !== i) {
      let temp = copyArr[i]
      copyArr[i] = copyArr[l]
      copyArr[l] = temp
    }
  }
  return copyArr
}
console.log(SelectSort(copyArr)) // [ -3, -2, 1, 2, 2, 3, 5, 8, 10, 88, 99, 101 ]
```

### 插入排序

插入排序的基本操作就是将一个数据插入到已经排好序的有序数据中，从而得到一个新的、个数加一的有序数据，算法适用于少量数据的排序。

插入排序又分直接插入排序和二分插入排序。后面分类举例

算法稳定性：

插入排序是在一个已经有序的小序列的基础上，一次插入一个元素。当然，刚开始这个有序的小序列只有1个元素，就是第一个元素。比较是从有序序列的末尾开始，也就是想要插入的元素和已经有序的最大者开始比起，如果比它大则直接插入在其后面，否则一直往前找直到找到它该插入的位置。如果碰见一个和插入元素相等的，那么插入元素把想插入的元素放在相等元素的后面。所以，相等元素的前后顺序没有改变，从原无序序列出去的顺序就是排好序后的顺序，所以插入排序是稳定的。

平均时间复杂度：O(n^2)
 
直接插入排序示例：

```javascript
// 直接插入排序是一种简单的插入排序法，其基本思想是：把待排序的元素按大小逐个插入到一个已经排好序的有序序列中，直到所有的记录插入完为止，得到一个新的有序序列
let arr = [10,2,5,8,3,2,88,99,101,-3,-2,1]
function DirectInsertSort (arr) {
  // needSortArr 待排序数组，sortedArr 已排序数组
  let needSortArr = arr.slice(), length = arr.length
  let sortedArr = needSortArr.splice(0, 1)
  while (sortedArr.length < length) {
    let insertIndex = sortedArr.length - 1
    let tempValue = needSortArr.splice(0, 1)[0]
    while (insertIndex >= 0) {
      if (tempValue >= sortedArr[insertIndex]) {
        break
      }
      insertIndex--
    }
    sortedArr.splice(insertIndex + 1, 0, tempValue)
  }
  return sortedArr
}
console.log(DirectInsertSort(arr)) // [ -3, -2, 1, 2, 2, 3, 5, 8, 10, 88, 99, 101 ]
```

折半插入排序（二分插入排序）示例：

```javascript
let arr = [10,2,8,5,3,2,88,99,101,-3,-2,1]
function HalfInsertSort (arr) {
  let needSortArr = arr.slice(), length = arr.length
  let sortedArr = needSortArr.splice(0, 1)
  while (sortedArr.length < length) {
    let insertIndex = sortedArr.length
    let tempValue = needSortArr.splice(0, 1)[0]
    // 二分查找 需要插入的位置
    while (true) {
      if (insertIndex === 0) {
        if (tempValue > sortedArr[0]) {
          insertIndex = 1
        }
        break
      } else {
        if (sortedArr[insertIndex - 1] > tempValue) {
          insertIndex = Math.floor(insertIndex / 2)
          continue
        } else if (sortedArr[insertIndex] < tempValue) {
          insertIndex += insertIndex / 2
        } else {
          break
        }
      }
    }
    sortedArr.splice(insertIndex, 0, tempValue)
  }
  return sortedArr
}
console.log(HalfInsertSort(arr)) // [ -3, -2, 1, 2, 2, 3, 5, 8, 10, 88, 99, 101 ]
```

### 快速排序

通过一趟排序将要排序的数据分割成独立的两部分，其中一部分的所有数据都比另外一部分的所有数据都要小，然后再按此方法对这两部分数据分别进行快速排序，整个排序过程可以递归进行，以此达到整个数据变成有序序列。

稳定性：快速排序不是一种稳定的排序算法，也就是说，多个相同的值的相对位置也许会在算法结束时产生变动。

时间复杂度：O(nlogn)

代码示例
```javascript
let arr = [10,2,8,5,3,2,88,99,101,-3,-2,1]
function QuickSort (arr) {
  let copyArr = arr.slice()
  function PartSort (arrArg) {
    let arr = arrArg.slice()
    let key = arr[0]
    let i = 0, j = arr.length - 1
    if (arr.length === 1 || arr.length === 0) {
      return arr
    }
    do {
      if (arr[j] < key) {
        arr[i] = arr[j]
        arr[j] = key
        do {
          if (arr[i] > key) {
            arr[j] = arr[i]
            arr[i] = key
            break
          }
          i++
        } while (i !== j)
      }
      j--
    } while (j >= i);
    return [...PartSort(arr.slice(0, i+1)), ...PartSort(arr.slice(i+1, arr.length))]
  }
  return PartSort(copyArr)
}
console.log(QuickSort(arr)) // [ -3, -2, 1, 2, 2, 3, 5, 8, 10, 88, 99, 101 ]
```
### 堆排序

堆排序(Heapsort)是指利用堆积树（堆）这种数据结构所设计的一种排序算法，它是选择排序的一种。可以利用数组的特点快速定位指定索引的元素。堆分为顶堆和小顶堆，是完全二叉树。大根堆的要求是每个节点的值都不大于其父节点的值，即A[PARENT[i]] >= A[i]。在数组的非降序排序中，需要使用的就是大顶堆，因为根据大顶堆的要求可知，最大的值一定在堆顶，反之，则使用小顶堆。

算法稳定性：不稳定

时间复杂度：O(nlogn)

> 由于建初始堆所需的比较次数较多，所以堆排序不适宜于元素数较少的数组

在进行算法时，有两个关键性问题：1、 怎么表示一个大顶堆，2、怎么调整数组元素成为一个合格的堆

代码示例
```javascript
// 对数组所有元素操作，是数组成为一个大顶堆
Array.prototype.buildMaxHeap = function () {
  for (let i = Math.floor(this.length / 2) - 1; i >= 0; i--) {
    this.heapAdjust(i, this.length)
  }
}
// 交换元素
Array.prototype.swap = function (i, j) {
  let tmp = this[i]
  this[i] = this[j]
  this[j] = tmp
}
// 排序
Array.prototype.heapSort = function () {
  this.buildMaxHeap()
  for (let i = this.length - 1; i > 0; i--) {
    this.swap(0, i)
    this.heapAdjust(0, i)
  }
  return this
}
// 调整一个子二叉树，选择最大的数作为二叉树顶点，之后调整以被交换的子节点作为父节点的二叉树，重复这个操作
Array.prototype.heapAdjust = function (i, j) {
  let largest = i
  let left = 2 * i + 1
  let right = 2 * i + 2

  if (left < j && this[largest] < this[left]) {
    largest = left
  }
  if (right < j && this[largest] < this[right]) {
    largest = right
  }
  if (largest != i) {
    this.swap(i, largest)
    this.heapAdjust(largest, j)
  }
}
let arr = new Array()
arr.push(10, 2, 8, 5, 3, 2, 88, 99, 101, -3, -2, 1)
console.log(arr.heapSort()) // [ -3, -2, 1, 2, 2, 3, 5, 8, 10, 88, 99, 101 ]
```

### 希尔排序

希尔排序(Shell's Sort)是插入排序的一种又称“缩小增量排序”（Diminishing Increment Sort），是直接插入排序算法的一种更高效的改进版本。简单的插入排序中，如果待排序列是正序时，时间复杂度是O(n)，如果序列是基本有序的，使用直接插入排序效率就非常高。希尔排序就利用了这个特点。基本思想是：先将整个待排记录序列分割成为若干子序列分别进行直接插入排序，待整个序列中的记录基本有序时再对全体记录进行一次直接插入排序，具体做法是：先取一个小于n的整数d1作为第一个增量，把文件的全部记录分组。所有距离为d1的倍数的记录放在同一个组中。先在各组内进行直接插入排序；然后，取第二个增量d2 < d1重复上述的分组和排序，直至所取的增量 dt =1( dt < dt-1 … < d2 < d1)，即所有记录放在同一组中进行直接插入排序为止。

算法稳定性： 非稳定排序

时间复杂度：希尔排序的分析是复杂的，时间复杂度是所取增量的函数，这涉及一些数学上的难题。但是在大量实验的基础上推出当n在某个范围内时，时间复杂度可以达到O(n^1.3)

代码示例
```javascript
let arr = [10, 2, 8, 5, 3, 2, 88, 99, 101, -3, -2, 1]
Array.prototype.InsertSort = function (step) {
  for (let i = 0; i < step; i++) {
    let j = i + step
    while (j < this.length) {
      for (let k = j - step; k >= i; k -= step) {
        if (this[k] > this[j]) {
          this.swip(k, j)
          break
        }
      }
      j += step
    }
  }
}
Array.prototype.ShellSort = function () {
  let step = Math.floor(this.length)
  // 当step为1时，是对整个数组进行插入排序，也就是最后的一次排序
  while (step > 0) {
    this.InsertSort(step)
    step = Math.floor(step / 2)
  }
  return this
}
Array.prototype.swip = function (i, j) {
  let temp = this[i]
  this[i] = this[j]
  this[j] = temp
}
console.log(arr.ShellSort()) // [ -3, -2, 1, 2, 8, 5, 3, 2, 10, 88, 99, 101 ]
```

### 归并排序

归并排序（MERGE-SORT）是建立在归并操作上的一种有效的排序算法,该算法是采用分治法（Divide and Conquer）的一个非常典型的应用,速度仅次于快速排序，为稳定排序算法，一般用于对总体无序，但是各子项相对有序的数列.

算法稳定性：稳定

时间复杂度：O(nlogn)

代码示例
```javascript
let arr = [10, 2, 8, 5, 3, 2, 88, 99, 101, -3, -2, 1]
// 合并两个数组
function MergeTwoArray (arrA, arrB) {
  let i = j = 0
  let final = []
  while (arrA.length > 0 && arrB.length > 0) {
    if (arrA[0] <= arrB[0]) {
      final.push(arrA.shift())
    } else {
      final.push(arrB.shift())
    }
  }
  return final.concat(arrA).concat(arrB)
}
function MergeSort (arr) {
  let copyArr = arr.slice()
  for (let i = 1; i < copyArr.length; i *= 2) {
    for (let j = 0; j < copyArr.length; j += i * 2) {
      let arrA = copyArr.slice(j, j + i)
      let arrB = copyArr.slice(j + i, j + 2 * i)
      if (!arrB) {
        arrB
      }
      copyArr.splice(j, i * 2, ...MergeTwoArray(arrA, arrB))
    }
  }
  return copyArr
}
console.log(MergeSort(arr)) // [ -3, -2, 1, 2, 2, 3, 5, 8, 10, 88, 99, 101 ]

// 递归做法，不推荐，容易堆栈爆炸
// 合并两个数组
function merge(left, right) {
  let result = []
  while (left.length > 0 && right.length > 0) {
    if (left[0] < right[0]) {
      /*shift()方法用于把数组的第一个元素从其中删除，并返回第一个元素的值。*/
      result.push(left.shift())
    } else {
      result.push(right.shift())
    }
  }
  return result.concat(left).concat(right)
}
function mergeSort(items) {
  if (items.length == 1) {
    return　items
  }
  let　middle = Math.floor(items.length / 2)
  let left = items.slice(0, middle)
  let right = items.slice(middle)
  return merge(mergeSort(left), mergeSort(right))
}
console.log(mergeSort(arr)) // [ -3, -2, 1, 2, 2, 3, 5, 8, 10, 88, 99, 101 ]
```

### 计数排序

虽然前面基于比较的排序的下限是O(nlogn)，但是计数排序不是比较排序。当输入的元素是n个0到k之间的整数时，它的运行时间O(n+k),快于任何比较排序算法。其基本思想是，用待排序的数作为计数数组的下标，统计每个数字的个数。然后依次输出即可得到有序序列。计数排序对于数据范围很大的数组，需要大量时间和内存。

算法稳定性：稳定

时间复杂度：O(n+k)

```javascript

let arr = [10, 2, 8, 15, 23, 2, 88, 99, 101, 3, 2, 41]
Array.prototype.CountSort = function() {
  let C = []
  for(let i = 0; i < this.length; i++) {
    if (C[this[i]]) {
      C[this[i]]++
    } else {
      C[this[i]] = 1
    }
  }
  let D = []
  for (let j = 0; j < C.length; j++) {
    if (C[j]) {
      while(C[j] > 0) {
        D.push(j)
        C[j]--
      }
    }
  }
  return D
}
console.log(arr.CountSort())
```

### 桶排序

桶排序（Bucket sort）或所谓的箱排序，是一个排序算法，工作的原理是将数组分到有限数量的桶里。每个桶再个别排序（有可能再使用别的排序算法或是以递归方式继续使用桶排序进行排序）。桶排序是鸽巢排序的一种归纳结果。桶排序并不是比较排序，他不受到 {\displaystyle O(n\log n)} {\displaystyle O(n\log n)}下限的影响。

算法稳定性： 稳定

时间复杂度： O(n)

示例代码
```javascript
Array.prototype.swip = function (i, j) {
  let temp = this[i]
  this[i] = this[j]
  this[j] = temp
}
Array.prototype.BucketSort = function(num) {
  let max = Math.max(...this)
  let min = Math.min(...this)
  let bucketSize = Math.floor((max - min) / num)
  let bucket = []
  for (let i = 0; i < this.length; i++) {
    let index = Math.floor(this[i] / bucketSize)
    if (!bucket[index]) {
      bucket[index] = []
    }
    bucket[index].push(this[i])
    let l = bucket[index].length
    while (l > 0) {
      if (bucket[index][l] < bucket[index][l - 1]) {
        bucket[index].swip(l, l - 1)
      }
      l--
    }
  }
  this.splice(0, this.length)
  while(bucket.length > 0) {
    let arr = bucket.shift()
    if (arr) {
      this.push(...arr)
    }
  }
  return this
}
let arr = [10, 2, 8, 15, 23, 2, 88, 99, 101, 3, 2, 41]
console.log(arr.BucketSort(20))
console.log(arr)
```

### 基数排序

基数排序（英语：Radix sort）是一种非比较型整数排序算法，其原理是将整数按位数切割成不同的数字，然后按每个位数分别比较。由于整数也可以表达字符串（比如名字或日期）和特定格式的浮点数，所以基数排序也不是只能使用于整数。

算法稳定性：稳定

时间复杂度：O(k*n),其中n是排序元素个数，k是数字位数

代码示例：
```javascript
Array.prototype.radixSort = function() {
  let arr = this.slice(0)
  const max = Math.max(...arr)
  let digit = `${max}`.length
  let start = 1
  let buckets = []
  while(digit > 0) {
    start *= 10
    for(let i = 0; i < arr.length; i++) {
      const index = arr[i] % start
      !buckets[index] && (buckets[index] = [])
      buckets[index].push(arr[i])
    }
    arr = []
    for(let i = 0; i < buckets.length; i++) {
      buckets[i] && (arr = arr.concat(buckets[i]))
    }
    buckets = []
    digit --
  }
  return arr
}
const arr = [1, 10, 100, 1000, 98, 67, 3, 28, 67, 888, 777]
console.log(arr.radixSort())
```


