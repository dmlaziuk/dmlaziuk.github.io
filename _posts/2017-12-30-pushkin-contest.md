---
title:  "Pushkin contest"
date:   2017-12-30
---
[![Pushkin]({{ "/images/pushkin.gif" | absolute_url }})][Pushkin]

The resulting project in Ruby courses was Pushkin contest.

Again dedicated server sends us questions and our clients have to respond answers.

There are 8 levels. You have to answer 100 questions to jump to another level.

1.
**Question**: One line from any verse.  
**Answer**: The title of the verse.
2.
**Question**: One line from any verse with one word replaced by %WORD%.  
**Answer**: Replaced word.
3.
**Question**: Two lines in succession from any verse with one word replaced by %WORD% on each line.  
**Answer**: Two replaced words separated by comma.
4.
**Question**: Three lines in succession from any verse with one word replaced by %WORD% on each line.  
**Answer**: Three replaced words separated by commas.
5.
**Question**: One line from any verse with one word replaced by any other word from pushkin vocabulary.  
**Answer**: Right word and replaced one separated by comma.
6.
**Question**: One line from any verse with shuffled words order (punctuation removed).  
**Answer**: Original line.
7.
**Question**: One line from any verse with shuffled letterss order (punctuation removed).  
**Answer**: Original line.
8.
**Question**: One line from any verse with shuffled letterss order (punctuation removed) and one letter changed.  
**Answer**: Original line.





Results:
![Pushkin]({{ "/images/pushkin.png" | absolute_url }})

Source files are available on [GitHub][GitHub].

[Pushkin]: http://pushkin.rubyroidlabs.com
[GitHub]: https://github.com/dmlaziuk/pushkin-contest-bot.git
