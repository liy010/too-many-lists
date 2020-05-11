# An Ok Singly-Linked Stack

In the previous chapter we wrote up a minimum viable singly-linked
stack. However there's a few design decisions that make it kind of sucky.
Let's make it less sucky. In doing so, we will:
> 在上一章中，我们写了一个可以运行的最小的单向链表栈，不管怎样，其中的一些代码设计决策使它有点糟糕。
> 为了让它看起来更好，我们将做下面这些工作：

* Deinvent the wheel
* Make our list able to handle any element type
* Add peeking
* Make our list iterable
> * 发明轮子
> * 使我们的列表能够处理任何元素类型
> * 
> * 让我们的链表可迭代

And in the process we'll learn about
> 在这个过程中，我门将学习以下知识

* Advanced Option use
* Generics
* Lifetimes
* Iterators
> * 高级的 Option 使用方式
> * 范型
> * 生命周期
> * 迭代器

Let's add a new file called `second.rs`:
> 在 src 文件夹下填加 second.rs 文件
```rust ,ignore
// in lib.rs

pub mod first;
pub mod second;
```

And copy everything from `first.rs` into it.
> 从上一章中的 first.rs 复制所有代码到本章的 first.rs 文件
