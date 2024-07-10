- defer的执行时间：执行return语句之后，return为非原子性，需要两步，第一步是得到返回值（为返回值赋值），第二步是把返回值返回调用处。**defer的是在第一步之后，第二步之前。**
- defer使用场景：
	1. **defer后面跟着无传参函数调用**：

	```Go
func f() {
	defer func(){fmt.Println(1)}()
}
func main() {
	f()
}	
	```

	**注意事项**：
	- 匿名函数定义后面一定要加()，不然不会在最后函数返回的时候调用。
	- 执行时间就是在f函数结束的时候，打印1

	2. **defer后面跟着传参函数调用**：
	```Go
func f() {
	i := 0
	defer func(i int) {fmt.Println(i)}()
	i++
}
	```

	**注意事项**：
	- f函数调用之后打印出0，因为如果defer后面的函数有参数传入，那么会直接在定义defer的地方进行参数解析，即defer代码处传入参数i=0，后续的i++对该函数已经无影响。

	3. **defer后面跟着方法调用**：

	```Go
func f() {
	i := 0
	defer fmt.Println(i)
	i++
	return
}
	```

	**注意事项**：
	- 方法调用的时候大部分情况是有参数传入的，因此跟第二种情况一样，会直接进行参数解析，比如上述例子就会打印出0而不是1。

- 几种比较特殊的情况：
```Go
func func1(s string) (n int, err error) {
	defer func() {
		log.Printf("func1(%q) = %d, %v", s, n, err)
	}()
	return 7, io.EOF
}

func main() {
	func1("Go")
}
```
这个的输出结果是```Output: 2011/10/04 10:46:11 func1("Go") = 7, EOF```，当返回值命名之后，相当于提前定义了变量，return语句把n=7，err=io.EOF，然后执行defer后面的函数，此时的n和err就是return赋值之后的结果。如果改为如下代码：
```Go
func func1(s string) (n int, err error) {
	defer log.Printf("func1(%q) = %d, %v", s, n, err)
	return 7, io.EOF
}

func main() {
	func1("Go")
}
```
这个输出就是```Output: 2011/10/04 10:46:11 func1("Go") = 0, nil```，因为这种情况就是上述的第三种情况——方法的调用，会直接进行参数解析，而把返回值命名相当于提前定义了该变量，因此此时获取到的n和err都是默认初始化值。