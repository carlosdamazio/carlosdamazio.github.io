It's been a while ~~2 months~~ since I made a post about Python.

![Imnoteven](https://media.giphy.com/media/X0oyaVY9dOiBy/giphy.gif)

Now I'll tackle for a brief moment on the topic about dealing with
Decorators on Python of which are on a Linux system environment.

If you thought that I'd stop with the gifs, you got it wrong.
## Decorators? What's the matter of not using them?

_If you throw in a function, I'd be something like this:_

![Imnoteven](https://media.giphy.com/media/P3aaEHlnOuj6M/giphy.gif)

_Maybe._

On a formal manner, a decorator is a pattern used to extend a certain
behaviour on a function without actually changing it explicitelly. In a way, you wrap
the function with another function to get the behaviour you want. It's mainly used
by developers for replicate a certain behaviour to other functions.

_Not good enough. Explain why I need to use it!_

Imagine that you are handling with parallelism and concurrency modules and wanted to time
them to come up with the best approach for the scenario. And I meant to use "behaviour" a lot
because that's what being used and I can't stress it enough.

{% highlight python %}
# testing.py

def func_thread(*args, **kwargs)
    exec_instruction_1()
    exec_instruction_2()
    exec_instruction_3()

    return True

def func_worker(*args, **kwargs)
    exec_instruction_1()
    exec_instruction_2()
    exec_instruction_3()

    return True

def func_pools(*args, **kwargs)
    exec_instruction_1()
    exec_instruction_2()
    exec_instruction_3()

    return True
{% endhighlight %}

You have 3 functions that you want to time them to know which of them are being the
most performatic. You'd come up with an approach to code fairly enough to time them
on each function, right?

{% highlight python %}
# testing.py

import time

def func_thread(*args, **kwargs)
    time_begin = time.time()

    exec_instruction_1()
    exec_instruction_2()
    exec_instruction_3()

    time_end = time.time()

    print("It took {} on func_thread".format(time_end - time_begin))

    return True

def func_worker(*args, **kwargs)
    time_begin = time.time()

    exec_instruction_1()
    exec_instruction_2()
    exec_instruction_3()

    time_end = time.time()

    print("It took {} on func_worker".format(time_end - time_begin))

    return True

def func_pools(*args, **kwargs)
    time_begin = time.time()

    exec_instruction_1()
    exec_instruction_2()
    exec_instruction_3()

    time_end = time.time()

    print("It took {} on func_pools".format(time_end - time_begin))

    return True
{% endhighlight %}

But I can list 3 problems of which you are certainly going to address on
this regard:

- If this behaviour is used for testing only, you are going to take it off anyways. Which is
more lines to delete and being careful to not delete anything more.
- Code repetition, you are writting the same stuff over and over if you
want to apply this behaviour on, maybe, 30 functions.
- If you want to change this behaviour on those 30 functions, for any reason,
you have to look for those 30 functions and change it by hand. _And good luck though._

![Imnoteven](https://media.giphy.com/media/cwTtbmUwzPqx2/giphy.gif)

_This is the face of someone who's going to deal with this code. It could be you or me._

You DON'T need to do these things! Please don't.

## The solution. Decorate (almost) everything!
A programming language may say they have first class functions if they treat them as
"first class citizens", which are objects that can fit in, virtually, anywhere on the
coding process (data structure, variables, parameters, etc...). Lucky you, Python is a
programming language that treats functions as "first class citizens".

{% highlight python %}
# first_classy.py

def hello_there():
    print("hello there.")

if __name__ == "__main__":
    var = hello_there
    print(type(var))
    var()
{% endhighlight %}

Since Python treats functions as first class citizens, you can pass it as parameters.
So let's try to solve the problem that I presented earlier with this pattern, known as
Decorator. I'll show you the ugly way of doing it and then the pretty way.

{% highlight python %}
# ugly_dec.py

import time


def time_decorator(some_func):

    def wrapper(*args, **kwargs):
        t1 = time.time()
        some_func(*args, **kwargs)
        t2 = time.time()
        print("Time it took: {}".format(t2 - t1))

    return wrapper



def func_thread(*args, **kwargs)
    exec_instruction_1()
    exec_instruction_2()
    exec_instruction_3()

    return True

def func_worker(*args, **kwargs)
    exec_instruction_1()
    exec_instruction_2()
    exec_instruction_3()

    return True

def func_pools(*args, **kwargs)
    exec_instruction_1()
    exec_instruction_2()
    exec_instruction_3()

    return True

if __name__ == "__main__":
    func_thread = time_decorator(func_thread)
    func_thread()

    func_worker = time_decorator(func_worker)
    func_worker()

    func_pools = time_decorator(func_thread)
    func_thread()
{% endhighlight %}

![Imnoteven](https://media.giphy.com/media/26xBI73gWquCBBCDe/giphy.gif)

_Is this even a decorator?_

Yes it is. It's in it's primal form.

The decorator function receives an function as a parameter and on the inside,
we have an inner function "wrapper" that receives the parameters of the function to be decorated.
When the behaviour is wrapped and set, the outer function returns the inner function, a new
single function decorated with the behaviour.

There's a little problem with this approach. Still, on the code, we needed to code the decorator on
execution to wrap the original function and call the new function.
To solve this issue, there's a sugar syntax for the decorator, which is the "pie" (@) above the function
to be decorated, so it solves the redoing of redeclaring our function with the decorator:

{% highlight python %}
# beautiful_dec.py

import time


def time_decorator(some_func):

    def wrapper(*args, **kwargs):
        t1 = time.time()
        some_func(*args, **kwargs)
        t2 = time.time()
        print("Time it took: {}".format(t2 - t1))

    return wrapper


@time_decorator
def func_thread(*args, **kwargs)
    exec_instruction_1()
    exec_instruction_2()
    exec_instruction_3()

    return True

@time_decorator
def func_worker(*args, **kwargs)
    exec_instruction_1()
    exec_instruction_2()
    exec_instruction_3()

    return True

@time_decorator
def func_pools(*args, **kwargs)
    exec_instruction_1()
    exec_instruction_2()
    exec_instruction_3()

    return True

if __name__ == "__main__":
    func_thread()
    func_worker()
    func_thread()

{% endhighlight %}

See? You don't need to modify all of those functions to get what you want.
Just set the decorator and you're good to go.

![Imnoteven](https://media.giphy.com/media/ENagATV1Gr9eg/giphy.gif)

On the next post, I'll give something more practical with Python on Linux enviroment!

See ya!