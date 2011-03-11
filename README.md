Resty
=====

Resty is a tiny script wrapper for [curl](http://curl.haxx.se/). It
provides a simple, concise shell interface for interacting with
[REST](http://en.wikipedia.org/wiki/Representational_State_Transfer) services.
Since it is implemented as functions in your shell and not in its own separate
command environment you have access to all the powerful shell tools, such
as perl, awk, grep, sed, etc. You can use resty in pipelines to process data
from REST services, and PUT or POST the data right back.  You can even pipe
the data in and then edit it interactively in your text editor prior to PUT 
or POST.

Cookies are supported automatically and stored in a file locally. Most of
the arguments are remembered from one call to the next to save typing. It
has pretty good defaults for most purposes. Additionally, resty allows you
to easily provide your own options to be passed directly to curl, so even
the most complex requests can be accomplished with the minimum amount of
command line pain.

[Here is a nice screencast showing resty in action](http://blog.fupps.com/2010/04/26/resty/) (by Jan-Piet Mens).

Quick Start
===========

You have `curl`, right? Okay. 

      curl -L http://github.com/micha/resty/raw/master/resty > resty

Source the script before using it. (You can put this line in your `~/.bashrc`
file if you want, or just paste the contents of the `resty` script right in
there. Either way works.)

      . resty

Set the REST host to which you will be making your requests (you can do this
whenever you want to change hosts, anytime).

      resty http://127.0.0.1:8080/data

Make some HTTP requests.

      GET /blogs.json
      PUT /blogs/2.json '{"title" : "updated post", "body" : "This is the new."}'
      DELETE /blogs/2
      POST /blogs.json '{"title" : "new post", "body" : "This is the new new."}'

Usage
=====

      source resty [-W] [remote]              # load functions into shell
      resty                                   # prints current request URI base
      resty <remote>                          # sets the base request URI

      HEAD [path] [curl opts]                 # HEAD request
      OPTIONS [path] [curl opts]              # OPTIONS request
      GET [path] [-Z] [curl opts]             # GET request 
      DELETE [path] [-Z] [curl opts]          # DELETE request 
      PUT [path] [data|-V] [-Z] [curl opts]   # PUT request
      POST [path] [data|-V] [-Z] [curl opts]  # POST request
      TRACE [path] [-Z] [curl opts]           # TRACE request

      Options:

      -W            Don't write to history file (only when sourcing script).
      -V            Edit the input data interactively in 'vi'. (PUT and POST
                    requests only, with data piped to stdin.)
      -Z            Raw output. This disables any processing of HTML in the
                    response.

Request URI Base
================

The request URI base is what the eventual URI to which the requests will be
made is based on. Specifically, it is a URI that may contain the `*` character
one or more times. The `*` will be replaced with the `path` parameter in the
`OPTIONS`, `HEAD`, `GET`, `POST`, `PUT`, or `DELETE` request as described above.

For example:

      resty 'http://127.0.0.1:8080/data*.json'

and then

      GET /5

would result in a `GET` request to the URI `http://127.0.0.1:8080/data/5.json`.

If no `*` character is specified when setting the base URI, it's just added
onto the end for you automatically.

HTTPS URIs
----------

HTTPS URIs can be used, as well. For example:

      resty 'https://example.com/doit'

URI Base History
----------------

The URI base is saved to an rc file (_~/.resty/host_) each time it's set,
and the last setting is saved in an environment variable (`$_resty_host`).
The URI base is read from the rc file when resty starts up, but only if the
`$_resty_host` environment variable is not set.  In this way you can make
requests to different hosts using resty from separate terminals, and have
a different URI base for each terminal.

If you want to see what the current URI base is, just run `resty` with no
arguments. The URI base will be printed to stdout.

The Optional Path Parameter
===========================

The HTTP verbs (`OPTIONS`, `HEAD`, `GET`, `POST`, `PUT`, and `DELETE`) first 
argument is always
an optional URI path. This path must always start with a `/` character. If
the path parameter is not provided on the command line, resty will just use
the last path it was provided with. This "last path" is stored in an
environment variable (`$_resty_path`), so each terminal basically has its
own "last path".


URL Encoding Of Path Parameter
------------------------------

Resty will always [URL encode]
(http://www.blooberry.com/indexdot/html/topics/urlencoding.htm) the path,
except for slashes. (Slashes in path elements need to be manually encoded as
`%2F`.) This means that the `?`, `=`, and `&` characters will be encoded, as
well as some other problematic characters. See the query string howto below
for the way to send query parameters in GET requests.

POST/PUT Requests and Data
==========================

Normally you would probably want to provide the request body data right on
the command line like this:

      PUT /blogs/5.json '{"title" : "hello", "body" : "this is it"}'

But sometimes you will want to send the request body from a file instead. To
do that you pipe in the contents of the file:

      PUT /blogs/5.json < /tmp/t

Or you can pipe the data from another program, like this:

      myprog | PUT /blogs/5.json

Or, interestingly, as a filter pipeline with 
[jsawk](http://github.com/micha/jsawk):

      GET /blogs/5.json | jsawk 'this.author="Bob Smith";this.tags.push("news")' | PUT

Notice how the `path` argument is omitted from the `PUT` command.

Edit PUT/POST Data In Vi
------------------------

With the `-V` options you can pipe data into `PUT` or `POST`, edit it in vi,
save the data (using `:wq` in vi, as normal) and the resulting data is then
PUT or POSTed. This is similar to the way `visudo` works, for example.

      GET /blogs/2 | PUT -V

This fetches the data and lets you edit it, and then does a PUT on the
resource. If you don't like vi you can specify your preferred editor by
setting the `EDITOR` environment variable.

Errors and Output
=================

For successful 2xx responses, the response body is printed on stdout. You
can pipe the output to stuff, process it, and then pipe it back to resty,
if you want.

For responses other than 2xx the response body is dumped to stderr.

In either case, if the content type of the response is `text/html`, then
resty will try to process the response through either `lynx`, `html2text`,
or, finally, `cat`, depending on which of those programs are available on
your system.

Raw Output (-Z option)
----------------------

If you don't want resty to process the output through lynx or html2text you
can use the `-Z` option, and get the raw output.

Passing Command Line Options To Curl
====================================

Anything after the (optional) `path` and `data` arguments is passed on to 
`curl`.

For example:

      GET /blogs.json -H "Range: items=1-10"

The `-H "Range: items=1-10"` argument will be passed to `curl` for you. This
makes it possible to do some more complex operations when necessary.

      POST -v -u user:test

In this example the `path` and `data` arguments were left off, but `-v` and
`-u user:test` will be passed through to `curl`, as you would expect.

Here are some useful options to try:

  * **-v** verbose output, shows HTTP headers and status on stderr
  * **-j** junk session cookies (refresh cookie-based session)
  * **-u \<username:password\>** HTTP basic authentication
  * **-H \<header\>** add request header (this option can be added more than 
    once)
  * **-d/-G** send query string parameters with a GET request (see below)

Query Strings For GET Requests
------------------------------

Since the path parameter is URL encoded, the best way to send query parameters
in GET requests is by using curl's commnand line arguments. For example,
to make a GET request to `/Something?foo=bar&baz=baf` you would do:

      GET /Something -d foo=bar -d baz=baf -G

This sends the name/value pairs specified with the `-d` options as a query
string in the URL.

Per-Host/Per-Method Curl Configuration Files
--------------------------------------------

Resty supports a per-host/per-method configuration file to help you with
frequently used curl options. Each host (including the port) can have its
own configuration file in the _~/.resty_ directory. The file format is

      GET [arg] [arg] ...
      PUT [arg] [arg] ...
      POST [arg] [arg] ...
      DELETE [arg] [arg] ...

Where the `arg`s are curl command line arguments. Each line can specify
arguments for that HTTP verb only, and all lines are optional.

So, suppose you find yourself using the same curl options over and over. You
can save them in a file and resty will pass them to curl for you. Say this
is a frequent pattern for you:

      resty localhost:8080
      GET /Blah -H "Accept: application/json"
      GET /Other -H "Accept: application/json"
      ...
      POST /Something -H "Content-Type: text/plain" -u user:pass
      POST /SomethingElse -H "Content-Type: text/plain" -u user:pass
      ...

It's annoying to add the `-H` and `-u` options to curl all the time. So
create a file _~/.resty/localhost:8080_, like this:

_~/.resty/localhost:8080_

      GET -H "Accept: application/json"
      POST -H "Content-Type: text/plain" -u user:pass

Then any GET or POST requests to localhost:8080 will have the specified 
options prepended to the curl command line arguments, saving you from having
to type them out each time, like this:

      GET /Blah
      GET /Other
      ...
      POST /Something
      POST /SomethingElse
      ...

Sweet! Much better.

Exit Status
===========

Successful requests (HTTP respose with 2xx status) return zero.
Otherwise, the first digit of the response status is returned (i.e., 1 for
1xx, 3 for 3xx, 4 for 4xx, etc.) This is because the exit status is an 8 bit
integer---it can't be greater than 255. If you want the exact status code
you can always just pass the `-v` option to curl.

Using Resty In Shell Scripts
============================

Since resty creates the REST verb functions in the shell, when using it from a script you must `source` it before you use any of the functions. However, it's likely that you don't want it to be overwriting the resty host history file, and you will almost always want to set the URI base explicitly.

      #!/usr/bin/env bash

      # Load resty, don't write to the history file, and set the URI base
      . /path/to/resty -W 'https://myhost.com/data*.json'

      # GET the JSON list of users, set each of their 'disabled' properties
      # to 'false', and PUT the modified JSON back
      GET /users | jsawk 'this.disabled = false' | PUT

Here the `-W` option was used when loading the script to prevent writing to the history file and an initial URI base was set at the same time. Then a JSON file was fetched, edited using [jsawk](http://github.com/micha/jsawk), and re-uploaded to the server.

Working With JSON or XML Data
=============================

JSON REST web services require some special tools to make them accessible
and easily manipulated in the shell environment. The following are a few
scripts that make dealing with JSON data easier.

  * [Jsawk](http://github.com/micha/jsawk) can be used to process and filter
    JSON data from and to resty, in a shell pipeline. This takes care of
    parsing the input JSON correctly, rather than using regexes and sed,
    awk, perl or the like, and prints the resulting output in correct JSON
    format, as well.

    `GET /blogs.json |jsawk -n 'out(this.title)' # prints all the blog titles`

  * The included `pp` script will pretty-print JSON for you. You just need to
    install the JSON perl module from CPAN or you can use `pypp` if you have
    python 2.6 installed.

    `GET /blogs.json |pp # pretty-prints the JSON output from resty`

  * Another way to format JSON output:

      $ echo '{"json":"obj"}' | python -mjson.tool
      {
        "json": "obj"
      }

  * The `tidy` tool can be used to format HTML/XML:

      $ ~$ echo "<test><deep>value</deep></test>" | tidy -xml -q -i
      <test>
        <deep>value</deep>
      </test>
