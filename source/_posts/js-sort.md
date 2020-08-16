---
title: js排序算法
date: 2020-08-10 02:22:28
tags: Javascript
categories: Javascript
cover: /img/sort.jpg
---

# 冒泡排序

#### 外循环每排一次，最大或最小数浮到最后

外循环：N个数需排N-1次

内循环：N个数需交换N-1次

##### i递减

```javascript
var temp;
for(let i=arr.length-1;i>0;i--){
    for(let j=0;j<i;j++){
        if(arr[j]>arr[j+1]){
            temp=arr[j];
            arr[j]=arr[j+1];
            arr[j+1]=temp;
        }
    }
}
```

##### i递增

```javascript
var temp;
for(let i=0;i<arr.length-1;i++){
    for(let j=0;j<arr.length-i-1;j++){
        if(arr[j]>arr[j+1]){
            temp=arr[j];
            arr[j]=arr[j+1];
            arr[j+1]=temp;
        }
    }
}
```

### 优化

　　**对于已经有序或接近有序的集合时，会进行很多次不必要的循环比较**，为此，需要改进实现，**设置一个flag记录在一次循环比较中是否有交换操作**，如果没有说明集合已经有序。

```javascript
var temp;
boolean flag = true;
for(var i=0;(i<arr.length-1) && flag;i++){
	flag = false;	// 用flag作为标记
    for(var j=0;j<arr.length-i-1;j++){
        if(arr[j]>arr[j+1]){
            temp=arr[j];
            arr[j]=arr[j+1];
            arr[j+1]=temp;
            flag = true;	// 有数据交换则为true
        }
    }
}
```

### 复杂度分析

　　最好的情况下，也就是数组有序时，根据最后改进的代码，需要比较n-1次关键字，没有数据交换，时间复杂度为O(n)。最坏的情况下，即待排序记录全为倒序，此时比较1+2+3+4+…+(n-1) = n(n-1)/2次，并作等数量级的记录移动。所以时间复杂度为O(n2)。







# 选择排序

#### 从第一个数开始，找该数之后最大或最小的数，与之交换

外循环：N个数需排N-1次

内循环：找i后最大或最小的数

#### 找Min(从小到大)

```javascript
for(let i=0;i<arr.length-1;i++){
    let temp,
        min=arr[i],
        index=i;
    for(let j=i+1;j<arr.length;j++){
        if(arr[j]<min){
            min=arr[j]
            index=j;
        }
    }
    temp=arr[i];
    arr[i]=arr[index];
    arr[index]=temp;
}
```

#### 找Max(从大到小)

```javascript
for(let i=0;i<arr.length-1;i++){
    let temp,
        max=arr[i],
        index=i;
    for(let j=i+1;j<arr.length;j++){
        if(arr[j]>max){
            max=arr[j]
            index=j;
        }
    }
    temp=arr[i];
    arr[i]=arr[index];
    arr[index]=temp;
}
```

### 复杂度分析

　　从简单选择排序过程看，最大的特点是减少了移动数据的次数，这样节约了时间。无论最好还是最差的情况下，比较次数都是一样的，第i趟要比较n-i次关键字，共需要比较(n-1)+(n-2)+…+2+1=n(n-1)/2次，最好情况下，即有序时，交换0次，最坏情况下，即逆序时，交换n-1次。最终排序时间为比较和移动的总和，时间复杂度为O(n2)。尽管与冒泡排序同为O(n2)，但简单选择排序的性能还是要略优于冒泡排序。







# 归并排序

**思路**：先递归分解数列，再根据**两者选最小原则**合并两个数列（分治思想的典型应用）

　　（1）递归分解数列直到每个小组只有一个元素为止。

　　（2）可以视小组为有序，比较每个小组的第一个元素，两者选最小，push到新数组。

```javascript
function mergeSort ( array ) {
    let len = array.length;
    if( len < 2 ){
        return array;
    }
    let middle = Math.floor(len / 2),
        left = array.slice(0, middle),
        right = array.slice(middle);
    return merge(mergeSort(left), mergeSort(right));
}
function merge(left, right)
{
    let result = [];
    while (left.length && right.length) {
        if (left[0] <= right[0]) {
            result.push(left.shift());
        } else {
            result.push(right.shift());
        }
    }
    while (left.length)
        result.push(left.shift());
    while (right.length)
        result.push(right.shift());
    return result;
}
```

### 时间复杂度：O(N*logN)







# 快速排序

## 左右指针法（推荐记忆）

1. 选取一个关键字(key)作为枢轴，一般取整组记录的（首、中、尾三数取中），这里选取序列第一一个数为枢轴。
2. 设置两个变量left = 0;right = N - 1;
3. 从left一直向右走，直到找到一个大于key的值，right一直向左走，直至找到一个小于key的值，然后交换这两个数。
4. 重复第三步，一直往后找，直到left和right相遇，这时将key放置left的位置即可。

![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTcxMTI5MTUzMzMzNzkx?x-oss-process=image/format,png)

```javascript
function quickSort(a, l=0, r=a.length-1)	//注意r=a.length-1
{
    if(l>=r)
        return;

    let h=l,
        e=r,
        keyIndex=l,
        key=a[keyIndex],
        temp;
    while(l<r){
        while(l < r && a[r] >= key)
            --r;
        while(l < r && a[l] <= key)
            ++l;
        temp=a[l];
        a[l]=a[r];
        a[r]=temp;
    }
    temp=a[l];
    a[l]=key;
    a[keyIndex]=temp;
    quickSort(a,h,l-1);
    quickSort(a,l+1,e)
    return a;
}
```

## 挖坑法1

#### （基准值可随意取）

1. 选取一个关键字(key)作为枢轴，一般取整组记录的（首、中、尾三数取中），这里选取序列第一一个数为枢轴，也是初始的坑位。
2. 设置两个变量left = 0;right = N - 1;
3. 从right一直向前走，直到找到一个小于key的值，然后将该数放入坑中，坑位变成array[right]。
4. 从left一直向后走，直到找到一个大于key的值，然后将该数放入坑中，坑位变成array[left]。
5. 重复3和4的步骤，直到left和right相遇，然后将key放入最后一个坑位。

![这里写图片描述](https://imgconvert.csdnimg.cn/aHR0cDovL2ltZy5ibG9nLmNzZG4ubmV0LzIwMTcxMTI5MTYzNzM2NjUy?x-oss-process=image/format,png)

```javascript
function quickSort(a, l=0, r=a.length-1)
{
    if(l>=r)
        return;

    var h=l,e=r
    var key=a[l];
    while(l<r){
        while(l < r && a[r] >= key)
            r--;
        if(l<r)
            a[l]=a[r];
        while(l < r && a[l] <= key)
            l++;
        if(l<r)
            a[r]=a[l];
    }
    a[l]=key;
    quickSort(a,h,l-1);
    quickSort(a,l+1,e)
    return a;
}
quickSort(arr);
```

## 挖坑法2（带左右指针）

#### （下面代码写法，基准数只能取第一或最后）

1. i =L; j = R; 将基准数挖出形成第一个坑a[i]。
2. j--由后向前找比它小的数，找到后挖出此数填前一个坑a[i]中。
3. i++由前向后找比它大的数，找到后也挖出此数填到前一个坑a[j]中。
4. 再重复执行2，3二步，直到i==j，将基准数填入a[i]中。

```javascript
//快速排序
function quickSort(a, l=0, r=a.length-1)
{
    if (l >= r)
        return;

    var i = l, j = r, x = a[l];
    while (i < j)
    {
        while(i < j && a[j] >= x) // 从右向左找第一个小于x的数
            j--;
        if(i < j)
            a[i++] = a[j];

        while(i < j && a[i] < x) // 从左向右找第一个大于等于x的数
            i++;
        if(i < j)
            a[j--] = a[i];
    }
    a[i] = x;
    quickSort(a, l, i - 1); // 递归调用
    quickSort(a, i + 1, r);
    return a;
}
quickSort(arr);
```







# 插入排序

**思路：**

​		N个数排N-1次。

　　假设前n个数组已经排列完成，第n+1个元素（记为插入值） 与前n个元素倒序比较，比插入值大则往后挪让位，小则插入其后。

```javascript
function insertSort(arr){
    for(var i=1;i<arr.length;i++){
        var inserted = arr[i];
        for(var j=i-1;j>=0 && inserted<arr[j];j--)	//注意j>=0
            arr[j+1]=arr[j];
        arr[j+1]=inserted;	//注意j不能用let声明
    }
    return arr;
}
```

#### 时间复杂度：O(n2)







# 希尔排序（缩小增量插入排序）

**思路：**每隔增量inc分成一小组，小组内插入排序。逐渐减半增量直至向下取整小于0

```javascript
function shellSort(arr) {
    for(var inc=Math.floor(arr.length/2); inc>0;inc=Math.floor(inc/2)){
        for(var i= inc;i<arr.length;i++){
            var inserted = arr[i];
            for(var j=i-inc;j>=0 && inserted<arr[j];j-=inc)
                arr[j+inc]=arr[j];
            arr[j+inc]=inserted;
        }
    }
    return arr;
}
```







# 堆排序

https://www.cnblogs.com/chengxiao/p/6129630.html

