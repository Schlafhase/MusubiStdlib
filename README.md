# Musubi standard library

![time spent](https://hackatime.hackclub.com/api/v1/badge/U081Z7D7YJK/stdlib)

Musubi's standard library provides powerful definitions that allow using it
almost like a modern functional programming language.

## Table of contents

- [Installation](#installation)
- [Encodings of datatypes](#encodings-of-datatypes)
  - [Scott encoding](#scott-encoding)
  - [Church numerals](#church-numerals)
- [Modules](#modules)

## Installation

Go to `~/.musubi` (or `C:\Users\{you}\.musubi` on windows).

In that directory, run
`git clone https://github.com/Schlafhase/MusubiStdlib stdlib/`. **It's important
that the directory is called `stdlib`** because that's what the modules in the
library expect.

That's it. The standard library can now be included in any Musubi program like
this:

```
#include stdlib
```

> [!NOTE]
>
> I plan to add a `musubi install` command which will automatically put the
> contents of a github repo into a subfolder of ~/.musubi

## Encodings of datatypes

### Scott encoding

The scott encoding is very useful to define recursive datatypes. Every scott
encoded datatype has a finite set of constructors that can take as many
arguments as they need. Take a numeral for example:

We have a "zero" constructor and a "successor of n" constructor (which takes one
parameter `n`). The scott encoding encodes this as a lambda that takes these two
constructors as parameters and uses them to return the appropriate value.

```
zero := \z.\s.z, ; zero is just the zero constructor (which takes no arguments)
succ :=
  \n.
    \z.\s.s n ; the successor of n is constructed using the s (successor) constructor of n
```

These are "Scott numerals" and are, at the moment, the preferred encoding for
numerals.

We can do the same thing for lists. Let's have a "nil" constructor for an empty
list and a "cons x xs" constructor for a list with "x" as head and "xs" as tail:

```
nil := \n.\c.n,
cons :=
  \x.\xs.
    \n.\c.c x xs
```

The scott encoding is really beautiful for pattern matching because we can pass
a handler for every constructor. The handler is a lambda that takes the same
parameters as the constructor (if any) and returns the value that should be
returned when that constructor is matched.

Let's use this to define the predecessor function on scott numerals. In the
zero-constructor case, return 0 and in the "successor of `p`"-case return `p`.

```
pred := \n.n 0s (\p.p)
```

To verify that this works:

```
  pred 2
= pred (\z.\s.s (\z.\s.s (\z.\s.z))) ; expand 2
= (\n.n 0s (\p.p)) (\z.\s.s (\z.\s.s (\z.\s.z))) ; expand pred
= (\z.\s.s (\z.\s.s (\z.\s.z))) 0s (\p.p) ; apply pred to 2 (replace occurences of `n` in `pred`s body with 2)
= (\p.p) (\z.\s.s (\z.\s.z)) ; apply 2 first to 0s (which does nothing because `z` isn't used in 2) and then to (\p.p)
= (\z.\s.s (\z.\s.z)) ; apply (\p.p) which is the identity function to (\z.\s.s (\z.\s.z))
= 1 ; number representation of (\z.\s.s (\z.\s.z))
```

### Church numerals

A church numeral `n` is defined as a lambda that takes an `f` and an `x` and
applies that `f` to `x` `n` times.

```
zero := \f.\x.x,
succ :=
  \n.
    \f.\x.f (n f x) ; (n f x) is `f` applied to `x` `n` times, so we add one more `f` around it
```

## Modules

The main module (`stdlib/module.mbim`) which is included using `#include stdlib`
includes the following other modules:

- `stdlib/generalMacros.mbim`
- `stdlib/pipe.mbim`
- `stdlib/recursion.mbim`
- `stdlib/lazy.mbim`
- `stdlib/scottArithmetics.mbim`
- `stdlib/useScottEncoding.mbim`
- `stdlib/boolean.mbim`
- `stdlib/lists.mbim`
- `stdlib/conversions.mbim`

All available modules have a documentation comment explaining what it
does. Most defined functions also have a documentation comment above.
