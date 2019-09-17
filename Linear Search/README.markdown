# Linear Search

Goal: Find a particular value in an array.

We have an array of generic objects. With linear search, we iterate over all the objects in the array and compare each one to the object we're looking for. If the two objects are equal, we stop and return the current array index. If not, we continue to look for the next object as long as we have objects in the array.

## An example

Let's say we have an array of numbers `[5, 2, 4, 7]` and we want to check if the array contains the number `2`.

We start by comparing the first number in the array, `5`, to the number we're looking for, `2`. They are obviously not the same, and so we continue to the next array element.

We compare the number `2` from the array to our number `2` and notice they are equal. Now we can stop our iteration and return 1, which is the index of the number `2` in the array.

## The code

Here is a simple implementation of linear search in Swift:

```swift
func linearSearch<T: Equatable>(_ array: [T], _ object: T) -> Int? {
  for (index, obj) in array.enumerated() where obj == object {
    return index
  }
  return nil
}
```

Put this code in a playground and test it like so:

```swift
let array = [5, 2, 4, 7]
linearSearch(array, 2) 	// This will return 1
```

## Performance

Linear search runs at **O(n)**. It compares the object we are looking for with each object in the array and so the time it takes is proportional to the array length. In the worst case, we need to look at all the elements in the array.

The best-case performance is **O(1)** but this case is rare because the object we're looking for has to be positioned at the start of the array to be immediately found. You might get lucky, but most of the time you won't. On average, linear search needs to look at half the objects in the array.

## See also

[Linear search on Wikipedia](https://en.wikipedia.org/wiki/Linear_search)

*Written by [Patrick Balestra](http://www.github.com/BalestraPatrick)*

# 中文版

# 线性搜索

目标：在数组中查找特定值。  

我们有一个包含普通对象的数组。对它使用线性搜索时，需要遍历数组中的所有对象，并将每个对象与我们要查找的对象进行比较。如果两个对象相等，则停止并返回对象在数组中的索引。如果没有，并且数组还未遍历到最后一个，就继续寻找下一个对象。

## 举个例子

假如我们有一个数组 `[5，2，4，7]` 要检查其中是否包含数字 `2` 。

我们首先将数组中的第一个数字 `5` 与要查找的数字 `2` 进行比较。它们显然不一样，所以我们继续下一个数组元素。

接下来，把数组中第二个元素 `2` 和 `2` 进行比较，发现他们是相等的。这时就可以结束遍历，并把数组中的 `2` 的索引返回，即： `1`

## 代码如下

下面是 swift 中线性搜索的一个简单实现：

```swift
func linearSearch<T: Equatable>(_ array: [T], _ object: T) -> Int? {
  for (index, obj) in array.enumerated() where obj == object {
    return index
  }
  return nil
}
```
把上面这段代码放到 playground 中，然后进行如下测试：


```swift
let array = [5, 2, 4, 7]
linearSearch(array, 2) 	// This will return 1
```

线性搜索的时间复杂度为 **O(n)**。这种算法会遍历数组中每个对象，所以，它花费的时间是和数组的长度成正比的。在最坏的情况下，我们甚至需要遍历数组中所有元素。

## 扩展

[线性搜索-维基百科](https://en.wikipedia.org/wiki/Linear_search)    
*作者 [帕特里克·巴莱斯特拉](http://www.github.com/BalestraPatrick)*



