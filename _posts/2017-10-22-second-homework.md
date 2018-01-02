---
title:  "Second homework"
date:   2017-10-22
---
The second homework was about [King of the Dot][KOTD] rap battles.

[![King of the Dot]({{ "/images/kotd.jpg" | absolute_url }})][KOTD]

The task was to determine winner in each battle.
Lyrics were taken from [Genius.com][Genius].
The criteria is the number of letters spelled by each participant in the battle.
You may also choose name of the participant and your own criteria.

```
$ NAME=DNA CRITERIA=you ruby kotd.rb
```

Source code for `kotd.rb`:
```ruby
require 'mechanize'
require_relative 'kotd_battle'

class Kotd
  START_PAGE = 'https://genius.com/artists/songs?for_artist_page=117146&id=King-of-the-dot&page=1&pagination=true'.freeze

  attr_reader :links, :name, :criteria

  def initialize(name = nil, criteria = nil)
    @links = []
    @name = name
    @criteria = criteria
    parse_links
  end

  def run
    if @name
      run_by_name
    else
      run_all
    end
  end

  private

  def parse_links
    puts 'Initializing...'
    puts
    agent = Mechanize.new
    page = agent.get(START_PAGE)
    page.links_with(href: /\w+-lyrics/).each { |link| @links << link }
    next_page = page.links_with(class: 'next_page').first
    while next_page
      page = next_page.click
      page.links_with(href: /\w+-lyrics/).each { |link| @links << link }
      next_page = page.links_with(class: 'next_page').first
    end
  end

  def run_all
    @links.each do |link|
      battle = KotdBattle.new(link.click, @criteria)
      puts battle
      puts
    end
  end

  def run_by_name
    wins = 0
    battles = @links.select { |link| link.text.scan(@name).size >= 1 }
    battles.each do |link|
      battle = KotdBattle.new(link.click, @criteria)
      wins += 1 if battle.winners.include?(@name.to_sym)
      puts battle
      puts
    end
    loses = battles.size - wins
    puts @name + ' wins ' + wins.to_s + ' times, loses ' + loses.to_s + ' times'
  end
end

name = ENV['NAME']
criteria = ENV['CRITERIA']
Kotd.new(name, criteria).run
```

Source code for `kotd_battle.rb`:
```ruby
require 'mechanize'

class KotdBattle
  attr_reader :title, :lyrics, :uri, :winners, :score

  def initialize(page = nil, criteria = nil)
    @title = ''
    @lyrics = ''
    @count = {}
    @winners = []
    @score = 0
    @criteria = criteria
    return if page.nil?
    init_by_page(page)
    battle
    guess_winner
  end

  def to_s
    str = @title.to_s + ' - ' + @uri.to_s
    if @count
      str += "\n"
      @count.each { |key, value| str += key.to_s + ' - ' + value.to_s + "\n" }
      str += @winners.join(' and ').to_s + ' WINS!' + "\n"
    end
    str
  end

  private

  def init_by_page(page)
    @uri = page.uri
    @title = page.title
    @title.gsub!('King of the Dot –', ' ')
    @title.gsub!('Lyrics | Genius Lyrics', ' ')
    @title.strip!
    @lyrics = page.css('.lyrics').text.strip
    # remove [?]
    @lyrics.gsub!(/\[\?\]/, ' ')
    # remove [...]
    @lyrics.gsub!(/\[\.\.\.\]/, ' ')
    @lyrics.gsub!(/\[…\]/, ' ')
    # remove [*text*]
    @lyrics.gsub!(/\[\*.*?\*\]/, ' ')
  end

  def battle
    round = @lyrics.scan(/\[.*?\]/)
    text = @lyrics.split(/\[.*?\]/)
    text.shift # first element is always "" (zero string)
    (0...round.count).each do |i|
      performer = round[i][1...-1]
      performer.gsub!(/Round\s\d\s?[:|\-|\u2013]*\s*/, ' ')
      performer.strip!
      key = performer.to_sym
      if text[i]
        t = text[i] # duct tape to avoid rubocop "line is too long"
        counter = @criteria ? t.scan(@criteria).count : t.scan(/[A-Za-z]/).count
      end
      if counter
        @count[key] = @count[key] ? @count[key] + counter : counter
      end
    end
  end

  def guess_winner
    return if @count.empty?
    @score = @count.values.max
    @count.each { |key, value| @winners << key if value == @score }
  end
end
```

The sample output:
```
$ NAME=DNA CRITERIA=you ruby kotd.rb
Initializing...

AR & Talksic vs DNA & Charron - https://genius.com/King-of-the-dot-ar-and-talksic-vs-dna-and-charron-lyrics
DNA - 22
Charron - 34
Talksic - 0
100 Bulletz - 0
Charron WINS!

A. Ward vs DNA - https://genius.com/King-of-the-dot-a-ward-vs-dna-lyrics
DNA - 47
A. Ward - 50
A. Ward + DNA - 23
A. Ward WINS!

Charron vs DNA - https://genius.com/King-of-the-dot-charron-vs-dna-lyrics
Charron - 61
DNA - 60
Charron WINS!

Dizaster vs DNA - https://genius.com/King-of-the-dot-dizaster-vs-dna-lyrics
Dizaster - 254
DNA - 292
Diz: DNA's a faggot! - 0
Diz: Canadians are faggots! *Crowd boos* - 5
DNA WINS!

Dizaster vs DNA (2016) - https://genius.com/King-of-the-dot-dizaster-vs-dna-2016-lyrics
DNA - 52
Dizaster - 63
Dizaster WINS!

DNA vs Dirtbag Dan - https://genius.com/King-of-the-dot-dna-vs-dirtbag-dan-lyrics
DNA - 67
Dirtbag Dan - 91
Dirtbag Dan WINS!

DNA vs Eurgh - https://genius.com/King-of-the-dot-dna-vs-eurgh-lyrics
Eurgh - 69
DNA - 91
DNA WINS!

DNA vs Jimz - https://genius.com/King-of-the-dot-dna-vs-jimz-lyrics
Jimz - 67
DNA - 52
Crowd - 0
Jimz WINS!

DNA vs. Rone (2014) - https://genius.com/King-of-the-dot-dna-vs-rone-2014-lyrics
Intro: Organik - 2
DNA - 47
Organik - 0
Rone - 37
DNA WINS!

Illmaculate vs DNA - https://genius.com/King-of-the-dot-illmaculate-vs-dna-lyrics
DNA - 69
Illmaculate - 54
DNA WINS!

The Saurus vs DNA - https://genius.com/King-of-the-dot-the-saurus-vs-dna-lyrics
The Saurus - 41
DNA - 97
DNA WINS!

DNA wins 5 times, loses 6 times
```

Sample video is available on [RecordIt][RecordIt].

Source files are available on [GitHub][GitHub].

[KOTD]: https://www.youtube.com/user/KingOfTheDot
[Genius]: https://genius.com/artists/King-of-the-dot
[RecordIt]: http://recordit.co/tQAyvEpFmL
[GitHub]: https://github.com/dmlaziuk/bsuir-courses/tree/dm-homework-2/2017/DmLaziuk/2
