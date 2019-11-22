---
title: "Basic Go"
date: 2019-11-22T16:31:44+01:00
draft: false 
author: "Jacqueline"
---

When learning Golang and coming from other (scripting) languages, that asterisk and ampersant may strike as a bit odd.  
This is a beginner Golang post!

Let's start with a basic Golang program outline and in the main func, let's create a variable named a and assign it the value Three-toed sloth.

```golang
package main
import (
    "fmt"
)
func main() {
  a := "Three-toed sloth"
  fmt.Println(a)
}
```

Notice the : before the = and that no type was specified. The Go compiler is able to derive the type based on the literal value of the variable. So it knows it's a string.

I could also have declared the variable like this:

```golang
var a string = 'Three-toed slothCopy
```

Ok, so the program will return:

`Three-toed sloth`

So you see that a is assigned a value of Three-toed sloth. It has stored that value in memory. And we can get the memory address by adding the & (ampersant).

Like this:

```golang
package main
import (
    "fmt"
)
func main() {
  a := "Three-toed sloth"
  fmt.Println(a)
  fmt.Println(&a)
}
```

This will return:

```
Three-toed sloth
0xc0000861c0
```
We can get the types like so:

```
fmt.Printf("%T\n",a)
fmt.Printf("%T\n",&a)
```

Adding this to our func will return:

```
 string
*string
```

We can also get the value of what is stored on that memory address like so (with the asterisk!):

```
  b := &a
  fmt.Println(*b)Copy
```

This will return:

```
Three-toed sloth
```

And finally we can do:

```
fmt.Println(*&b)
```

Now what would that return?

Exactly:

```
0xc0000861c0
```

Anyways, here is the complete code of this very useful exercise:

```
package main
import (
    "fmt"
)

func main() {
    a := "Three-toed sloth"
    fmt.Println(a)
    fmt.Println(&a)
    fmt.Printf("%T\n",a)
    fmt.Printf("%T\n",&a)
    b := &a
    fmt.Println(*b)
    fmt.Println(*&b)
}
```

So the gist of this post comes down to:

The & will give you the memory address.    
The asterisk will give you the value stored at an address, aka the pointer to the memory location where the value of the variable is stored.  
Now consider this piece of code without any pointers:  

```
package main
import (
    "fmt"
)

func main() {
    x := 100
    blah(x)
    fmt.Println(x)
}

func blah(y int) {
    fmt.Println(y)
    y = 12
    fmt.Println(y)
}
```

This will return

```
100 # from blah
12 # from blah
100
```

But when we change the signature of 'blah' so that it will take in a memory address instead of an actual int

```
package main
import (
    "fmt"
)

func main() {
    x := 100
    blah(&x)
    fmt.Println("From main: ", x)
}

func blah(y *int) {
    fmt.Println("1.From foo:", y)
    *y = 12
    fmt.Println("2.From foo:", y)
}
```

This will return

```
1.From foo: 0xc00001c0c8
2.From foo: 0xc00001c0c8
From main:  12
```

That value will be never be the value of 100 because the blah function assigns the memory address the value of 12.

Finally a tip for a great Golang mentor: https://twitter.com/Todd_McLeod
