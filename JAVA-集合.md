## 栈与队列
JAVA中的栈是Stack类，但是官方已经不推荐使用Stack，而是推荐更高效的ArrayDeque。
JAVA中的队列是Queue接口,同样当需要使用队列时，也推荐使用ArrayDeque了（次选是LinkedList）。

### 1.Deque接口：即双端队列，它既可以当作栈使用，也可以当作队列使用。

下表列出了*Deque*与*Queue*相对应的接口：

| Queue Method | Equivalent Deque Method | 说明 |
|--------|--------|--------|
| `add(e)` | `addLast(e)` | 向队尾插入元素，失败则抛出异常 |
| `offer(e)` | `offerLast(e)` | 向队尾插入元素，失败则返回`false` |
| `remove()` | `removeFirst()` | 获取并删除队首元素，失败则抛出异常 |
| `poll()` | `pollFirst()` | 获取并删除队首元素，失败则返回`null` |
| `element()` | `getFirst()` | 获取但不删除队首元素，失败则抛出异常 |
| `peek()` | `peekFirst()` | 获取但不删除队首元素，失败则返回`null` |

下表列出了*Deque*与*Stack*对应的接口：

| Stack Method | Equivalent Deque Method | 说明 |
|--------|--------|--------|
| `push(e)` | `addFirst(e)` | 向栈顶插入元素，失败则抛出异常 |
| 无 | `offerFirst(e)` | 向栈顶插入元素，失败则返回`false` |
| `pop()` | `removeFirst()` | 获取并删除栈顶元素，失败则抛出异常 |
| 无 | `pollFirst()` | 获取并删除栈顶元素，失败则返回`null` |
| `peek()` | `peekFirst()` | 获取但不删除栈顶元素，失败则抛出异常 |
| 无 | `peekFirst()` | 获取但不删除栈顶元素，失败则返回`null` |

从表中可以看出，Deque接口对添加，删除，取值都有两套方法。其区别是对失败情况的处理不同。**一套接口遇到失败就会抛出异常，另一套遇到失败会返回特殊值（`false`或`null`）。**

### .ArrayDeque的实现

-	ArrayDeque底层通过**循环数组（circular array）**实现，也即数组的任何一点都可能被看作起点或者终点。
-	ArrayDeque是非线程安全的（not thread-safe），当多个线程同时使用的时候，需要程序员手动同步。
-	ArrayDeque不允许放入`null`元素。


 