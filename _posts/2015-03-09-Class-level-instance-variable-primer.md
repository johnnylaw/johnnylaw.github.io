---
layout: post
title: Class-level instance variables in Ruby; a primer
---

There's a high likelihood that if you've found this page, it's because you're experiencing at least a little bit of pain that comes with unexpected results around class variables, designated in Ruby with names beginning in `@@`. As it turns out, although you may want to store a value on a class itself rather than on each instance of a class, it is almost NEVER the case that you want to use these simple class variables to do it.

It's likely that you've come to the point where you are ready to understand what is referred to in Ruby as the "class-level instance variable," a class-level variable whose results are likely to be more in line with what you expect and were hoping for.

### "The problem" with class variables
Take the example of the code below:

```ruby
class Animal
  def self.locomotion_technique
    @@locomotion_technique
  end
end

class Dog < Animal
  @@locomotion_technique = :quadrupedal
end

class Fish < Animal
  @@locomotion_technique = :undulatory_propulsion
end
```

Again, since you found this article, you may be having trouble with class variables that get overwritten by seemingly unrelated code (code in a subclass). What happens in this code above?  Calling the `locomotion_technique` method on any of the three classes above will all result in the same answer, namely :undulatory_propulsion, which is not a dog's favorite form of moving around, however much joy it might give humans to watch them try.

In fact, calling the `locomotion_technique` method on any class that is a subclass of `Animal` will yield the same result as calling it on any other, and to make matters worse, every time you assign a value in one subclass of `Animal`, you'll be overwriting the value for the `Animal` class and every one of its subclasses.

### How to fix it (Class-Level Instance Variables 101)
The answer is simple: Rewrite the code above, replacing each occurence of `@@` with `@`. That's it; problem solved. The code looks like this, and it probably does what you want it to:

```ruby
class Animal
  def self.locomotion_technique
    @locomotion_technique
  end
end

class Dog < Animal
  @locomotion_technique = :quadrupedal
end

class Fish < Animal
  @locomotion_technique = :undulatory_propulsion
end
```
 In all honesty, if you're in a hurry to crank out a feature, there may be no reason to read further.  I remember, however, that when I really grasped the concept of a "class-level instance variable" in Ruby, a whole new level of abstraction and meta-programming was opened up and I got real smiley, so if you wanna see how deep the rabbit hole goes, read on...

### Understanding it (Class-Level Instance Variables 202)
So let's start where I suspect you might be confused.  I just told you to return an instance variable from a class method and that that would fix your problem.  But that's just plain stupid, because if we know one thing about Ruby, it's that `@some_variable` is an *instance* variable and `@@some_variable` is a *class* variable, and *class* methods do not have access to *instance* variables, because the *class* method has no knowledge of an *instance*  and therefore no knowledge of that non-instance's variables (run-on sentence = rant)!

This thinking is not wrong; it's just that the instance variable we're returning isn't an instance variable of an instance of an `Animal`; it's an instance variable of an instance of something else.  You may see where this is going.

For just a sec, let's jump to 10,000 feet. Remember that everything in Ruby is an object. What does 'everything' include?  It's a trick question, but in this case, we want to highlight that it includes the `Animal` class (and all classes).  After defining the classes as above -- I would expect the following input to yield the following output:

```ruby
2.2.1 :014> animal = Animal.new
2.2.1 :015> animal.class
 => Animal
2.2.1 :016> Animal.class  # or Dog.class or Fish.class
 => Class
```
The `Animal` class can be seen to be an instance of the Class class.  **Every class** is in fact an instance of the Class class.

When we return an *instance* variable from a *class* context, i.e.

```ruby
class MyClass
  def self.my_method  # notice self.
    @my_property      # notice single @
  end
end
```
the variable `@my_property` is -- as we have clarified above -- in fact an *instance* variable.  The *instance* on which it exists is not an *instance* of MyClass, but rather it exists on `MyClass` *itself*, and as we just saw `MyClass` is an instance of `Class`.  Re-read from "When we return..." until you get it; the meat is in here!

Those of you familiar with the somewhat popular web development framework called Rails may have seen this little beauty at work and failed to register what was working on your behalf.  Ever notice how ActiveRecord seems to keep track of the names of the database tables that corresponds to your models?  The `ActiveRecord::Base` class takes advantage of this class-level instance variable in the form of a variable called `table_name`.  It can be left to be inferred (convention) or overridden (like many things in Rails) in a manner such as:

```ruby
class SomethingOrAnother < ActiveRecord::Base
  self.table_name = :things
end
```

If we cracked open the `ActiveRecord::Base` class and found a property called `@@table_name`, then -- as we now know -- changing it in one place would change it for every database-backed model in the entire application, and all hell would break loose.  Instead, we'd expect -- and we'd be right -- to find a property called `@table_name`, which would give each subclass its own copy to manipulate and report on without any effect on the others.

### The shorthand (Class-Level Instance Variables 303 or "Code Like a Boss")
It turns out that there's a shorthand way to give ourselves a getter and a setter for class-level instance variables, just as there is for instance variables; in fact it's virtually the same and simply requires opening up the singleton class as follows:

```ruby
class Animal
  class << self
    attr_accessor :locomotion_technique  # attr_reader and addr_writer also work as expected
  end
end
```
Before this conversation, it would be natural to assume that this snippet would actually define `@@locomotion_technique`, but it does in fact do what we would ALMOST ALWAYS prefer that it do by creating `@locomotion_technique` at the class level.  To my knowledge there's no shorthand in Ruby for defining regular class variable getters and setters, and nor should there be.  That is all.