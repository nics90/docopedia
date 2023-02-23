
a) Array and Slice Examples :

```
package main

import "fmt"

func main() {
   
   //  Create an array 
	var a [5]int = [5]int{1, 2, 3, 4, 5}
	var b [7]int
	
	fmt.Println(a, b)
	
   // Create a slice
	var c []int = []int{1, 2, 3}
	d := make([]int, 2, 2)
	d[0], d[1] = 4, 5

    fmt.Println(c)
	fmt.Println(d)
}

```


b)  Slice Pointer, Length and Capacity

```

package main

import "fmt"

func main() {
	e := make([]int, 5)
	e[0], e[1], e[2], e[3], e[4] = 1, 2, 3, 4, 5
	
	e1 := e[2:4]
	fmt.Println(len(e1), cap(e1))
	fmt.Println(e1[:cap(e1)])
	
    e2 := make([]int, 2)
	copy(e2, e[2:4])
	fmt.Println(e2)
	fmt.Println(e2[:cap(e2)])

    var a [5]int = [5]int{1, 2, 3, 4, 5}
	b := a[2:3]
	fmt.Println(b)
	fmt.Println(len(b), cap(b))
	fmt.Println(b[0:cap(b)])

	v := make([]int, 5, 5)
	v[0] = 10
	v[4] = 20
	fmt.Println(v)

	
}

```

c) Go Map

```

// You can edit this code!
// Click here and start typing.
package main

import "fmt"

func main() {
	
	/*  Initialize Map Method 1*/
	
	var x map[string]int = make(map[string]int)
	x["key1"] = 10
	x["key2"] = 20
	fmt.Println(x)
	
	/*  Initialize Map Method 2*/
	y := make(map[string]int)
	y["key1"] = 100
	y["key2"] = 200
	fmt.Println👍
	
	/*  Initialize Map Method 3*/
	z := map[string]int{
		"key1": 1000,
		"key2": 2000,
	}
	fmt.Println(z)

	if v, ok := z["key2"]; ok {
		fmt.Println(v)
		fmt.Println("Found key")
	}
	
	delete(z, "key1")
	fmt.Println(z)
}

```

d) Map Real Life Example

```

    var marks map[string]int = make(map[string]int)
	fmt.Println(marks)
	marks["english"] = 90
	marks["maths"] = 96
	marks["science"] = 75
	fmt.Println(marks)

	runrate := map[string]float32{"Kohli": 78.9, "Surya": 90.2}
	fmt.Println(runrate)
	player_name := "Rohit"
	var player_exist bool
	_, player_exist = runrate[player_name]
	fmt.Println(player_exist)
	if player_exist {
		fmt.Printf("%v run rate is %v", player_name, runrate[player_name])
	} else {
		fmt.Printf("%v have not played this match", player_name)
	}
	
```

e) Go Struct 

```

package main

import "fmt"

type player struct {
	name       string
	age        int
	strikeRate float32
	matches    int
}

func main() {
	player1 := player{
		name:       "M.S. Dhoni",
		age:        39,
		strikeRate: 98.5,
		matches:    300,
	}

	fmt.Printf("At age %v also %v strike rate in %v matches is %v and that is really awesome", player1.age, player1.name, player1.matches, player1.strikeRate)
}

```