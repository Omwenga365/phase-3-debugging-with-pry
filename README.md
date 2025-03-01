# Debugging with Pry

## Learning Goals

- Explain how Pry is a more flexible REPL than IRB.
- Install Pry on your computer
- Debug a program using `binding.pry` within the body of your file

## Introduction

We'll cover Pry, a type of REPL, and discuss how to install and use it to debug
a program.

## What Is a REPL?

You've already been introduced to REPLs through using IRB (Interactive Ruby).
REPL stands for _Read, Evaluate, Print, Loop_. It is an interactive programming
environment that takes a user's input, evaluates it and returns the result to
the user.

Ruby installs with its own REPL, which is IRB, that you've already been using.
Every time you type `irb` into your terminal, you're entering into a REPL.

## What Is Pry?

Pry is another Ruby REPL with some added functionality. When you enter IRB, you
are entering a brand new interactive environment. Any code you want to play with
in IRB, you have to write in IRB or copy and paste into IRB. Pry, on the other
hand, is like a REPL that you can inject into your program.

Pry is far more flexible than IRB. Once you install the Pry library (via the Pry
gem—we'll walk through installation in a bit), you can use a `binding.pry`
anywhere in your code.

## Wait... What's 'binding'?

Binding is a built-in ruby class whose objects can encapsulate the context of
your current scope (variables, methods etc.), and retain them for use outside of
that context.

Calling `binding.pry` is essentially 'prying' into the current binding or
context of the code, from outside your file.

`binding.pry` gives you similar functionality to using `debugger` in a
JavaScript application, in that it lets you set a **breakpoint** in your code
that will pause the execution of your program at a certain point so you can
inspect the variables, methods, and other context available at a specific place
in your code.

So when you place the line `binding.pry` in your code, that line will get
interpreted at runtime (as your program is executed). When the interpreter hits
that line, your program will actually _freeze_ and your terminal will turn into
a REPL that exists right in the middle of your program, wherever you added the
`binding.pry` line.

Let's take a look. In this repository, you'll see a file called
`pry_is_awesome.rb`.

## Using `binding.pry`

First, let's make sure Pry is installed on your system:

```console
$ gem install pry
```

This will install the Pry gem system-wide, so you only need to run this command
once.

Look at the code in `lib/pry_is_awesome.rb`. You should see the following code:

```rb
require 'pry'

def prying_into_the_method
  inside_the_method = "We're inside the method"
  puts inside_the_method
  puts "We're about to stop because of pry!"
  binding.pry
  this_variable_hasnt_been_interpreted_yet = "The program froze before it could read me!"
  puts this_variable_hasnt_been_interpreted_yet
end

prying_into_the_method
```

At the top of the file, `require 'pry'` is a method that loads the `pry` library
in this file when our application runs, similar to using the `import` keyword in
JavaScript.

After requiring `pry`, _which you must do to use pry_, we define a method, and
then call that method.

In the directory of this repo, in your terminal, run the file by typing
`ruby lib/pry_is_awesome.rb`. Now, look at your terminal. You should see
something like this: lib/pry_is_awesome

```txt
  3: def prying_into_the_method
     4:     inside_the_method = "We're inside the method"
     5:     puts inside_the_method
     6:     puts "We're about to stop because of pry!"
 =>  7:     binding.pry
     8:     this_variable_hasnt_been_interpreted_yet = "The program froze before it could read me!"
     9:     puts this_variable_hasnt_been_interpreted_yet
    10: end
[1] pry(main)>
```

You have frozen your program _as it executes_ and are now inside a REPL _inside
your program_. You basically just stopped time! How cool is that?

> **NOTE**: Depending on the size of your terminal window, Pry might not be able
> to fit your entire method on the screen, in which case you'll see a screen
> like this:
>
> ```txt
>   3: def prying_into_the_method
>      4:     inside_the_method = "We're inside the method"
>      5:     puts inside_the_method
>      6:     puts "We're about to stop because of pry!"
>      7:     binding.pry
> :
> ```
>
> If you ever get into this situation where you see a `:` as the terminal prompt
> instead of `pry(main)>`, just type the letter `q` in the terminal to exit out.

In the terminal, in your pry console, type the variable name `inside_the_method`
and hit enter. You should see a return value of `"We're inside the method"`

You are able to explore the data _inside_ the method in which you've placed your
binding. Any variables or methods that are available from the line you placed
your `binding.pry` will be available to you to explore from the terminal.

Now, in the terminal, in your pry console, type the variable name
`this_variable_hasnt_been_interpreted_yet`. You should see a return value of
`nil`. That's because the binding you placed on line 7 actually froze the
program on line 7 and the variable you just called hasn't been interpreted yet.
Consequently, our REPL doesn't know about it.

Now, in the terminal, type `exit`, and you'll leave your pry console and the
program will continue to execute.

## Using Pry to Debug

In addition to _exploring_ code inside Pry, you can also manipulate variables
and try code out. This is where Pry really becomes helpful for debugging. If you
have a method that isn't doing what it's supposed to do, instead of making
changes in your text editor and running the tests over and over until you get it
working, you can put a binding in your code and try things out. Once you've
figured out how to fix the problem, you then update the code in your text editor
accordingly.

Let's walk through an example together. In this repository, you'll see a `spec`
folder containing a file `pry_debugging_spec.rb`. This is a test for the file
`lib/pry_debugging.rb`.

In `pry_debugging.rb`, we have a broken method. Run `learn test` to see the
failing test. You should see the following:

```txt
  1) #plus_two takes in a number as an argument and returns the sum of that number and 2
     Failure/Error: expect(plus_two(3)).to eq(5)

       expected: 5
            got: 3

       (compared using ==)
     # ./spec/pry_debugging_spec.rb:6:in `block (2 levels) in <top (required)>'
```

So what's happening? In the second line (the line starting with
`Failure/Error`), we can see that the test is calling the `plus_two` method and
passing in `3` as an argument. Below that we can see that the test is expecting
`5` to be returned, but that `3` is being returned instead. Remember that the
return value of a method in Ruby is generally the value of the last line of the
method, in this case, `num`:

```ruby
def plus_two(num)
    num + 2
    num
end
```

So while our method is adding 2 to `num` on the second line, it appears that it
is not _updating_ `num`. We have Pry required at the top of our
`spec/pry_debugging_spec.rb` file so we can use it to verify this. Let's place a
`binding.pry` in our code, right after that line:

```ruby
def plus_two(num)
    num + 2
    binding.pry
    num
end
```

Now, run the test suite again and drop into your Pry console. Your terminal
should look like this:

```txt
    1: def plus_two(num)
    2:  num + 2
 => 3:  binding.pry
    4:  num
    5: end

[1] pry(#<RSpec::ExampleGroups::PlusTwo>)>
```

Let's check our current return value by typing `num` at the Pry prompt. You
should see something like this:

```txt
[1] pry(#<RSpec::ExampleGroups::PlusTwo>)> num
=> 3
[2] pry(#<RSpec::ExampleGroups::PlusTwo>)>
```

By checking the value of the variable inside our pry console, we can confirm
that `num` is still equal to `3` and, as a result, the method is returning `3`.

How can we modify the code on line 2 so that the method behaves in the expected
way? We need to _update_ the value of our `num` variable so that it's equal to
the sum of itself and 2. Play around inside your Pry console: try code that you
think will update `num` as needed, then check the value of `num` to see if it
worked. Once you figure it out you can type `exit` in your terminal to get out
of Pry, update the code in your text editor, and rerun the test to verify it's
passing. Be sure to remove the `binding.pry`!

It can take a little while to get the hang of using Pry, so don't worry if it's
still a little confusing. As you start working with more complex methods and
data structures, you'll find it can be a very helpful tool.

## Resources

- [Pry documentation](http://pry.github.io/)
- [Pry wiki](https://github.com/pry/pry/wiki)
- [Debugging with Pry: A Beginner's Guide](https://dev.to/elimerrell/debugging-with-pry-a-beginners-guide-3p99)
