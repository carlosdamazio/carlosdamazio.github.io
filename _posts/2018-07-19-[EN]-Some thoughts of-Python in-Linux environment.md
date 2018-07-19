I know what you're thinking. I'm a big evangelist for the Python programming
language and I'll pick every single thing that the language has to offer
and nitpick others for their flaws.

![Imnoteven](https://media.giphy.com/media/xT0xeIlVtsY2kiaMXC/giphy.gif)

Nope, I have a stack of my own on my
toolbox and Python happens to be one of my tools (as Go, JS [and it's ecossystem],
C...). But I don't care, because all that matters the most to me is to talk
about my craft, my tools and help other people. And this is one of those days
to post something that I find useful.

Today is the day to talk about Linux and how you can handle it's
environment into Python. But before jumping directly into it, let's talk
about this Python module called **Subprocess**.

## What is it? How and why is it related to Linux administration anyway?

Subprocess is a Python module which can spawn processes, run system commands
and parse arguments ~~(I answered 3 questions in a row, moving on)~~.

*What if I said that you can automate system tasks and develop software that interacts with
it's environment? For experienced developers it's like "Really? Big deal." but to
novices is like "Geez, programming is really like painting".*

But really, it is a big deal to know that Python could be as powerful as shell scripting
manually. Mind if you try something like this:

{% highlight python %}
# example_sys.py
import subprocess

out = subprocess.run(["ls", "-lha", "nope_nopedity_nope"], capture_output=True)

print(out)
'''
Outputs: CompletedProcess(args=['ls', '-lha', 'nonono'], returncode=2,
         stdout=b'', stderr=b"ls: cannot access 'nonono': No such file or directory\n")
'''

print(out.returncode)
'''
Outputs: 2
'''

print(out.stderr)
'''
Outputs: b"ls: cannot access 'nonono': No such file or directory\n"
'''

subprocess.run(["mkdir", "nope_nopedity_nope"])\
if subprocess.run(["ls", "-lha", "nope_nopedity_nope"]).returncode != 0 else None
{% endhighlight %}

If you're familiar with shell scripting or such, you might know a bit about return codes
and output streams. You're basically automating basic everyday Linux command-line into
your code. You can create conditional statements based on those commands!

![Imnoteven](https://media.giphy.com/media/oYtVHSxngR3lC/giphy.gif)

Basically, even if you didn't know that Python is basically a language with the main
implementation made in C programming language (CPython) which is shipped in most Linux distros,
you'd get a pretty good idea where it's coming from. Now, let me show you a problem.

## Handling directories, files and permissions

I faced a problem few weeks ago regarding handling files within Linux environment, and
most of them are Ansible's configuration files. I wanted to create a simple inventory
based on groups and hosts within a Django module (which I'm doing) to automate Ansible on web environment
without importing wrappers and creating classes.

It's pretty straight forward, right? Just open the file with the statement "with" and
etcetera. But what happens if the file is owned by some other user? What if it's owned by **root**
and the group **root**? When you run a Python script, it's still being ran by your own user which, even though it's in
the **sudoers**, it's still need to be promped a password, and an application which requires to enter
the sudo password every once in a while it's no good.

I looked for solutions, people were storing their sudo passwords in environment variables and calling
with os.environ['SUDO_PASSWORD'] or storing their passwords within the code were totally insecure! We need
a proper implementation which we can find a middle ground of
secure but unpractical for testing to insecure.

Humberto, who's an awesome developer, gave me a light
(you should check his [blog](https://humberto.io/blog/)). And it was simple, since I was just testing my code and it won't be on production anyway:

- Create a new-group where only the owner of the file and the user running the script are the participants;
- Change the group of the directories and files that are being targeted on the script;

This solution should be temporary and only for testing purposes, because extending root priviledges can be
harmful for the system and it's security. But in production, it's adviced to tackle this problem with the
complete solution described by a user in [here](https://askubuntu.com/a/155827), which I've liked it
~~but the os.system()? Not so much~~!

And see you next time, folks!
