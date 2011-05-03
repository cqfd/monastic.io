---
layout: post
title: Special relativity is subtle!
---

# Special relativity is subtle!

I've recently started doing a little physics tutoring, and the kid I'm working
with is learning special relativity. It's been a while since I've thought about
SR, so I've been refreshing my memory by reworking a bunch of the standard
problems.

In this post I want to give a fun example of just how subtle SR can be: the
"velocity addition formula". I decided to rederive it for myself as part of my
reintroduction to SR. Two points about my derivation:

* It assumes you know how length contraction and time dilation work.

* It's totally wrong. The mistake is a bit subtle, and it's a fun exercise to
try to spot it.

## A specious derivation

<p>
Suppose that we're sitting in an inertial frame in the depths of outer space.
To our left, our friend Alice floats along with velocity \(v\), and to our right,
our friend Bob floats along with velocity \(-v\).
</p>

<p>
Bob intends to measure Alice's velocity in his own inertial frame by timing how
long it takes her to float past his handy measuring stick, which has length \(L\)
in his frame.
</p>

This means we're interested in two events: when Alice gets to the front of
Bob's measuring stick, and when Alice gets to the end of Bob's measuring stick.

<p>
Let's start by looking at these events in our own inertial frame. Due to length
contraction, Bob's measuring stick has contracted length \(L/\gamma\), where
\(\gamma\) is special relativity's ubiquitous scaling factor,
</p>

<div class="latex">
  \( \gamma = \frac{1}{\sqrt{1 - v^2/c^2}} \)
</div>

<p>
Let's calculate the time that elapses in our frame between these two events. If
Alice reaches the start of the measuring stick at \(x = 0\), then she'll reach
the end of the stick at \(x = L/(2\gamma)\). She's floating with velocity
\(v\), so this will take time
</p>

<div class="latex">
  \( \Delta t = \frac{L}{2 v \gamma} \)
</div>

<p>
Now let's look at things from Bob's perspective. Given that he's moving with
speed \(v\), his clocks will tick slower than ours by a factor of \(1/\gamma\);
that is, he will measure the elapsed time between the two events as
</p>

<div class="latex">
  \( \Delta t' = \frac{\Delta t}{\gamma} = \frac{L}{2 v \gamma^2} \)
</div>

<p>
Furthermore, in Bob's frame the rod still has length \(L\), so if it takes
Alice time \(\Delta t'\) to traverse a distance \(L\), then her velocity in
Bob's frame must be
</p>

<div class="latex">
  \( v' = \frac{L}{\Delta t'} = 2 v \gamma^2 = \frac{2 v}{1 - v^2/c^2} \)
</div>

<p>
Unfortunately, this is wrong. The correct result is
</p>

<div class="latex">
  \( v' = \frac{2 v}{1 + v^2/c^2}  \)
</div>

So, where's the mistake?
