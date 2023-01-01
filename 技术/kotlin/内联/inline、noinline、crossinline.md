#### inline

- 为了解决每次调用高阶函数时给它传递lambda表达式时会创建匿名内部类引入了inline关键字
- inline修饰的函数，不仅会把函数体的内容进行复制铺平，还会把函数类型参数的内容复制铺平；

#### noinline

- 当在内联函数中想把某个函数类型的参数进行传递或者返回，这时就不能把这个参数铺平，所以引入了noinline关键字

#### return与inline之间的约定

- 在非inline函数中，return无法使用（必须使用return@xxx 指明返回的函数），只有在inline函数中可以使用return语句这样就不会有异议；

#### crossinline

- crossinline是告诉IDE，这个参数的lambda不能写return语句