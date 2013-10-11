---
layout: post
title: "Dot(.) Send Me to the Moon"
date: 2013-10-10 18:20
comments: true
categories: ruby, methods, code, useless
---
One of the greatest things about Ruby is its never-ending ability to momentarily bewilder you before exploding your brain all over your face. Whether it's a seemingly supurfluous bit of syntactic sugar (point for alliteration to me, in case you're counting) or a questionable method call where one doesn't seem necessary, Ruby constantly has me scratching my head mere seconds before blowing me away.

My most recent moment of *ermahgerd* came when discovering the `.send` method. Here's what it looks like when called on an `Array`:

```ruby
array = [1, 2, 3, 4]
array.send(:[]=, 4, 5)

=> [1, 2, 3, 4, 5]
```

(The parenthesis aren't necessary, but I like them for clarity. Up in the code. Not surrounding this sentence, silly.)

So what exactly is this doing? It looks wacky, fo sho. To break it down, let's look at its equilavent in the way we normal human beings would most likely write it:

```ruby
array = [1, 2, 3, 4]
array[4] = 5

=> [1, 2, 3, 4, 5]
```

So yeah, that's different. To understand what's going on with `.send`, which, by the way, 'sends a message to a receiver', we have to remember that `array[whatever]` is really `array.[]=(index, value)`. Just because brackets are involved doesn't mean a method isn't being called (or that a message isn't being sent). `.send` then, basically passes a method to an object, along with a couple of arguments, and then invokes that method.

**Mini-break**: What does this whole "sends a message to a receiver" or "passes a method to an object" stuff mean? At first, it's kind of annoy...erm...I mean difficult to understand the oft-repeated refrain that "everything in Ruby is an object." First of all, what the heck is an object? And second of all, so what? Here's the quick and dirty deets, supremely over-simplified—when we say "object" we really just mean "thing". And now I'm guilty of saying it myself: everything really is an object. Like, *all the things* are ojects. Every last one of them. And they're just things that chill and wait for you to tell them to do stuff. Which is really cool, because all of these things can respond to a zillion different methods. And methods are little messages you "send" to these objects that tell them to do stuff. An object either understands a method or it doesn't. So if we think about it really abstractly, let's say I'm a Ruby object. (Sweet!) You could send me a method like `.karate` and I'd look at you like you're crazy because my parents never let me do martial arts when I was younger and that was sad and I'll stop rambling about that right now. But if you sent me a method like `.drink_water` I'd know exactly what to do—I can drink a glass of water like a boss—and I'd do it. This, in a nutshell, is what objects and methods are all about in Ruby. (Well, kinda. Maybe. You know what? Just keep moving...)

**Now, back to your regular programming (hah!)**

Cool.

Wait, why is it cool? Doesn't it seem like an unnecessary step? After all, as we just decided, the following code would look just as weird, get the same thing done, and not use the `.send` method at all:

```ruby
array = [1, 2, 3, 4]
array.[]=(4, 5)

=> [1, 2, 3, 4, 5]
```

(Ignore the weird syntax highlighting on that first bracket on line two—I think I broke the internet or something.)

So why bring `.send` into the mix at all? It just seems like extra…stuff, right? Well, what happens if we think about dynamically calling methods based on stuff that happens in our code? Or, even better, what if we want to call methods depending on user input? Then we can do some *really* cool stuff.

Well, I mean, somebody could do really cool stuff. I'm just going to show you how to make a silly, over-simplified, bot-type-thingy that sorta-kinda responds to user input. Ish.

```ruby
class ChatterBox
  COMMANDS = [:hi, :how, :why, :when, :who, :where, :what, :bye]

end
```

Boom…done! Ok, kidding. But we've gotten started at least. Here we're just defining a `class` called `ChatterBox` that we'll be using to demonstrate a neat little use of `.send` that comes in handy. Maybe not brain-explode-all-over-your-face handy, but huh-would-you-look-at-that handy. The `COMMANDS = [blah, blah, blah]` bit is going to be useful in just a second. Actually, it's going to be useful right now.

Let's pretend for a second that we have some fancy pants command line interface that we can wrap around this sucker and that everytime a user types something in, that sentence gets passed to a `.call` method on our `ChatterBox`. Cool? Cool.

```ruby
class ChatterBox
  COMMANDS = [:hi, :how, :why, :when, :who, :where, :what, :bye]
  
  def call
  
  end
end
```

Sweet. Now let's make it do something. I'm going to throw a bunch of code in there, and then we'll chat (hah again!) about it:

```ruby
class ChatterBox

  COMMANDS = [:hi, :how, :why, :when, :who, :where, :what, :bye]

  def call(sentence)
    sentence.split.each_with_index do |word, i|
      if self.class::COMMANDS.include(word.downcase.gsub(/[.?!,']/,'').to_sym)
        self.send("#{word.downcase.strip.gsub(/[.?!,']/,'')}", sentence.downcase.gsub(/[.?!,']/,'').split)
        break
      elsif i == sentence.length - 1
        puts "I have no idea what you're talking about."
      end
    end
  end

end
```

The first line is pretty basic: it takes the passed in argument, sentence, splits it up (by default that happens at the spaces) using the `.split` method, and iterates through that newly-minted `array`, while keeping track of the index.

And now we move on to the dopeness that is `.send`.

`COMMANDS` is just our array of possible commands we want to accept from the user. We first check to see if any word in the sentence that was passed in, when converted to a `sybmol` matches one of our possible commands. If it does, we get to break out the magic. `.send`, remember, works like this:

```ruby
object.send("method_as_string", arguments)
```

So in this case, our `object` is our instance of the `ChatterBox` class, and we send it, as a method, the word that matched one of our possible commands. Then, we send the rest of the sentence along as arguments. `.send` takes care of all of this for us. It converts the `string` version of the command into a symbol and uses that to call the method of the same name that we've written into the class. To that method, it passes the other non-matching part of the sentence.

In other words, say we do something like this:

```ruby
c = ChatterBox.new
c.call("What are you doing tonight?")
```

In the sentence we passed to the `.call` method, there is one of our commands: 'What'. `.send` then gets a hold of this in our `.call` method and turns it into this:

```ruby
c.send("what", ["are", "you", "doing", "tonight"])
```

And remembering how the `.send` method works, we know that this is going to call a method, `.what` and send it `["are", "you", "doing", "tonight"]` as an argument.

So it's essentially doing this:

```ruby
c.what(["are", "you", "doing", "tonight"])
```

But we didn't actually have to hard-code in a call to that `.what` method.

Pretty neat, right? Based on a passed in sentence, we can fire off a specific method and then use that method to parse a sentence in any way that we want.

One of these (reallyverymessymethodsthatyoushouldonlylookatforliketwosecondssoyoureyesdon'tmelt) methods might look something like this:

```ruby
class ChatterBox

  COMMANDS = [:hi, :how, :why, :when, :who, :where, :what, :bye]

  def what(args)
    if args.include?("is") && args.include?("your")
      puts "Personal! Sheesh!"
    elsif args.include?("is") || args.include?("are")
      puts "Get a dictionary, bonehead. I'm not Wikipedia over here."
    elsif args.include?("should")
      puts "I ain't a fortune teller, kid. And I certainly don't work for free."
    elsif args.include?("are")
      puts "That's none of your business. I do what I do. Ya herd?"
    else
      puts "Huh?"
    end
  end

  ...

end
```

I'll let you go through that and see how poorly a person can actually write a bunch of `if` statements, but it's pretty fun to run if you add similar methods for the other possible commands, if I do say so myself. Here's some sample output from this (ignore the syntax highlighting again):

```
t = ChatterBox.new

t.call("What is your favorite color?")
=> Personal! Sheesh!
t.call("Hi there!")
=> Well hi there! Nice to see you. :)
t.call("Why can't we be friends?")
=> We'd never work together. It's so obvious.
t.call("How old are you?")
=> Younger than yo mama!
t.call("What is the meaning of life?")
=> Get a dictionary, bonehead. I'm not Wikipedia over here.
t.call("Where were you born?")
=> Probably on the moon.
t.call("Who am I?")
=> How in the world could I know that? You didn't tell me your name!
t.call("When can I meet your family?")
=> A long time from now.
t.call("How is cheese made?")
=> I'm not too sure how...I'm a computer, after all.
t.call("Gobledygook blah pants")
=> I have no idea what you're talking about.
```

Hey, look! We made a really pathetic AIM bot, minus AIM and the internet and other stuff. Ok, really we just made a totally useless program to create a contrived example with which to demonstrate `.send`. Still, though, it's really awesome. `.send` is really awesome, to clarify, not this program. And supremely useful if you want to dynamically call methods based on user input (or in other situations where you don't really know what method is going to be called).

Not too shabby, Ruby. Not too shabby.