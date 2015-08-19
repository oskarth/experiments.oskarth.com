+++
date = "2015-08-15T17:00:00+02:00"
title = "A short overview of the file system"
+++

This is the sixth post in my series on Grokking xv6. In this post we
will give a brief overview of the xv6 file system, with special focus
on the buffer cache layer.

<!--more-->

A file system is used to store and access data. That data is stored on
a *hardware disk*. A hardware disk is divided into 512-byte *blocks*
or *sectors*. The first one is the *boot sector*, which contains the
code necessary to boot up the operating system. After that comes the
*super block*, which contains information about the rest of the
blocks.

```
| boot | super | inodes... | bit map... | data... | log... |
   0       1        2...
```

Here's what superblock looks like:

```
struct superblock {
  uint size;         // Size of file system image (blocks)
  uint nblocks;      // Number of data blocks
  uint ninodes;      // Number of inodes
  uint nlog;         // Number of log blocks
};
```

Thus the specific boundaries for the *inodes*, *bit map*, *data* and
*log* blocks, respectively, can be inferred from looking at the super
block. What purpose these blocks serve will be explained as it is
necessary, but for now it's enough to have a coarse mental model of
how the file system is laid out.

## Tower of abstractions

File systems do a lot of things: they provide persistence, recovery
from crashes, caching for increased performance, and coordination for
concurrent access. Thus they can be quite complex. One good way of
dealing with complexity is through abstraction in multiple layers. We
don't want the the code dealing with file descriptors to deal with
specifics of a certain disk driver, for example. Here's an
illustration of the different layers, from the top layer to the bottom
layer:

```
File descriptor
Pathname
Directory
Inode
Logging
Buffer cache
Disk
```

From the file system's point of view, the data lives on some disk
somewhere. The file system communicates with this disk via a *device
driver*. Since there are many different types of *devices* (disks,
graphic cards, keyboard, monitors, etc), for a real operating system
we need a lot of device drivers. Frequently these device drivers take
up the majority of an operating system's code base, measured in lines
of code.

We will now go through one layer, the *buffer cache*, in more detail,
and then briefly touch on the other layers. The general purpose of
most layers is the same - to abstract away implementation details and
provide an interface for the layers above.

## Buffer cache

One layer up from the disk we have the *buffer cache*, which consists
of a list of recently used *buffers*. A buffer provides an *in-memory*
copy of a specific disk sector. There are two main things you can do
with a buffer: read and write to it. A buffer can be in one of three
states: busy, valid and dirty. If a buffer is dirty it means it has
been changed and needs to be written to disk, if it's valid it has
been read from disk, and if it's busy it means a process is using the
buffer right now. Once you are done with a buffer you have to release
it so other processes can use it.

Often we access the same piece of data multiple times, and reading
from disk every time would be very slow. The buffer cache solves this
problem with an *LRU cache* (least-recently used), implemented as a
*doubly-linked list* in xv6. More efficient caching mechanisms exist
and are frequently used in real operating systems, at the cost of
implementation complexity. Here's what a buffer and the buffer cache
look like in xv6:

```
struct buf {
  int flags;         // B_BUSY, B_VALID, B_DIRTY
  uint dev;          // device number
  uint blockno;      // block number
  struct buf *prev;  // LRU cache list
  struct buf *next;
  struct buf *qnext; // disk queue
  uchar data[BSIZE]; // actual data, 512 bytes
};
#define B_BUSY  0x1  // buffer is locked by some process
#define B_VALID 0x2  // buffer has been read from disk
#define B_DIRTY 0x4  // buffer needs to be written to disk

struct {
  struct spinlock lock; // lock to synchronize access
  struct buf buf[NBUF]; // array of buffers
  struct buf head;      // most recently used buffer
} bcache;
```

This means that `bcache->head` points to the most recently used
buffer, and so on. If we haven't used the data located in a specific
disk sector recently, the corresponding buffer is not in our cache
anymore. We thus have to read the data from disk.

By caching recently used pieces of data, the buffer layer
significantly increases the speed of reading and writing data. It also
allows multiple processes to get the data they want without
accidentally disrupting each other.

## Other layers

The *logging layer* wraps multiple system calls' writes and reads into
one *transaction* and then does a so called *group commit*. This
ensures that the file system is always in a consistent state, so if
our computer crashes in the middle of writing a file it either writes
it or it doesn't - in other words it is *atomic*. It does this by
storing the writes of a transaction in intermediate *log blocks*. Only
when all system calls of a transaction have been logged are the writes
committed to their respective blocks. If the computer crashes, a
recovery function is run that checks if all writes were logged or
not. If they were, all writes are replayed, and if they were not, no
writes are performed.

The *inode layer* gives us files without names, and keeps track of
where the file's content is stored on disk. Any given file can have
multiple names, but it's always one inode underneath. We identify
inodes by their inode number. If all references to an inode are gone,
we delete the inode. Similarly to how buffers correspond to blocks on
disk, inodes have an in-memory representation as well as an on-disk
representation. For performance reasons, we want to deal with the
in-memory version as much as possible.

A file's content is stored in *data blocks*. The *bit map blocks*
tells us which of the data blocks are available to store data in, a
process which is called *block allocation*.

The *directory layer* gives us support for directories. A directory
consists of multiple *directory entries*, each having an inum, which
is its inode number, and a name. When we look up a directory we
iterate over its inode's data, which consists of directory entries
laid out contiguously. Here's a directory entry:

```
struct dirent {
  ushort inum;
  char name[DIRSIZ];
};
```

The *path layer* gives us support for looking up a path like
`/usr/bin/emacs` in the file system. It does this by successively
looking up directory entries. For relative paths, *current working
directory* is a property of the process where the system call is made
from.

The *file descriptor* layer gives us support for treating many
different things as files - standard streams, devices, pipes, and real
files - uniformly. This is the implementation of the interface we used
back in the second installment of this series, *What is a shell and
how does it work?*. There's a global *ftable* which keeps track of all
open files:

```
struct {
  struct spinlock lock;
  struct file file[NFILE];
} ftable;
```

A file here is usually an inode with some additional metadata, such as
whether it is readable or writable, what type it is, how many
references to the file there are, etc.

## Conclusion

We have seen a short overview of the file system, starting with where
things are on disk, and then looking at the layers that make up the
file system.

This is the last explanatory post in this series. Next week we will
look at testing the first of the original hypotheses given in
*Grokking xv6*.

(If you liked this, you might enjoy
[Grok LOC?](http://experiments.oskarth.com/unix06/). To stay up to
date on my experiments, consider [subscribing](http://eepurl.com/bvtdfj).)
