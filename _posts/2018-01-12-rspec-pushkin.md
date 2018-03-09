---
title:  "RSpec my Pushkin"
date:   2018-01-12
header:
  teaser: /images/rspec.png
toc: true
toc_label: Index
toc_icon: "hand-point-right"
tags:
  - ruby
  - testing
---
Finaly added RSpec to Pushkin Contest Bot.

![RSpec][RSpecPushkin]{: .align-center}

## Objective

If I'd done this before, the results of Pushkin Contest would be much better.
The reason is almost all tests fail when I run each level `ATTEMPTS=100` times.

## Installation

To install add to `Gemfile`:

```
gem 'rspec'
```

Then run:

```
$ bundle install
$ rspec --init
```

## Description

Definition of `Word`:

```ruby
# word_spec.rb
require 'rspec'
require_relative '../lib/word'

describe Word, 'behaviour' do
  let(:test_word) { 'дядя' }
  subject { Word.new(test_word) }
  it 'should save original word' do
    expect(subject.word).to eq(test_word)
  end
  it 'should save hash of the word' do
    expect(subject.word_hash).to eq(test_word.hash)
  end
  it 'should save length of the word' do
    expect(subject.word_length).to eq(test_word.length)
  end
  it 'should save array of hashes of each character' do
    expect(subject.chars_hash_arr).to eq(test_word.scan(/./).map(&:hash))
  end
end
```

Definition of `Line`:

```ruby
# line_spec.rb
require 'rspec'
require_relative '../lib/line'

describe Line, 'behaviour' do
  let(:test_line) { '«Мой дядя самых честных правил,' }
  let(:test_words_arr) { test_line.scan(/[\p{Word}\-]+/) }
  let(:test_line_wo_punctuation) { test_words_arr.join(' ') }
  subject { Line.new(test_line) }
  it 'should save original line' do
    expect(subject.line_orig).to eq(test_line)
  end
  it 'should save line without punctuation' do
    expect(subject.line).to eq(test_line_wo_punctuation)
  end
  it 'should save hash of line without punctuation' do
    expect(subject.line_hash).to eq(test_line_wo_punctuation.hash)
  end
  it 'should save words count' do
    expect(subject.words_count).to eq(test_words_arr.size)
  end
  it 'should save words array' do
    expect(subject.words_arr.map(&:word)).to eq(test_words_arr)
  end
end
```

Definition of `Verse`:

```ruby
# verse_spec.rb
require 'rspec'
require_relative '../lib/verse'

describe Verse, 'behaviour' do
  let(:test_title) { 'Бакуниной' }
  let(:test_text) do
    ['Напрасно воспевать мне ваши именины',
     'При всем усердии послушности моей;',
     'Вы не милее в день святой Екатерины',
     'Затем, что никогда нельзя быть вас милей.']
  end
  subject { Verse.new(test_title, test_text) }
  it 'should save title' do
    expect(subject.title).to eq(test_title)
  end
  it 'should save lines array' do
    expect(subject.lines_arr.map(&:line_orig)).to eq(test_text)
  end
end
```

Definition of `Pushkin`:

```ruby
# pushkin_spec.rb
require 'rspec'
require 'yaml'
require_relative '../lib/pushkin'

ATTEMPTS = 1
my_verses = YAML.load(File.read('lyrics3.yml'))
my_pushkin = Pushkin.new
my_verses.each { |verse| my_pushkin.add(verse[0], verse[1]) }
my_pushkin.init_hash

describe Pushkin, 'behaviour' do
  context 'on start' do
    let(:verses) { my_verses }
    subject { Pushkin.new }
    it 'should be empty' do
      expect(subject.to_s).to eq('')
    end
    it 'should add verses' do
      verse = verses.sample
      title = verse[0]
      text = verse[1]
      subject.add(title, text)
      expect(subject.to_s).to eq("#{title}\n\n#{text.join("\n")}\n\n\n")
    end
    it 'should generate hashes' do
      verse = verses.sample
      title = verse[0]
      text = verse[1]
      subject.add(title, text)
      expect(subject.init_hash).to eq(text.first.scan(/[\p{Word}\-]+/).join(' ').hash)
    end
  end

  context 'on contest' do
    let(:verses) { my_verses }
    subject(:pushkin) { my_pushkin }

    it 'should answer level 1 questions' do
      ATTEMPTS.times do |i|
        verse = verses.sample
        verse_title = verse[0]
        verse_text = verse[1]
        question = verse_text.sample
        right_answer = verse_title
        answer = subject.run_level1(question)
        puts "#{i}:#{question}:#{answer}:#{right_answer}"
        expect(answer).to eq(right_answer)
      end
    end

    it 'should answer level 2 questions' do
      ATTEMPTS.times do |i|
        verse = verses.sample
        verse_text = verse[1]
        line = verse_text.sample
        words = line.scan(/[\p{Word}\-]+/)
        random_number = rand(words.size)
        right_answer = words[random_number]
        words[random_number] = '%WORD%'
        question = words.join(' ')
        answer = subject.run_level2(question)
        puts "#{i}:#{line}:#{question}:#{answer}:#{right_answer}"
        expect(answer).to eq(right_answer)
      end
    end

    it 'should answer level 3 questions' do
      ATTEMPTS.times do |i|
        verse = verses.sample
        verse_text = verse[1]
        line_number = rand(verse_text.size - 2)
        line = []
        right_answer = []
        question = []
        2.times do |j|
          line[j] = verse_text[line_number + j]
          words = line[j].scan(/[\p{Word}\-]+/)
          random_number = rand(words.size)
          right_answer[j] = words[random_number]
          words[random_number] = '%WORD%'
          question[j] = words.join(' ')
        end
        answer = subject.run_level3(question.join("\n"))
        puts "#{i}:#{line}:#{question}:#{answer}:#{right_answer.join(',')}"
        expect(answer).to eq(right_answer.join(','))
      end
    end

    it 'should answer level 4 questions' do
      ATTEMPTS.times do |i|
        verse = verses.sample
        verse_text = verse[1]
        line_number = rand(verse_text.size - 3)
        line = []
        right_answer = []
        question = []
        3.times do |j|
          line[j] = verse_text[line_number + j]
          words = line[j].scan(/[\p{Word}\-]+/)
          random_number = rand(words.size)
          right_answer[j] = words[random_number]
          words[random_number] = '%WORD%'
          question[j] = words.join(' ')
        end
        answer = subject.run_level4(question.join("\n"))
        puts "#{i}:#{line}:#{question}:#{answer}:#{right_answer.join(',')}"
        expect(answer).to eq(right_answer.join(','))
      end
    end

    it 'should answer level 5 questions' do
      ATTEMPTS.times do |i|
        verse = verses.sample
        verse_text = verse[1]
        random_word = verses.sample[1].sample.scan(/[\p{Word}\-]+/).sample
        line = verse_text.sample
        words = line.scan(/[\p{Word}\-]+/)
        random_number = rand(words.size)
        right_word = words[random_number]
        right_answer = "#{right_word},#{random_word}"
        words[random_number] = random_word
        question = words.join(' ')
        answer = subject.run_level5(question)
        puts "#{i}:#{line}:#{question}:#{answer}:#{right_answer}"
        expect(answer).to eq(right_answer)
      end
    end

    it 'should answer level 6 questions' do
      ATTEMPTS.times do |i|
        verse = verses.sample
        verse_text = verse[1]
        line = verse_text.sample
        words = line.scan(/[\p{Word}\-]+/)
        right_answer = line
        question = words.shuffle.join(' ')
        answer = subject.run_level6(question)
        puts "#{i}:#{line}:#{question}:#{answer}:#{right_answer}"
        expect(answer).to eq(right_answer)
      end
    end

    it 'should answer level 7 questions' do
      ATTEMPTS.times do |i|
        verse = verses.sample
        verse_text = verse[1]
        line = verse_text.sample
        words = line.scan(/[\p{Word}\-]+/)
        right_answer = line
        question = words.join(' ').split('').shuffle.join
        answer = subject.run_level7(question)
        puts "#{i}:#{line}:#{question}:#{answer}:#{right_answer}"
        expect(answer).to eq(right_answer)
      end
    end

    it 'should answer level 8 questions' do
      ATTEMPTS.times do |i|
        verse = verses.sample
        verse_text = verse[1]
        line = verse_text.sample
        words = line.scan(/[\p{Word}\-]+/)
        right_answer = line
        question = words.join(' ').split('').shuffle.join
        random_char = verses.sample[1].sample.scan(/[\p{Word}\-]+/).sample.split('').sample
        random_number = rand(question.size)
        question[random_number] = random_char
        answer = subject.run_level8(question)
        puts "#{i}:#{line}:#{question}:#{answer}:#{right_answer}"
        expect(answer).to eq(right_answer)
      end
    end
  end
end
```

## Runing tests

To run tests:

```
$ rspec
```

To pretty format output:

```
$ rspec --format doc
```

[RSpec]: http://rspec.info
[RSpecPushkin]: {{ "/images/rspec-pushkin.png" | absolute_url }}
