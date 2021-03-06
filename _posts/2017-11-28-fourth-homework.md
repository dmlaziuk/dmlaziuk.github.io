---
title:  Fourth homework
date:   2017-11-28
header:
  teaser: /images/b2b.png
toc: true
toc_label: Index
toc_icon: hand-point-right
tags:
  - Ruby
---

The main theme of this homework is to connect people affected by hype in both bitcoins and bonsticks.

[![B2B]({{ "/images/b2b.png" | absolute_url }}){: .align-center}][AWS]

## Objective

This is simple Rails B2B application. In our context it transcribes as Bonstick2Bitcoin.

The task was to develop billboard with notices.
Here everyone can write what he want to sell or buy (bonsticks or bitcoins).
Comments to notices are allowed. In comments you can add contact information or just chat.

Pagination is implemented by `gem kaminari`.

Real exchange rate between bitcoin and BYR taken from web-API and stored in yaml format.

```ruby
# btc.rb
require 'yaml'
require 'rest-client'
require 'json'

response = RestClient.get('https://free.currencyconverterapi.com/api/v5/convert?q=BTC_BYR&compact=y')
pars = JSON.parse(response)
btc_byr = pars['BTC_BYR']['val']
puts btc_byr
File.open("btc_byr.yml", "w") {|f| f.write(btc_byr.to_yaml) }
```

## Result

The site is up and runing on [AWS][AWS].

## Video

{% include video id="249877559" provider="vimeo" %}

Source files are available on [GitHub][GitHub].

[AWS]: http://ec2-18-217-123-149.us-east-2.compute.amazonaws.com/
[GitHub]: https://github.com/dmlaziuk/bsuir-courses/tree/dm-homework-4/2017/DmLaziuk/b2b
