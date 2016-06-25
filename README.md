viasock
=======

Avoid startup overhead by automagically serverizing your scripts.

What it does
------------

Have a Unix command-line program that you call frequently with small input and
output? Does it take a long time to start up, e.g. because it needs to read
large parameter files? Turn it into a server process that keeps running in the
background while handling multiple inputs and outputs.

How it works
------------

For example, instead of this:

    $ cat file1.txt | ./parse -m model > file1.parse
    $ cat file2.txt | ./parse -m model > file2.parse
    $ cat file3.txt | ./parse -m model > file3.parse

Do this:

    $ cat file1.txt | viasock run ./parse -m model > file1.parse
    $ cat file2.txt | viasock run ./parse -m model > file2.parse
    $ cat file3.txt | viasock run ./parse -m model > file3.parse

All it takes is to insert `viasock run` before the command. The first time this
is called, it will spin up the `parse` program as a server in the background.
On every subsequent call, it will just use that server. If the `parse`
program takes a long time to start up, e.g. reading in its model, this now
happens only once, not every time it is used.

Viasock will also

* shut down the server if it is not used for a while (currently 5 seconds),
* start a new server if any of the files the command might need (such as
  `./parse` or `model`) are updated, making sure you use the
  current version.

There are similar projects that use TCP sockets, which may expose your program
to security threats. Viasock uses Unix domain sockets, which are more secure
because they are protected by file system permissions.

Input/output format and limitations
-----------------------------------

Programs that can be called through Viasock must

* accept input from STDIN,
* write output to STDOUT,
* output exactly one output record for each input record, and output it before
  it blocks waiting for the next input record.

A record is a single line by default, but this can be configured with the `-t`
and `-T` options.

The output may contain a number of extra records at the beginning which don’t
correspond to any input records and should be repeated at the beginning of the
output for every client. Specify the number of such “prelude” records using the
`-P` option.

Run `viasock run -h` for help and details.

Getting involved
----------------

* Check out the list of issues or submit your own at
  https://github.com/texttheater/viasock/issues
