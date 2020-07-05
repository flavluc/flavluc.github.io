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

If you have arrows:

```
a --> b --> c
  (f)   (g)
```

Then there must be an arrow:

```
a ----> c
  (g∘f)
```

Which is composition of the two arrows (reads g after f).

Properties of Composition

 - Associativity: h ∘ (g ∘ f ) = (h ∘ g) ∘ f = h ∘ g ∘ f
 - Identity: an arrow which is a unit of composition, f ∘ id A = f and id B ∘ f = f

## Isomorphism

Let two morphisms f and g in any category such that:

```
f :: a -> b
g :: b -> a
```

f is an isomorphism if there exist g such that:

```
 g . f = id_a
 f . g = id_b
```

### Inverse function

In category theory we can't look inside of sets, so we can't tell if a function is injective or surjective by looking at the object, but...

 - injective function = monic -> monomorphisim
 - surjective function = epic -> epimorphisism

These are defined for any category, let's look at what they are.
 
Suppose objects A, B and C:

  - If for every object C and every pair of morphisms `g1, g2 :: B -> C` such that for a morfism `f :: A->B`, if `g1.f = g2.f` then `g1 = g2`, then `f` is an **epimorphisim** (surjective)
  - If for every object C and every pair of morphisms `g1, g2 :: C -> A` such that for a morfism `f :: A->B`, if `f.g1 = f.g2` then `g1 = g2`, then `f` is a **monomorphisim** (surjective)

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
 - assosiativity? yes

#### Hom-set

Before get into exemples let's define a **hom-set**. It's nothing more than a set of morphisms between two objects, say a and b, then it's written as C(a, b) (or Hom_C(a, b)).

#### Preorders

A binary relation <= on a set X is a **preorder** iff the relation <= is both **reflexive** (for all a, a R a) and **transitive** (for all a,b and c, a R b and b R c -> a R c).

We can think of reflexivity and transitivity in terms of identity and composition. Indeed a preorder is a category.

A preorder is a category where theres 0 or 1 morphism between every pair of objects. We call such category a **thin** category. A **preorder is a thin category**. Therefore, every hom-set in a preorder is either empty or a singleton. You may, however, have cycles in a preorder.

**The counter example**: The preorder (or a thin category) is an interesting category because every morphisms is an epimorphisim and a monomorphisim, BUT, in a thin category a bimorphism is not necessarily invertible (i.e. an isomorphism).

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

* It's responsability of the programmer to make sure the monoidal properties are satisfied.

#### Monoid as a Category

Every monoid can be described as a single object category with a set of morphisms that follow appropriate rules of composition.

For examples we can think of an object as a set of natural numbers and the set of morphisms as "adders". The identity morphism would be a 0-adder. So a composition of the 5-add with the 7-adder would be a 12-adder.

For every monoidal category M, we have a hom-set M(m, m) of the single object m in the category. We can easily define a binary operator in this set. The monoidal product of two set-elements is the element corresponding to the composition of the corresponding elements. The composition always exist because the source and the target object are the same. And the identity morphisim is the neutral element of this product.

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

The idea is that the log will be aggregated between function calls. Here's a more practical example, suppose we want to create a function to upper case strings while still having a log property. First, let's create a `Writer` template to enpasulate the `embelishment` of the function:

```
template<class A>
using Writer = pair < A, string > ;
```

And here's our embelished functions:

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

Now the aggregation does not depend on a global state, instead, they're produced locally and externaly concatenated into a larger log.

A fact is that writing a program in this style would be a pain in the ass with a lot of repetitive code. A way to work around it is to abstract function composition itself. Before we do that, we should take a look at a category theory perspective of this problem.

#### The Writer Category

Here we're still talking about categories of types and functions, however, instead a normal composition function, we now have an embelished element. So we have to redefine our composition abstraction and that's how we do it:

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

We can say that each Kleisli category defines its own way of composing morphisms with embelishment, as well as the identity morphisms with respect to that composition.

In particular, our examples used a writer monad for logging the execution of functions. It's also an example of a more general mechanism for embedding effects in pure computations.

### Products and Coproducts

#### Universal Construction

There's a common construction in category theory caled the **universal construction** for defining objects in terms of their relationship. It basically boils down to a two-step proccess:

 - Pick a pattern in terms of objects and morphisms and find all the matching elements in the category.
 - Establish a kind of ranking between those hits and find the best fit.

#### Initial Object

Consider the following example for the above method: think about the pattern as an object and the ranking relation as `a` is more initial than `b` if there's a single morphism a -> b. Then we may have a "best" object (there's no guarantee that this object exist) which we define as an **initial object**.

The **initial object** is the object that has one and only one morphism going to any object in the category.

This definition guarantees an **uniquiness up to isomorphisim** on the initial objects (there's no uniquiness guarantee). Some examples of such objects are the `least element` in a poset (partially ordered set) and an empty set in a category made of sets and functions. Notice that on a poset there's no guarantee of the existence of a initial object (for exemple in the set of all positive integers).

#### Terminal Objects

Consider now the same pattern as before, but changing the ranking relation by reverting the direction. In other words, an object `a` is more terminal than `b` if there's a single morphism b -> a.

The **terminal object** is the object with one and only one morphism coming to it from any object in the category.

Again we have an uniquiness up to isomorphisms. As an example, in a poset, the terminal object, if it exists, is the biggest object as well as in the set category, the terminal object is a singleton (set with only one element).

#### Duality

Duality is a very important property in category theory. For every constructino you may come up with, there will be an opposite in which you can get by reversing the arrows and redefining the compositions.

Such opposite categories are often prefixed with "co", so you have products and coproducts, monads and comonads and so on.

Given this definition, it's clear enough that initial and terminal objects are in a duality relationship, as in the universal construction proccess we just inverted the morphisms in each ranking relation.

#### Isommorphisms

As for an intuition, we can think as isomorphism as a one-to-one mapping between two objects, such that every part of one object has a correspondance in the other one.

In category theory, isomorphism means invertable morphisms, so a formal definition in term of composition and identity can be thought as:

```
f . g = id
g . f = id
```

When we said that initial and terminal objects are unique up to isomorphisim, we actually meant that they're unique up to unique isomorphisim. In principle, there can be many isomophisms between two objects (think about {true, false}, {black, white}), but for the initial and terminal objects, the uniquiness property of the f and g morphisms guarantees a unique isomorphism for any two objects (see proof in the book).

In fact, this “uniqueness up to unique isomorphism” is the important property of all universal constructions.

#### Product

A cartesian product of any two sets is a set of pairs. In categories, we can define a pattern to generalize the product definition using universal contruction.

All we can say, in the category world, is that there are two functions, the projections, from the product to each of the constituents (in Haskell they're `fst` and `snd`).

Now, a pattern for choosing candidate objects is an object `c` and two morphisms `p` and `q` such that:

```
p :: c -> a
q :: c -> b
```

The ranking relation can be thought as: given two candidates, let it be (c, p, q) and (c', p', q'), we would like to say that c is "better" than c' if there's a morphisim m from c' to c and, therefore, the projections p' and q' such that:

```
p’ = p . m
q’ = q . m
```

We can also say that m factorizes p' and q', for examples on how m shrinks information from any other product candidate check out the book section.

We should notice as well that a product is also unique up to unique isomorphisim.

A high order function that produces the factorizing function m from two candidates is sometimes called the **factorizer**.

#### Coproduct
