---
layout: post
title: "understanding rebasing"
date: 2013-09-24 16:48
comments: true
sharing: true
footer: false
author: Logan Hasson
---
Today was full of pulling and fetching and branching and merging and commiting and, yes, the dreaded **rebasing**. I'll admit: before today, though I'd read about it on multiple occasions, rebasing in Git still made my brain physically hurt. And even as today wore on, I found myself more and more perplexed by this scary, repo-busting command.

Until I had an epiphany.

Git rebasing makes sense when you realize that *it does exactly what the name says it'll do.* Stick with me here.

Let's, for a second, imagine we have a repo that looks something like this:

![A possible Git repository](/images/repo-1.jpg)

Now, let's throw out all we know about rebasing (or think we know about rebasing), and just concentrate on what the command actually says. If we rebase `feature` onto `master`, we are literally changing the base (or parent!) of `feature`.

Let's look at the repo again, but this time with a bit more descriptors:

![A more verbose Git repository](/images/repo-2.jpg)

As it stands, `feature` begins it's life as a child of our second commit. If we `rebase` it onto `master` we are essentially saying, "Nope, let's rewrite history. Let's pretend that `feature` began it's life as a child of our fifth commit (the most current commit in the `master` branch)." So to make this rebase happen, we run the following commands:

```
git checkout feature
git rebase master
```

What Git does, to rewrite history, is take every commit between **C2** and **C5** (inclusive) and apply it to the `feature` branch. Because, for the `feature` branch to have a new base, it needs to know the whole history of that particlar base (parent).

Now, our repo looks like this:

![A Git repository after rebasing](/images/repo-3.jpg)

Woah! Now we have a nice clean history (it's all linear and stuff!), but `feature` is ahead of `master`. Why is that?

This is what initially confused the heck out of me, once I thought I understood rebasing. There's two things to keep in mind for this to make sense:

* The whole "rebase onto" nomenclature is, in and of itself, somewhat confusing.
* If we hadn't rebased or merged, the `master` branch would never have any concept of the `feature` branch after our second commit.

Let's tackle these points one at a time.

1) *The whole "rebase onto" nomenclature is, in and of itself, somewhat confusing.*

Here's the thing. Everything I've written before this is based upon the fact that the `rebase` command actually does what it's name implies. Cool. However, when we add the word "onto" into the mix, things get confusing. To me, `rebase onto` (combined with the myriad descriptions of rebasing as the rewinding of and playing back of commits onto one branch or another) makes it seem as if the branch being, erm, rebased onto, is being acted upon in some way.

**It isn't.**

2) *If we hadn't rebased or merged, the `master` branch would never have any concept of the `feature` branch after our second commit.*

Let's look at another version of our original repo, but with some files added to the mix. (Each time you see a file name next to a commit, this means the file was created and commited in that commit. These are simplistic commits for explanation purposes.)

![A Git repository with some files](/images/repo-4.jpg)

Cool. Now let's look at the same repository, but this time with the files carried through from one commit to the next. In other words, now we'll look at what files each commit knows about:

![A Git repository with some files and inheritance](/images/repo-5.jpg)

Now, if we created a new branch off of `master` right now, what files would it inherit?

If you said *files 1, 2, 3, 8, 9, and 10*, then you'd be right.

So if we rebase a branch onto `master` now, it too would know about those files. (It's new parent is our current `master` branch, and thus inherits everything from that branch.)

![A Git repository with some files and inheritance after rebase](/images/repo-6.jpg)

But if you notice from that image above, our `master` branch has no clue that *files 4, 5, 6, and 7* exist. Rebasing our `feature` branch, which does know about those files, onto `master` just means that now `feature` knows about *files 1, 2, 3, 8, 9, and 10*, just as any new branch would at this point!

Aha! So in order to catch `master` up on all the lovely things that we've been working on in `feature` branch, we still need to merge after the rebase is complete. And then, we get this:

![A Git repository after rebase and merge](/images/repo-7.jpg)

But, bonus side effect: we don't have to worry about conflicts. Sweet.

Phew. Git rocks.