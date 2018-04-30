I was speaking to a friend about Python and we eventually came down
to the topic about building projects. When I asked him if he was going to use
a certain framework in his web development project, he told me he was going
to build it in Django and was going to install it rightaway. I asked him
if he was going to install all the dependencies in a separated environment
or in the core environment (in the S.O). Unfortunately, he told me he wasn't
going to bother dealing with separate environments.

When we think about it, it might be troublesome to deal with a lot of stuff
before we actually can code the project itself. But in here, I'll explain why it
is recommended to deal with it.

![tell me more](https://media.giphy.com/media/eKDp7xvUdbCrC/giphy.gif)

## Problem: Multiple Projects

When you write some code in any programming language, there might be a need (it depends)
that you want a library or two to work within your code. It varies from language to
language. In Python, we have a package manager called PIP, so you can install your libraries
to work in your project.

So far so good. Let's say that in your company, they have a legacy web project written in Python 2.7
and Django 1.11, they want you to work in it. You install the Django version you'll work with.
Now, there's a particular project you want to work with Django 2.0, with Python 3. So you install the
Django 2.0 package.

Notice that the Django 2.0 and Django 1.11 shares some differences, hence the routing module:

{% highlight python %}
'''
Django 2.0 - Uses path and string on routing declaration instead of
regex and url.
'''

from django.contrib import admin
from django.urls import path

urlpatterns = [
    path('admin/', admin.site.urls),
]

{% endhighlight %}

{% highlight python %}
'''
Django 1.11 - Uses url and regex on routing declaration instead of
string and path.
'''

from django.conf.urls import include, url
from django.contrib import admin

urlpatterns = [
    url(r'^admin/', admin.site.urls),
]
{% endhighlight %}

If you're not careful enough of using pip2 and pip3 for Python versions, you just
used pip alone to install Django.


What happened when you installed Django 2.0?
You have overwritten the Django 1.11 library, so you can't execute the legacy project.
This is a problem when you have to work with projects who shares the same library but with
different versions. If you work on multiple projects, expect to have headaches with this.

## Another one: Project Dependencies

So, didn't bother you to be troubled with the first problem?

![worse](https://media.giphy.com/media/khhZNnQb0mamk/giphy.gif)

After reinstalling the Django 1.11
to work on the legacy project, you went to code in it. As you get home, you felt that you wanted
to work on the Django 2.0 project. Ok, just reinstall the Django 2.0 and let it roll. Now, imagine that
you wanted a library to work in your project. Ok, just pip install it, right?
Now, you wanted another one, right? Pip install it. Another one? Install.

The next day, you come to work and wanted to add that dependency on the Django 1.11 that you've
discussed with your colleagues, they agreed and you just pip freeze it to a file called "requirements".
This file contains all the project dependencies from pip that it needs to get it working.
To add a dependency, if you're in Linux, you just go to your terminal and type:

{% highlight bash %}

rm -rf requirements.txt
pip freeze >> requirements.txt

{% endhighlight %}


What happened there? You got your dependencies alright, but with all the other modules from
your Django 2.0 and you've just put it in the legacy project. Now the legacy, that originally
had 99 dependencies, it should have 100, right? Wrong, now it's 200.

In the worst case scenario: You've forgotten to reinstall the Django 1.11 module.
On the production server, you've overwritten the dependency with the Django 2.0, and now your
legacy project is broken.

*PS: In a production server,*
*you should not import modules that it's NOT supposed to work in production.*
*Hence iPython.*

![dead](https://media.giphy.com/media/jSxK33dwEMbkY/giphy.gif)

## Solution: Virtual Environments

After reading this cases, you might come to a conclusion: IT'S A FREAKING MESS.

Programming is not hard because you have to code. It's hard because you keep switching project and
have to get your head around your thoughts to know where you were coding on each one. Mind you that
you need to memorize all the dependencies for each project. Imagine that.

![nope](https://media.giphy.com/media/4SLBtw4UXOAq4/giphy.gif)

No fear my brother or sister.
That's why it's recommended to isolate each project environment from your core
environment with this little guy that we call:

**Virtual Environment**.

In Python 3.3 and onwards,
it comes by default as **venv**.
You should check the proper [documentation](https://docs.python.org/3/library/venv.html) for your
Python version. But it doesn't come by default in versions before 3.3!

Here's a not complete list of Virtual Environment tools that you might use:

- venv;
- virtualenv (with virtualenvwrapper sauce);
- conda.

In my case, I use the **virtualenv with virtualenvwrapper**. It works like a charm. Check out my
workflow with it in Debian 9. I doubt it'll be any different on different Linux distributions:

{% highlight bash %}

# After installing these beasts and setting them up,
# I want to work in a new project. So I have to create
# a new virtual environment.

mkvirtualenv --python=/path/to/python <name of your virtualenv>

# Now I want to use it.

workon <name of your virtualenv name>

# What if I want to get into my project directory? How to do I set it up?

setvirtualenvproject <name of your virtualenv> /path/to/your/project

# How do I go to the project directory without sweat?

cdproject

# I'm tired, I want to take a break.

deactivate

# I've finished this project long ago, I don't want this virtual environment
# anymore!

rmvirtualenv <name of your virtualenv>

# That's it! K TNX BYE
{% endhighlight %}

![standing ovation](https://media.giphy.com/media/xUOwGcs8QdIr6TpNU4/giphy.gif)

That's it folks. Thanks for reading and see ya next time!