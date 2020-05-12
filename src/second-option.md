# Using Option
> 使用 Option

Particularly observant readers may have noticed that we actually reinvented
a really bad version of Option:
> 一些细心的读者可能注意到，我们其实在使用重新发明的设计不良的 Option：

```rust ,ignore
enum Link {
    Empty,
    More(Box<Node>),
}
```

Link is just `Option<Box<Node>>`. Now, it's nice not to have to write
`Option<Box<Node>>` everywhere, and unlike `pop`, we're not exposing this
to the outside world, so maybe it's fine. However Option has some *really
nice* methods that we've been manually implementing ourselves. Let's *not*
do that, and replace everything with Options. First, we'll do it naively
by just renaming everything to use Some and None:
>Link 其实就是 `Option<Box<Node>>`. 现在，好的是不需要把`Option<Box<Node>>`写的到处都是，
>与`pop`不同的是，我们没有 pub 它到外面，所以也许很好。不管怎样，使用 Option 都是一个极好的
>方式，虽然我们之前使用的都是我们自己的实现。首先，让我们天真的以仅仅重命名的方式使用 Some 和 None

```rust ,ignore
use std::mem;

pub struct List {
    head: Link,
}

// yay type aliases!
type Link = Option<Box<Node>>;

struct Node {
    elem: i32,
    next: Link,
}

impl List {
    pub fn new() -> Self {
        List { head: None }
    }

    pub fn push(&mut self, elem: i32) {
        let new_node = Box::new(Node {
            elem: elem,
            next: mem::replace(&mut self.head, None),
        });

        self.head = Some(new_node);
    }

    pub fn pop(&mut self) -> Option<i32> {
        match mem::replace(&mut self.head, None) {
            None => None,
            Some(node) => {
                self.head = node.next;
                Some(node.elem)
            }
        }
    }
}

impl Drop for List {
    fn drop(&mut self) {
        let mut cur_link = mem::replace(&mut self.head, None);
        while let Some(mut boxed_node) = cur_link {
            cur_link = mem::replace(&mut boxed_node.next, None);
        }
    }
}
```

This is marginally better, but the big wins will come from Option's methods.
> 这看起来更好一点，成功的地方在于我们使用了 Options

First, `mem::replace(&mut option, None)` is such an incredibly
common idiom that Option actually just went ahead and made it a method: `take`.
> 第一， `mem::replace(&mut option, None)` 真是有点难以置信我们使用这种方式，其实Option里有更好的方式
> 那就是 `take`.

```rust ,ignore
pub struct List {
    head: Link,
}

type Link = Option<Box<Node>>;

struct Node {
    elem: i32,
    next: Link,
}

impl List {
    pub fn new() -> Self {
        List { head: None }
    }

    pub fn push(&mut self, elem: i32) {
        let new_node = Box::new(Node {
            elem: elem,
            next: self.head.take(),
        });

        self.head = Some(new_node);
    }

    pub fn pop(&mut self) -> Option<i32> {
        match self.head.take() {
            None => None,
            Some(node) => {
                self.head = node.next;
                Some(node.elem)
            }
        }
    }
}

impl Drop for List {
    fn drop(&mut self) {
        let mut cur_link = self.head.take();
        while let Some(mut boxed_node) = cur_link {
            cur_link = boxed_node.next.take();
        }
    }
}
```

Second, `match option { None => None, Some(x) => Some(y) }` is such an
incredibly common idiom that it was called `map`. `map` takes a function to
execute on the `x` in the `Some(x)` to produce the `y` in `Some(y)`. We could
write a proper `fn` and pass it to `map`, but we'd much rather write what to
do *inline*.
> 第二，对于`match option { None => None, Some(x) => Some(y) }` 我们可以用 `map` 方法代替。
> `map`需要一个函数，并用从`Some(x)`中拿到的 x 当作参数去执行这个函数，在函数内产生 `y`
> 并返回 `Some(y)`。我们需要写一个函数 fn 传递给 map。但是我们宁愿写些什么做*inline*

The way to do this is with a *closure*. Closures are anonymous functions with
an extra super-power: they can refer to local variables *outside* the closure!
This makes them super useful for doing all sorts of conditional logic. The
only place we do a `match` is in `pop`, so let's just rewrite that:
> 做到这一点的方法是使用 *闭包*，闭包是一个匿名函数，强大的是它能引用外部的局部变量。
> 这使它们对于执行各种条件逻辑超级有用, 我们只要改以下pop内的match就可以了， 让我们改一下：
```rust ,ignore
pub fn pop(&mut self) -> Option<i32> {
    self.head.take().map(|node| {
        self.head = node.next;
        node.elem
    })
}
```

Ah, much better. Let's make sure we didn't break anything:
> 嗯，好多了。确保我们没有破坏任何东西, 测试一下：
```text
> cargo test

     Running target/debug/lists-5c71138492ad4b4a

running 2 tests
test first::test::basics ... ok
test second::test::basics ... ok

test result: ok. 2 passed; 0 failed; 0 ignored; 0 measured

```

Great! Let's move on to actually improving the code's *behaviour*.
