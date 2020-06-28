+++
title = "An introduction to Rust implementing a functional list"
description = "In this post I'll implement a functional linked list using the Rust programming language. The goal of it is to provide an overview and explanatory view of the language, so I won't be focusing much in the data structure itself. Anyway, I hope you have some fun reading it =D"
date = 2020-06-27
+++

## Introduction

Before diving into any Rust code, we need to learn two fundamental concepts that make Rust unique:

### Ownership:
  - Each value in Rust has a variable that's called its owner.
  - There can only be one owner at a time.
  - When the owner goes out of scope, the value is dropped.

  ```
  let n = String::from("hello, world");   // 'n' owns the string 'hello, world'
  let m = n;                              // now 'm' owns the string, what about n?
  println!("{}", m);                      // prints 'hello, world'
  println!("{}", n);                      // **compilation error**
  ```

### Borrowing:
  - At any given time, you can either have one mutable reference or any number of immutable references.
  ```
    let mut a = String::from("hello, world");
    let b = &mut a;         // mutable reference to 'a'
    let c = &a;             // immutable reference to 'a'
    println!("{}", c);      // prints 'hello, world'
    println!("{}", b);      // **compilation error**
  ```
  - References must always be valid (lifetimes).
  ```
    let b;
    {
        let a = String::from("hello, world");
        b = &a;         // b is a reference to a
    }                   // a dies here, so b is invalid
    println!("{}", b);  // **compilation error**
  ```
## Hands on coding

With this out of our way, we can start writing our List implementation.

First let's create our project using `cargo`, Rust's packager manager, with the command:

```
cargo init --lib list
```

This will generate a startup workspace to get the us running. Then open the file `src/lib.rs` which is the root of our crate (think of `crate` as a `package` for now), put in the following code to export our about to be created module:

```
// src/lib.rs
pub mod list;
```

Now we can create a `list.rs` file in the `src` folder and start to write our code.

Let's first define our `Node` struct, the semantics of a struct in Rust is pretty similar to C/C++.

```
#[derive(Debug)]      // that's just a macro attribute that provide us default trait implementation (more on this later)
pub struct Node<T> {  // we'll be building a generic list
  elem: T,
  next: ??            // what should we put here?
}
```

Well, let's try to follow our first-glance intuition and write something like this:

```
#[derive(Debug)]      
pub struct Node<T> {
  elem: T,
  next: Node<T>
}
```

When compiling we get:
```
error[E0072]: recursive type `Node` has infinite size
  --> src/main.rs:8:1
   |
8  | pub struct Node<T> {
   | ^^^^^^^^^^^^^^^^^^ recursive type has infinite size
9  |     elem: T,
10 |     next: Node<T>,
   |     ------------- recursive without indirection
   |
   = help: insert indirection (e.g., a `Box`, `Rc`, or `&`) at some point to make `Node` representable
```

It looks like we can't have a recursive definition of `Node`, that because the compiler must know the size of the struct to allocate the amount of memory needed for it. How to solve it?

## References, Lifetimes and Box

The compiler have already told us how! We need indirection. If we use a pointer-like value in the `next` field of our struct the compiler will know exactly the amount of memory needed because pointers have a fixed size. So let's hurry up and put a reference in there:

```
#[derive(Debug)]      
pub struct Node<T> {
  elem: T,
  next: &Node<T>      // '&' is the syntax for an immutable reference
}
```

If we run we get:

```
error[E0106]: missing lifetime specifier
  --> src/main.rs:10:11
   |
10 |     next: &Node<T>,
   |           ^ expected named lifetime parameter
   |
help: consider introducing a named lifetime parameter
   |
8  | pub struct Node<'a, T> {
9  |     elem: T,
10 |     next: &'a Node<T>,
   |
```

Ok now there's this new concept called `lifetime parameter`, let's see what it is.

Remember in the "Borrowing" session when we said that "references must always be valid", well that's accomplished by lifetime parameters. Most of the time the Rust compiler will infer the lifetime parameters (check out the 'Lifetime Elision' session in the [Rust Book](https://doc.rust-lang.org/nomicon/lifetime-elision.html)), however there are some special cases when we need to explicitly tell it about them. Here's how we do it:

```
// implicit
fn foo(x: &i32) {
}

// explicit
fn bar<'a>(x: &'a i32) {  // 'a' is just a name for the parameter
}
```

In our case, it happens because, thus far, Rust has preferred to err on the side of explicitness, modulo a few areas that were deemed an ergonomic hit without some implicitness. There’s been some talk about eliding lifetimes in more places (check [this](https://internals.rust-lang.org/t/lang-team-minutes-elision-2-0/5182) out).

Then, following the compiler suggestion, let's change our code to:

```
#[derive(Debug)]
pub struct Node<'a, T> {    // that's the syntax to declare a lifetime parameter for a struct
  elem: T,
  next: &'a Node<'a, T>,  // we say that if the 'next' field lives 'a' then our struct must live at least 'a'
}
```

The code compiles! Great! However, we can make it better. Sometimes things can get too complicated with references and lifetime parameters, a better solution would be using the heap, the simplest way to do this in Rust is with the `Box` pointer. From the Rust official documentation:

_"Box<T>, casually referred to as a 'box', provides the simplest form of heap allocation in Rust. Boxes provide ownership for this allocation, and drop their contents when they go out of scope."_

So let's use it:

```
#[derive(Debug)]
pub struct Node<T> {
  elem: T,
  next: Box<Node<T>>,
}
```

Done! We now say that our `Node` is a recursively defined struct, but with a fixed size. There's just one problem with that: how do we know when we have reached the end or our list? Rust doesn't have a `NULL` pointer as we would use in C/C++, but it has something way more sophisticated, the `Option` enum. It is defined as:

```
pub enum Option<T> {
  None,
  Some(T),
}
```

In Rust, enums represent one of many possible variants. Many languages support an enum syntax that works like this. Rust lets us go a bit further and add data associated with each of the variants. In our example, we have a generic enum, which means that for some type T, our enum may be a `None` variant or a `Some` variant carrying a value of type `T`.

With data added to variants, this style of enum takes on many of the properties of union types in other languages. The combination of enum and union-like behavior is why this sort of type is often referred to as a “tagged union”. In type theory, it’s referred to as a sum type, and is one of many algebraic types such as product types and quotient types.

Let's use this new type in our `Node` struct:

```
#[derive(Debug)]
pub struct Node<T> {
  elem: T,
  next: Box<Option<Node<T>>>,
}
```

Cool huh? We just added the following semantics to our code: if the `next` field is the enum variant `None` then we've reached the end of our list.

This whole `Box<Option<Node<T>>>` is a little annoying to look at though, there's something called `type` aliases in Rust, we can use just as we would in our bash terminal.

```
type Link<T> = Option<Box<Node<T>>>;

#[derive(Debug)]
pub struct Node<T> {
  elem: T,
  next: Link<T>,
}
```

Perfect!

## Creating an API

Now that we have our basic structure to work with recursive lists we can add some behavior to it.

What about we creating our `List` struct now? As we're building a simple linked list we can simply define our list as:

```
pub struct List<T> {
  head: Link<T>,
}
```

Let's define some basic operations:

```
impl<T> List<T> {
  pub fn new() -> Self {}
  pub fn push(&mut self, elem: T) {}
  pub fn peek(&self) -> Option<&T> {}
  pub fn pop(&mut self) -> Option<T> {}
}
```

Ok, that's a lot of new code, let's understand each of these lines.

 - `impl<T> List<T>`: the `impl` keyword is used to define implementation on types, here we are using a generic implementation `impl<T>` for a generic `List<T>`.

 - `pub fn new() -> Self {}`: defining a static method (note that it doesn't use the `self` parameter as the other ones) that returns `Self` which is the implementing type of our `impl` block, for example it can be `List<i32>`, `List<String>` and so on.

 - I'll assume the other methods are straightforward, the `self` keyword behaves just as a `this` pointer in C/C++.


If we try to compile this code it won't work, that because the Rust compiler expect us to return an `Option` value from the last two methods. So let's write the body of our methods:

```
impl<T> List<T> {
  pub fn new() -> Self {
    List { head: None } // that's basically our constructor
  }

  pub fn push(&mut self, elem: T) {
    let node = Box::new(Node {
      elem,
      next: self.head.take(),   // the `take` method takes the value out of the option, leaving a `None` in its place
    });
    self.head = Some(node);
  }

  pub fn peek(&self) -> Option<&T> {  // what happens if we return the value instead of a reference?
    self.head.as_ref().map(|node| &node.elem) // the `map` method maps an Option<T> to Option<U> by applying a function to a contained value
  }

  pub fn pop(&mut self) -> Option<T> {
    self.head.take().map(|node| {
      self.head = node.next;
      node.elem
    })
  }
}
```

We've finally used Rust closures for the first time. Closures are fundamentally a pointer to code together with a pointer to a table of free variables, it allows us to treat functions as first-class citizens. In Rust we can define them by the simple syntax `| vars | { code }` or we can optionally omit the enclosing curly brackets for a single expression `| vars | expr`.

## Testing

OK, so our basic implementation is supposed to work. But how do we test it?

Rust let us write testing functions, unit tests go into a tests mod with the `#[cfg(test)]` attribute. Test functions are marked with the `#[test]` attribute. Let's write some:

```
#[cfg(test)]
mod test {
  use super::List;

  #[test]
  fn basics() {
    let mut list = List::new();

    // Check empty list behaves right
    assert_eq!(list.pop(), None);

    // Populate list
    list.push(1);
    list.push(2);
    list.push(3);

    // Check normal removal
    assert_eq!(list.pop(), Some(3));
    assert_eq!(list.pop(), Some(2));

    // Push some more just to make sure nothing's corrupted
    list.push(4);
    list.push(5);

    // Check normal removal
    assert_eq!(list.pop(), Some(5));
    assert_eq!(list.pop(), Some(4));

    // Check exhaustion
    assert_eq!(list.pop(), Some(1));
    assert_eq!(list.pop(), None);
  }
}
```

Here everything is pretty straightforward right? There's just this `assert_eq!` we still didn't talk about, but think about it as a function that takes two parameters and checks if they're equal;

## Cleaning memory and Traits

Do we need to worry about cleaning up our list? Well, no, the Rust ownership system handles it to us automatically. However, we may have some problems if we just let the Rust compiler to handle it.

The problem here is with our recursive definition. When dropping the `Box` pointer, first we need to drop it's content (which is recursive) and then deallocating the pointer itself, because of that, we can't have a tail recursive optimization and, therefore, we might blow the stack for really big lists.

To work around it we have to implement an iterative "destructor" for our data structure. This can be accomplished by the `Drop` trait available in the `std`.

So what is a trait? For now, think about it as an `interface` in Java or C++ (or if you're a Haskeller we can trace a better comparison with `type classes`). This is how we implement a trait for our list:

```
impl<T> Drop for List<T> {
  fn drop(&mut self) {
    let mut cur_link = self.head.take();
    while let Some(mut boxed_node) = cur_link {
      cur_link = boxed_node.next.take();
      // boxed_node goes out of scope and gets dropped here;
    }
  }
}
```

## Shared Ownership and an Immutable List

What if we want our list to be immutable? Like those used in functional language like Haskell or ML.

Let's look at a simple example:
```
list1 = A -> B -> C -> D
list2 = tail(list1) = B -> C -> D
list3 = push(list2, X) = X -> B -> C -> D
```

In this case, the memory should look like this:
```
list1 -> A ---+
              |
              v
list2 ------> B -> C -> D
              ^
              |
list3 -> X ---+
```

This won't work with `Box` pointers anymore, in this case the ownership of B is shared. A basic implementation of shared ownership in Rust is done by reference counting, in other words, using the `Rc` pointer.

First of all let's rewrite our interface as the methods `push` and `pop` doesn't make much sense now.

```
impl<T> List<T> {
  pub fn new() -> Self {}
}
pub fn head(list: &List<T>) -> Option<&T> {}
pub fn tail(list: &List<T>) -> List<T> {}
pub fn cons(list: &List<T>, elem: T) -> List<T> {}
```

Note that we've given up on using `self` because we'll get a functional style composing these defined functions. Now let's introduce our `Rc` pointer in our `Link` type:

```
use std::rc::Rc;

type Link<T> = Option<Rc<Node<T>>>;
```

The semantics of the pointer is pretty simple, it keeps an internal counter and every time we call `.clone()` it increases the pointer by one. When the counter gets to 0 the value is dropped.

Let's implement our methods:

```
impl<T> List<T> {
  pub fn new() -> Self {
    List { head: None }
  }
}

pub fn head<T>(list: &List<T>) -> Option<&T> {
  list.head.as_ref().map(|node| &node.elem)
}

pub fn tail<T>(list: &List<T>) -> List<T> {
  List {
    head: list.head.as_ref().and_then(|node| node.next.clone()),
  }
}

pub fn cons<T>(list: &List<T>, elem: T) -> List<T> {
  List {
    head: Some(Rc::new(Node {
      elem,
      next: list.head.clone(),
    })),
  }
}
```

The code is, again, straightforward, so I'll leave you to read it by yourself.

Now let's test it!

```
#[cfg(test)]
mod test {
  use super::*;

  #[test]
  fn basics() {
    let list: List<i32> = List::new();
    assert_eq!(head(&list), None);

    let list = cons(&cons(&cons(&List::new(), 1), 2), 3);
    assert_eq!(head(&list), Some(&3));

    let list = tail(&list);
    assert_eq!(head(&list), Some(&2));

    let list = tail(&list);
    assert_eq!(head(&list), Some(&1));

    let list = tail(&list);
    assert_eq!(head(&list), None);
  }
}

```

Last we have to define our iterative drop:

```
impl<T> Drop for List<T> {
  fn drop(&mut self) {
    let mut head = self.head.take();
    while let Some(node) = head {
      // unwraps if it's unique
      match Rc::try_unwrap(node) {
        Ok(mut node) => {
          head = node.next.take();
        }
        Err(_) => {
          break;
        }
      }
    }
  }
}
```

The only unknown operator here is the `match`, it's used in Rust to simulate patter matching just like in functional languages.

## Iterator and Append

To finish the tutorial, let's implement two more cool functionalities to our list. 

An iterator is a pattern that abstract the logic of transversing in a container, particularly lists. In Rust std library we have a `Iterator` trait that helps us implementing the patter, let's take a look.


```
pub trait Iterator {
  type Item;
  fn next(&mut self) -> Option<Self::Item>;
}
```

The new kid on the block here is `type Item`. This is declaring that every implementation of Iterator has an associated type called Item. In this case, this is the type that it can spit out when you call `next`. Here it goes our implementation:

```
pub struct Iter<'a, T> {
  next: Option<&'a Node<T>>,
}

impl<'a, T> Iterator for Iter<'a, T> {
  type Item = &'a T;

  fn next(&mut self) -> Option<Self::Item> {
    self.next.map(|node| {
      self.next = node.next.as_ref().map(|node| &**node);
      &node.elem
    })
  }
}
```

And here are some tests:

```
#[test]
fn iter_t() {
  let list = cons(&cons(&cons(&List::new(), 1), 2), 3);

  let mut iter = iter(&list);
  assert_eq!(iter.next(), Some(&3));
  assert_eq!(iter.next(), Some(&2));
  assert_eq!(iter.next(), Some(&1));
}
```

Now let's build our last piece of the puzzle, the append function. Its semantics is supposed to be quite simple, given two `List<T>` return a new `List<T>` which is the result of appending the final of the first list to the beginning of the second. Let's code it:

```
pub fn append<T>(l1: &List<T>, l2: &List<T>) -> List<T>
where
  T: Clone,
{
  match head(l1) {
    None => List {
      head: l2.head.clone(),
    },
    Some(e) => {
      let mut l = tail(&l1);
      l = append(&l, l2);
      cons(&l, e.clone())
    }
  }
}
```

You should be able to understand the whole code by now, except for the `ẁhere` clause. What it does is basically set a trait bound to the generic parameter `T`, which means that every possible `T` must implement a trait called `Clone`, in fact, we need to clone some elements in order to append the two lists. If you know Haskell, it's as simple as defining a type class bound, for example: `Clone a => List a -> List a -> List a`.

That's how we test this new function:

```
#[test]
fn append_t() {
  let l1 = cons(&cons(&List::new(), 1), 2);
  let l2 = cons(&cons(&List::new(), 5), 6);
  let list = append(&l1, &l2);

  let mut iter = iter(&list);
  assert_eq!(iter.next(), Some(&2));
  assert_eq!(iter.next(), Some(&1));
  assert_eq!(iter.next(), Some(&6));
  assert_eq!(iter.next(), Some(&5));
}
```

And that's it !!! You did build a functional linked list using an imperative (multi paradigm) language!!! If you've read this until the end, I hope you have enjoyed it. See you in the next post if you're sticking around.

Here's the full code for reference:

```
use std::rc::Rc;

type Link<T> = Option<Rc<Node<T>>>;

#[derive(Debug)]
pub struct List<T> {
  head: Link<T>,
}

#[derive(Debug)]
struct Node<T> {
  elem: T,
  next: Link<T>,
}

impl<T> List<T> {
  pub fn new() -> Self {
    List { head: None }
  }
}

pub fn head<T>(list: &List<T>) -> Option<&T> {
  list.head.as_ref().map(|node| &node.elem)
}

pub fn tail<T>(list: &List<T>) -> List<T> {
  List {
    head: list.head.as_ref().and_then(|node| node.next.clone()),
  }
}

pub fn cons<T>(list: &List<T>, elem: T) -> List<T> {
  List {
    head: Some(Rc::new(Node {
      elem,
      next: list.head.clone(),
    })),
  }
}

pub fn append<T>(l1: &List<T>, l2: &List<T>) -> List<T>
where
  T: Clone,
{
  match head(l1) {
    None => List {
      head: l2.head.clone(),
    },
    Some(e) => {
      let mut l = tail(&l1);
      l = append(&l, l2);
      cons(&l, e.clone())
    }
  }
}

pub fn iter<T>(list: &List<T>) -> Iter<T> {
  Iter {
    next: list.head.as_ref().map(|node| &**node),
  }
}

impl<T> Drop for List<T> {
  fn drop(&mut self) {
    let mut head = self.head.take();
    while let Some(node) = head {
      match Rc::try_unwrap(node) {
        // unwraps if it's unique
        Ok(mut node) => {
          head = node.next.take();
        }
        Err(_) => {
          break;
        }
      }
    }
  }
}

pub struct Iter<'a, T> {
  next: Option<&'a Node<T>>,
}

impl<'a, T> Iterator for Iter<'a, T> {
  type Item = &'a T;

  fn next(&mut self) -> Option<Self::Item> {
    self.next.map(|node| {
      self.next = node.next.as_ref().map(|node| &**node);
      &node.elem
    })
  }
}

#[cfg(test)]
mod test {
  use super::*;

  #[test]
  fn basics() {
    let list: List<i32> = List::new();
    assert_eq!(head(&list), None);

    let list = cons(&cons(&cons(&List::new(), 1), 2), 3);
    assert_eq!(head(&list), Some(&3));

    let list = tail(&list);
    assert_eq!(head(&list), Some(&2));

    let list = tail(&list);
    assert_eq!(head(&list), Some(&1));

    let list = tail(&list);
    assert_eq!(head(&list), None);
  }

  #[test]
  fn iter_t() {
    let list = cons(&cons(&cons(&List::new(), 1), 2), 3);

    let mut iter = iter(&list);
    assert_eq!(iter.next(), Some(&3));
    assert_eq!(iter.next(), Some(&2));
    assert_eq!(iter.next(), Some(&1));
  }

  #[test]
  fn append_t() {
    let l1 = cons(&cons(&List::new(), 1), 2);
    let l2 = cons(&cons(&List::new(), 5), 6);
    let list = append(&l1, &l2);

    let mut iter = iter(&list);
    assert_eq!(iter.next(), Some(&2));
    assert_eq!(iter.next(), Some(&1));
    assert_eq!(iter.next(), Some(&6));
    assert_eq!(iter.next(), Some(&5));
  }
}

```