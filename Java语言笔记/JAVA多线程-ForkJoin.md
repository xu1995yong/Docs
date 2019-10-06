# ForkJoin框架

ForkJoin框架主要涉及三大核心组件：`ForkJoinPool`（线程池）、`ForkJoinTask`（任务）、`ForkJoinWorkerThread`（工作线程），外加`WorkQueue`（任务队列）：

- **ForkJoinPool**：ExecutorService的实现类，负责工作线程的管理、任务队列的维护，以及控制整个任务调度流程；
- **ForkJoinTask**：Future接口的实现类，fork是其核心方法，用于分解任务并异步执行；而join方法在任务结果计算完毕之后才会运行，用来合并或返回计算结果；
- **ForkJoinWorkerThread**：Thread的子类，作为线程池中的工作线程（Worker）执行任务；
- **WorkQueue**：任务队列，用于保存任务；

## 