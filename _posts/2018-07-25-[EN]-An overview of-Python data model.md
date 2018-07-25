What does "pythonic" mean? Many of things that I've heard when something
is being called this way is when it's simple and straight-forward, even when
it's so simple that it doesn't seem right. But in reality, what conditions
should one achieve to being considered such thing?

Reading some answers, I came across some thoughts of differences that we may
do in Python that is syntatically impossible in some languages. In this C
code, we are summing up array values and then print them up on screen along with their lenghts:

{% highlight c %}
#include <stdio.h>
#include <stdlib.h>


double array_sum(double array[], int n_elems);

double array_sum (double array[], int n_elems) {
    double sum = 0;

    for (int i = 0; i < n_elems; i++) {
        sum += array[i];
    }

    return sum;
}

int main(void) {
    double array_elem [] = {10.1, 20.2, 30.3, 40.4};
    double sum = array_sum(array_elem, sizeof(array_elem)/sizeof(double));

    printf("%d %.2f\n", sizeof(array_elem)/sizeof(double), sum);
    return 0;
}
{% endhighlight %}

The for syntax is where i'd like to point out when we're running through
the array. Now take a look at the same thing Python does:

{% highlight python %}
#test.py

array_elem = [10.1, 20.2, 30.3, 40.4]
size = 0

# First try. Not pythonic enough.

for i in range(0, 3):
    summed += array_elem[i]

# Second try. Still not pythonic enough.
import operator
from functools import reduce

summed = reduce(operator.add, array_elem)

# Let's do the most pythonic way as possible.

sum(array_elem)
len(array_elem)

{% endhighlight %}

You see? You might realize there's something wrong, doesn't it? If you think about Java,
we have <iterable>.length(), but in this case, we would have length(<iterable>).

This is what people see as a Pythonic code, which doesn't follow standard syntax rules but
more acceptable for the community as a whole. You can do it with pratically everything.
The first condition for making such types or objects to behave with the most idiomatic feature
is to get them programatically to behave with it. And the way to do it is looking at
the **Python Data Model**.

## Python data model

>"(...)if you learned another object oriented language before Python, you may have
>found it strange to spell len(collection) instead of collection.len() . This apparent
>oddity is the tip of an iceberg which, when properly understood, is the key to everything
>we call Pythonic."
>
>Luciano Ramalho - "Fluent Python" (2015)

As quoted above, when we call len(something) instead of
something.length() is pratically the secret of what we might call "pythonic". Objects that
can behave accordingly with the true behaviour of Python as types can get to. You probably realized it
when we create classes, we often use a "constructor" (AKA __init__), you knew that if you initialize
a class with parameters as set into this method, you'd create a class with those parameters.

These "dunder methods" implements behaviours, behaviours that can interact with the core language itself.
 __init__, when called upon a class instantiation, customizes
an instance __with__ the set of parameters (see __new__ method for creating a cls instance).
Dunder methods are the core of Python Data Model, which we can see it as an API for objects to
interact with the most Python resources without reinventing the wheel.

If you ever read my other posts, you know that I love to give examples, because keeping theoretical is
boring! :/

Imagine an library. A really good library (OF BOOKS).

{% highlight python %}
# library.py

import collections


Book = collections.namedtuple('Book', ['title', 'author', 'price'])


class Library(object):

    def __init__(self, books=[]):
        self.books = books

    def __len__(self):
        return len(self.books)

    def __getitem__(self, pos):
        return self.books[pos]

if __name__ == '__main__':
    library = Library([
        Book("Sherlock Holmes complete series", "DOYLE", "R$200,00"),
        Book("The Crow", "POE", "R$170,00"),
        Book("Testing with Unittest.", "Documentation", "R$0,00")
    ])
{% endhighlight %}

Check how we can use our basic and most idiomatic form of Python to get some bits of
information without much effort:

{% highlight python %}
# We can iterate over it.

for book in library:
    print(book)

# We can check how many books are in it.

print(len(library))

# And pick a book in it.

print(library[1])
{% endhighlight %}

- When we implemented __len__ method, we choose what attribute we could measure
its size on the abstraction of a Library. In this case, we chose an attribute as an array of books;
- With the __getitem__, we can make this object iterable choosing what attribute we could
iterate over. Same as above though;
- And finally, you might be familiar with the __init__, we implemented our customization of the object
just after it's instantiation, given the option for pass an argument with a list of books, otherwise, it'll be empty as said on the kwargs books.

It's a simple example of how we can make an object being as simple to handle for developers without
creating things such as an method to check it's length, because we've abstracted it before.
We don't need to remember how we can iterate over an object's attribute, because we've abstracted it before.
At last, we don't need to remember which attribute we can extract an item, because we've abstracted it before.

I'll give it a try on this next example. Though it's not going to be perfect, I'd like to show some more
of the data model examples. We'll create an abstraction of an Latitude reading:

{% highlight python %}
# latitude.py

class Latitude(object):

    def __init__(self, degrees, minutes, seconds, hemisphere):
        if not self._check_attrs_instance_(degrees, minutes, seconds, hemisphere):
            raise Exception("Not a valid latitude assignment.")

        self.degrees = degrees
        self.minutes = minutes
        self.seconds = seconds
        self.hemisphere = hemisphere

    def __repr__(self):
        return 'Latitude {}º {}\' {}" {}'.format(
            self.degrees,
            self.minutes,
            self.seconds,
            self.hemisphere
        )

    def __add__(self, other):
        if self.hemisphere.lower() != other.hemisphere.lower():
            raise Exception("Cannot add latitudes with different hemispheres.")

        deg = self.degrees + other.degrees
        minu = self.minutes + other.minutes
        sec = self.seconds + other.seconds

        if sec >= 60:
            sec = sec - 60
            minu += 1

        if minu >= 60:
            minu = minu - 60
            deg += 1

        if deg >= 90 and minu != 0 and sec != 0:
            if deg == 90:
                deg = 90 - (deg - 90) - 1
            else:
                deg = 90 - (deg - 90)

            minu = 60 - minu
            sec = 60 - sec

            hemisphere = 'S' if self.hemisphere.lower() == 'n' and \
                other.hemisphere.lower() == 'n' else hemisphere = 'N'

        return Latitude(deg, minu, sec, hemisphere)

    @staticmethod
    def _check_attrs_instance_(degrees, minutes, seconds, hemisphere):
        if degrees == 90 and minutes != 0 and seconds != 0:
            return False

        if (0 > degrees > 90) or (0 > minutes > 59) or (0 > seconds > 59) \
        or (hemisphere.lower() != 'n' and hemisphere.lower() != 's'):
            return False

        return True


if __name__ == '__main__':
    lat1 = Latitude(30, 20, 20, 'S')
{% endhighlight %}

{% highlight python %}
print(lat1)

'''
Output: Latitude 30º 20' 20" S
'''

lat3 = lat1 + Latitude(60, 0, 0, 'S')
print(lat3)

'''
Output: Latitude 89º 40' 40" N
'''

lat3 = lat3 + Latitude(30, 0, 1, 'S')

'''
Output:

Original exception was:
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "latitude.py", line 22, in __add__
    raise Exception("Cannot add latitudes with different hemispheres.")
Exception: Cannot add latitudes with different hemispheres.
'''
{% endhighlight %}

Did you notice something about this object behaviour? Maybe it can be summed?
Or we have a printable representation to it?

![](https://media.giphy.com/media/LxPsfUhFxwRRC/giphy.gif)

- With the implementation of __add__, we have added an representation of a
numerical operation such as adding, same thing we do with integers, floating, doubles
and other common types that supports arithmetical operations. Note that we can customize
this representation with exceptions and return a brand new object affected by the operand!
- __repr__ gives a printable representation to our object when it's called on our code!
Instead of having a printable representation of <___main___.Latitude object at [address]>,
we could get the object, when printed, some representation that does make sense!

## Interesting resources

- For a more in-depth look over Python Data-Model, and even Meta-classes, decorators, context managers,
etcetera, you might want to watch this talk "So you want to be a Python expert?" by James Powell:
[![VIDEO](http://img.youtube.com/vi/cKPlPJyQrt4/0.jpg)](https://www.youtube.com/watch?v=cKPlPJyQrt4)
- Most of my examples were influenced by the book "Fluent Python" by Luciano Ramalho. If you're doing
okay with Python and want to improve it, you should definitely check it out [here](https://www.amazon.com/_/dp/1491946008?tag=oreilly20-20);
- These methods for data model "API" shown here are just a few of a whole lot
explained in the [documentation](https://docs.python.org/3/reference/datamodel.html).

Next time I'll post something different, related with Angular and Typescript. See ya!
