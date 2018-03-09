---
title:  "Interview at Rubyroid Labs"
date:   2018-01-16
header:
  teaser: /images/job_answers.png
tags:
  - interview
---
My first interview.

![Interview][Interview]{: .align-center}

It was thrilling to pass IT interview for the first time.
I have a lot of questions to ask about new job, my responsibilities and what will be my first project?

At Rubyroid Labs site there were a [blog post][RubyroidlabsInterview] on how to interview Ruby on Rails developers.
I used this article as a survey for preparation.
But real interview questions differs from that.

The first quiz on Ruby:

```ruby
{}.class[[].class.new(2) {|_| [[_],[_]]}]
```

It took me almost 5 minutes to describe all the hidden classes, Array initialization and Array to Hash mapping.

The other one was all about internet infrastructure:

_Describe full path of user request from the browser through all protocols to server response and then back to browser and DOM rendering._

It was a long story of packets journey that took me almost 20 to 30 minutes.

And the final one was on JavaScript knowledge:

```js
for(var i = 0; i < 10; i++) {
  setTimeout(function(){
    console.log(i)
  },0)
}
```

I didn't use javascript much, but the answer was obvious.

We also had a long conversation on future Ruby 3 features, attitude to Node.js and other urgent technologies.

Finally we talked on my wage.
I had no wage expectations just because I was jumping from Building to IT industry.
So I agreed with the offer.

Hurrah!!!  
Now I'm a Software Developer!

![Cowsay][Cowsay]{: .align-center}

[Interview]: {{ "/images/job_interview.png" | absolute_url }}
[RubyroidlabsInterview]: https://rubyroidlabs.com/blog/2016/12/how-we-interview-ruby/
[Cowsay]: {{ "/images/cowsay.png" | absolute_url }}
