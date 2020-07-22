+++
title = "[WIP] Study notes: Category Theory"
description = "This report is supposed to hold a summary of my studies in Category Theory. I'll be using the material of the professor Bartosz Milewski, it can be found at his website, feel encouraged to check it out (just google it and you'll find out)."
date = 2020-07-05
+++

A category is just a bunch of dots (**objects**) connected with arrows (**morphisms**) in a specific way to meet some axioms/conditions.

There must be:

- Composition
- Identity
- Associativity

## Composition

Properties of Composition

 - associative: h ∘ (g ∘ f ) = (h ∘ g) ∘ f = h ∘ g ∘ f
 - identity: an arrow which is a unit of composition, f ∘ id A = f and id B ∘ f = f

## Isomorphism

Let two morphisms `f` and `g` in any category such that:

```
f :: a -> b
g :: b -> a
```

 - f is an isomorphism if there exist g such that:

 g . f = id_a
 f . g = id_b

### Inverse function

In category theory we can't look inside of sets, so we can't tell if a function is injective or surjective by looking at the object, but...

 - injective function = monic -> monomorphism
 - surjective function = epic -> epimorphism

These are defined for any category, let's look at what they are.
 
Suppose objects A, B and C:

  - If for every object C and every pair of morphisms `g1, g2 :: B -> C` such that for a morfism `f :: A->B`, if `g1.f = g2.f` then `g1 = g2`, then `f` is an **epimorphism** (surjective)
  - If for every object C and every pair of morphisms `g1, g2 :: C -> A` such that for a morfism `f :: A->B`, if `f.g1 = f.g2` then `g1 = g2`, then `f` is a **monomorphism** (surjective)

* OBS: it's not true that if a morfism is epi and mono then it's an isomorphism (in set theory this is true, but here it's not).

## Examples of Categories

### No Objects

A category with no objects, and therefore, no morphisms. This is pretty useless in itself. But it make sense in a context, for example, the category of all categories.

### Simple Graphs

Start with a directed graph and keep adding stuff until you make it into a category:

- add identity arrows
- for any composable pair of arrows add a composition arrow
- for any added arrow, consider its composition with others

Such process is called a **free construction**, which is the process of completing a given structure by extending it with a minimum number of items to satisfy a property.

### Orders

Let's have a quick look at order theory.

First, can an order be defined in terms of a category? Look at it this way, the category where morphisms are relations between objects: less than or equal. Let's check the conditions:

 - identity? a <= a
 - composition? a <= b <= c -> a <= c
 - associativity? yes

#### Hom-set

Before get into examples let's define a **hom-set**. It's nothing more than a set of morphisms between two objects, say a and b, then it's written as C(a, b) (or Hom_C(a, b)).

#### Preorders

A binary relation <= on a set X is a **preorder** iff the relation <= is both **reflexive** (for all a, a R a) and **transitive** (for all a,b and c, a R b and b R c -> a R c).

We can think of reflexivity and transitivity in terms of identity and composition. Indeed a preorder is a category.

A preorder is a category where theres 0 or 1 morphism between every pair of objects. We call such category a **thin** category. A **preorder is a thin category**. Therefore, every hom-set in a preorder is either empty or a singleton. You may, however, have cycles in a preorder.

**The counter example**: The preorder (or a thin category) is an interesting category because every morphisms is an epimorphism and a monomorphism, BUT, in a thin category a bimorphism is not necessarily invertible (i.e. an isomorphism).

#### Equivalence Relations

An **equivalence relation** is a preorder that is also symmetric. A binary relation R is **symmetric** if a R b -> b R a.

An equivalence relation ≡ divides a set S into **equivalence classes**. The equivalence class of an element x is [x]≡ = {y | x ≡ y}.

* Every preorder <= induces a natural equivalence relation  ≡_<=:
 - a ≡_<= b iff a <= b and b <= a

#### Partial Order

A **partial order** is a preorder that is antisymmetric. A binary relation R is **antisymmetric** if a R b and b R a implies a = b. Partial orders can't have cycles, you can look at it as a DAG (Directed Acyclic Graph).

A **partially ordered set** or **poset** is a set X equipped with a partial order <=, often described as the pair (X, <=).

In Haskell, we can define a partial order through type classes:

```
class PartialOrder t where
 (⊑) :: t -> t -> Bool
```

#### Total Orders

A **total order** <= is a binary relation that is **total**, **transitive** and **antisymmetric**.

A binary relation R is total if for any two values a and b, a R b or b R a.

Totality implies reflexivity, which means that every total order is also a **partial order**.

### Monoid

Monoid is a really simple but extremely powerful concept. It's quite influent in arithmetics (addition and multiplication are monoids), but also in programming. Strings, lists, foldable data structures, futures in concurrent programming, and events in functional reactive programming are all examples of monoids.

#### Monoid as a Set

Monoid is defined as a set with a binary operation. We require that the operation must be associative and there must be one special unit (identity) element.

Examples:

 - Natural numbers with zero form a monoid under adition.
 - String's a monoid under string concatenation (a non comutative one)

Here's a declaration of a `Monoid` typeclass in Haskell

```
class Monoid m where
    mempty  :: m
    mappend :: m -> m -> m
```

* It's responsibility of the programmer to make sure the monoidal properties are satisfied.

#### Monoid as a Category

Every monoid can be described as a single object category with a set of morphisms that follow appropriate rules of composition.

For examples we can think of an object as a set of natural numbers and the set of morphisms as "adders". The identity morphism would be a 0-adder. So a composition of the 5-add with the 7-adder would be a 12-adder.

For every monoidal category M, we have a hom-set M(m, m) of the single object m in the category. We can easily define a binary operator in this set. The monoidal product of two set-elements is the element corresponding to the composition of the corresponding elements. The composition always exist because the source and the target object are the same. And the identity morphism is the neutral element of this product.

Clearly there's a one-to-one correspondence between Monoidal categories and sets.


### Kleisi Categories

Let's see how to model non-pure functions, or side effects in category theory. For example functions that log their execution, something that would likely be implemented by mutating some global state:

```
string logger;

bool negate ( bool b) {
  logger += "Not so! ";
  return ! b;
}
```

We can make this function pure by using pairs returning a log string:

```
pair <bool , string > negate( bool b) {
  return make_pair( ! b, "Not so! ");
}
```

The idea is that the log will be aggregated between function calls. Here's a more practical example, suppose we want to create a function to upper case strings while still having a log property. First, let's create a `Writer` template to enpasulate the `embellishment` of the function:

```
template<class A>
using Writer = pair < A, string > ;
```

And here's our embellished functions:

```
Writer < string > toUpper(string s) {
  string result;
  int ( * toupperp)( int ) = & toupper;
  transform(begin(s), end(s), back_inserter(result), toupperp);
  return make_pair (result, "toUpper ");
}

Writer < vector < string >> toWords(string s) {
  return make_pair(words(s), "toWords ");
}
```

What if we want to compose these two functions?

```
Writer < vector < string >> process(string s) {
  auto p1 = toUpper(s);
  auto p2 = toWords(p1.first);
  return make_pair (p2.first, p1.second + p2.second);
}
```

Now the aggregation does not depend on a global state, instead, they're produced locally and externally concatenated into a larger log.

A fact is that writing a program in this style would be a pain in the ass with a lot of repetitive code. A way to work around it is to abstract function composition itself. Before we do that, we should take a look at a category theory perspective of this problem.

#### The Writer Category

Here we're still talking about categories of types and functions, however, instead a normal composition function, we now have an embellished element. So we have to redefine our composition abstraction and that's how we do it:

```
auto const compose = []( auto m1, auto m2) {
  return [m1, m2]( auto x) {
    auto p1 = m1(x);
    auto p2 = m2(p1.first);
    return make_pair (p2.first, p1.second + p2.second);
  };
};
```

*Obs: the above code uses C++14 features such as generalized lambda functions with return type deduction.

So now we have our category with newly defined composition. Another question is: what are our identity functions?

```
template<class A> Writer < A > identity(A x) {
  return make_pair(x, "");
}
```

This category is also trivially associative (composition + string concatenation), so we indeed have a category here. In fact, we could've changed the string concatenation for any other monoid, we just have to replace **mappend** inside compose and **mempty** inside identity (in place of + and "" ).

* A good library writer should be able to identify the bare minimum of constraints that make the library work — here the logging library’s only requirement is that the log have monoidal properties.

#### Writer in Haskell

Here's the same code we defined above, but in Haskell.

```
type Writer a = (a, String )

( >=> ) :: (a -> Writer b) -> (b -> Writer c) -> (a -> Writer c)
m1 >=> m2 = \ x ->
  let (y, s1) = m1 x
    (z, s2) = m2 y
  in (z, s1 ++ s2)

return :: a -> Writer a -- the identity function
return x = (x, "")

upCase :: String -> Writer String
upCase s = (map toUpper s, "upCase ")

toWords :: String -> Writer [ String ]
toWords s = (words s, "toWords ")

process :: String -> Writer [ String ]
process = upCase >=> toWords
```

In fact, things are a little more terse and we get a lot of help from the compiler.

#### Kleisli Categories

The category we just defined is called `Kleisli Category`, a category based on a monad.

We can say that each Kleisli category defines its own way of composing morphisms with embellishment, as well as the identity morphisms with respect to that composition.

In particular, our examples used a writer monad for logging the execution of functions. It's also an example of a more general mechanism for embedding effects in pure computations.

### Products and Coproducts

#### Universal Construction

There's a common construction in category theory called the **universal construction** for defining objects in terms of their relationship. It basically boils down to a two-step process:

 - Pick a pattern in terms of objects and morphisms and find all the matching elements in the category.
 - Establish a kind of ranking between those hits and find the best fit.

#### Initial Object

Consider the following example for the above method: think about the pattern as an object and the ranking relation as `a` is more initial than `b` if there's a single morphism a -> b. Then we may have a "best" object (there's no guarantee that this object exist) which we define as an **initial object**.

The **initial object** is the object that has one and only one morphism going to any object in the category.

This definition guarantees an **uniqueness up to isomorphism** on the initial objects (there's no uniqueness guarantee). Some examples of such objects are the `least element` in a poset (partially ordered set) and an empty set in a category made of sets and functions. Notice that on a poset there's no guarantee of the existence of a initial object (for example in the set of all positive integers).

#### Terminal Objects

Consider now the same pattern as before, but changing the ranking relation by reverting the direction. In other words, an object `a` is more terminal than `b` if there's a single morphism b -> a.

The **terminal object** is the object with one and only one morphism coming to it from any object in the category.

Again we have an uniqueness up to isomorphisms. As an example, in a poset, the terminal object, if it exists, is the biggest object as well as in the set category, the terminal object is a singleton (set with only one element).

#### Duality

Duality is a very important property in category theory. For every constructing you may come up with, there will be an opposite in which you can get by reversing the arrows and redefining the compositions.

Such opposite categories are often prefixed with "co", so you have products and coproducts, monads and comonads and so on.

Given this definition, it's clear enough that initial and terminal objects are in a duality relationship, as in the universal construction process we just inverted the morphisms in each ranking relation.

#### Isomorphisms

As for an intuition, we can think as isomorphism as a one-to-one mapping between two objects, such that every part of one object has a correspondence in the other one.

In category theory, isomorphism means invertible morphisms, so a formal definition in term of composition and identity can be thought as:

```
f . g = id
g . f = id
```

When we said that initial and terminal objects are unique up to isomorphism, we actually meant that they're unique up to unique isomorphism. In principle, there can be many isomorphisms between two objects (think about {true, false}, {black, white}), but for the initial and terminal objects, the uniqueness property of the f and g morphisms guarantees a unique isomorphism for any two objects (see proof in the book).

In fact, this “uniqueness up to unique isomorphism” is the important property of all universal constructions.

#### Product

A cartesian product of any two sets is a set of pairs. In categories, we can define a pattern to generalize the product definition using universal construction.

All we can say, in the categorical world, that there are two functions, the projections, from the product to each of the constituents (in Haskell they're `fst` and `snd`).

Now, a pattern for choosing candidate objects is an object `c` and two morphisms `p` and `q` such that:

```
p :: c -> a
q :: c -> b
```

The ranking relation can be thought as: given two candidates, let it be (c, p, q) and (c', p', q'), we would like to say that c is "better" than c' if there's a morphism m from c' to c and, therefore, the projections p' and q' such that:

```
p’ = p . m
q’ = q . m
```

We can also say that m factorizes p' and q', for examples on how m shrinks information from any other product candidate check out the book section.

We should notice as well that a product is also unique up to unique isomorphism.

A high order function that produces the factorizing function m from two candidates is sometimes called the **factorizer**.

#### Coproduct

As a dual of Product, when we reverse the morphisms we end up with two **injections**, let's call them i and j:

```
i :: a -> c
j :: b -> c
```

The ranking relation is also inverted, so object c is "better" than c' if there's an unique morphism m from c to c' that factorizes the injections:

```
i' = m . i
j' = m . j
```

As in Products, the best object is called coproduct and, if it exists, it's unique up to unique isomorphism.

In the category of sets a coproduct is a **disjoint union** of two sets. In a category of types, it's called a **tagged union** or **variant** of two types.

In C++ the unions are untagged, so we must keep track of the union member, one could implement a tagged union using enumeration like this:

```
struct Contact {
    enum { isPhone, isEmail } tag;
    union { int phoneNum; char const * emailAddr; };
};
```

And then we can implement the projections as constructors or functions:

```
Contact PhoneNum(int n) {
    Contact c;
    c.tag = isPhone;
    c.phoneNum = n;
    return c;
}
```

In Haskell things are way simpler to define a sum types, we just use a `data` keyword:

```
data Contact = PhoneNum Int | EmailAddr String
```
#### Algebraic Data Types

Product and sum types can be really powerful when combined (composition). We have two commutative monoidal structures underlying the type system:

 - Sum type with `Void` as the neutral element
 - Product type with the unit type `()` as the neutral element

We can think of them as analogous to addition and multiplication where `Void` = 0 and `()` = 1. Let's look at some algebraic properties and how we'd implement them by using types:

 - a*0 = 0 --> `(a, Void) = Void`
 - a * (b + c) = a * b + a * c --> `(a, Either b c) = Either (a, b) (a, c)`

In fact the equality holds up to an isomorphism. We can implement functions to convert them from one side to another and these functions will be the inverse of each other.

We can call this structure a **ring**, which consists of a set equipped with two binary operations that generalize the arithmetic operations of addition and multiplication. However, a true ring must have an additive inverse, in our case we don't actually have it, we do have sum (sum types) but no subtraction. The given name for such structure is **semiring** or **rig**.

The list type is quite interesting, because it’s defined as a solution to an equation. The type we are defining appears on both sides of the equation:

```
List a = Nil | Cons a (List a)
```

If we do our usual substitutions, and also replace List a with x, we get the equation: `x = 1 + a * x`.

We can’t solve it using traditional algebraic methods because we can’t subtract or divide types. But we can try a series of substitutions, where we keep replacing x on the right hand side with (1 + a*x), and use the distributive property. This leads to the following series:

```
x = 1 + a*x
x = 1 + a*(1 + a*x) = 1 + a + a*a*x
x = 1 + a + a*a*(1 + a*x) = 1 + a + a*a + a*a*a*x
...
x = 1 + a + a*a + a*a*a + a*a*a*a...
```

We end up with an infinite sum of products (tuples), which is a geometric series.

One thing worth mentioning is that there's also a mapping between types and a semiring formed by logigal **or** and **and** operators. This analogy goes deeper, and is the basis of the Curry-Howard isomorphism between logic and type theory. 

### Functors

A functor is just a mapping that preserves structure between categories. Structures on categories are morphisms and compositions. So a functor between two categories must map every object as well as every morphism, preserving the connections.

Example:

Say we have categories C and D, then for every morphism in C `f :: a -> b`, in D we must have a mapping `F f :: F a -> F b`.

Also, if h is a composition `h = g . f`, then `F h = F g . F f` must be a composition of the two images under the functor F.

Finally, the identity must also be mapped to identities: `F id_a = id_F a`.

These conditions make functors very restrictive, they may smash objects together, it may glue multiple morphisms into one, but it may never break things apart.

In programming, we can think about functors mapping the categories of types to itself, such functors are called **endofunctors**.

#### The Maybe functor

`Maybe` is a type constructor, so it maps types to types. To make it a functor we must also map `morphisms` (functions). So, for any function `f :: a -> b` we want to produce a function `f' :: Maybe a -> Maybe b`, and we do it by implementing `fmap`. We say that `fmap` lifts a function, in this case to a `Maybe` value.

To show that `Maybe` is actually a functor we must prove the functors laws (that means, that it preserves the structure), they are: composition and identity. To do that we're going to use *equation reasoning*.

#### Equational Reasoning

This is a common proof technique in haskell, we can use it because Haskell is pure, therefore we can inline and refactor a function application without changing the program's semantics.

For the identity, let's prove `fmap id = id`:

```
fmap id Nothing ==inlining== Nothing ==refactoring== id Nothing

fmap id (Just x) ==inlining== Just (id x) ==inlining== Just x ==refactoring== id (Just x)
```

For the composition we must prove `fmap (g . f) = fmap g . fmap f`:

```
fmap (g . f) Nothing ==inlining== Nothing ==refactoring== fmap g Nothing ==refactoring== fmap g (fmap f Nothing)

fmap (g . f) (Just x) ==inlining== Just ((g . f) x) ==inlining== Just (g (f x)) ==refactoring== fmap g (Just (f x)) ==refactoring== fmap g (fmap f (Just x)) ==refactoring== (fmap g . fmap f) (Just x)
```

#### Functors as Containers

Functors can be thought as containers with values inside. However, this intuition may seems to not embrace some specific functors. In fact, in Haskell, functions can be memoized, so we can turn it into a table lookup as there's no side effects. This might seem impractical as functions may have infinite domains and codomains, but in category theory we can handle infinity. Therefore this table is nothing more than a container with the memoized functions acting as a functor.

According to this interpretation, a functor object is something that may contain a value or values of the type it’s parameterized upon. Or it may contain a recipe for generating those values, in the case of infinite values.

All we are interested in is to be able to manipulate those values using functions. If the values can be accessed, then we should be able to see the results of this manipulation. If they can’t, then all we care about is that the manipulations compose correctly and that the manipulation with an identity function doesn’t change anything (the functors' laws). We don’t really care about being able to access the values inside a functor object.

#### Functor Composition

As well as functions in sets compose, functors in categories also compose. Let's look at an example of a `Maybe` list datatype:

```
square x = x * x

mis :: Maybe [Int]
mis = Just [1, 2, 3]

mis2 = fmap (fmap square) mis
```

The compiler, after analyzing the types, will figure out that, for the outer fmap, it should use the implementation from the Maybe instance, and for the inner one, the list functor implementation.

The above code can be rewritten as:

```
mis2 = (fmap . fmap) square mis
```

Functors can be viewed as morphisms in categories (objects). So a category of categories (small) is called **cat**.

### Functoriality

Let's see now which type constructors (which corresponds to mapping between objects in categories) can be extended to functors (which include mappings between morphisms).

#### Bifunctors

A bifunctor is just a functor of two arguments. On objects, a bifunctor map every pair of objects say in categories C and D to an object in category E. `C x D -> E`. To fully define functoriality we must also map morphisms. Here we're mapping pair of morphisms (from CxD), to a single morphism (to E), they can be defined as a single morphism in the cartesian category, this is how we compose them:

`(f, g) ∘ (f', g') = (f ∘ f', g ∘ g')`

It clearly is associative and has an identity, so the cartesian product of categories is indeed a category.

An easier way to think about bifunctors would be to consider them functors in each argument separately. So instead of translating functorial laws — associativity and identity preservation — from functors to bifunctors, it would be enough to check them separately for each argument. However, in general, separate functoriality is not enough to prove joint functoriality. Categories in which joint functoriality fails are called **premonoidal**.

In Haskell we can define a bifunctor in its category of types like this:

```
class Bifunctor f where
    bimap :: (a -> c) -> (b -> d) -> f a b -> f c d
    bimap g h = first g . second h
    first :: (a -> c) -> f a b -> f c b
    first g = bimap g id
    second :: (b -> d) -> f a b -> f a d
    second = bimap id
```

When declaring an instance of `Bifunctor`, you have a choice of either implementing `bimap` and accepting the defaults for `first` and `second`, or implementing both `first` and `second` and accepting the default for `bimap`.

#### Product and Coproduct Bifunctors

An example of bifunctor is the categorical product, if the product exists for any pair of objects, the mapping from those objects to the product is bifunctorial. Here's the `Bifunctor` instance for a pair constructor (product type):

```
instance Bifunctor (,) where
    bimap f g (x, y) = (f x, g y)
```

By duality, a coproduct, if it’s defined for every pair of objects in a category, is also a bifunctor. In Haskell, this is exemplified by the Either type constructor being an instance of Bifunctor:

```
instance Bifunctor Either where
    bimap f _ (Left x)  = Left (f x)
    bimap _ g (Right y) = Right (g y)
```

Now, remember when we talked about monoidal categories? A monoidal category defines a binary operator acting on objects, together with a unit object. I mentioned that Set is a monoidal category with respect to cartesian product, with the singleton set as a unit. And it’s also a monoidal category with respect to disjoint union, with the empty set as a unit. What I haven’t mentioned is that one of the requirements for a monoidal category is that the binary operator be a bifunctor (**tensor product ⊗**). This is a very important requirement — we want the monoidal product to be compatible with the structure of the category, which is defined by morphisms. We are now one step closer to the full definition of a monoidal category (we still need to learn about naturality, before we can get there).

#### Functorial Algebraic Data Types

We have just seen that sums and products are functorial. We also know that functors compose. So if we can show that the basic building blocks of ADTs are functorial, we’ll know that parameterized ADTs are functorial too.

First, there are the items that have no dependency on the type parameter of the functor:

```
data Const a b = Const a

instance Functor (Const a) where
  fmap _ (Const a) = Const a
```

Then there are the elements that simply encapsulate the type parameter itself:

```
data Identity a = Identity a

instance Functor Identity where
    fmap f (Identity x) = Identity (f x)
```

Everything else in algebraic data structures is constructed from these two primitives using products and sums:

```
data Maybe a = Nothing | Just a
type Maybe a = Either (Const () a) (Identity a)
```

So Maybe is the composition of the bifunctor Either with two functors, Const () and Identity. (Const is really a bifunctor, but here we always use it partially applied.)

We will now show that a composition of this bifunctor with two functors is also a bifunctor. Given two morphisms, we simply lift one with one functor and the other with the other functor. We then lift the resulting pair of lifted morphisms with the bifunctor. In Haskell we can express this as:

```
newtype BiComp bf fu gu a b = BiComp (bf (fu a) (gu b))

instance (Bifunctor bf, Functor fu, Functor gu) =>
  Bifunctor (BiComp bf fu gu) where
    bimap f1 f2 (BiComp x) = BiComp ((bimap (fmap f1) (fmap f2)) x)
```

That means that our `BiComp` type is a bifunctor only if `bf` is a bifunctor and `fu` and `gu` are functors. The x in the definition of bimap has the type:

```
bf (fu a) (gu b)
```

which is quite a mouthful. The outer bimap breaks through the outer bf layer, and the two fmaps dig under fu and gu, respectively. If the types of f1 and f2 are:

```
f1 :: a -> a'
f2 :: b -> b'
```

Then the final result is of type `bf (fu a') (gu b')`.

#### Covariant and Contravariant Functors

Remembering the reader functor, it's just a function constructor partially applied:

```
(->) r

type Reader r a = r -> a

instance Functor (Reader r) where
    fmap f g = f . g
```

So, we know that it is a functor when partially applied, but is this a bifunctor in both parameter? Let’s try to make it functorial in the first argument. We’ll start with a type synonym — it’s just like the Reader but with the arguments flipped:

```
type Op r a = a -> r
```

When implementing `Functor` we would need to have an `fmap` f type:

```
fmap :: (a -> b) -> (a -> r) -> (b -> r)
```

Well, this is kind of impossible right? It would be different if we somehow invert the first function...

**A short recap**: For every category C there is a dual category Cop. It’s a category with the same objects as C, but with all the arrows reversed.

Now consider a mapping G that maps from category C to D, but when mapping the morphisms it reverses them. It takes a morphism f:: b -> a in C and first map it to the opposite f_op :: a -> b and then uses a functor F that maps from C_op to D to get F f_op :: F a -> F b. Considering that the object mapping of F and G are the same, we can say that G is just a functor with a twist G f :: (b -> a) -> (G a -> G b), we call it **contravariant functor**. It's just a regular functor from the opposite category, the regular functor is called **covariant functor**.

Here's the definition of a contravariant functor in Haskell:

```
class Contravariant f where
    contramap :: (b -> a) -> (f a -> f b)
```

#### Profunctors

We’ve seen that the function-arrow operator is contravariant in its first argument and covariant in the second. If the target category is **Set** the we call this to be a profunctor. A profunctor is defined as `C_op x D -> Set`.

Here's the definition of a profunctor typeclass in Haskell:

```
class Profunctor p where
  dimap :: (a -> b) -> (c -> d) -> p b c -> p a d
  dimap f g = lmap f . rmap g
  lmap :: (a -> b) -> p b c -> p a c
  lmap f = dimap f id
  rmap :: (b -> c) -> p a b -> p a c
  rmap = dimap id
```

Just as for Bifunctor we can provide one of the implementations and get the other by default. Below is how we implement the Profunctor class for the function-arrow operator:

```
instance Profunctor (->) where
  dimap ab cd bc = cd . bc . ab
  lmap = flip (.)
  rmap = (.)
```

Profunctors have their application in the Haskell lens library.

### Function Types

A function between a and b (`a -> b`) is just the hom-set between the objects a and b (hom(a, b)) in the Set category (category of sets), so a function type is also an object in this category, because it's after all, just a set. For other categories this is not true because the hom-set is **external** to the category, but there's a way to construct hom-sets as objects, they are then called **internal hom-sets**.

#### Universal Construction

Let's try to construct a function type, or more generally, a internal hom-set using universal construction. We need a pattern that involves three objects: the function object (a=>b), the argument (a) and the result (b). This pattern is called to be an __evaluation__. We now take candidates of the product of the function type a=>b and argument type a (a=>b x a), this is an object, so we pick, as our application morphism, an arrow `eval` from this object to b. 

So that’s the pattern: a product of two objects a=>b and a connected to another object b by a morphism `eval`.

The last piece of our universal construction is a ranking relation.  This is usually done by requiring that there be a mapping between candidate objects — a mapping that somehow factorizes our construction. In our case, we’ll decree that a=>b together with the morphism `eval` from a=>b×a to b is better than some other z with its own application g, if and only if there is a unique mapping h from z to a=>b such that the application of g' factors through the application of `eval`.

We know that the product (a=>b x a) is a functor (functoriality of the product), since we are not touching the second component of the product z × a, we will lift the pair of morphisms (h, id), where id is an identity on a. 

So, here’s how we can factor one application, `eval`, out of another application g:

```
g = eval ∘ (h × id)
```

Of course, there is no guarantee that such an object a⇒b exists for any pair of objects a and b in a given category. But it always does in Set. Moreover, in Set, this object is isomorphic to the hom-set Set(a, b). This is why, in Haskell, we interpret the function type a->b as the categorical function object a⇒b.

#### Currying

If we look at the candidates for the function object, we think of the morphism g as a function of two variables. In fact, in Set, g is a function from pairs of values, one from the set z and one from the set a:

```
g :: z × a -> b
```

The universal property tells us that for each such g there is a unique morphism h that maps z to a function object a=>b. In Set, this just means that h is a function that takes one variable of type z and returns a function from a to b. That makes h a higher order function:

```
h :: z -> (a⇒b)
```

Given a g there's always an unique h, and given any h, you can recreate the two-argument function g using the formula:

```
g = eval ∘ (h × id)
```

This gives us a one-to-one correspondence between the curried (h) and uncurried (g) versions of the same function.


#### Exponentials

