## Spring中的事务



### 事务的传播行为

事务的传播行为是指，当前事务方法调用另一个事务方法时，当前方法的事务如何传播到被调用的方法中。

Spring中定义有7种事务的传播行为：

- PROPAGATION_REQUIRED：加入到当前事务，如果当前没有事务，就新建一个事务。这是最常见的选择。 

- PROPAGATION_SUPPORTS：支持当前事务，如果当前没有事务，就以非事务方式执行。 

- PROPAGATION_MANDATORY：使用当前事务，如果当前没有事务，就抛出异常。 

- PROPAGATION_REQUIRES_NEW：新建事务，如果当前存在事务，把当前事务挂起。 

- PROPAGATION_NOT_SUPPORTED：以非事务方式执行操作，如果当前存在事务，就把当前事务挂起。 

- PROPAGATION_NEVER：以非事务方式执行，如果当前存在事务，则抛出异常。 

- PROPAGATION_NESTED：如果当前存在事务，则在嵌套事务内执行。如果当前没有事务，则执行与PROPAGATION_REQUIRED类似的操作。