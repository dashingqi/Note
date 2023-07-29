#### inline

- 为了解决每次调用高阶函数时给它传递lambda表达式时会创建匿名内部类引入了inline关键字
- inline修饰的函数，不仅会把函数体的内容进行复制铺平，还会把函数类型参数的内容复制铺平；
- 减少一次函数调用，少创建了一个匿名内部类；
- inline修饰的高阶函数中不能调用私有方法(这个是确定的)
- 如果内联函数是internal那么它内部就能调用internal修饰的普通函数；

```kotlin
inline fun performOperation(a: Int, b: Int, crossinline operation: (Int, Int) -> Int): Int {
    return operation(a, b)
}

fun performMain() {
    performOperation(1, 2) { a, b ->
        a + b
    }
}


public static final int performOperation(int a, int b, @NotNull Function2 operation) {
      int $i$f$performOperation = 0;
      Intrinsics.checkNotNullParameter(operation, "operation");
      return ((Number)operation.invoke(a, b)).intValue();
   }

   public static final void performMain() {
      byte a$iv = 1;
      int b$iv = 2;
      int $i$f$performOperation = false;
      int var5 = false;
      int var10000 = a$iv + b$iv;
   }
```

#### noinline

- 当在内联函数中想把某个函数类型的参数进行传递或者返回，这时就不能把这个参数铺平，所以引入了noinline关键字

#### return与inline之间的约定

- 在非inline函数中，return无法使用（必须使用return@xxx 指明返回的函数），只有在inline函数中可以使用return语句这样就不会有异议；

```kotlin
inline fun inlineFunction(block: () -> Unit) {
    println("Before inline")
    block()
    println("After inline")
}

fun main() {
    println("Start")
    inlineFunction {
        println("Inside inline")
        return // 非局部返回，从 main 函数返回
    }
    println("End")
}
```

#### crossinline

- crossinline是告诉IDE，lambda表达式的局部不能使用return

```kotlin
inline fun performOperation(a: Int, b: Int, crossinline operation: (Int, Int) -> Unit) {
    operation(a, b)
}

fun performMain() {
    performOperation(1, 2) { a, b ->
        a + b
        // 此处使用 return 语句来中断代码执行是不被允许的，因为crossinline关键字的作用
    }
    Log.d("TAG", "performMain: ")
}
```

#### JMH 测试 inline

###### 调用一次

<img src="https://raw.githubusercontent.com/dashingqi/DQPicBeg/main/inline-jmh-1.png" alt="inline-jmh-1" style="zoom:200%;" />

###### 嵌套调用10次

```kotlin
@BenchmarkMode(Mode.Throughput)
@Warmup(iterations = 3)
@Measurement(iterations = 5, time = 2, timeUnit = TimeUnit.SECONDS)
@OutputTimeUnit(TimeUnit.MILLISECONDS)
open class InlineBenchmark {

    private fun foo(block: () -> Unit) {
        block()
    }

    private inline fun fooInline(block: () -> Unit) {
        block()
    }

    @Benchmark
    fun testNoInline() {
        var i = 0
        foo {
            foo {
                foo {
                    foo {
                        foo {
                            foo {
                                foo {
                                    foo {
                                        foo {
                                            foo {
                                                i++
                                            }
                                        }
                                    }
                                }
                            }
                        }
                    }
                }
            }

        }
    }

    @Benchmark
    fun testInline() {
        var i = 0
        fooInline {
            fooInline {
                fooInline {
                    fooInline {
                        fooInline {
                            fooInline {
                                fooInline {
                                    fooInline {
                                        fooInline {
                                            fooInline {
                                                i++
                                            }
                                        }
                                    }
                                }
                            }
                        }
                    }
                }
            }

        }
    }
}

fun main() {
    val options = OptionsBuilder()
        .include(InlineBenchmark::class.java.simpleName)
        .resultFormat(ResultFormatType.JSON)
        .build()

    Runner(options).run()
}
```

<img src="https://raw.githubusercontent.com/dashingqi/DQPicBeg/main/inline-jmh.png" alt="inline-jmh" style="zoom:200%;" />

可以看到当处于嵌套多次调用时，使用inline关键字的性能要远好于noinline
