---
layout: post
title: "Jason Arhart Talks RSpec"
date: 2013-10-07 23:31
comments: true
categories: ruby, presentations, testing, rspec
---
We've been spending a lot of time thinking about and implementing test-driven development
using RSpec at [Flatiron School](http://www.flatironschool.com) these past several days.
While I certainly love the *idea* of loving TDD, I've found myself struggling to understand exactly
how to effectively start dipping my toe into the Ruby testing pool.

It's not that I don't grasp the concept of testing first. Rather, I haven't been able to
figure out the appropriate tests to write before I actually start coding. I often find myself combining
the following steps:

    1. Pseudo coding a solution to the programming problem
    2. Thinking of how I could test that pseudo code
    3. Thinking of how I could turn that pseudo code into Ruby
    4. Thinking of tests to write to test that code

Sadly, this creates a very un-TDD workflow. I either write tests that are completely
unrelated to my eventual code, or I end up writing code and then creating tests that
conform to that code.

I came across a wonderful [deck by Jason Arhart](https://speakerdeck.com/lvrug/rspec-for-beginners-jason-arhart) today, though,
that has finally (sort of) given me a sense of how to approach this whole TDD thing. This slide in particular was my "aha"
moment...


<script async class="speakerdeck-embed" data-slide="11" data-id="ec123320ddad0130679a56e32e49313d" data-ratio="1.74446337308348" src="//speakerdeck.com/assets/embed.js"></script>

It seems so simple, but the line "focus on **what** before **how**" is quite a brilliant way of putting it.

I'd been struggling to write effective RSpec tests (let alone even come up with tests *at all*) because
I was approaching the whole process with a "how" mindset. I thought of my tests in the context of how I'd be solving the
programming problem that was before me, rather than as a way to clearly document and test the "what" of the problem. The distinction is nuanced, but the way to properly test is to think, "what should my methods/objects/whatevers be doing?" instead of "how am I going to get my methods/objects/whatevers to solve this problem and also prove to some test that they are working?"

Of course, I still couldn't whip together a good test suite right now, but this presentation has given me much more clarity as to how I should actually approach writing passable (hah!) test stuites in the near future.