---
title: Proving Theorems About Nat
---

Before going to the next `datatype` -- `List`, let's prove some theorems about `Nat`,
to show off the power of dependent types!

How about the **commutative property of addition** for natural number?

It says,

> When doing addition for natural number,
> the order of the numbers can be changed without changing the answer.
>
> x + y = y + x

We import `Nat` and `add` from previous chapter first.

```cicada
import {
  Nat, add,
  zero, add1,
} from "./nat.md"
```

- We also imported `zero` and `add1` to simplify our discussion.

Then we can express the following theorems by a type in our programming language,

> For all `x` and `y` in `Nat`, `add(x, y)` is equal to `add(y, x)`.

```cicada
compute (x: Nat, y: Nat) -> Equal(Nat, add(x, y), add(y, x))
```

# Let's prove it!

To prove the theorem,
let's follow Pólya's advices about problem solving steps.

- Thanks, George Pólya (1887 – 1985),
  for your great teachings,
  and your lovely book "How to Solve It" (1945).

## 1. Understanding the problem

**What is the unknown?**

We need to prove a theorem in our programming language,
the theorem is expressed by a type, and to prove it
is to construct an element of that type.

> A theorem expressed by a type, is like a puzzle --
> a game between us and the type checker.
>
> If we can construct an element of that type, we win!

And we see the type is a pi type,
so its element will probably be a function.

The result type of the pi type is constructed by `Equal`,
we might need to use `Equal`'s constructors in our proof.

**What are the data?**

We have the definition of `Nat`, we already know a lot about it.

And we have the definition of `add`,
which tells us how the computation of addition is performed,
both in the case of `zero` and in the case of `add1`.

We also know something about the built-in type `Equal`,
but maybe we need to learn more about it.

## 2. Devising a plan

How should we prove the commutative property of addition in English mathematical prose?

Here is my plan, I want to review the proof in prose,
and try to translate it to our programming language.

> For all `x` and `y` in `Nat`, `add(x, y)` is equal to `add(y, x)`.

**Proof by mathematical induction over `x`.**

- **Base case, suppose `x` is `zero`.**

  It is obvious that

  ```
  add(zero, y) = add(y, zero)
  ```

  because both sides are equal to `y`.

- **Inductive step, suppose `x` is `add1(prev)`.**

  We have inductive hypothesis

  ```
  add(prev, y) = add(y, prev)
  ```

  We need to prove

  ```
  add(add1(prev), y) = add(y, add1(prev))
  ```

  If for the left hand side we have

  ```
  add(add1(prev), y) = add1(add(prev, y)) // Lemma 1
  ```

  and for the right hand side we have

  ```
  add(y, add1(prev)) = add1(add(y, prev)) // Lemma 2
  ```

  Then we can use inductive hypothesis to prove

  ```
  add1(add(prev, y)) = add1(add(y, prev))
  ```

  by applying `add1` to inductive hypothesis

  ```
       add(prev, y)  =      add(y, prev)
  add1(add(prev, y)) = add1(add(y, prev))
  ```

  Now we turn to prove Lemma 1 and Lemma 2.

  - **Proof of Lemma 1.**

    It is obvious that

    ```
    add(add1(prev), y) = add1(add(prev, y))
    ```

    because by the definition of `add` in the case of `add1`,
    we have the following

    ```
    add(add1(prev), y) =
    add1(almost.prev) =
    add1(add(prev, y))
    ```

    **Q.E.D. Lemma 1.**

  - **Proof of Lemma 2, by mathematical induction over `y`.**

    Remind Lemma 2

    ```
    add(y, add1(prev)) = add1(add(y, prev))
    ```

    - **Base case, suppose `y` is `zero`.**

      It is obvious that

      ```
      add(zero, add1(prev)) = add1(add(zero, prev))
      ```

      because both sides are equal to `add1(prev)`.

    - **Inductive step, suppose `y` is `add1(y_prev)`.**

      We have inductive hypothesis

      ```
      add(y_prev, add1(prev)) = add1(add(y_prev, prev))
      ```

      We need to prove

      ```
      add(add1(y_prev), add1(prev)) = add1(add(add1(y_prev), prev))
      ```

      By definition of `add` in the case of `add1`,
      the left hand side is equal to

      ```
      add1(add(y_prev, add1(prev)))
      ```

      and the right hand side is equal to

      ```
      add1(add1(add(y_prev, prev)))
      ```

      We can prove them equal by applying `add1` to inductive hypothesis

      ```
           add(y_prev, add1(prev)) =       add1(add(y_prev, prev))
      add1(add(y_prev, add1(prev))) = add1(add1(add(y_prev, prev)))
      ```

    **Q.E.D. Lemma 2.**

  Since both Lemma 1 and Lemma 2 are proved,
  the inductive step of the original proof is finished.

**Q.E.D.**

## 3. Carrying out the plan

The outline of our proof will be

```
function add_is_commutative(
  x: Nat, y: Nat,
): Equal(Nat, add(x, y), add(y, x)) {
  return induction (x) {
    motive (x) => Equal(Nat, add(x, y), add(y, x))
    case zero =>
      // Base case.
      ...
    case add1(prev, almost) =>
      // Induction step.
      // `almost.prev` is the inductive hypothesis.
      ...
  }
}
```

Note that, we need to explicitly writing down
our `motive` of this induction, namely

```
(x) => Equal(Nat, add(x, y), add(y, x))
```

Where the argument is the `target` of the induction,
and the return expression is the return type of the induction,
which depends on the `target`.

We will extract two helper functions,
one for the **Base case**,
one for **Lemma 2**.

Before we dive into them, we need a utility function about `Equal` first -- `equalMap`.

```cicada
import {
  equalMap,
} from "../equality/equal-utilities.md"
```

It maps a function to the elements on both sides of an `Equal` type.

Let's practice using it first.

```cicada
check same(zero):
  Equal(Nat, zero, zero)

check equalMap(same(zero), add1):
  Equal(Nat, add1(zero), add1(zero))
```

Intuitive, right?

In the chapter [Equal Utilities](../equality/equal-utilities.md),
we will discuss the utility functions about `Equal` in details.

**Base case.**

The Base case itself is a meaningful theorem, which reads like

> For all `x` in `Nat`, `add(zero, x)` is equal to `add(x, zero)`.

```cicada
function add_is_commutative_on_zero(
  x: Nat
): Equal(Nat, add(zero, x), add(x, zero)) {
  return induction (x) {
    motive (x) => Equal(Nat, add(zero, x), add(x, zero))
    case zero => refl
    case add1(prev, almost) =>
      equalMap(almost.prev, add1)
  }
}
```

**Lemma 2.**

The Lemma 2 itself is also a meaningful theorem, which reads like

> For all `x` and `y` in `Nat`, `add(x, add1(y))` is equal to `add1(add(x, y))`.

i.e. the inner `add1` applied to the second argument of `add`,
can be moved out of `add`.

```cicada
function add_is_commutative_on_add1(
  x: Nat, y: Nat,
): Equal(Nat, add(x, add1(y)), add1(add(x, y))) {
  return induction (x) {
    motive (x) => Equal(Nat, add(x, add1(y)), add1(add(x, y)))
    case zero => refl
    case add1(_prev, almost) =>
      equalMap(almost.prev, add1)
  }
}
```

**Back to the original proof.**

We need two more utility functions about `Equal`.

```cicada
import {
  equalSwap,
  equalCompose,
} from "../equality/equal-utilities.md"
```

Let's practice using them.

`equalSwap` can swap the left and right hand side of the `Equal`.

```cicada
import { one } from "./nat.md"

check same(add1(zero)):
  Equal(Nat, add(one, zero), add1(zero))

check equalSwap(same(add1(zero))):
  Equal(Nat, add1(zero), add(one, zero))
```

`equalCompose` can compose two `Equal`s,
if the right hand side of the first `Equal`,
matchs the left hand side of the second `Equal`.

```cicada
check same(add1(zero)):
  Equal(Nat, add(one, zero), add1(zero))

check same(add1(zero)):
  Equal(Nat, add1(zero), add(zero, one))

check equalCompose(same(add1(zero)), same(add1(zero))):
  Equal(Nat, add(one, zero), add(zero, one))
```

Now we can finish the original proof.

```cicada
function add_is_commutative(
  x: Nat, y: Nat,
): Equal(Nat, add(x, y), add(y, x)) {
  return induction (x) {
    motive (x) => Equal(Nat, add(x, y), add(y, x))
    case zero =>
      // Base case.
      add_is_commutative_on_zero(y)
    case add1(prev, almost) =>
      // Induction step.
      // `almost.prev` is the inductive hypothesis.
      equalCompose(
        equalMap(almost.prev, add1),
        equalSwap(add_is_commutative_on_add1(y, prev)),
      )
  }
}
```

## 4. Looking back

We can import some example `Nat` to test our proof.

```cicada
import { two, three, four } from "./nat.md"

check add_is_commutative(two, three):
  Equal(Nat, add(two, three), add(three, two))

check add_is_commutative(three, four):
  Equal(Nat, add(three, four), add(four, three))
```

Note that, our function `add_is_commutative`
proves the commutative property of addition _for all `Nat`_,
while applying the function to specific `Nat`s give us,
commutative property of addition for specific `Nat`s.

Some questions for **Looking back**:

- **Can you derive the result differently?**

  Maybe we can make the **Lemma 1** more explicit in our formal proof.

  Maybe there are different ways to use the utility functions about `Equal`.

- **Can you see it at a glance?**

  If we practice our intuition about `Equal` and `inductive`,
  maybe the proof will be more obvious to us.

- **Can you use the result, or method, for some other problem?**

  Maybe we can use similar idea
  to prove **the commutative property of multiplication**
  for natural number.

---

# Summary

Phew ~

That's a lot of lemmas and inductions.

Compare our formal proof in programming language,
with the proof in English mathematical prose,
how do you feel?

Do you find more conciseness and clarity in formal proofs?

Probably not yet.

But you see,
when we prove in a programming language,
the checker can give us feedback.
And we can play with our proofs,
apply them and test them.
We also can package our works
to be imported by others easily.

> Proof assistants [programming languages]
> can be extremely useful,
> just like musical instrument,
> you can use it to practise,
> in the circumstance where you do not have a teacher.
>
> [**Thus spake Vladimir Voevodsky**](https://readonly.link/articles/xieyuheng/xieyuheng/-/persons/vladimir-voevodsky/how-i-became-interested-in-foundations-of-mathematics.md)
