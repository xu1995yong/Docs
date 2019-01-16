# ThreadLocal

ThreadLocal，也即：线程局部变量。

```java

final ThreadLocal<ArrayList> threadLocal = new ThreadLocal<ArrayList>() {


		@Override
		protected ArrayList initialValue() {
			return new ArrayList();
		}
	};
	ExecutorService executors = Executors.newFixedThreadPool(5);
	Callable setTask = () -> {
		ArrayList list = threadLocal.get();
		System.out.println(Thread.currentThread().getName());
		list.add(Thread.currentThread().getName());
		return null;
	};

	Callable getTask = () -> {
		synchronized (MainTest.class) {
			ArrayList<String> list = threadLocal.get();
			System.out.println();
			System.out.println(Thread.currentThread().getName());
			// System.out.println(threadLocal);
			// System.out.println(System.identityHashCode(list));
			// for (String str : list) {
			// System.out.println(str);
			// }
			System.out.println(list.size());
			System.out.println();
			return null;
		}

	};

	for (int i = 0; i < 10; i++) {
		executors.submit(setTask);
	}
	// Thread.sleep(500);
	// for (int i = 0; i < 10; i++) {
	// executors.submit(getTask);
	// }

	executors.shutdown();
```



## ThreadLocal的实现原理

Thread类中的成员变量，ThreadLocalMap