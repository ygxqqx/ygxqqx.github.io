---  
published: true  
layout: post  
title: JavaScript数组的浅复制和深复制
date: 2016-05-24  
category: work  
---  

## 浅拷贝 & 深拷贝
在JavaScript中当把一个数组赋给另外一个数组时,只是为被赋值的数组增加了一个新的引用。
当你通过原引用修改了数组的值,另外一个引用也会感知到这个变化。下面的代码展示了这
种情况：

```
var nums = [];
for (var i = 0; i < 100; ++i) {
	nums[i] = i+1;
}
var samenums = nums;
nums[0] = 400;
console.log(samenums[0]); // 显示400
```

这种行为被称为浅复制,新数组依然指向原来的数组。一个更好的方案是使用深复制,
将原数组中的每一个元素都复制一份到新数组中。可以写一个深复制函数来做这件事：

```
function copy(arr1, arr2) {
	for (var i = 0; i < arr1.length; ++i) {
		arr2[i] = arr1[i];
	}
}
```

这样,下述代码片段的输出就和我们希望的一样了：

```
var nums = [];
for (var i = 0; i < 100; ++i) {
	nums[i] = i+1;
}
var samenums = [];
copy(nums, samenums);
nums[0] = 400;
console.log(samenums[0]); // 显示 1

```

















