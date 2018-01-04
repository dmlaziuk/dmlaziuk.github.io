---
title:  "Tetris contest"
date:   2017-10-26
---
On the second workshop we were struggling on tetris contest.

[![codenjoy.com]({{ "/images/codenjoy.jpg" | absolute_url }})][Server]

We had one dedicated server. All of us divided by pairs were clients.

Every second server sends us a string containing current figure `figure=O`, coordinates of the figure `x=4`, `y=20`, next 4 figures `next=OOOO` and content of the glass `glass=.{#{SIZE}}`.

`"figure=O&x=4&y=20&glass=      ****      ****                                                                                                                                                                                    &next=OOOO"`

Our clients had to respond a commands `left=` or `right=` then `rotate=` and `drop` separated by commas.

Instructions to run server locally described on [codenjoy.com](http://codenjoy.com/portal/?cat=3), source code located on [GitHub](https://github.com/tdd-elevator-training/tetris.git).
Client boilerplate for Ruby located on [GitHub](https://github.com/FUT/codenjoy_client.git).

To start the server type `$ java -jar start.jar`. The server starts at localhost:8080.

To run a client you have to install `gem 'codenjoy_connection'`.
Then you need to register at `localhost:8080` and type your `name`, `port` and `host_ip` in client.rb.

Source code of `client.rb`:
```ruby
require 'codenjoy_connection'
require_relative 'player'

host_ip = '127.0.0.1' # ip of host with running tetris-server
port = '8080' # this port is used for communication between your client and tetris-server
user = 'dml' # your username, use the same for registration on tetris-server

opts = { username: user, host: host_ip, port: port, game_url: 'ws?' }

player = Player.new
CodenjoyConnection.play(player, opts)
```

Source code of `player.rb`:
```ruby
class Player

  MAX_X = 10
  MAX_Y = 20
  SIZE = MAX_X * MAX_Y

  # initialize your player
  def initialize
    # @x_glass -- straight (vertical standing) glass
    @x_glass = Array.new(MAX_Y) { '' }
    # @y_glass -- transposed (horizontal) glass
    @y_glass = Array.new(MAX_X) { ' ' * MAX_Y }
  end

  # process data for each event from tetris-server
  def process_data(data)
    @figure = data[/figure=.+?/][7, 1].to_sym
    @x = data[/x=\d+/][2, 2].to_i
    @y = data[/y=\d+/][2, 2].to_i
    @next = data[/next=\w{4}/][5, 4]
    glass = data[/glass=.{#{SIZE}}/][6, 6 + SIZE]
    MAX_Y.times { |y| @x_glass[y] = glass[y * MAX_X, MAX_X] }
    MAX_X.times { |x| MAX_Y.times { |y| @y_glass[x][y] = @x_glass[y][x] } }
    MAX_Y.times { |y| p @x_glass[MAX_Y - 1 - y] }
    #MAX_X.times { |x| p @y_glass[x] }
  end

  # This method should return string like left=0, right=0, rotate=0, drop'
  def make_step
    case @figure
    when :O then step_o
    when :I then step_i
    when :L then step_l
    when :J then step_j
    when :S then step_s
    when :Z then step_z
    when :T then step_t
    end
  end

  private

  #
  # Find lowest level for block with given width
  #
  def find_lowest(width)
    y = MAX_Y - 1
    x = 0
    # process every column and find first free cell
    y_cur = @y_glass.map do |column|
      i = column.reverse.index('*')
      i ? MAX_Y - i : 0
    end
    # find continuous line with 'width' number of free cells
    (MAX_X - width + 1).times do |x_cur|
      if y_cur[x_cur..x_cur + width - 1].uniq.size == 1 && y_cur[x_cur] < y
        y = y_cur[x_cur]
        x = x_cur
      end
    end
    # find not continuous line with 'width' number of free cells
    (MAX_X - width + 1).times do |x_cur|
      y_max = y_cur[x_cur..x_cur + width - 1].max
      if y_max < y - 2
        y = y_max
        x = x_cur
      end
    end

    # @x_glass.each_with_index do |row, y|
    #   (MAX_X)
    #   i = row.index(' ' * width)
    #   next if i.nil?
    #   next if @y_glass[i][y...MAX_Y].index('*')
    # end
    [x, y]
  end

  #
  # Forming result string
  #
  def result_string(x, rotate = 0)
    str = ''
    if x < @x
      str << "left=#{@x - x}, "
    elsif x > @x
      str << "right=#{x - @x}, "
    end
    str << "rotate=#{rotate}, " if rotate > 0
    str << 'drop'
  end

  # Process O block
  #
  #   **
  #   **
  #
  def step_o
    x, y = find_lowest(2)
    result_string(x)
  end

  # Process I block
  #
  #   *
  #   *
  #   *
  #   *
  #
  def step_i

    # Process vertical I block
    x_vert, y_vert = find_lowest(1)

    # Process horizontal I block
    x_horiz, y_horiz = find_lowest(4)

    if y_vert < y_horiz
      result_string(x_vert)
    else
      result_string(x_horiz + 2, 1)
    end
  end

  # Process L block
  #
  #   *
  #   *
  #   **
  #
  def step_l

    # Process rotate = 0
    x_1, y_1 = find_lowest(2)

    # Process rotate = 3
    x_2, y_2 = find_lowest(3)

    if y_1 < y_2
      result_string(x_1)
    else
      result_string(x_2 + 1, 3)
    end
  end

  # Process J block
  #
  #     *
  #     *
  #    **
  #
  def step_j

    # Process rotate = 0
    x_1, y_1 = find_lowest(2)

    # Process rotate = 1
    x_2, y_2 = find_lowest(3)

    if y_1 < y_2
      result_string(x_1 + 1)
    else
      result_string(x_2 + 1, 1)
    end
  end

  # Process S block
  #
  #    **
  #   **
  #
  def step_s
    x, y = find_lowest(2)
    result_string(x + 1)
  end

  # Process Z block
  #
  #    **
  #     **
  #
  def step_z
    x, y = find_lowest(2)
    result_string(x)
  end

  # Process T block
  #
  #    *
  #   ***
  #
  def step_t
    x, y = find_lowest(3)
    result_string(x + 1)
  end
end
```

Sample video is available on [RecordIt][RecordIt].

Source files are available on [GitHub][GitHub].

[Server]: {{ site.url }}/data/tetris-servers.zip
[RecordIt]: http://recordit.co/e5P3wznF32
[GitHub]: https://github.com/dmlaziuk/codenjoy_client/tree/master/ruby
