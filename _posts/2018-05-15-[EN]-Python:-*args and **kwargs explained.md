I ~~gathered some courage~~ decided to finally make a post with something more
specific and technical about programming in general.
Today's post is about Python (what a surprise).
I know this is getting a bit repetitive, but calm your butts, someday I might
post something about Golang, JavaScript, C++ or C. ~~Maybe C#~~.

~~Today I'm going to ease on the GIF's usage aswell to maintain our focus on what
matters~~ (Beautiful lie).

![Imnoteven](https://media.giphy.com/media/4WkCTDMpgjsUU/giphy.gif)

Let's get started on what every newcomer might be confused on an
ongoing Python project and when they see this kind of thing, he/she might throw
himself out of the 9th floor on frustration of not understanding what it might
be. Yes, I'm talking about **args* and kwargs****.

## What the hell is this anyway?

You need to focus on the " * " of the args* and kwargs**, because the names are
just a convention for naming Arguments and Keyworded-Arguments of a function.

{% highlight python %}
'''
A reminder for Arguments and Keyworded-Arguments in a function.

First parameter on __init__ function declaration is an Argument and
Keyworded-Argument is the second one.
'''

class Person(object):
    def __init__(self, name, last_name=''):
        self.name = name
        self.last_name = last_name

{% endhighlight %}

On Python, * and ** are called packing and unpacking operators for arbitrary
argument lists and dictionaries.

## Packing arguments and keyworded arguments

When you do arguments and keyworded arguments packing,
it means that you want to pass a good amount of arguments and "optional" arguments all at once.
How does it work?

Imagine that you have a function with 3 parameters, or maybe, a function with
unknown number of parameters and I want to make it to print a phrase made based on
the arguments:

{% highlight python %}
def phrase_args(a, b, c):
    print(a + ' ' + b + ' ' + c)
{% endhighlight %}

Look at that function. Even if I know the number of parameters that I want to
pass, what if the requirements change? What if I pass only 2 arguments?

{% highlight python %}
phrase_args('Hello', 'cruel', 'world')
# It works!
phrase_args('Hello', 'world')
# TypeError. It breaks!
{% endhighlight %}

Because the function expected 3 arguments and you gave only 2, it returned a TypeError.
To make it work, you might have to change the whole function to work within
the new requirements (to work with 2 strings instead of 3).
And it is the same story for keyworded arguments. BUT NO, we don't want that!

![Imnoteven](https://media.giphy.com/media/3ohs7N0iXvc180MiB2/giphy.gif)

You see, this function is static. With the use of the packing operator on the function
definition, we change the function from static to actually being dynamic.

Note that our function is working towards iteration within arguments. Instead of
declaring each parameter in a function definition, we can simply pass a tuple of a * and
declare it as parameter *args. You might understand it more clearly when we talk
about Unpacking.

{% highlight python %}
def phrase_args(*args):
    print(' '.join([arg for arg in args]))

phrase_args('Hello', 'cruel', 'world')
# It works!
phrase_args('Hello', 'world')
# It works aswell!
{% endhighlight %}

And it's the same principle for keyworded arguments, but it works with a dictionary
of ** and we conventionally declare it as a **kwargs:

{% highlight python %}
# Breakable function.
def phrase_kwargs(a="Hello", b="world"):
    print(a + ' ' + b)

phrase_kwargs() # It works.
phrase_kwargs(a="Hello", b="world!") # Works aswell.
phrase_kwargs(a="Hello", b="cruel", c="world.") # TypeError

{% endhighlight %}

Now for the packing feature:

{% highlight python %}
# Functioning function (no pun intended).
def phrase_kwargs(**kwargs):
    print(' '.join([value for value in kwargs.values()]))

phrase_kwargs() # It works.
phrase_kwargs(a="Hello", b="world!") # Works aswell.
phrase_kwargs(a="Hello", b="cruel", c="world.") # Perfect!

{% endhighlight %}

## Unpacking arguments and keyworded arguments

Well, so far so good. Now let's make it interesting, you might
understand this topic more clearly.

Imagine that we have a list of strings to make a phrase.
But we can't simply pass this list to the function. We need to pass it as normal arguments
(because it is expected to get normal string arguments), but I feel that we don't
want to do it because it's boring and dreadful. **Note that we're not allowed to change
the function anymore.**

Examples?

{% highlight python %}
string_list = ['I', 'am', 'in', 'a', 'hurry!']

phrase_args([string_list]) # Nope.
phrase_args(['I', 'am', 'in', 'a', 'hurry!']) # Not on my watch.
phrase_args('I', 'am', 'in', 'a', 'hurry!') # It's simply what we're trying to avoid!
{% endhighlight %}

The unpacking operation is quite magical. Because when we put a list on unpacking operator,
it interprets as each element of a list as an individual argument, so instead of putting
all arguments into the function call, we just put our list with a unpacking operator, which
is the same for packing. See the example when we call our string_list as a *string_list into the function:

{% highlight python %}
string_list = ['I', 'am', 'in', 'a', 'hurry!']

phrase_args(*string_list) # It works!
{% endhighlight %}

And the unpacking operator for keyworded argument follows the same logic, but for dictionaries:

{% highlight python %}
string_dict = {"greetings": "Good morning!", "name": "Carlos!"}

phrase_kwargs(string_dict) # Nope.
phrase_kwargs({"greetings": "Good morning!", "name": "Carlos!"}) # Geez.
phrase_kwargs(greetings="Good morning!", name="Carlos") # We want to avoid it!
phrase_kwargs(**string_dict) # It works!
{% endhighlight %}

The main difference of packing operator and unpacking operator is when it's used.

* If it's used in a function call, it's an unpacking operator.
* If it's used on the function definition, it's a packing operator.


That's it folks! See ya next time!