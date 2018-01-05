---
title:  "First homework"
date:   2017-10-09
---
The first homework was challenging. It was about traversing binary trees.

The task was to parse data stored in binary tree and visualise the tree.

For example given array:
```ruby
arr = [1 ,[[2 ,[4 , 5 ]],[3,[6,7]]]]
```
You have to output something like this:
```
      1
     /  \
    2    3
   / \  / \
  4  5 6  7
```
There were 3 levels:
1. Read trees from zip-file, select tree with given name `$ NAME=mel ruby trees.rb` and print it out to console.
2. If not given a NAME print out all trees in ascending order.
3. Same as previous, but warn if depth of the tree greater then 5 or summ of all leaves is greater then 5000.

The code for all 3 levels is here:
```ruby
require 'zip'
require 'json'
# require_relative 'tree_hash'

class TreeHash
  attr_reader :depth, :sum

  def initialize(arr = [])
    @depth = 0
    @sum = 0
    @hash = Hash.new([0, 0])
    parse(arr, 0, [0])
  end

  def to_s
    str = []
    # initialize each string with 4 spaces for 2**@depth elements
    (0..2 * @depth).each { |i| str[i] = '    ' * 2**@depth }
    # print branches /\
    (0...@depth).each do |y|
      (0...2**y).each do |x|
        # curr_str -- every odd line in str[]
        curr_str = str[2 * y + 1]
        # curr_x -- offset from the beginning of line
        curr_x = 2**(@depth + 1 - y) + x * 2**(@depth + 2 - y) + 1
        curr_str[curr_x, 2] = '/\\'
      end
    end
    # print nodes with values
    @hash.each do |yx, value|
      # (y, x) -- coordinates of node
      y = yx[0]
      x = yx[1]
      # curr_str -- every even line in str[]
      curr_str = str[2 * y]
      curr_x = 2**(@depth + 1 - y) + x * 2**(@depth + 2 - y)
      curr_str[curr_x, 4] = format('%3d ', value)
    end
    str.join("\n")
  end

  # parse -- recursively parse all nodes of the tree, stored in arr[]
  #          and form hash table @hash = { [y, x] = value }
  #            y -- level of the tree (0 -- root)
  #            x -- x position in tree (0...2**y)
  #
  # arguments:
  #   arr -- array representing tree
  #          for example
  #          arr = [1 ,[[2 ,[4 , 5 ]],[3,[6,7]]]]
  #          is for tree:
  #             1
  #            /  \
  #           2    3
  #          / \  / \
  #         4  5 6  7
  #
  #   lvl -- current level
  #          start parse from lvl = 0 (root)
  #
  #   x -- array of current x for current level y
  #       x[y] = current x
  #
  def parse(arr, lvl, x)
    if lvl > @depth
      @depth = lvl
      x << 0
    end
    left = arr[0]
    right = arr[1]
    if left.is_a?(Integer) && right.is_a?(Array)
      # node [Integer, Array]
      @hash[[lvl, x[lvl]]] = left
      @sum += left
      x[lvl] += 1
      parse(right, lvl + 1, x)
    elsif left.is_a?(Array) && right.is_a?(Array)
      # node [Array, Array]
      parse(left, lvl, x)
      parse(right, lvl, x)
    else
      # ending branches
      if left
        @hash[[lvl, x[lvl]]] = left
        x[lvl] += 1
        @sum += left
      end
      if right
        @hash[[lvl, x[lvl]]] = right
        x[lvl] += 1
        @sum += right
      end
    end
  end
end

CUT_SUM = 5000
TRIM_DEPTH = 5

# environment variable NAME='tree_name'
name = ENV['NAME']

if name
  Zip::File.open('trees.zip') do |zip_file|
    if zip_file.find_entry("trees/#{name}.tree")
      arr = JSON.parse(zip_file.read("trees/#{name}.tree"))
      tree = TreeHash.new(arr)
      puts name + '.tree'
      puts tree
      if tree.sum > CUT_SUM
        puts 'Cut.'
      elsif tree.depth > TRIM_DEPTH
        puts 'Trim.'
      else
        puts 'Leave.'
      end
    else
      puts 'There\'s no such tree in our forest.'
    end
  end
else
  trees = 0
  trim = 0
  cut = 0
  leave = 0
  Zip::File.open('trees.zip') do |zip_file|
    zip_file.glob('trees/*.tree').sort_by(&:name).each do |entry|
      arr = JSON.parse(entry.get_input_stream.read)
      tree = TreeHash.new(arr)
      trees += 1
      puts entry.name
      puts tree
      if tree.sum > CUT_SUM
        puts 'Cut.'
        cut += 1
      elsif tree.depth > TRIM_DEPTH
        puts 'Trim.'
        trim += 1
      else
        puts 'Leave.'
        leave += 1
      end
      puts 'Continue? [y/n]'
      q = gets
      break if q[0, 1] == 'n'
    end
    puts 'Proceded ' + trees.to_s + ' trees, which:'
    puts '-- leave ' + leave.to_s
    puts '-- trim ' + trim.to_s
    puts '-- cut ' + cut.to_s
    puts 'Thanks for being in our forest'
  end
end
```

Sample output looks like this:
```
$ ruby trees.rb
trees/alica.tree
                                 30
                                 /\
                  1                              35
                 /\                              /\
          5              19              46              22
         /\              /\              /\              /\
     10      38      10      29      36      44      19      46
     /\      /\      /\      /\      /\      /\      /\      /\
   14  38   7  18  19  16   8   7  19  22   7  15  17  42  16   7
Leave.
Continue? [y/n]
n
Proceded 1 trees, which:
-- leave 1
-- trim 0
-- cut 0
Thanks for being in our forest
```
Sample video:

{% include video id="249877592" provider="vimeo" %}

Source files are available on [GitHub][GitHub].

[GitHub]: https://github.com/dmlaziuk/bsuir-courses/tree/dm-homework-1/2017/DmLaziuk/1
