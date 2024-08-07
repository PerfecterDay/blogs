# 不可变类
{docsify-updated}
- [不可变类](#不可变类)
  - [Final](#final)
  - [好处](#好处)


不可变对象是一种在完全创建后内部状态保持不变的对象。这意味着，一旦将对象赋值给变量，我们就无法通过任何方式更新引用或更改内部状态。

### Final
Java 的 final 关键字保证基础类型的值不会改变，所有原始类型变量都是如此。
针对引用类型，可以保证引用关心不可变，也就是说引用只能指向同一个对象，但是对象的状态能否改变取决于对象本身。

### 好处
1. 线程安全，由于不可变对象的内部状态在时间上保持不变，因此我们可以在多个线程之间安全地共享它。
2. 可以自由地使用它，而引用它的对象都不会察觉到任何不同，因此我们可以说不可变对象是没有副作用的。
