---
layout: post
title: Some fun with recursion, and a little Haskell
---

# Some fun with recursion, and a little Haskell

The other day I started reading Thomas Judson's open source algebra book,
[Abstract Algebra: Theory and Applications](http://abstract.ups.edu/index.html),
and came across a familiar identity for n choose k:

<div class="latex">
  \( \binom{n}{k} = \binom{n - 1}{k} + \binom{n - 1}{k - 1} \)
</div>

As Judson mentions, you can check that this is true with a little arithmetic:
plug in 

<div class="latex">
  \( \binom{n}{k} = \frac{n!}{k! (n - k)!} \) 
</div>

and then chug.

But is there a nicer way? The identity is, after all, a very simple recursive
definition for n choose k; presumably there's an elegant, intuitive explanation
for why it should be true.

## Some philosophizing

There's been some discussion
[lately](http://news.ycombinator.com/item?id=2440364)
about recursion on Hacker News, and whether or not it's an inherently difficult
or abstruse concept. I confess that it was initially not obvious to me why this
recursion relation for n choose k should be true; I've always used the formula
with factorials, which has a nice and intuitive derivation of its own.
Now, I _love_ recursion, so I think it's interesting that it had never even
occurred to me to look for a recursion relation for n choose k.  If recursion
is so obvious, why hadn't I thought to use it?

My claim is that recursion is _not_ particularly obvious, and that human
brains, unaided by courses on functional programming, just aren't very good at
it. The "wait... what? That works? ... wait what?" feeling that most people
have when first exposed to recursion is indicative of the fact that
recursion is very rarely a natural way for a human being to solve a problem.

For instance, after I figured out why the recursion relation works for n choose
k, I excitedly told a friend. He's very numerate but not a programmer, and he
stared at me a little blankly before asking, "... why not just use the
factorial version?"

And yes, as a human being, you would never want to actually compute n
choose k using recursion; it would take forever. You may object that factorials
are naturally defined recursively, so you're still using recursion at the end
of the day. However, I would argue that most people tend to think of factorials
_textually_, not recursively; 5! = 5 * 4 * 3 * 2 * 1, all at once. And
factorials are one of the simplest examples of recursion!

If you want to get to the point where recursion seems natural, you need to
practice it, and be on the lookout for patterns that yield to recursive
solutions. Thinking about n choose k recursively added another pattern to my
collection, so let's do some math :)

## A derivation

Abstractly, n choose k is defined to be the number of ways of choosing k
elements out of a collection of n elements, ignoring order. Alternatively, you
could phrase that as the number of distinct k-element subsets of a set with n
elements.

For instance, if you have a set of three elements, {1, 2, 3}, and want to pick
two of them, there are three ways to do it: {1,2}, {1,3}, and {2,3}. This means
that "three choose two" is three.  By convention we'll say that three choose
zero is one, because every set technically has the empty set as its lone
zero-element subset, and three choose any number larger than three is zero.

The goal is to start with this intuitive notion for what n choose k means, and
show that

<div class="latex">
  \( \binom{n}{k} = \binom{n - 1}{k} + \binom{n - 1}{k - 1} \)
</div>

Let's say S is a set with n elements. The trick is the following: pick some
element x in S (your "favorite", doesn't matter which one), and observe that
every k-element subset of S either contains x, or does not contain x. Kind of
trivial, but true. This quality of containing x or not containing x partitions
the subsets of S.  If we can count the number of k-element subsets that contain
x and we can count the number of k-element subsets that don't contain x, then
we can count the total number of k-element subsets by adding the two together.

<p>
Here's the magic: how many k-element subsets are there that don't contain x?
That's equivalent to picking k elements out of the set S - {x}, which contains
n - 1 elements; but that's simply \( \binom{n - 1}{k} \), by definition! And
what about k-element subsets that do contain x? That's equivalent to picking k
- 1 elements (we've already picked x) from S - {x}; and that's simply \(
\binom{n-1}{k-1} \)!
</p>

And that's it! Along with the base cases, this gives us a perfectly executable
definition of n choose k. Here's what it looks like in Haskell:

<code>
{% highlight haskell %}
choose :: Int -> Int -> Int
choose n 0 = 1
choose 0 k = 0
choose n k = choose (n - 1) k + choose (n - 1) (k - 1)
{% endhighlight %}
</code>

<br/>

## The pattern

I really like math puzzles that suddenly seem trivial once you realize
the trick; oftentimes the trick is applicable to a bunch of other problems.

The trick in this problem was figuring out how to partition the k-element
subsets of S into two disjoint classes; having done so, we can then recursively
count the two classes, add their counts together, and still have time to write
a blog post about how recursion is really cool.

## Practice puzzles

Here are a bunch of other puzzles that, broadly speaking, yield to the same
trick: figure out how to split the problem into disjoint subproblems, and then
recurse.

Puzzle #1: Write a function that explicitly computes all k-element subsets
(sublists) of a list with length n. You can check that you've generated all of
them by making sure the number of sublists is n choose k. This is also a nice
follow-on to the previous discussion, since the code will look a lot like the
code from above.

Puzzle #2: Write a function that computes the powerset of a set (the set of all
subsets of the set). As an example, the powerset of {1,2} should be { {}, {1},
{2}, {1,2} }. Note that the empty set, {}, is a subset of any set.

Puzzle #3: Write a function that computes permutations. For example, the
permuations of {1,2,3} should be {1,2,3}, {1,3,2}, {2,1,3}, {2,3,1}, {3,1,2},
and {3,2,1}. Starter problem: how would you count permutations?

Puzzle #4: Write a function that counts the number of ways to partion a list.

Puzzle #5: Write a function that explicitly generates partitions. For example,
your function should map {1,2} --> { { {1}, {2} }, { {1,2} } }.

For me at least, the partition puzzles were the hardest.

## Puzzle #1: Computing all k-element sublists

The strategy here is almost identical to what we did for n choose k: given a
list of the form (x:xs), any k-element sublist either contains x or it doesn't.

We can generate all k-element sublists that do contain x by first generating
all sublists of xs with k - 1 elements, and then consing x onto all of them.

And we can generate all k-element sublists that don't contain x by generating
all k-element sublists of xs.

Here's the code. Hopefully the base cases make intuitive sense.

{% highlight haskell %}
sample :: Int -> [a] -> [[a]]
sample 0 _ = [[]]
sample _ [] = []
sample k (x:xs) = 
  let withFirst    = map (x:) $ sample (k - 1) xs
      withoutFirst = sample k xs
  in withFirst ++ withoutFirst
{% endhighlight %}
<br/>

## Puzzle #2: Computing powersets

Consider a list of the form (x:xs). As we keep doing, we can split the sublists
of (x:xs) into two disjoint sets: those that contain x, and those that don't.

We can generate the list of sublists containing x by first generating the
powerset of xs, and then consing x onto all of those sublists.

And the list of sublists not containing x is simply the powerset of xs.

{% highlight haskell %}
powerset :: [a] -> [[a]]
powerset [] = [[]]
powerset (x:xs) = 
  let ps = powerset xs
  in map (x:) ps ++ ps
{% endhighlight %}

However, we can also compute powersets with our sample function. Another
perfectly valid way to split the sublists into disjoint subsets is by their
length. The powerset of a list with n elements with include sublists with
length zero (the empty set) all the way up to n (the entire list).

Intuitively, what we'd like to do then is compute 
{% highlight haskell%}sample k (x:xs){% endhighlight%} for all k
between 0 and n, and then join together all of the results.

Here's the code, using Haskell's list monad:

<code>
{% highlight haskell %}
pset :: [a] -> [[a]]
pset xs = do
  k <- [0..length xs] 
  sample k xs
{% endhighlight %}
</code>

If you haven't gotten to monads yet, don't worry. In words, the code says to
sample k xs for all k in the span [0..length xs], and then collect all of
the results into one big list. So even if the monad syntax makes zero sense,
the idea behind the code is actually very simple.

## Puzzle #3: Computing permutations

As usual, we want to find a way to split the possible permutations of a given
list (x:xs) into disjoint subsets. One way is to split the permutations based
on what element they start with. For example, the permutations of {1,2,3} split
nicely into {1,2,3}, {1,3,2}; {2,1,3}, {2,3,1}; and {3,1,2}, {3,2,1}.

We can generate all of the permutations that start with x by consing x onto all
of the permutations of the rest of the list---the part that doesn't include x.

Here's one way, again using the list monad. To use this you'll need to import
the Data.List module, which includes the "\\\\" function; it does "list
subtraction", if you will.

<code>
{% highlight haskell %}
perms :: (Eq a) => [a] -> [[a]]
perms [] = [[]]
perms xs = do
  x <- xs
  perm <- perms (xs \\ [x])
  return (x:perm)
{% endhighlight %}
</code>

In words, the code says to loop over every x in xs, generate every possible
permutation of xs - {x}, cons x onto every one of those permutations, and then
collect everything into a big list.

## Puzzles #4 and #5: Partitions

These are really beautiful. Let S be a set with n elements, labelled from 1
to n. A partition of S is a set of non-empty subsets of S that span S; for
instance, one partition of {1,2,3} is { {1}, {2,3} }.

How can we split the partitions of S into nice disjoint subsets? Well, every
partition must include a subset that contains your favorite element of S, say
1; if it didn't, then the partition wouldn't span all of S.

The trick is that a given partition can only include one subset containing 1;
subsets of the form {1, ...} partition the partitions of S!

An example might help to explain this: lets look at the partitions of {1, 2,
3}. We'll use this idea of labelling partitions by which {1, ...} subset they
include to enumerate all the partitions.

First, the ones that include {1}:
* { {1}, {2,3} }
* { {1}, { {2}, {3} } }

Next, the ones that include {1,2} or {1,3}:
* { {1,2}, {3} }
* { {1,3}, {2} }

And finally, there's the partition that includes {1,2,3}, e.g. the whole set.
Evidently there are five partitions of {1,2,3}.

Let's call this initial {1,...} subset the anchor of a partition; any partition
of S has a unique anchor. We can generate a (k + 1)-element anchor by first
choosing any k elements from S - {1}, and then consing 1 onto the front. We can
get all of the possible anchors for a set by doing this for all k between zero
and and one less than the length of the list.

Here's some Haskell code to that effect:

<code>
{% highlight haskell %}
anchors :: [a] -> [[a]]
anchors [] = [[]]
anchors (x:xs) = do
  k <- [0..length xs]
  map (x:) $ sample k xs
{% endhighlight %}
</code>

Having specified the anchor, the rest of the partition consists of any
partition of the rest of the elements in S---which is where recursion comes in!
For every choice of anchor, we need to recursively generate all of the
partitions of the remaining elements in S, cons the anchor onto the front of
all of them, and then join everything together into a big list.

Once again, it would probably help to illustrate this with an example.
Returning to {1,2,3}, the first anchor is {1}. We'll generate the partitions
including {1} by generating all partitions of the rest of the set, {2,3}.
Rather than go through another layer of recursion, let's just enumerate them:

* { {2}, {3} } 
* { {2,3} } 

The next step is to cons {1} onto the front of each partition, which yields

* { {1}, {2}, {3} }
* { {1}, {2,3} }

Doing the same thing for the other anchors yields the rest of the partitions:

* { {1,2}, {3} }
* { {1,3}, {2} }
* { {1,2,3} }

To finish, just collect all of the partitions into a big list.

Here's the Haskell. I'll explain it line by line, but the cool thing is that 
"the code follows the words" to a pretty remarkable extent.

<code>
{% highlight haskell %}
parts :: (Eq a) => [a] -> [[[a]]]
parts [] = [[]]
parts xs = do
  anchor <- anchors xs
  partition <- parts (xs \\ anchor)
  return (anchor : partition)
{% endhighlight %}
</code>

I think this is a really cool example of the list monad in action. The anchor
xs line says to generate every possible anchor from the xs. The next line says
for every such anchor, recursively generate every possible partition from (xs
\\\ anchor), i.e. from the xs minus the elements already consumed by the
anchor.  The final line says to cons the anchor onto the partition, and collect
_all_ such anchor : partition combinations into a list.

This post is getting a little long, but I want to talk quickly about counting
partitions; I'll let you write the Haskell :)

Let C(n) denote the number of partitions of an n-element set. The plan is to
count the partitions of an (n + 1)-element set S by adding up the partitions
labelled by each anchor.

Every partition of S, per the Haskell code above, is of the form a : p, where a
is an anchor, and p is a partition of the elements in S not eaten up by the
anchor.  All anchors contain a common element, say x, so a (k + 1)-element
anchor will contain k elements drawn from the set S - {x}. The remaining
partition p must then be a partition of the remaining (n + 1) - (k + 1) = n - k
elements.

At the risk of adding one two many variables to the mix, let i = n - k; i is
the number of elements contained in p. Just trust me, it makes things look a
bit cleaner. A (k + 1)-element anchor will then be an (n - i + 1)-element
anchor.

Okay, how many partitions are there of an i-element set? By induction, C(i).

How many (n - i + 1)-element anchors are there? We need to draw (n - i)
elements from the set S - {x} (because we've already implicitly drawn x), which
contains (n + 1) - 1 = n elements, so there must be n choose (n - i) different
(n - i + 1)-element anchors.

For each of these anchors there will be C(i) partitions, so we can sum over all
i between zero and n to get

<div class="latex">
  \( C(n + 1) = \sum \binom{n}{n - i} C(i) \)
</div>

We can write this in a nicer form by noting that n choose k = n choose (n - k).
Intuitively, choosing k elements from an n-element set is equivalent to _not_
choosing (n - k) elements. Thus we wind up with the following lovely formula:

<div class="latex">
  \( C(n + 1) = \sum \binom{n}{i} C(i) \)
</div>

## Wrapup

Hopefully this has been a fun. In the future, I'd like to use this blog to
write about similar things: interesting math, stuff about programming
languages, maybe a little physics.

I've [submitted](http://news.ycombinator.com/item?id=2468467) this to Hacker
News, so please let me know if anything is unclear, mistaken, or hideously
inefficient---I'm mostly self-taught and haven't really thought much about
performance yet. 
