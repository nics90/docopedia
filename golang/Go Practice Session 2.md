
a) Go Array and Slices

```
package main

import "fmt"

func main() {
	var arr [5]int = [5]int{10, 20, 30, 40, 50}
	var slc []int = arr[0:2]
	fmt.Println(slc)
	for i, j := range slc {
		fmt.Println(i, j)
	}
	for i := 0; i < len(slc); i++ {
		//fmt.Println(slc[i])
		fmt.Println(cap(slc), len(slc))
	}
	var arr1 []int = make([]int, 6)
	fmt.Println(arr1, len(arr1), cap(arr1))
	var arr2 []int = make([]int, 3, 5)
	fmt.Println(arr2, cap(arr2), len(arr2))
	arr2 = arr2[0:1]
	fmt.Println(arr2, cap(arr2), len(arr2))

	b := make([]int, 0, 5) // len(b)=0, cap(b)=5
	fmt.Println(b, len(b), cap(b))
	b = b[:cap(b)] // len(b)=5, cap(b)=5
	fmt.Println(b, len(b), cap(b))
	b = b[1:] // len(b)=4, cap(b)=4
	fmt.Println(b, len(b), cap(b))

}

```

b) Go 