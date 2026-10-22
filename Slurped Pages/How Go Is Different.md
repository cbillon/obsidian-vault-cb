---
link: https://www.amazingcto.com/how-go-golang-is-different-for-loop-embedding-structs-inheritance/
byline: Stephan Schmidt
site: Amazing CTO
date: 2025-08-12T00:00
excerpt: Go looks like C but behaves differently. Inline conditionals, implicit
  interfaces, composition over inheritance - here's what Java and Rust devs get
  wrong.
twitter: https://twitter.com/@KingOfCoders
slurped: 2026-10-20T21:00
title: How Go Is Different
---

Go looks just like many other programming languages, especially from the C tradition. This and it’s focus on simplicity leads to people’s misunderstanding of Go. At least I misunderstood some parts of Go, and it was not goroutines and channels. When I started Go I used Go in a way I would have used Java, Rust or JavaScript. But while it looks familiar, Go is different in some ways.

## 1. Inline Conditional Initialization and Checking

Go has the nice property of not requiring parenthesis `(cond)` in `if` statements. Obvious to everyone. But instead of

```
x, err := x()
if err != nil {
    ...
}
```

you can write

```
if ok := x(); ok {
   ...
}
```

Here, `ok` is assigned the result of `x()`, and then it’s checked in the `if` condition. This idiom is commonly seen in Go, especially when dealing with functions that return a value and an error. It helps in keeping the scope of the variable `ok` limited to the `if` block, enhancing readability and avoiding clutter in the outer scope. This also works with more than one return value and keeps the variables to a scope.

```
if x, err := x(); err != nil {
    ...
}
```

The downside in this case is that `x` does not escape the scope, so you can’t do this:

```
if x, err := x(); err != nil {
    ...
}
x.foo() // does not work
```

## 2. Embedding Structs/Composition Over Inheritance

In Go you can directly embed `structs` in other `structs`:

```
type X struct {
   x string
}

type Y struct {
  X
}
```

Whereas in some other languages you would need to write

```
type Y struct {
  x X
}
```

to embed it.

This demonstrates Go’s approach to [composition over inheritance](app://obsidian.md/abstraction-as-a-developer-footgun/). In Go, instead of classical inheritance (as seen in object-oriented languages like Java or C++), you “embed” one struct into another. This means that `type Y` includes all fields and methods of `type X` implicitly. This way, methods and fields can be directly accessed on the embedded struct:

```
func (x X) foo() string {
  return "Hello"
}

func main() {
	y := Y{
		X: X{x: "Hello"},
	}
	// prints "Hello"
	fmt.Println(y.foo())
	// prints "Hello"
	fmt.Println(y.x)
}
```

instead of

```
  fmt.Println(y.X.foo())
  fmt.Println(y.X.x)
```

It’s a way to achieve a similar outcome to inheritance but with a composition-based approach. And as we all know, better compose than inherit. This encourages a more modular and flexible design, as it focuses on what the structs do (behaviors) rather than where they are in an inheritance hierarchy.

## 3. Interfaces Implemented Implicitly

In Go, a type implements an interface by implementing the interface’s methods, without explicitly declaring that it does so. This is different from languages like Java or C#, where a class must explicitly specify which interfaces it implements.

```
type Reader interface {
	Read(p []byte) (n int, err error)
}

type MyReader struct {
    // MyReader implicitly implements Reader if it has the Read method.
	func (m MyReader) Read(p []byte) (n int, err error) {
}
```

This [reduces explicit coupling](app://obsidian.md/modulith-vs-monolith-vs-microservices/) and makes development easier.

Implicity also works for `interfaces` and embedded structs:

```
type Foo interface {
	foo() string
}

func x(foo Foo) {
	fmt.Println(foo.foo())
}

func main() {
	y := Y{
		X: X{x: "Hello"},
	}
	// prints "Hello"
	x(y)
}
```

## 4. Error Handling via Returned Values

Unlike many other languages that use exceptions for error handling, Go handles [errors by returning them](app://obsidian.md/best-way-to-handle-errors-for-a-programming-language/) as a regular value from functions. This approach encourages explicit error checking and makes the control flow easier to follow—although a bit noisy sometimes.

```
func readFile(name string) ([]byte, error) {
    data, err := ioutil.ReadFile(name)
    if err != nil {
       return nil, err
    }
    return data, nil
}

data, err := readFile("example.txt")
if err != nil {
    log.Fatal(err)
}
```

It is also sometimes awkward when you return a value instead of a pointer:

```
func foo(f string) (Foo, error) {
	if len(f) == 0 {
	    // does not work
		return nil,errors.New("f is empy")
	}
	...
}
```

## 5. Deferred Function Execution

The `defer` statement in Go schedules a function call to be run after the function that contains the `defer` statement has completed, regardless of whether the containing function exits via a return statement or panics. This is particularly useful for resources cleanup.

There is one big benefit compared to `finally` in Java for example: The `defer` call is local and can easily be seen, and is not many lines down where it is hard to spot.

```
func readFile(name string) ([]byte, error) {
     f, err := os.Open(name)
     if err != nil {
             return nil, err
     }
     defer f.Close()  // f.Close will be called when readFile returns
     return ioutil.ReadAll(f)
}
```

## 6. Go has only for loops

While other languages have `while` or `repeat` or other constructs for looping, Go only has `for`.

The many ways to use `for` in Go:

**Counting Loop**: This is a basic loop where you iterate over a range of numbers.

```
for i := 0; i < 10; i++ {
  fmt.Println(i)
}
```

**Infinite Loop**: This loop will run forever unless it is broken out of with a `break` statement or an external interrupt.

```
    // Do something forever
    for {
       // Use 'break' to exit the loop
    }
```

**Loop Over a Range**: You can use a `for` loop to iterate over a range of values with range:

```
for i := range make([]int, 5) {
   fmt.Println(i) // Will print 0 to 4
}
```

**Loop Over a Map**: When looping over a map, you get both the key and value for each entry in the map.

```
m := map[string]int{"a": 1, "b": 2, "c": 3}
for key, value := range m {
    fmt.Printf("Key: %s, Value: %d\n", key, value)
}
```

**Loop Over a String**: Iterating over a string using `range` will give you the index and the rune at that index.

```
for index, runeValue := range "Go" {
    fmt.Printf("%#U starts at byte position %d\n", runeValue, index)
}
```

**Loop Over a Slice or Array**: Similar to a range loop, you can iterate over each element in a slice or array.

```
nums := []int{1, 2, 3, 4, 5}
for i, num := range nums {
    fmt.Printf("Index: %d, Value: %d\n", i, num)
}
```

**Loop over integers**: With Go 1.22 Go can now iterate over integers.

```
// iterate from 0 to 9
for i := range 10 {
  fmt.Println(10 - i)
}
```

## 7. Zero Values

In Go, every type has a zero value. If a variable is declared without an explicit initial value, it is given its zero value. This sounds normal, but Go goes way beyond. `len(uninizialized)` returns `0` instead of breaking. `map` and slices `[]` just work.

```
var i int        // Zero value is 0
var f float64    // Zero value is 0.0
var s string     // Zero value is ""
var slice []int  // Zero value of []int(nil) not nil
```

It is far less common in Go to use constructors than in other languages. `Structs` are expected to work as-is. It is perfectly fine to have a struct without a constructor in Go.

## 8. Named Return Values

In Go, return values can be named. This way, the `return` statement does not have parameters but the named variables are returned:

```
func sum(a, b int) (result int, err error) {
    result = a + b
    // 'err' is implicitly returned as nil if not assigned
    return // No need to explicitly return 'result' and 'err'
}
```

## 9. Go Has Slices And Arrays

Go has arrays and slices, which seem to be the same but are different.

```

    // defines a slice
    slice := []int{1, 1, 2}

    // defines an array
    var arr [3]int
    arr[0] = 1
    arr[1] = 1
    arr[2] = 2
```

**Slices**

- _Dynamic Size_ A slice can grow and shrink in size. Slices are more flexible and commonly used compared to arrays.
- _Reference Type_ A slice is a reference to an underlying array. When you pass a slice to a function, it receives a reference to the slice, not a copy of its elements.
- _Header_ A slice has a header that includes a pointer to the elements, the length of the slice, and its capacity (the maximum size it can grow to without reallocating memory).

**Arrays**

- _Fixed Size_ An array has a fixed size. Once declared, its size cannot be changed.
- _Value Type_ An array is a value type. When you pass an array to a function, it receives a copy of the array (not a reference).
- _Defined Length_ The length of an array is part of its type:

```
func foo(bar [3]int) {
	fmt.Println(bar[2])
	return
}
```

## Conclusion

I do ❤️ Go. But the road was rocky as I’ve assumed it would be like most other mainstream languages, while the designers of Go made different choices. Some of them rubbed me the wrong way for a long time until I’ve learned to ❤️ them.

**About me:** Hey, I'm Stephan, I help [CTOs with Coaching](app://obsidian.md/cto-coaching-from-experienced-cto/), with 40+ years of software development and 25+ years of engineering management experience. I've coached and mentored 100+ CTOs and founders in more than 1200 sparring and coaching sessions. I've founded 3 startups. 1 nice exit. I help CTOs and engineering leaders grow, scale their teams, gain clarity, lead with confidence and navigate the challenges of fast-growing companies.

Most of the CTOs I coach didn't know CTO coaching was a thing until they were already drowning. It is a thing - here's [what it is](app://obsidian.md/what-is-cto-coaching/).