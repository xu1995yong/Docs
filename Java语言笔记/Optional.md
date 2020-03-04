# Optional类

为了更好的解决和避免常见的 NPE 问题，Java 8 中引入了一个新的类 java.util.Optional。Optional类对于 null 判定提供了一种更加优雅的实现，从而避免 NPE 问题。

## 为什么不喜欢NPE

因为NPE属于运行时异常，只在程序运行时出现，这是不可控的，会对程序的稳定运行产生极大的威胁。



## Optional类的思想

Optional类是个容器，其可以包装一个对象，无论该对象是不是null指针。并且Optional类提供了很多有用的方法，使开发人员不用进行显式的空值判断。同时可以根据传入对象是否是null指针，采取不同的处理手段。



## Optional对象的创建

- **Optional.of(T value):** 返回一个包装了value值的Optional。其中value值不可为null，否则抛出NPE。
- **Optional.ofNullable(T value):** 返回一个包装了value值的Optional。其中value值可为空。
- **Optional.empty():** 返回一个值为null的Optional对象。

## Optional其他API

- **T get()：**如果在这个Optional中包含这个值，返回值，否则抛出NoSuchElementException异常。
- **void ifPresent(Consumer consumer)：**如果值存在则使用该值调用 consumer , 否则不做任何事情。
- **boolean isPresent()：**判断值是否存在，如果存在则返回true，否则返回 false。
- **Optional map(Function mapper)：**如果值存在则对其执行调用映射函数得到返回值。并且如果返回值不为 null，则创建包含映射返回值的Optional作为map方法返回值，否则返回空Optional。
- **T orElse(T other)：**如果该值存在则返回值， 否则返回 other。
- **T orElseGet(Supplier other)：**如果该值存在则返回值， 否则触发 other，并返回 other 调用的结果。
- ** T orElseThrow(Supplier exceptionSupplier)：**如果该值存在则返回值，否则抛出由 Supplier 继承的异常。
- **Optional filter(Predicate predicate)：**如果值存在，并且这个值匹配给定的 predicate，返回一个包装了该值的新Optional，否则返回一个空的Optional。



## Optional类最佳实践



- 使用Optional取值并在值不存在的时候抛出异常

```
public String getCity(User user) throws Exception{
    return Optional.ofNullable(user)
                   .map(u-> u.getAddress())
                   .orElseThrow(()->new Exception("取指错误"));
```



- 使用Optional类取值并do something

```
  Optional.ofNullable(user).ifPresent(u->{
                                        dosomething(u);
                                      });
```