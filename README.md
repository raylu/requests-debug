This is a debugging exercise based on the actual requests project.
There is a simple bug in the `cookie` branch and a more involved bug in the `content-length` branch.
You will need to set up a development environment and then wade through multiple levels of
abstraction to find the cause of the bugs.
Only then will you be able to perform the trivial task of changing some code to fix them.

## cookie

The original text of the issue:

> **"in" operation on resp.cookies should not raise CookieConflictError**

> Hosts which have the same cookie define for multiple domains should not raise 
"CookieConflictError: There are multiple cookies with name," when performing "in" 
It should short cut and return True immediately when found.

The last commit on the `cookie` branch adds a failing test called
`test_cookie_duplicate_names_different_domains`.
Get this test running, then fix.

## content-length

The original text of the issue:

> **Content-Length header is duplicated on Python3**

> Content-Length header is duplicated if specified on Python3, leading to 400 responses:

> <pre><code>Python 3.3.0 (default, Apr 30 2013, 06:45:58)
[GCC 4.2.1 Compatible Apple Clang 4.0 ((tags/Apple/clang-421.0.60))] on darwin
Type "help", "copyright", "credits" or "license" for more information.
&gt;&gt;&gt; import requests
&gt;&gt;&gt; requests.post("http://httpbin.org/post", headers={"Content-Length": 0})
&lt;Response [400]&gt;</code></pre>

> We need to go deeper:

> <pre><code>&gt;&gt;&gt; import six
&gt;&gt;&gt; six.moves.http_client.HTTPConnection.debuglevel = 1
&gt;&gt;&gt; requests.post("http://httpbin.org/post", headers={"Content-Length": 0})
send: b'POST /post HTTP/1.1\r\nHost: httpbin.org\r\nAccept-Encoding: identity\r\nContent-Length: 0\r\nUser-Agent: python-requests/1.2.0 CPython/3.3.0 Darwin/12.3.0\r\nContent-Length: 0\r\nAccept: */*\r\nAccept-Encoding: gzip, deflate, compress\r\n\r\n'
reply: 'HTTP/1.1 400 BAD_REQUEST\r\n'
header: Content-Length header: Connection &lt;Response [400]&gt;</code></pre>

> That's what happens:

> <pre><code>POST /post HTTP/1.1
Host: httpbin.org
Accept-Encoding: identity
Content-Length: 0
User-Agent: python-requests/1.2.0 CPython/3.3.0 Darwin/12.3.0
Content-Length: 0
Accept: */*
Accept-Encoding: gzip, deflate, compress</code></pre>

> Two content-lengths, one 400 response :-(
On Python2 everything's OK.
