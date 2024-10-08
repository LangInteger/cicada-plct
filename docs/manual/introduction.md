---
title: Introduction
---

**Cicada Language** is a
_dependently typed programming language_ and an
_interactive theorem prover_.

> The aim of cicada project is to help people understand that developing
> software and developing mathematics are increasingly the same kind of
> activity, and people who practice these developments can learn from
> each other, and help each other in very good ways.

[ [**Homepage**](https://cicada-lang.org/)
| [**Installation**](./installation.md) ]

# Source Code

Source code of this manual is available at
the `docs/manual/` directory of [cicada repository](https://github.com/cicada-lang/cicada).

Welcome to give feedback :)

# Manual as Standard Library

This manual also serve as as the _standard library_ of cicada language.

Example usage:

```cicada
import { Nat, zero, add1, add } from "./datatypes/nat.md"

check add: (Nat, Nat) -> Nat

let two = add1(add1(zero))

compute add(two, two)
```

# References

## The Little Typer

I learned about how to implement type system from Dan's little books.

My design of cicada language can be viewed as an exercise after "The Little Typer".

- Thanks, Daniel P. Friedman (1944 -), and David Thrane Christiansen,
  for your great dialogs and teachings.

## Vladimir Voevodsky's talk

I recommend a talk of Vladimir Voevodsky:

- ["How I became interested in foundations of mathematics"](https://readonly.link/articles/xieyuheng/xieyuheng/-/persons/vladimir-voevodsky/how-i-became-interested-in-foundations-of-mathematics.md)

It summarises the motivation behind cicada project beautifully.

- Thanks, Vladimir Voevodsky (1966 - 2017)
