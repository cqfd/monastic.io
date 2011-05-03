---
layout: post
title: Continuations for the Cranky and Confused, Part I
---

# Continuations for the Cranky and Confused, Part I

Of all the programming language concepts I'm come across so far, none has made
me feel quite as cranky and confused as continuations. Everything else has gone
down pretty smoothly, from closures to higher-order functions to monads to
funky Ruby metaprogramming. But continuations have been confusing as hell.

In the hopes that one more tutorial will get you over the hump, here are my own
quirky thoughts. In hindsight, I don't think continuations should be all that
scary; existing tutorials just emphasized things that didn't really click for
me.

The plan is to play around a bit in
[Racket](www.racket-lang.org), a nifty new descendant of Scheme, and then in
a subsequent post we'll build all the way up to tackling Haskell's rather
badass continuation monad.

## Continuations in Racket

It's traditional at this point in a continuations tutorial to say a bunch of
mumbo jumbo about reifying control contexts, or something. Let's skip that
part. As a beginner it's totally incomprehensible.

Instead, let's look at the following bit of code:

{% highlight scheme %}
(+ 2 3)
{% endhighlight %}

Now, imagine that you're the number 3. You're minding your own ternary
business when suddenly a weird s-exp thing comes along and gobbles you up. It 
smooshes you together with a + and a 2 and out pops a 5. Don't worry, you're
a [persistent data
structure!](http://existentialtype.wordpress.com/2011/04/09/persistence-of-memory/)

This weird gobbling s-exp thing is known as a continuation. You can think of it
as a regular old procedure:

{% highlight scheme %}
(lambda (x)
  (+ 2 x))
{% endhighlight %}

You, the number 3, get passed to this procedure and out pops a 5.

Languages like Racket ask a really interesting question: what if you, the
programmer, had access to this implicit procedure, this continuation? In this
simple case we were able to synthesize the procedure by hand, but what if the
programming language would just give it to you for free?

With that thought in mind, let's look at some more code:

{% highlight scheme %}
(+ 2 (call-with-composable-continuation
       (lambda (k)
         3)))
{% endhighlight %}

You can look up the official treatment of
<code>call-with-composable-continuation</code> in the Racket
[docs](http://docs.racket-lang.org/guide/Continuations.html). Don't worry
though, I'll try to explain everything here as well.

Running the code will return 5, as before. But what's going on?

Racket's <code>call-with-composable-continuation</code> takes a function as
an argument; this function should in turn take a single argument, which, _by
black magic_, will be set to the continuation. More on that in a bit. In the
previous example, the function that we've passed to
<code>call-with-composable-continuation</code> is this guy:

{% highlight scheme %}
(lambda (k) 3)
{% endhighlight %}

The idea is that <code>call-with-composable-continuation</code>
will call this function, setting the argument k to the continuation. Again,
black magic! In this case the function we're passing to
<code>call-with-composable-continuation</code> is kind of lame and doesn't even
do anything with the continuation, but we can change that.

{% highlight scheme %}
(+ 2 (call-with-composable-continuation
       (lambda (k)
         (display (k 10))
         (display "\n")
         3)))
{% endhighlight %}

Running this guy will print 12 and then return 5. Interesting!

As you may have guessed, k is precisely the implicit procedure I mentioned
above. That is, k is identical to

{% highlight scheme %}
(lambda (x)
  (+ 2 x))
{% endhighlight %}

Calling <code>(display (k 10))</code> runs k with 10 as its argument, returning
12, and then prints it. Then we print a newline for aesthetics, and finally
return 3 to the surrounding addition, getting 5.

The thing I want to emphasize as being so cool about this is that Racket *gives
us the continuation for free*. How this works under the hood is a
question for Racket's designers, or at least for another post; the important
part is that somebody, back in the day, decided that it would be pretty cool if
things worked this way.
