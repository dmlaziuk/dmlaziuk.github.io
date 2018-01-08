---
title:  "Rack"
date:   2017-11-19
header:
  teaser: /images/rack-teaser.png
---
I wanted to understand how HTTP protocol works.

![Rack]({{ "/images/rack.png" | absolute_url }}){: .align-center}

To find this out the best recipe is to read standards.
So I found [RFC2617][RFC2617].

To act as a client I send requests to some server.
Here is the client app `http.rb`:
```ruby
require 'open3'

def request(str)
  cmd = 'nc localhost 9292'
  Open3.pipeline_w(cmd) { |stdin| stdin << str }
end

request "OPTIONS * HTTP/1.1\r\n\r\n"
request "HEAD / HTTP/1.1\r\nHost: localhost\r\nConnection: Close\r\n\r\n"
request "GET / HTTP/1.1\r\nHost: localhost\r\nConnection: Close\r\n\r\n"
request "POST / HTTP/1.1\r\nHost: localhost\r\nContent-Length: 6\r\n"\
"Connection: Close\r\n\r\nPOST\r\n"
```

To respond to the requests I had to start my own web server.
It's not nginx or apache, but an application server built in Ruby.
It called WEBrick.

But to build a web application you need some interface to communicate to http requests and send responses.
This interface in Ruby is called Rack.

The interface is simple.
You have to implement one function named `call(env)` with only one parameter `env`.

The other way to use Rack is middleware. You can build stack of Rack apps with various functionality.

My Rack app is Rack::Lobster, and my Rack middleware is simple "Hello world".

Source code of `rack_app.rb`:
```ruby
class RackApp

  def initialize(app)
    @app = app
  end

  def call(env)
    status, header, body = @app.call(env)
    res = Rack::Response.new(body, status, header)
    res.write "<p>Hello, lobster!</p>\n"
    res.finish
  end
end
```

To start web server you have to write `rackup` in command line.

`rackup` reads configuration from `config.ru`:
```ruby
require 'rack'
require 'rack/lobster'
require_relative 'rack_app'

use Rack::Head
use Rack::Reloader, 0
use Rack::CommonLogger
use Rack::ShowExceptions
use RackApp
run Rack::Lobster.new
```

Let's explain Rack middleware stack:
- Rack::Head — without this middleware WEBrick doesn't respond to HTTP request HEAD
- Rack::Reloader, 0 — you can modify rack_app.rb without reloading WEBrick
- Rack::CommonLogger — log HTTP requests and responses
- Rack::ShowExceptions — shows Ruby exceptions in browser
- RackApp — my middleware, that adds "Hello, lobster!" at the end of the body

The output of my client `http.rb`:
* for `OPTIONS *` request response is:
```
HTTP/1.1 200 OK
Allow: GET,HEAD,POST,OPTIONS
Server: WEBrick/1.3.1 (Ruby/2.4.1/2017-03-22)
Date: Sun, 19 Nov 2017 01:47:53 GMT
Content-Length: 0
Connection: close
```
* for `HEAD /` request response is:
```
HTTP/1.1 200 OK
Content-Length: 615
Server: WEBrick/1.3.1 (Ruby/2.4.1/2017-03-22)
Date: Sun, 19 Nov 2017 01:47:53 GMT
Connection: close
```
* for `GET /` request response is:
{% highlight html %}
HTTP/1.1 200 OK
Content-Length: 615
Server: WEBrick/1.3.1 (Ruby/2.4.1/2017-03-22)
Date: Sun, 19 Nov 2017 01:47:53 GMT
Connection: close

<title>Lobstericious!</title><pre>                             ,.---._
                   ,, ,     /       `,
                    \\\\   /    '\_  ;
                     |||| /\/``-.__\;'
                     ::::/\/_
      {`-.__.-'(`(^^(^^^(^ 9 `.========='
    { { {  { ( ( (  (   (-----:=
      {.-'~~'-.(,(,,(,,,(__6_.'=========.
                     ::::\/\
                     |||| \/\  ,-'/,
                    ////   \ `` _/ ;
                   ''''     \  `  .'
                             `---'
</pre><p><a href='?flip=left'>flip!</a></p><p><a href='?flip=crash'>crash!</a></p>
<p>Hello, lobster!</p>
{% endhighlight %}
* for `POST /` request response is:
{% highlight html %}
HTTP/1.1 200 OK
Content-Length: 615
Server: WEBrick/1.3.1 (Ruby/2.4.1/2017-03-22)
Date: Sun, 19 Nov 2017 01:47:53 GMT
Connection: close

<title>Lobstericious!</title><pre>                             ,.---._
                   ,, ,     /       `,
                    \\\\   /    '\_  ;
                     |||| /\/``-.__\;'
                     ::::/\/_
      {`-.__.-'(`(^^(^^^(^ 9 `.========='
    { { {  { ( ( (  (   (-----:=
      {.-'~~'-.(,(,,(,,,(__6_.'=========.
                     ::::\/\
                     |||| \/\  ,-'/,
                    ////   \ `` _/ ;
                   ''''     \  `  .'
                             `---'
</pre><p><a href='?flip=left'>flip!</a></p><p><a href='?flip=crash'>crash!</a></p>
<p>Hello, lobster!</p>
{% endhighlight %}

Source files are available on [GitHub][GitHub].

[RFC2617]: http://www.ietf.org/rfc/rfc2616.txt
[GitHub]: https://github.com/dmlaziuk/rack.git
