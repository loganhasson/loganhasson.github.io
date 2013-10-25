---
layout: post
title: "All Bound Up"
date: 2013-10-23 10:56
comments: true
categories: ruby, erb, binding, scope
---
When getting started with ERB (Embeded Ruby), you'll undoubtedly come accross a weird
little thing called `binding`. It won't pop up all the time, but you'll be minding your own business
one day, reading tutorials, when suddenly you'll see this:

```ruby
require 'erb'

template = ERB.new(File.open('/blah/blah.erb').read)
some_variable = "This is a variable"

File.open('/some/html/file.html', 'w+') do |f|
  f << template.result(binding)
end
```

And then you're like, "Whoa, whoa, whoa, what the heck is binding?" And then the tutorial says that you can't
access `some_variable` inside your `'/blah/blah.erb'` template file unless you pass your `binding` along as an argument 
with the `.result` method.

And then you're like, "Ok, but whoa, whoa, whoa, what is binding?" And the tutorial says that you don't have to
really worry about it, and that you can do this instead:

```ruby
require 'erb'

template = ERB.new(File.open('/blah/blah.erb').read)
@some_variable = "This is an instance variable"

File.open('/some/html/file.html', 'w+') do |f|
  f << template.result
end
```

And you get some explanation along the lines of, "Your binding is like a snapshot of your current environment.
If you want your `template.erb` file to have access to any local variables, you have to pass along your `binding`
with the `.result` method. This lets your template have access to your current environment and all variables defined
within it."

And you go, "Oh, ok. I guess that makes sense. Cool."

And your friendly tutorial person ends with the note, "But really, you can just always define instance variables
and your template.erb file will automatically have access to them without all this passing of binding around because it elevates
their scope."

And you think you get it. And you think the concept of `binding` is really cool. But you know that it's weird to pass it around, so you just start whipping out instance variables, assigning them, and moving along with your day.

Until this happens (Let's ignore for a second that this weird Dog knows how to make HTML...pretend it's a
special class that only generates HTML):

```ruby
require 'erb'

class Dog
  
  def make_me_some_html
    template = ERB.new(File.open('/dog/dog_show.erb').read)
    @name = "Pooch"

    File.open('dog.html', 'w+') do |f|
      f << template.result
    end
  end

end

Dog.make_me_some_html
```

You open up, excitedly, `dog.html`, and...nothing. Literally nothing. No "Pooch", no
error. Nothing.

And you go, "Oh, well, I must have made a typo. There's probably an error sitting in my
terminal." But there isn't.

And then, after debugging for what seems like days, you try tacking on a `(binding)` to
`template.result` as a last ditch effort to save face. So now your code looks like this:

```ruby
require 'erb'

class Dog
  
  def make_me_some_html
    template = ERB.new(File.open('/dog/dog_show.erb').read)
    @name = "Pooch"

    File.open('dog.html', 'w+') do |f|
      f << template.result(binding)
    end
  end

end

Dog.make_me_some_html
```

Huzzah! It worked. It worked! Wait...why in the world didn't it work before? It should have worked.

Here's the thing. Binding is, indeed, a snapshot of your current environment. And before, when we defined
instance variables and didn't pass our binding with `.result`, it *seemed* like they were accessible by our `template.erb`
because they were part of our current binding. And that was true.

The problem is, `.result` doesn't actually take your current binding by default. Its signature reads as follows:

`result(b=new_toplevel)`

What's actually going on here is our *top level binding* is being passed along with `.result` by default. And what
is our top level binding? It's essentially the environment in which `main` exists. So everything that is defined in our `main` object, except for local variables, (all method signatures outside of a class, all class definitions, all instance variables --
fun fact, `main` does indeed have instance variables!, and all other constants) get passed as a parameter of `.result` by default as our binding. It just so happened that before, our top level binding *was* our current binding.

With this in mind, it makes sense that the above example with our `Dog` class only worked properly when we eplicitly sent
the current binding as `.result`'s argument. The `@name` instance variable is defined *within* the scope of the `make_me_some_html` method in the `Dog` class. It is not part of the `main` binding, and therefore doesn't exist in our template file without
some help.