---
title: Built-in Types
---

# Type

`Type` is a `Type`.

We can use the `check` statement to make assertion about an expression's type.

```cicada
check Type: Type
```

Don't forget that code blocks on our website are interactive,
hovering over them to see a menu button.

# String

`String` is a `Type`.

```cicada
check String: Type
```

We use double-quoted `String`.

```cicada
check "Hello, World!": String
```

We can use the `let` statement to do assignment.

```cicada
let my_name = "Xie Yuheng"
```

We can use the `compute` statement to compute the value of an expression.

```cicada
compute my_name
```

# Trivial

`Trivial` is a `Type`, and `sole` is its only element.

```cicada
check Trivial: Type
check sole: Trivial
```

`let` can be nested in `{ ...; return ... }`.

```cicada
let result = {
  let x = sole
  let y = x
  let z = y
  return z
}

compute result
```

# Pair

`Pair` taken two `Type`s, is a `Type`.

```cicada
check Pair(String, Trivial): Type
```

We can use `cons` to construct `Pair`.

```cicada
check cons("abc", sole): Pair(String, Trivial)
```

And using `car` to get a `Pair`'s first element,
using `cdr` to get a `Pair`'s second element.

- `cons`, `car` and `cdr`.

  Yes, that's Lisp's DNA, dancing in our cells.

  Thanks, John McCarthy (1927 - 2011).

```cicada
let pair: Pair(String, Trivial) = cons("abc", sole)

check car(pair): String
check cdr(pair): Trivial
```

We use `Both` as another name of `Pair`,
because we read `Pair(A, C)` as

> Both A and C are true.

The following two expressions are of the same `Type`.

```cicada
check Pair(String, Trivial): Type
check Both(String, Trivial): Type
```

We can write nested `Pair`.

```cicada
check Pair(Type, Pair(String, Trivial)): Type
check cons(String, cons("abc", sole)): Pair(Type, Pair(String, Trivial))
```

# Sigma type

We write **sigma type** as `exists (x: A) C`,
where `x` might occur in `C`, or say, `C` depends on `x`.

This is why sigma type is also called **dependent pair type**.

We read `exists (x: A) C` as

> There exists `x` in `A`, such that `C` is true.

For `Pair` the second type is fixed,
while for sigma type, when the first expression is different,
the second type can change,

```cicada
check exists (x: Pair(Type, Type)) car(x): Type

check cons(cons(String, Trivial), "ABC"): exists (x: Pair(Type, Type)) car(x)
check cons(cons(Trivial, String), sole): exists (x: Pair(Type, Type)) car(x)
```

`Pair` is actually a special form of sigma type.
The following two types are the same.

We can use `same_as_chart` to assert that
many expressions of a given type are the same.

```cicada
compute same_as_chart (Type) [
  Pair(String, Trivial),
  exists (_: String) Trivial,
]
```

# Function

We write **function type** as `(A) -> R`,
where `A` is the **argument type**,
and `R` is the **return type**.

```cicada
check (String) -> Trivial: Type
check (Pair(String, Trivial)) -> Pair(Trivial, String): Type
```

We use `(x) => ...` to construct **anonymous function**,
which is also famously called **lambda**.

```cicada
check (x) => sole: (String) -> Trivial

check (pair) => cons(cdr(pair), car(pair)):
  (Pair(String, Trivial)) -> Pair(Trivial, String)
```

When using the `let` statement, we can give a type.

```cicada
let very_trivial: (String) -> Trivial = (x) => sole

check very_trivial("any string"): Trivial
```

We can also use `function` to define a function.

```cicada
function swap_pair(
  pair: Pair(String, Trivial)
): Pair(Trivial, String) {
  return cons(cdr(pair), car(pair))
}

check pair: Pair(String, Trivial)
check swap_pair(pair): Pair(Trivial, String)
```

# Pi type

We write **pi type** as `(x: A) -> R`,
just like function type, `A` is the **argument type**, and `R` is the **return type**,
but this time, `x` might occur in `R`, or say, `R` depends on `x`.

This is why pi type is also called **dependent function type**.

Another way of writing `(x: A) -> R` is `forall (x: A) R`.

And we read it `forall (x: A) R` as

> For all `x` in `A`, `R` is true.

For function type the return type is fixed,
while for pi type, when the argument expression is different,
the return type can change.

```cicada
check (T: Type) -> (T) -> T: Type
check forall (T: Type) (T) -> T: Type
```

The return type of the above pi type is `(T) -> T`,
it depends on the argument expression `T`.

We have a built-in function of the above type,
and we call this function `the`.

```cicada
compute the(String, "abc")
compute the(Trivial, sole)
compute the((T: Type) -> (T) -> T, the)
```

It is defined as the following:

```cicada
function the(T: Type, x: T): T {
  return x
}
```

# Absurd

`Absurd` is a very special `Type`, because it has no elements.

```cicada
check Absurd: Type
```

We have a built-in function called `from_falsehood_anything`,

If you have a element of type `Absurd`,
you can use it to prove anything.

```cicada
check from_falsehood_anything:
  (target: Absurd, motive: Type) -> motive
```

When we want to express a proposition is not true,
we say that proposition leads to absurd.

```cicada
function Not(T: Type): Type {
  return (T) -> Absurd
}
```

# Equal

Given a `Type`, and two expressions of that `Type`,
we can create a new `Type` to express the two expressions are the same.

```cicada
check Equal(String, "abc", "abc"): Type
check Equal(Trivial, sole, sole): Type
```

We can use `the_same` to construct elements of this type.

```cicada
check the_same(String, "abc"): Equal(String, "abc", "abc")
check the_same(Trivial, sole): Equal(Trivial, sole, sole)
```

If we want to omit the first argument, we can use `same`.

```cicada
check same("abc"): Equal(String, "abc", "abc")
check same(sole): Equal(Trivial, sole, sole)
```

If we want to omit all arguments, we can use `refl`, which means "reflection".

- We can omit all arguments, because after all, all the information is already in the type.

```cicada
check refl: Equal(String, "abc", "abc")
check refl: Equal(Trivial, sole, sole)
```

If the two elements are actually not the same,
we can still use `Equal` to create a `Type`,
but we will not be able to construct any elements of this type.

```cicada
// NOTE There is no way by which we can construct
//   an element of the following type.
check Equal(String, "abc", "de"): Type
```

By the way, we use `// ...` to write comments in code,
just like the two lines above.

---

# Summary

Above are all the built-in types of cicada language!

Not enough, right?

Beside them, we will use `datatype` and `class` to construct new compound types.

But before that, let's see how to view a file as a module,
and use `import` to import published works.
