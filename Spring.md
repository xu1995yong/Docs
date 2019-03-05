## 常见概念

1. 控制反转（IOC）

控制反转是一种设计思想。在传统的Java程序中，直接通过new关键字创建对象，这样导致程序的强耦合性。为了降低耦合，将对象的创建过程控制权交给了容器，由容器进行注入组合对象。而程序只是被动的接受对象

2. 依赖注入（DI)
3. 面向切面编程（AOP)









```flow
st=>start: Start
e=>end
op=>operation: 调用reflush()
cond=>condition: Yes or No?

st->op->cond
cond(yes)->e
cond(no)->op
```

