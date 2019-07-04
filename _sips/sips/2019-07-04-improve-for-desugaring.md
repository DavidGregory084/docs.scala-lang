---
layout: sips
discourse: true
title: SIP-NN - Improve Desugaring of For Comprehensions
---

**By: David Gregory**

## History

| Date          | Version       |
|---------------|---------------|
| Jul 4th 2019  | Initial Draft |

## Motivation

For comprehensions are a wonderful feature of the Scala language. They allow us to eliminate nesting in code which makes heavy use of the collections operations  `flatMap` and `map`, and provide a shorthand syntax for filtering collections.

However, the for comprehension currently has some rough edges which we can eliminate by changing the way that this syntax is desugared by the compiler.

These rough edges have been discussed on multiple occasions previously, most notably in [this Dotty ticket][2] which refers to the desugaring of for comprehensions as a "Scala Wart".

## Motivating Examples

### Superfluous map operation

Every for comprehension of the following form introduces a superfluous `map` operation:

{% highlight scala %}
for {
  x <- xs
} yield x
{% endhighlight %}

This desugars to the following:

{% highlight scala %}
xs.map(((x) => x))
{% endhighlight %}

Which is equivalent to the following:

{% highlight scala %}
xs
{% endhighlight %}

### Patterns and withFilter

The desugaring of patterns in for comprehensions requires that the collection type that is used implements the `withFilter` operation:

{% highlight scala %}
val either: Either[Throwable, (Int, Int)] = Right((1, 2))

for {
  (x, y) <- either
} yield x
// value withFilter is not a member of Either[Throwable,(Int, Int)] 
{% endhighlight %}

In this case there is no reason to apply the `withFilter` operation since `either` contains `(Int, Int)` for which `(x, y)` is a valid constructor pattern.

Unfortunately, the desugaring of for comprehensions happens prior to typechecking, so there's currently no way to determine that this is the case at the point that desugaring occurs.

### Complicated desugaring of value definitions



## Design


## Implementation


## Counter-Examples


## Drawbacks

### Eliminating superfluous `map` operations

Eliminating the `map` operation in for comprehensions which yield their last generator's result might alter the behaviour of code which does not comply with the monad right identity law.

Within the Scala collections and the functional programming libraries we can reasonably expect that `xs.map(x => x)` is equivalent to `xs`, but it's quite possible that Scala users have written lots of code where this is not the case.

## Alternatives

The community could continue to use [oleg-py](https://github.com/oleg-py)'s excellent compiler plugin.

## References

1. [oleg-py's better-monadic-for plugin][1]
2. [Scala Wart: Convoluted de-sugaring of for-comprehensions][2]
3. [Scala Contributors Discussion][3]

[1]: https://github.com/oleg-py/better-monadic-for "better-monadic-for"
[2]: https://github.com/lampepfl/dotty/issues/2573 "Scala Wart: Convoluted de-sugaring of for-comprehensions"
[3]: http://contributors.scala-lang.org/ "Contributors discussion"
