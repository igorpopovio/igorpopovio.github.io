---
title: Pipe Viewer - the Linux flow meter
layout: post
tags : [pipe, viewer, linux]

---

**Pipe viewer** is just **a flow meter for Linux command line programs**.

![digital flow meter](/images/pipe-viewer-digital-flow-meter.jpg)

It allows you to see the progress of a long running command. For example if you extract a big archive using `tar` you are left in the dark regarding how much was already extracted, how much is left and when it will finish. Pipe viewer adds a progress bar much like `wget` when it shows the download progress.

## A simple example

    igor@vm:~/temp$ pv archive.tar.gz | tar -xz
    35MB 0:00:03 [10.3MB/s] [==========>                          ] 32% ETA 0:00:06

## Explanation
**Pipe viewer** works very much like `cat`. Indeed, if you'll replace `pv` with `cat` you'll get exactly the same result, but without the progress indication. Here is the previous example, but this time using `cat`:

    igor@vm:~/temp$ cat archive.tar.gz | tar -xz

Normally, in `*nix` systems all the command line programs are designed to read from standard-in and write to standard-out (`stdin` and `stdout`). This allows for a very cool trick: you can "**pipe**" (using the `|` operator) the output of a program into another program. This allows most of the `*nix` commands to focus on just one thing. This is the case with **Pipe Viewer** too: it just shows progress information, but the actual decompression (from the previous example) is done by a separate command: `tar`.

Perhaps it's easier if you think about **data flow**: you have a long pipe (a real-world pipe), through one end you feed data and through the other end you get the processed data. You can consider each `*nix` program from the (software) pipe as a **filter** you put on the real-world pipe.

The simplest (software) "**pipe**" is `cat` which copies standard-in to standard-out and doesn't do any transformations on the input data.

## Other examples

**Restoring a MySQL database dump**

    pv mysql-dump.sql | mysql -u root -p db-name

**Copy big files locally**

    pv big-file.txt > /path/to/new/location

**Copy big files from local to remote computer**  
This will create the archive first then pipe it to **Pipe Viewer** and send it to the remote computer using `netcat` (`-l number` means listen on port `number` - used on the server to listen for connections, `-q number` - after EOF on stdin, wait the specified number of seconds and then quit. If seconds is negative, wait forever).

    # local (ip: 192.168.33.10)
    tar -cf – /path/to/dir | pv | nc -l 12321 -q 5

    # remote
    nc 192.168.33.10 12321 | pv | tar -xf -

You'll want to use this instead of `scp` (secure copy) when copying files on the internal network and you're concerned only with getting a file as fast as possible on the new computer (using just the network, without external storage).

**Using named pipe viewers**

    igor@vm:~/temp$ pv -N archive sql-dump.tar.gz | tar -xz
      archive: 23.6MB 0:00:02 [15.3MB/s] [====>                     ] 21% ETA 0:00:07

**Using more than one pipe viewer**

    igor@vm:~/temp$ pv -cN reading sql-dump.tar.gz | \
        > tar -xz --to-stdout | \
        > pv -cN writing > sql-dump.sql
      reading: 63.1MB 0:00:05 [10.1MB/s] [=============>            ] 57% ETA 0:00:03
      writing: 69.2MB 0:00:05 [11.8MB/s] [   <=>                                    ]



# How to create big files fast
If you want to follow along with the examples described here you'll need some big files. One way is to find some files you already have or to generate some on the fly. This is how you can generate a file having 10 GB:

    fallocate -l 10G big-file.bin


## Resources

[http://www.catonmat.net/blog/unix-utilities-pipe-viewer/](http://www.catonmat.net/blog/unix-utilities-pipe-viewer/)  
[http://davedash.com/2009/09/16/getting-started-with-pipe-viewer/](http://davedash.com/2009/09/16/getting-started-with-pipe-viewer/)  
[http://securfox.wordpress.com/2009/07/03/pv-pipe-viewer/](http://securfox.wordpress.com/2009/07/03/pv-pipe-viewer/)  
[http://www.ibm.com/developerworks/aix/library/au-spunix_pipeviewer/](http://www.ibm.com/developerworks/aix/library/au-spunix_pipeviewer/)
