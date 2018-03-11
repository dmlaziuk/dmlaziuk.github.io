---
title:  Craft Academy
date:   2018-01-09
header:
  teaser: /images/craftacademy.png
toc: true
toc_label: Index
toc_icon: hand-point-right
tags:
  - Linux
  - Git
  - Ruby
  - JavaScript
---

Interesting resource for (self-)education.

[![CraftAcademy][CALogo]{: .align-center}][CA]

## Books

I used this resource during preparation for interview, also for self-education.
There are several Craft Academy books available at GitBook:

1. [Learn To Code Workshop][CAWorkshop]
2. [Coding as a Craft Bootcamp Prep][CAPrepCraft]
3. [Coding as a Craft][CACraft]
4. [Coding as a Craft 2.0][CACraft2]

Code Academy [GitHub link][CAGitHub].

I've learned lots of techniques form that courses:

## Links

### Command prompt with git status

```
$ brew install bash-git-prompt
```

```bash
# ~/.bash_profile
if [ -f "$(brew --prefix)/opt/bash-git-prompt/share/gitprompt.sh" ]; then
    source "$(brew --prefix)/opt/bash-git-prompt/share/gitprompt.sh"
fi
```

### Git usage

`.gitconfig`:

```
[user]
  name = dmlaziuk
  email = dm.laziuk@gmail.com
[alias]
    a = !git add . && git status
   aa = !git add . && git add -u . && git status
   ac = !git add . && git commit
  acm = !git add . && git commit -m
alias = !git config --list | grep 'alias\.' | sed 's/alias\.\([^=]*\)=\(.*\)/\1\ => \2/' | sort
   au = !git add -u . && git status
   br = branch
    c = commit
   ca = commit --amend
   cm = commit -m
   co = checkout
    d = diff
 dump = cat-file -p
 hist = log --pretty=format:"%h %ad | %s%d [%an]" --graph --date=short
    l = log --graph --all --pretty=format:'%C(yellow)%h%C(cyan)%d%Creset %s %C(white)- %an, %ar%Creset'
   lg = log --color --graph --pretty=format:'%C(bold white)%h%Creset -%C(bold green)%d%Creset %s %C(bold green)(%cr)%Creset %C(bold blue)<%an>%Creset' --abbrev-commit --date=relative
   ll = log --stat --abbrev-commit
  llg = log --color --graph --pretty=format:'%C(bold white)%H %d%Creset%n%s%n%+b%C(bold blue)%an <%ae>%Creset %C(bold green)%cr (%ci)' --abbrev-commit
master = checkout master
    s = status
spull = svn rebase
spush = svn dcommit
 type = cat-file -t

[core]
  editor = atom --wait
[filter "lfs"]
  clean = git-lfs clean -- %f
  smudge = git-lfs smudge -- %f
  process = git-lfs filter-process
  required = true
```

- [How to Write a Git Commit Message][GitCommit]

### Linux

[Command Line][CLI]

Key/Command|Description
----------:|:----------
Ctrl + A   |Go to the beginning of the line you are currently typing on
Ctrl + E   |Go to the end of the line you are currently typing on
Ctrl + L   |Clears the Screen
Command + K|Clears the Screen
Ctrl + U   |Clears the line before the cursor position. If you are at the end of the line, clears the entire line.
Ctrl + H   |Same as backspace
Ctrl + R   |Lets you search through previously used commands
Ctrl + C   |Kill whatever you are running
Ctrl + D   |Exit the current shell
Ctrl + Z   |Puts whatever you are running into a suspended background process. fg restores it.
Ctrl + W   |Delete the word before the cursor
Ctrl + K   |Clear the line after the cursor
Ctrl + T   |Swap the last two characters before the cursor
Ctrl + F   |Move cursor one character forward
Ctrl + B   |Move cursor one character backward
Esc + F    |Move cursor one word forward
Esc + B    |Move cursor one word backward
Esc + T    |Swap the last two words before the cursor
Tab        |Auto-complete files and folder names

### JavaScript

- [javascript basics][JS]
- [jquery basics][jquery]

### Other resources

- [GitHub Pages][GitHubPages]
- [Atom][Atom]
- motivated me to use RSpec for a primary development process (TDD)
- use [Jasmine][Jasmine] as a Behaviour Driven Development framework for testing JavaScript code.

[CA]: http://craftacademy.se/
[CALogo]: {{ "/images/craftacademy_logo.png" | absolute_url }}
[CAGitHub]: https://github.com/CraftAcademy
[CAPrepCraft]: https://www.gitbook.com/book/craftacademy/caa_precourse/details
[CACraft]: https://www.gitbook.com/book/craftacademy/coding-as-a-craft/details
[CACraft2]: https://www.gitbook.com/book/craftacademy/coding-as-a-craft-2-0/details
[CAWorkshop]: https://www.gitbook.com/book/craftacademy/workshop/details
[Atom]: https://atom.io
[GitHubPages]: https://help.github.com/articles/configuring-a-publishing-source-for-github-pages/
[jquery]: https://www.codecademy.com/learn/learn-jquery
[JS]: https://www.codecademy.com/learn/learn-javascript
[CLI]: https://www.codecademy.com/learn/learn-the-command-line
[GitCommit]: https://chris.beams.io/posts/git-commit/
[Jasmine]: https://jasmine.github.io
