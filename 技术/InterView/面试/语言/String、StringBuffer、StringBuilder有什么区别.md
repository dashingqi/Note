## String、StringBuffer、StringBuilder有什么区别

#### String

- String时Java中非常基础和重要的类，提供了管理和构造字符串的各种逻辑
- 它是Immutable（不可变的），被声明为final，类中的属性也都是final的。
- 由于它的不可变性，类似于拼接、切割字符串的操作都是要产生新的String对象。

#### StringBuffer

- StringBuffer是为了解决String拼接产生太多中间对象而提供的一个类。
- StringBuffer是一个线程安全的，保证了线程安全之余带来了额外的性能开销。
- 除非有线程安全的问题，不然可以使用StringBuilder

#### StringBuilder

- 是Java1.5中新增的，非线程安全的，相对StringBuffer性能好些。
- 也是绝大部份情况下进行字符串拼接的首选。

