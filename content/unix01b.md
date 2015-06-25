What is the shell and how does it work?

A shell is just another program. It is the main user interface to operating
systems that are similar to Unix. An operating system is responsible for having
several programs run on a single computer, and it also tries to abstract away
the specific hardware the computer is running on, so the same program can run on
many different types of computers. Unix is a special type of operating system
that was developed at Bell Labs.

The shell lives in user space, along with most other programs, as opposed to in
kernel space. It talks to the kernel by using system calls. A system call is
something you can tell the kernel to do, such as open a file or create a
process.

What happens when you type the following in the shell and press enter?

> ls

After the shell has parsed it into its internal representation, it knows ls is a
command. It then goes looking for the command in the ifle system, starting with
the present working directory, and then going through all the folders in the
user's PATH.

<not explained what cat does> But the shell is much more powerful than that.

What else can we do? Let's say we want to save this results to a file called
"foo". We can do so by typing "ls > foo" and pressing enter. This is called

> ls > foo

- redirection example

What about:

> cat /usr/share/dict/words | grep xylo

This goes through each line in the words dictionary 

What is the canonical example for piping? And how exactly is it different from
other thing?
