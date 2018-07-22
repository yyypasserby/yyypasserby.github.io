---
title: Go Learning Notes
date: 2017-06-09 23:09:33
categories:
  - practice
tags:
  - golang
thumbnail: /img/go-learning-notes/go.png
---

## Functions

```go
func compute(fn func(float64, float64) float64) float64 {
	return fn(3, 4)
}
 
func main() {
	hypot := func(x, y float64) float64 {
		return math.Sqrt(x*x + y*y)
	}
	fmt.Println(hypot(5, 12))
	fmt.Println(compute(hypot))
}
 
func adder() func(int) int {
	sum := 0
	return func(x int) int {
		sum += x
		return sum
	}
}
 
pos, neg = adder(), adder()
pos(2) // sum = 2
neg(-2) // sum = -2
```

1. Types come after variables names
2. Only capitalized name will be exported
3. Consecutive variables with same type can share
  `x int, y int` -> `x, y int`
4. Functions can return multiple string using `(int, int)`
5. Return variables can be named and used in the functions (must use parenthesis) `func Func(x int) (y int)`
6. Naked return (using return statement without arguments in a function with return values) will return the named return variables. Not recommended in long functions.
7. Functions can also be passed as values
8. Functions have closures which can access variables outside itself, and different functions have different closures.

## Variables

1. Variables can have initializer through `=`, if given, type can be omitted
2. `:=` is a short assignment, which can be used where `var` and type omitted
3. `fmt.Printf("Type: %T Value: %v\n", z, z)`
4. `const` variable has very high precision.

## Packages

1. Import name is the last word of the package
2. You should make all the import names unique, if it is not, using like `othername "path/to/conflictname"`

## Control Flow

1. In `if` and `for`, `{}` is always required, `()` is always not needed
2. No `while`, only `for $condition {}`
3. Like `for`, `if` and `switch` can have a short statement, and the variables declared can only be used until the end of `if` or `else`
4. No need to break in `switch`, it will stop when hits
5. `switch` with no condition is equivalent to `switch true`
6. `defer` defers the execution of a function until the surrounding function returns, but only the function call, the arguments are evaluated immediately
7. Multiple `defer`s goes into a stack, when function returns, it will pop out in a LIFO order

## Pointers

1. No pointer arithmetic
2. Pointer to a struct: `p.X = 1` instead of `(*p).X = 1`

## Types

```go
type Vertex struct {
	X, Y float64
}
 
v := Vertex{3, 4}
// v: = Vertex{X: 1} // Y: 0 is implicit
p := &Vertex{3, 4}
func (v Vertex) Abs() float64 {
	return math.Sqrt(v.X*v.X + v.Y*v.Y)
}
 
func (v *Vertex) Scale(f float64) {
	v.X = v.X * f
	v.Y = v.Y * f
}
 
v.Scale(10) // v.X = 30, v.Y = 40
v.Abs() // 50
```

1. Go doesn't have classes, but types with methods
2. Methods is just functions with a receiver
3. Methods can declared on non-struct type, like `type Myfloat float64`
4. You can just declare methods with a receiver whose type is defined in the same pacakage as the method
5. Methods' receiver can also be pointers, and only methods with pointer receivers can modify the value to which the receiver points.
6. Methods with pointer receivers can accept both pointers and values
7. In the inverse direction, methods with value receivers can also accept both pointers and values(Remember both can't modify the field in that type)

## Interface

```go
type I interface {
	M()
}
 
type T struct {
	S string
}
 
// This method means type T implements the interface I,
// but we don't need to explicitly declare that it does so.
func (t T) M() {
	fmt.Println(t.S)
}
 
func do(i interface{}) {
	switch v := i.(type) {
	case int:
		fmt.Printf("Twice %v is %v\n", v, v*2)
	case string:
		fmt.Printf("%q is %v bytes long\n", v, len(v))
	default:
		fmt.Printf("I don't know about type %T!\n", v)
	}
}
 
func main() {
	var a I = T{"hello"}
	a.M()
 
	var i interface{} = "hello"
	s := i.(string)
	fmt.Println(s)
	s, ok := i.(string)
	fmt.Println(s, ok)
	f, ok := i.(float64)
	fmt.Println(f, ok)
	f = i.(float64) // panic
	fmt.Println(f)
 
	// error code
	inte, err := strconv.Atoi("42")
	if err != nil {
    		fmt.Printf("couldn't convert number: %v\n", err)
    		return
	}
 
	fmt.Println("Converted integer:", inte)
 
	// reader
	r := strings.NewReader("Hello, Reader!")
	b := make([]byte, 8)
	for {
		n, err := r.Read(b)
		fmt.Printf("n = %v err = %v b = %v\n", n, err, b)
		fmt.Printf("b[:n] = %q\n", b[:n])
		if err == io.EOF {
			break
		}
	}
}
```

1. A interface method is defined as a set of method signature, and can hold any value that implements those methods
2. If the concrete value inside an interface is `nil`, the receiver in the method will be `nil`
3. Interface with zero methods are called empty interface, which are used to handle values of unknown types
4. A `Stringer` is a type that can describe itself as a string. The `fmt` package look for this interface to print values
5. Functions often return `error` value, and calling code should handle errors by testing whether the error equals `nil`

## Array

```go
var a [10]int // [n]T is an array of n values of T, and it will give you ten 0s
var s []int = a[1:5] // []T is a slice of an array
var as []int = a[1:] // a[5:]
 
ma := make([]int, 0. 5) // len(ma)=0, cap(ma)=5
// Create a tic-tac-toe board.
 
board := [][]string{
	[]string{"_", "_", "_"},
	[]string{"_", "_", "_"},
	[]string{"_", "_", "_"},
}
 
// The players take turns.
board[0][0] = "X"
board[2][2] = "O"
board[1][2] = "X"
board[1][0] = "O"
board[0][2] = "X"
append(as, 1, 2, 3) // add {1, 2, 3} to slice as
```

1. Slices are like references to arrays
2. Slices can't exist individually, if you build a slice, you build an array and create a slice referenced to it
3. Slices have both length `len(s)` and capacity `cap(s)`. If you change the back, the capacity will always maintain; but if you change the front the capacity will decrease, but anytime slices can't exceed the capacity which means you can re-slice it
4. If a slice reference to no array, it has 0 in both length and capacity, and also it equals to `nil`
5. `range` returns index and value for array in `for` and value can be dropped by not declaring it

## Map

```go
var m = map[string]string {
	"Debug": "sucks",
	"Test" : "rocks",
}
 
v, ok := m["Test"] // v = "rocks", ok = True
v, ok = m["Deploy"] // v = "", ok = False
m["Deploy"] = "rocks"
delete(m, "Deploy")
 
func WordCount(s string) map[string]int {
	ret := map[string]int {}
	fields := strings.Fields(s)
	for _, field := range fields {
		v, ok := ret[field]
		if ok == true {
			ret[field] = v + 1
		} else {
			ret[field] = 1
		}
	}
	return ret
}
```

## Goroutines

```go
go f(x, y, z)
ch <- v // Send v to channel ch.
v := <-ch // Receive from ch, and assign value to v.
ch := make(chan int, 1)
 
// close and range
func fibonacci(n int, c chan int) {
	x, y := 0, 1
	for i := 0; i < n; i++ {
		c <- x
		x, y = y, x+y
	}
	close(c)
}
 
func main() {
	c := make(chan int, 10)
	go fibonacci(cap(c), c)
	for i := range c {
		fmt.Println(i)
	}
}
 
// select
func fibonacci(c, quit chan int) {
	x, y := 0, 1
	for {
		select {
		case c <- x:
			x, y = y, x+y
		case <-quit:
			fmt.Println("quit")
			return
		}
	}
}
 
func main() {
	c := make(chan int)
	quit := make(chan int)
	go func() {
		for i := 0; i < 10; i++ {
			fmt.Println(<-c)
		}
		quit <- 0
	}()
	fibonacci(c, quit)
}
 
// SafeCounter is safe to use concurrently.
type SafeCounter struct {
	v   map[string]int
	mux sync.Mutex
}
 
// Inc increments the counter for the given key.
func (c *SafeCounter) Inc(key string) {
	c.mux.Lock()
	// Lock so only one goroutine at a time can access the map c.v.
	c.v[key]++
	c.mux.Unlock()
}
 
// Value returns the current value of the counter for the given key.
func (c *SafeCounter) Value(key string) int {
	c.mux.Lock()
	// Lock so only one goroutine at a time can access the map c.v.
	defer c.mux.Unlock()
	return c.v[key]
}
```
1. `goroutine` is a lightweight thread managed by the Go routine, `go func(x)`
1. Channels are a typed conduit through which you can send and receive values with the channel operator, `<-`
2. Like Maps and slices, channels must be created before use, `ch := make(chan int)`
3. By default, sends and receives block until the other side is ready. This allows gorountines to synchronize without explicit locks or condition variables
4. A sender can `close(ch)` a channel to indicate that no more value will be sent. And the receivers can use the second parameter to test whether the channel has been closed, `v, ok := <-ch`
5. The loop, `for x := range ch {}` for the channel will repeat until it is closed.
6. In `select`, `default` means non-blocking
