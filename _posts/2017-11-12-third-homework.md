---
title:  "Third homework"
date:   2017-11-12
---
Inspired by recent [coming out by Kevin Spacey](https://news.tut.by/culture/566702.html) the task was to write Telegram chat bot that finds out celebrities coming outs.

[![CominoutBot]({{ "/images/lgbt.png" | absolute_url }})][CominoutBot]

Instructions on creating Telegram bot and deploying it on Heroku described [here][Heroku].

Data parsed from [wikipedia.org][Wikipedia] and [IMDB.com][IMDB] are stored in Redis database in the following format:
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

[`gem 'ferret'`][FerretGem] is used for fuzzy search.

This bot runs on [heroku.com](https://herokuapp.com) and is available on Telegram as [@cominoutbot][CominoutBot]

Used books:
1. [The Little Redis Book][RedisBook]
2. [Ferret][FerretBook]

Source code of `Procfile`:
```Procfile
bot: ruby comingoutbot.rb
```

Source code of `Gemfile`:
```Gemfile
ruby '2.4.1'

gem 'ferret'
gem 'mechanize'
gem 'redis'
gem 'telegram-bot-ruby'
```

Source code of `comingoutbot.rb`:
```ruby
#!/usr/bin/env ruby
require_relative 'lib/comingout'

Comingout::Bot.new.run
```

Source code of `lib/comingout.rb`:
```ruby
require_relative 'comingout/constants'
require_relative 'comingout/comingout_db'
require_relative 'comingout/parse_imdb'
require_relative 'comingout/parse_wikipedia'
require_relative 'comingout/parse_ru_wikipedia'
require_relative 'comingout/bot'
```

Source code of `lib/comingout/constants.rb`:
```ruby
module Comingout
  DB = 'lgbt'.freeze
  DB_INDEX = 'lgbt:index'.freeze
  TELEGRAM_TOKEN = 'your token'.freeze

  RU_EN = { 'А' => 'A', 'а' => 'a', 'Б' => 'B', 'б' => 'b',
            'В' => 'V', 'в' => 'v', 'Г' => 'G', 'г' => 'g',
            'Д' => 'D', 'д' => 'd', 'Е' => 'E', 'е' => 'e',
            'Ё' => 'Jo', 'ё' => 'jo', 'Ж' => 'Zh', 'ж' => 'zh',
            'З' => 'Z', 'з' => 'z', 'И' => 'I', 'и' => 'i',
            'Й' => 'J', 'й' => 'j', 'К' => 'K', 'к' => 'k',
            'Л' => 'L', 'л' => 'l', 'М' => 'M', 'м' => 'm',
            'Н' => 'N', 'н' => 'n', 'О' => 'O', 'о' => 'o',
            'П' => 'P', 'п' => 'p', 'Р' => 'R', 'р' => 'r',
            'С' => 'S', 'с' => 's', 'Т' => 'T', 'т' => 't',
            'У' => 'U', 'у' => 'u', 'Ф' => 'F', 'ф' => 'f',
            'Х' => 'H', 'х' => 'h', 'Ц' => 'C', 'ц' => 'c',
            'Ч' => 'Ch', 'ч' => 'ch', 'Щ' => 'Sch', 'щ' => 'sch',
            'Ш' => 'Sh', 'ш' => 'sh', 'Ы' => 'Y', 'ы' => 'y',
            'Ь' => '', 'ь' => '', 'Ъ' => '', 'ъ' => '',
            'Э' => 'E', 'э' => 'e', 'Ю' => 'U', 'ю' => 'u',
            'Я' => 'Ja', 'я' => 'ja' }.freeze

  def self.translit(text)
    text.split('').map do |char|
      RU_EN[char] ? RU_EN[char] : char
    end.join('')
  end
end
```

Source code of `lib/comingout/comingout_db.rb`:
```ruby
require 'redis'
require 'ferret'
require_relative 'constants'

module Comingout
  class ComingoutDB
    attr_reader :redis, :ferret

    def initialize
      @redis = Redis.new(url: ENV['REDIS_URL']) # uncomment for  heroku.com
      @redis.flushall
      @ferret = Ferret::I.new(key: :id)
      @redis.setnx Comingout::DB_INDEX, 0
    end

    def add(name, uri, note)
      index = @redis.get Comingout::DB_INDEX
      @redis.hset "#{Comingout::DB}:#{index}", 'name', name
      @redis.hset "#{Comingout::DB}:#{index}", 'uri', uri
      @redis.hset "#{Comingout::DB}:#{index}", 'note', note
      prime_names = name.split(/\s/)
      prime_names.each do |each_name|
        @redis.rpush "#{Comingout::DB}:name:#{each_name.downcase}", index
      end
      @ferret << { id: index, name: name }
      @redis.incr Comingout::DB_INDEX
    end

    def addnx(name, uri, note)
      found = @ferret.search(name)
      max_score = found[:max_score]
      return nil if max_score > 1
      add(name, uri, note)
    end

    def get_by_name(name)
      @redis.lrange "#{Comingout::DB}:name:#{name}", 0, -1
    end

    def get_by_index(index)
      @redis.hgetall "#{Comingout::DB}:#{index}"
    end
  end
end
```

Source code of `lib/comingout/parse_imdb.rb`:
```ruby
require 'mechanize'
require_relative 'constants'
require_relative 'comingout_db'

module Comingout
  class ParseImdb
    IMDB_PAGE = 'http://www.imdb.com/list/ls072706884/'.freeze

    def initialize(db)
      @agent = Mechanize.new
      @db = db
    end

    def parse
      count = 0
      total_count = 0
      print 'Parsing IMDB.com '
      page = @agent.get(IMDB_PAGE)
      print '.'
      info = page.xpath('//*[@id="main"]/div/div[7]/div')
      info.each do |item|
        name = item.xpath('./div[3]/b').text
        uri = item.xpath('./div[3]/b/a/@href').text
        uri = "http://www.imdb.com#{uri}"
        note = item.xpath('./div[4]').text
        count += 1 if @db.addnx(name, uri, note)
        total_count += 1
      end
      puts "\n#{count} entries added out of #{total_count}"
    end
  end
end
```

Source code of `lib/comingout/parse_wikipedia.rb`:
```ruby
require 'mechanize'
require_relative 'constants'
require_relative 'comingout_db'

module Comingout
  class ParseWikipedia
    WIKI_PAGE = 'https://en.wikipedia.org/wiki/List_of_gay,_lesbian_or_bisexual_people'.freeze

    def initialize(db)
      @agent = Mechanize.new
      @db = db
      @count = 0
    end

    def parse
      print 'Parsing en.wikipedia.org '
      path = '//*[@id="mw-content-text"]/div/div[3]/ul/li/a'
      links = @agent.get(WIKI_PAGE).links_with(xpath: path)
      urls = links.map { |link| "https://en.wikipedia.org/wiki/#{link}" }
      urls.each do |url|
        print '.'
        page = @agent.get(url)
        parse_table(page)
      end
      puts "\n#{@count} entries added"
    end

    private

    def parse_table(page)
      tr = page.xpath('//*[@id="mw-content-text"]/div/table/tr')
      tr.shift # table header
      tr.each do |row|
        td = row.xpath('./td')
        next if td.size != 5 # duct tape for error in "Z" table
        name = td[0].xpath('.//a')
        ref = td[4].xpath('.//a/@href')
        ref_path = "//*[@id=\"#{ref.text[1..-1]}\"]"
        ref_path << '//span[@class="reference-text"]'
        ref_text = page.xpath(ref_path).text
        uri = "https://en.wikipedia.org#{name.xpath('@href')}"
        @db.add(name.text, uri, ref_text)
        @count += 1
      end
    end
  end
end
```

Source code of `lib/comingout/parse_ru_wikipedia.rb`:
```ruby
require 'mechanize'
require_relative 'constants'
require_relative 'comingout_db'

module Comingout
  class ParseRuWikipedia
    WIKI_PAGE = 'https://ru.wikipedia.org/wiki/Проект:ЛГБТ/Списки'\
      '/Известные_лесбиянки,_геи_и_бисексуалы_России'.freeze

    def initialize(db)
      @agent = Mechanize.new
      @db = db
      @count = 0
    end

    def parse
      print 'Parsing ru.wikipedia.org '
      print '.'
      page = @agent.get(WIKI_PAGE)
      parse_table(page)
      puts "\n#{@count} entries added"
    end

    private

    def parse_table(page)
      tr = page.xpath('//*[@id="mw-content-text"]/div/table/tr')
      tr.shift # table header
      tr.each do |row|
        td = row.xpath('./td')
        name = td[3].xpath('.//a')
        note = td[4].text
        note.gsub!(/\[\d*?\]/, '')
        uri = "https://ru.wikipedia.org#{name.xpath('@href')}"
        @db.add(name.text, uri, note)
        @count += 1
      end
    end
  end
end
```

Source code of `lib/comingout/bot.rb`:
```ruby
require 'telegram/bot'
require_relative 'constants'
require_relative 'comingout_db'

module Comingout
  class Bot
    def initialize
      @db = Comingout::ComingoutDB.new
    end

    def run
      Comingout::ParseWikipedia.new(@db).parse
      Comingout::ParseRuWikipedia.new(@db).parse
      Comingout::ParseImdb.new(@db).parse
      puts 'Starting bot'

      Telegram::Bot::Client.run(Comingout::TELEGRAM_TOKEN) do |bot|
        bot.listen do |chat|
          cmd = chat.text
          case cmd
          when '/start'
            msg = "Hello, #{chat.from.first_name}!\n"
            msg << 'This bot is for finding out celebrities coming out.'
            say(bot, chat, msg)
          else
            dialog(bot, chat)
          end
        end
      end
    end

    private

    def dialog(bot, chat)
      chat_text = chat.text
      chat_text.gsub!(/[*?]/, '') # remove wildcard search
      found = @db.ferret.search(chat_text, limit: :all) # strict search
      max_score = found[:max_score]
      hits = found[:hits]
      return do_you_mean(bot, chat) if hits.empty?
      hits_max = []
      hits.each { |hit| hits_max << hit if hit[:score] == max_score }
      return one_hit(bot, chat, hits_max.first[:doc]) if hits_max.size == 1
      return five_hits(bot, chat, hits) if hits.size < 5
      muli_hits(bot, chat, hits)
    end

    def do_you_mean(bot, chat)
      chat_text = Comingout.translit(chat.text.downcase)
      found = @db.ferret.search("#{chat_text}~", limit: :all) # fuzzy search
      max_score = found[:max_score]
      hits = found[:hits]
      return say(bot, chat, 'Found no data') if hits.empty?
      hits_max = []
      hits.each { |hit| hits_max << hit if hit[:score] == max_score }
      if hits_max.size == 1
        return if one_hit_right?(bot, chat, hits_max.first[:doc])
      end
      return five_hits(bot, chat, hits) if hits.size < 5
      muli_hits(bot, chat, hits)
    end

    def one_hit(bot, chat, hit)
      doc = @db.ferret[hit]
      person = @db.get_by_index(doc[:id])
      say(bot, chat, comeout(person))
    end

    def one_hit_right?(bot, chat, hit)
      doc = @db.ferret[hit]
      person = @db.get_by_index(doc[:id])
      msg = "Do you mean <b>#{person['name']}?</b>"
      ans = Telegram::Bot::Types::ReplyKeyboardMarkup.new(
        keyboard: [%w[yes no]], one_time_keyboard: true
      )
      bot.api.send_message(chat_id: chat.chat.id, text: msg, reply_markup: ans,
                           parse_mode: 'HTML')
      response = ''
      bot.listen { |req| response = req.text; break }
      if %w[yes да].include?(response.downcase)
        say(bot, chat, comeout(person))
        true
      end
    end

    def five_hits(bot, chat, hits)
      names = []
      msg = "There are #{hits.size} persons with given name:\n"
      hits.each do |db_index|
        doc = @db.ferret[db_index[:doc]]
        person = @db.get_by_index(doc[:id])
        names << person['name']
      end
      ans = Telegram::Bot::Types::ReplyKeyboardMarkup.new(
        keyboard: names, one_time_keyboard: true
      )
      bot.api.send_message(chat_id: chat.chat.id, text: msg, reply_markup: ans)
    end

    def muli_hits(bot, chat, hits)
      msg = "There are #{hits.size} persons with given name:\n"
      hits.each_with_index do |db_index, counter|
        doc = @db.ferret[db_index[:doc]]
        person = @db.get_by_index(doc[:id])
        msg << "#{counter + 1}. #{person['name']}\n"
      end
      say(bot, chat, msg)
    end

    def say(bot, chat, msg)
      bot.api.send_message(chat_id: chat.chat.id, text: msg, parse_mode: 'HTML')
    end

    def comeout(person)
      msg = "<a href='#{person['uri']}'>#{person['name']}</a>\n"
      msg << "<b>Coming out:</b> <i>#{person['note']}</i>"
    end
  end
end
```

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
