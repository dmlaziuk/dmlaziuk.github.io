---
title:  "Game of Life"
date:   2017-11-09
header:
  teaser: /images/gospers_glider_gun.gif
toc: true
toc_label: Index
toc_icon: "hand-point-right"
tags:
  - ruby
  - contest
---
On the third workshop we had to write our own implementation of the Game of Life.

[![Life]({{ "/images/gospers_glider_gun.gif" | absolute_url }}){: .align-center}][Life]

## Source code

```ruby
# life.rb
require_relative 'game_of_life'

life = GameOfLife.new
life.print_board
sleep(0.1)
loop do
  puts "\e[H\e[2J"
  life.run
  life.print_board
  sleep(0.1)
end
```

```ruby
# game_of_life.rb
class GameOfLife
  X_SIZE = 100
  Y_SIZE = 50
  def initialize
    @generation = [''] * Y_SIZE
    @generation.map! do
      str = ''
      X_SIZE.times { str << (rand(3).zero? ? '*' : ' ') }
      str
    end
  end

  def run
    next_generation = []
    (0...Y_SIZE).each do |y|
      str = ' ' * X_SIZE
      (0...X_SIZE).each do |x|
        current = @generation[y][x]
        count = count_neighbours(y, x)
        str[x] = '*' if current == ' ' && count == 3
        str[x] = '*' if current == '*' && [2, 3].include?(count)
      end
      next_generation << str
    end
    @generation = next_generation
  end

  def count_neighbours(y, x)
    count = 0
    count += 1 if @generation[(y + 1) % Y_SIZE][(x - 1) % X_SIZE] == '*'
    count += 1 if @generation[(y + 1) % Y_SIZE][x % X_SIZE] == '*'
    count += 1 if @generation[(y + 1) % Y_SIZE][(x+1) % X_SIZE] == '*'
    count += 1 if @generation[y % Y_SIZE][(x - 1) % X_SIZE] == '*'
    count += 1 if @generation[y % Y_SIZE][(x + 1) % X_SIZE] == '*'
    count += 1 if @generation[(y - 1) % Y_SIZE][(x - 1) % X_SIZE] == '*'
    count += 1 if @generation[(y - 1) % Y_SIZE][x % X_SIZE] == '*'
    count += 1 if @generation[(y - 1) % Y_SIZE][(x + 1) % X_SIZE] == '*'
    count
  end

  def print_board
    puts @generation
  end
end
```

## Video

{% include video id="249877584" provider="vimeo" %}

Source files are available on [GitHub][GitHub].

[Life]: https://en.wikipedia.org/wiki/Conway%27s_Game_of_Life
[GitHub]: https://github.com/dmlaziuk/bsuir-courses/tree/dm-life/2017/DmLaziuk/life
