+++
date = "2015-09-07T15:00:00+02:00"
title = "Netpowder, a mini-server in the browser" 
+++

The last few weeks I have been working on a little project called Netpowder, a
mini-server in the browser. In this post I will give some technical background
and give a short demonstration of what it is.

<!--more-->

## Introduction

If you want to see the demonstration first, you'll find it at the end of this
post.

When you create a side-project, why do you have to bother with servers? There's
no sensitive information, it's probably not going to get a lot of traffic, and
you just want to show people something quickly.

There are solutions out there, no doubt: containers, Heroku, Wordpress
hackery and everything under the sun. Yet I have never found myself using them,
for whatever reason. To me that's a sign that this is something people want, but
no one has quite hit on the right combination yet.

There are at least two key principles behind Netpowder, at this stage: (a) you
should only have to know the minimum to get the job done and (b) use the already
excellent shell to enable more complex tasks.

Netpowder is far from ready, but that's generally the right time to tell people
about it.

## What can you do?

In the browser you are presented with three views: an editor, a command prompt,
and a log. You can enter simple commands into the command prompt such as `open
<file>`, `save` and `serve <foo>`. The inspiration for these commands are the
IRC protocol. Eventually they might instead corresponds to other UI elements,
such as buttons.

There's also support for more complex shell commands by writing `shell <foo>`
which runs foo in a shell and returns the standard out to the log window. For
example, you can find your `nginx.conf` file by typing `shell locate
nginx.conf`.

As you type these commands, you get some output in the log view, such as whether
a file was opened, or the standard output for some shell command, etc. When you
edit text you do it in the editor, which is a standard Codemirror instance.

You can do pretty much anything you could do inside a standard unix-based
server. There are limitations for security reasons, but almost all of these are
network-based, thanks to the FreeBSD jails system.

## Technical overview

### Infrastructure

It starts with a standard FreeBSD server, where several FreeBSD jails are
running. A FreeBSD jail does not have direct access to the Internet, and is a
form of operating system virtualization. Each jail thinks it's its own instance
of a FreeBSD server, with root and everything, but it has access only to the
private network and can't see anything else in the other jails. This provides
isolation between each jail.

There are a lot of security details to this, and I'm sure my current
configuration is insecure to some attack. Security is tricky, and only time will
tell whether this is insurmountable or not. Right now I'm carefully optimistic,
given the effort that has gone into making FreeBSD jails secure. (The thing I am
currently most unsure about is the packet forwarding, using pf).

There is a reverse proxy running in one of the jails, which forwards HTTP and
Websocket (TLS coming soon, right now self-signed certificates are available)
connections to the various mini-servers.

### Handler and client

Each mini-server has a websocket connection to the browser. It uses websocketd,
which reads from standard in and prints to standard out. Attached to this is a
handler process written in Racket. This handler takes the IRC-like commands,
parses them, and executes the necessary shell utils or system calls to get the
right thing to show up on the client. For example, a `save` operation works as a
form of transaction, where we first send a `beginop save <filename>` message,
and then stream the file, line by line, until we get to the end of the file in
the browser editor, when we send a `endop` message. Likewise, the `serve`
command involves a whole lot of hackery with programmatically updating the
`nginx.conf` file using awk, and then reloading it.

## TODOs

There are a lot of interesting things to be done at each level:

1. The back-end infrastructure for automated creation of jails, taking
snapshots, making sure it's secure and people can't flood the network, etc.

2. The handler with defining a usable little language for accomplishing common
tasks and wrapping system-calls in a sane manner.

3. The client in terms of UX discoverability, getting more visual feedback on
actions taken, etc.

In short: there's a lot of work to do.

## Demonstration

Here's a short video demonstration of I/O and hosting a publicly viewable site.

<iframe width="560" height="315" src="https://www.youtube.com/embed/9NqEpsFeKDg"
frameborder="0" allowfullscreen></iframe>

(If you liked this, you might enjoy [Grok
LOC?](http://experiments.oskarth.com/unix06/).  To stay up to date on my
experiments, consider [subscribing](http://eepurl.com/bvtdfj).)
