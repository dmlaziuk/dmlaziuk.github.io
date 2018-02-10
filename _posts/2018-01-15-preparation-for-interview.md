---
title:  "Preparation for interview"
date:   2018-01-15
header:
  teaser: /images/mindmap.png
---
My preparation for Ruby on Rails interview.

![Mindmap][Mindmap]{: .align-center}

Summary of YouTube [lectures][Lectures] of Ruby on Rails Courses.

### 1. [Ubuntu install. Linux basics. RVM and Ruby install. Ruby Pros and Cons][Lecture1]

Ruby is interpreted, dymamic, object-oriented programming language.

* JIT — Just-in-Time compiling
* MRI — Metz Ruby Interpreter
* MJIT — MRI JIT
* YARV — Yet Another Ruby VM
* YARV-MJIT — modified MJIT
* LLRB — LLVM Ruby JIT
* LLVM — Low Level Virtual Machine
* GCC — GNU Compilers Collection
* JVM — Java virtual machine

RubyKaigi 2017, September 18..20, Hiroshima, Japan [#rubykaigi][Markov]

[Ruby 3 and JIT][Ruby3]: Where, When and How Fast?

REPL — Read Evaluate Play Loop — irb interactive Ruby

```bash
#!/usr/bin/env ruby
```

```
$ ruby —dump parse-tree
```

Book: "Agile Web Development with Rails 5"

Pipeline pattern

`gem 'interactor'`

### 2. [Ruby basics. Classes and modules][Lecture2]

#### Variables

* $global_variable
* local_variable
* @@class_variable
* @instance_variable
* CONSTANTS

.freeze

#### Functions

```ruby
arr.reduce(&:*) # multiply all elements of array
def func(first:, last:, a: nil, b:, c: true)
end
```

#### Classes

Book: "Ruby under microscope"

#### Modules

```ruby
Module Test
  include ‘test’ # instance methods
  prepend 'test' # precedence over include
  extend ‘test’  # class methods
  # ...
end
```

### 3. [Classes and Modules. Ruby Object Model. Lambdas vs Procs vs Blocks][Lecture3]

`self` — current recipient of the message

### Block, Proc, Lambda

1. Blocks — are not objects
2. Blocks are faster — no need for object creation
3. Proc, Lambda — are objects
4. Proc is insensitive to the number of arguments (may pass more on call)
5. Behavior on `return`:  
Anecdote:  
Proc and Lambda got into some sort of fight in a pub, and then  
Proc — flies off the pub  
Lambda — flies off the handle

to call `lambda` use `.call`, `[]`, `===`

```ruby
case
  when ->(v) { |v| v % 2 == 0 }
end
```

```ruby
a = Array.new(5) { |index| index }
```

### 4. [Metaprogramming][Lecture4]

```ruby
alias_method :old_plus, :+
# Pattern decorator
.send()
.define_method()
.undefine_method()
attr_reader
attr_writer
attr_accessor
.method_missing(name, *args, &block)
eval ‘’
.instance_eval ‘’
.class_eval ‘’
```

### 5. [Ruby Tips and Tricks. Array, Hash methods][Lecture5]

```ruby
arr.first(3) = arr.take(3)
arr.last() = arr.drop()
.each .reverse_each
.cycle
.sort_by(&:itself)
‘’.to_i(2)
.to_s(2)
```

### 6. [Gems. Ruby testing][Lecture6]

Semantic versioning:  
major.minor.patch

For example: ruby 2.4.1

Bundler

Minitest

RSpec

Timecop

stab — to substitute state,  
mock — to substitute behaviour

### 7. [Web basics. Rack][Lecture7]

TCP/IP стек:

* Application layer — HTTP, RTSP, FTP, DNS
* Transport layer — TCP, UDP, SCTP, DCCP
* Internet layer — Для TCP/IP это IP
* Link layer — Ethernet, IEEE 802.11 WLAN, SLIP, Token Ring, ATM и MPLS

OSI стек:

* 7: application — HTTP, FTP, SMTP, RDP, SNMP, DHCP
* 6: presentation — ASCII, EBCDIC, JPEG
* 5: session — RPC, PAP
* 4: transport — TCP, UDP, SCTP, PORTS
* 3: network — IPv4, IPv6, IPsec, AppleTalk
* 2: data link — PPP, IEEE 802.22, Ethernet, DSL, ARP, L2TP
* 1: physical — USB, twisted pair, coaxial cable, fibre cable

#### IP header

![IPheader][IPheader]{: .align-center}
￼
#### TCP header

![TCPheader][TCPheader]{: .align-center}
￼
CCNA Cisco

#### Rack

```ruby
run ->(env) { [200, {}, ['Hello world']] }
```

#### Node.js Event loop

![Node][Node]{: .align-center}

### 8. [HTML, CSS, JS basics. Sinatra introduction][Lecture8]

ERB

slim-lang.com

HTML2Slim

haml.info

CSS

SASS — Syntactically Awesome Stylesheets

JavaScript

CORS — cross-origin resource sharing

CoffeScript

Sinatra

### 9. ActiveRecord [Part 1][Lecture91], [Part 2][Lecture92]

ORM — Object-Relational Mapping

Pattern — Data mapper, Active record

`gem ‘activerecord’`

Migrations

```ruby
create table :cars do |t|
  # …
  t.ref references :user, foreign_key: true, index: true
end
```

```ruby
class User < ActiveRecord::Base
  has_many :cars, dependent: :destroy
end

class Car < ActiveRecord::Base
  belongs_to :user
end
```

```ruby
.find()
.find_by()
.where()
.order()
.limit()
.joins()
.pluck()
.destroy()
.save()
.update()
```

Callbacks

STI — Single Table Inheritance

### 10. [Padrino][Lecture10]

### 11. [Rails basics][Lecture11]

Rails:

* actioncable — to work with websocket
* actionmailer — work with mail
* actionpack —прослойка для работы приложения
* actionview — helpers to work with view
* activejob — to run background jobs
* activemodel — validations for Active Record
* activerecord — ORM
* activesupport — i18n, timezone, minitest
* railties —
* sprockets-rails —

Rails composer

### 12. [Rails routing and controllers][Lecture12]

CRUD — create, read, update, delete

routes shallow

params

cookies

session

flash

URI: `schema://user:password@host:port/path?query#fragment`

send_file

### 13. [Rails template engines. Assets pipeline. Rails form builders][Lecture13]

`erb` — embedded ruby

```ruby
respond_to do |format|
  format.html
  format.json do
    render json: @authors
  end
```

`gem oj` — optimized JSON (with native extensions)

[wkhtmltopdf][wkhtmltopdf]

`gem faker`

`gem ryba`

### 14. [Rails background jobs. Rails action cable][Lecture14]

```ruby
gem sidekiq
gem whenever

# Action Cable
gem anycable
```

### 15. [Deploying Rails Application][Lecture15]

`gem capistrano`

[Mindmap]: {{ "/images/mindmap.png" | absolute_url }}
[Lectures]: https://dmlaziuk.github.io/2017/10/04/youtube-lectures.html
[Markov]: http://rubykaigi.org/2017/presentations/vnmakarov.html
[Ruby3]: http://engineering.appfolio.com/appfolio-engineering/2017/12/26/ruby-3-and-jit-where-when-and-how-fast
[IPheader]: {{ "/images/ip_header.jpg" | absolute_url }}
[TCPheader]: {{ "/images/tcp_header.jpg" | absolute_url }}
[Node]: {{ "/images/node_event_loop.png" | absolute_url }}
[wkhtmltopdf]: https://wkhtmltopdf.org
[Lecture1]: https://youtu.be/x75YRjBV-w0
[Lecture2]: https://youtu.be/t7r9qeLuy5M
[Lecture3]: https://youtu.be/b_IgPqu8-7c
[Lecture4]: https://youtu.be/P4KvsTOE968
[Lecture5]: https://youtu.be/YGIQ7LR5oYU
[Lecture6]: https://youtu.be/S-7q2P4VpLg
[Lecture7]: https://youtu.be/HsVSI6ODXw4
[Lecture8]: https://youtu.be/Nqeepz1qw0o
[Lecture91]: https://youtu.be/_VeSBoAmOD8
[Lecture92]: https://youtu.be/mKKHXSPk8eo
[Lecture10]: https://youtu.be/OVSCX5Jh8jw
[Lecture11]: https://youtu.be/e7PPNijw1V4
[Lecture12]: https://youtu.be/iCOvrHBf4Jg
[Lecture13]: https://youtu.be/N4L1XPPrSMg
[Lecture14]: https://youtu.be/J3ibzAU4Dro
[Lecture15]: https://youtu.be/_xLn0juHZAU
