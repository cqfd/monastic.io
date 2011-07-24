--- 
layout: post 
title: Folds in Ruby 
---

# Reductions and folds in Ruby

## Some background on reduce

In this post I want to talk about Ruby's reduce method and its relation to
folds in functional programming. The plan is to first talk about how reduce
works in Ruby, then talk about how folds work in Haskell, and finally I'll show
you how to implement folds in Ruby.

Let's first recall what reduce does. Reduce comes from the Enumerable module,
and it allows you to loop over a collection and "reduce" it to a single value.
A simple example would be summing the values in an array:

{% highlight ruby %}
a = [1, 2, 3, 4, 5] 
a.reduce(0) { |sum, i| sum + i } # => 15 
{% endhighlight %}

Reduce takes one argument and a block. The block is invoked once for every
element in the collection, and the current element is passed as the block's
second argument, `i`. The block's first argument, `sum`, is set on its first
invocation to be the argument passed to reduce; on subsequent invocations, it's
set to the result of the previous invocation. The return value of the call to
reduce is the value of the final invocation of the block.

That's a bit of a mouthful, so let's go through the example step by step.
Afterwards, we'll implement our own version of reduce in Ruby.

We start by invoking the block on the first element of `a`, so that `i` is set
to be `1`. This is the first invocation, so `sum` is set to `0`, the argument
we passed to reduce. The block evaluates to `sum + i`, or `1`.

Next, invoke the block on the second element of `a`, by setting `i` to be `2`.
This is our second time invoking the block, so set `sum` to tbe result of the
last invocation, `1`. The block then evaluates to `sum + i`, or `3`.

Continuing in the same fashion, you see that at each invocation, `sum` is set
to be the sum of all the previous elements of the array. Adding the last
element of the array then sums the entire array.

## Implementing reduce in Ruby

Here's a simple implementation of this algorithm in Ruby, written as a
monkeypatch on the `Array` class:

{% highlight ruby %}
class Array 
  def reduce acc 
    each do |i| 
      acc = yield acc, i
    end 
    acc 
  end 
end 
{% endhighlight %}

The code loops over every element in the array you want to reduce, updating the
value of the passed argument each time. At the very end, it returns the fully
accumulated `acc` value. Also, notice that it uses a bit of syntactic sugar:
generally speaking (emphasis on generally), Ruby will let you get away without
appending an explicit receiver to a method call if that receiver is the current
object, `self`. So the bare `each` is equivalent to `self.each`.

## Some background on folds in Haskell

Ruby's reduce has a direct analogue in most functional programming languages:
folds. Like reduce, folds give you a way to accumulate a list of values into a
single value. For instance, Haskell comes with two functions, `foldl` and
`foldr`, that work pretty much exactly like reduce:

{% highlight haskell %}
foldl ( + ) 0 [1,2,3,4,5] -- 15
foldr ( * ) 1 [1,2,3,4,5] -- 120
{% endhighlight %}

In case you're not familiar with Haskell's syntactic sugar, `( + )` and `( * )`
are called sections. Given any infix binary operator `o`, `( o )` is the prefix
version; you can also do fun things like `(x o)`, which partially applies `x`
into the operator's first slot, or `(o x)` to partially apply `x` into the
second slot.

In case you're not familiar with Haskell's syntactic sugar, `( + )` and `( * )`
are called sections; 

The left fold, `foldl`, is so-named because it operates from the left. Its example
expands like this:

{% highlight haskell %}
((((0 + 1) + 2) + 3) + 4) + 5
{% endhighlight %}

The right fold, `foldr`, operates from the right. Its example expands like this:

{% highlight haskell %}
1 * (2 * (3 * (4 * (5 * 1))))
{% endhighlight %}
<br/>

## Implementing folds in Haskell

Let's start with the left fold. Unlike in Ruby, Haskell won't let us update an
accumulator value, as variables are immutable. This means we'll need
to use recursion.

You can get a little intuition for how to structure the recursion by observing
the following silly identity for the left fold example:

{% highlight haskell %}
((((0 + 1) + 2) + 3) + 4) + 5 = (((1 + 2) + 3) + 4) + 5
{% endhighlight %}

The left hand side of the equation is, by definition, equal to

{% highlight haskell %}
foldl ( + ) 0 [1,2,3,4,5]
{% endhighlight %}

However, that means the right hand side looks a lot like

{% highlight haskell %}
foldl ( + ) 1 [2,3,4,5]
{% endhighlight %}

Hmm. Continuing the thought, your advanced algebra-fu will convince you that

{% highlight haskell %}
(((1 + 2) + 3) + 4) + 5 = ((3 + 3) + 4) + 5
{% endhighlight %}

The right hand side of this equation looks a lot like

{% highlight haskell %}
foldl ( + ) 3 [3,4,5]
{% endhighlight %}

This suggests the following definition for `foldl`:

{% highlight haskell %}
foldl f z [] = z
foldl f z (x:xs) = foldl f (f z x) xs
{% endhighlight %}

The left fold peels off the first element of the list, `x`, and feeds it to the
folding function, `f`, along with the previous starter value, `z`. The result
is then passed into the recursive call as the new starter value. Once the list
is exhausted, the fully accumulated starter value is returned.

To my mind, the recursion behind the right fold is even simpler. Using more
algebra-fu, notice that

{% highlight haskell %}
foldr ( * ) 1 [1,2,3,4,5] = 1 * (2 * (3 * (4 * (5 * 1))))
                          = 1 * (foldr ( * ) 1 [2,3,4,5])
{% endhighlight %}

This suggests the following definition:

{% highlight haskell %}
foldr f z [] = z
foldr f z (x:xs) = f x (foldr f z xs)
{% endhighlight %}

## Back to Ruby

One of the fun things about Haskell is that it gives you a clean
syntax and semantics for exploring recursion. Ruby is less encouraging of
recursion, but it's doable. As an exercise, what would left and right
folds look like in Ruby? I really encourage you to stop reading and perform
the translation yourself.

Here's how I might implement them, again as monkeypatches on `Array`:

{% highlight ruby %}
class Array
  def foldl z, &f
    if length == 0
      z
    else
      drop(1).foldl f.call(z, first), &f
    end
  end

  def foldr z, &f
    if length == 0
      z
    else
      f.call first, (drop(1).foldr z, &f)
    end
  end
end
{% endhighlight %}

I think the biggest takeaway from this is that higher order programming is
awkward in Ruby because blocks aren't first class citizens. We need to use the
ampersand operator to grab a `Proc`-ified version of the block so that we can
reference it in the rest of the method. Not the end of the world, but good luck
passing two functions.

## Parting thoughts

There are lots of other interesting things to learn about folds. I'd encourage
you to check out 
[Learn You a Haskell](http://learnyouahaskell.com/higher-order-functions), as
well as Graham Hutton's [tutorial](http://www.cs.nott.ac.uk/~gmh/fold.pdf),
which includes all sorts of cool examples of implementing other functions in
terms of folds.
