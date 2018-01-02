---
title:  "Third homework"
date:   2017-11-12
---
[![CominoutBot]({{ "/images/lgbt.png" | absolute_url }})][CominoutBot]

Inspired by recent [coming out by Kevin Spacey](https://news.tut.by/culture/566702.html) the task was to write Telegram chat bot that finds out celebrities coming outs.


Instructions on creating Telegram bot and deploying it on Heroku described [here][Heroku].

Data parsed from [wikipedia.org][Wikipedia] and [IMDB.com][IMDB] and stored in Redis database in the following format:
```
lgbt:index (String) -- current index (increased on adding)

lgbt:__index__ (Hash) -- person's data
               name:  -- full name
               uri:   -- url link
               note:  -- coming out data

lgbt:name:__name__ (List) -- list of indexes names occurances in full names
```

For example current index is `lgbt:index => "13"`.
After adding "Kevin Spacey" and "Kevin Williamson" the following entries will be created:
```
lgbt:13 =>
    name: "Kevin Spacey"
    uri:  "...some uri..."
    note: "...some note..."
lgbt:name:kevin => "13"
lgbt:name:spacey => "13"

lgbt:14 =>
    name: "Kevin Williamson"
    uri:  "...some uri..."
    note: "...some note..."
lgbt:name:kevin => "13" "14"
lgbt:name:williamson => "14"
```

[`gem 'ferret'`][Ferret] is used for fuzzy search.
 
This bot runs on [heroku.com](https://herokuapp.com) and is available on Telegram as [@cominoutbot][CominoutBot]

Used books:
1. [The Little Redis Book][RedisBook]
2. [Ferret][FerretBook]

Sample video is available on [RecordIt][RecordIt].

Source files are available on [GitHub][GitHub].

[Heroku]: https://github.com/barbaramartina/ruby-telegram-bot
[Wikipedia]: https://en.wikipedia.org/wiki/List_of_gay,_lesbian_or_bisexual_people
[IMDB]: http://www.imdb.com/list/ls072706884/
[FerretGem]: https://rubygems.org/gems/ferret
[CominoutBot]: https://t.me/cominoutbot
[RedisBook]: https://github.com/kondratovich/the-little-redis-book/blob/master/en/redis.md
[FerretBook]: https://www.safaribooksonline.com/library/view/ferret/9780596519407/
[RecordIt]: http://recordit.co/iR7FXIFzzK
[GitHub]: https://github.com/dmlaziuk/bsuir-courses/tree/dm-homework-3/2017/DmLaziuk/3
