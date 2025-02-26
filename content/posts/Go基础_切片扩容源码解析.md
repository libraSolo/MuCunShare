+++
date = '2024-8-2T16:21:57+08:00'
draft = false
title = 'Go基础_切片扩容源码解析'
author = '木村凉太'
categories = 'Golang'
hiddenFromHomePage = true 
+++

# Go基础_切片扩容源码解析

```go
// newLen通常是你当前切片的长度（即已经包含的元素数量）加上你希望添加的元素数量。
func nextslicecap(newLen, oldCap int) int {
	newcap := oldCap
	doublecap := newcap + newcap
	if newLen > doublecap {
		return newLen
	}

	const threshold = 256
	if oldCap < threshold {
		return doublecap
	}
    // 这个循环确保newcap至少满足newLen的期望大小。
	for {
		// Transition from growing 2x for small slices
		// to growing 1.25x for large slices. This formula
		// gives a smooth-ish transition between the two.
    
        // 翻译：这个公式为从对小切片的小贝扩容切换到对大切片的1.25倍扩容提供了一个丝滑的切换。
		newcap += (newcap + 3*threshold) >> 2

		// We need to check `newcap >= newLen` and whether `newcap` overflowed.
		// newLen is guaranteed to be larger than zero, hence
		// when newcap overflows then `uint(newcap) > uint(newLen)`.
		// This allows to check for both with the same comparison.

        // 翻译：我们需要检查newcap >= newLen和newcap是否溢出。因为newLen保证大于0，所以当newcap溢出时，满足uint(newcap) > uint(newLen)。利用这个特性我们可以同时检查上面的两个条件是否满足。
        // 注：unit()会将int类型转换为uint，当newcap溢出时，会变为一个负数，使用unit会将其转换为一个非常大的正数，从而使满足而退出循环。
		if uint(newcap) >= uint(newLen) {
			break
		}
	}

	// Set newcap to the requested cap when
	// the newcap calculation overflowed.
  
    // 翻译：newcap溢出时，将newcap设置为newLen。
	if newcap <= 0 {
		return newLen
	}
	return newcap
}

```

1. 当`newLen`大于`oldcap`的两倍时，`newcap`的值等于`newLen`。
2. `newLen`小于等于`oldcap`的两倍时，当`oldcap`小于`256`，`newcap`的值为`oldcap`的两倍。
3. `newLen`小于等于`oldcap`的两倍时，当`oldcap`大于25，使用公式`newcap +=（newcap + 3 * threshold) >> 2`循环计算直到`newcap`至少等于`newLen`，如果计算溢出则`newcap`等于`newLen`。