---
layout: post
title: "Man vs Vim: Vim commands to survive the wild"
---

Vim is an amazing tool - fast, minimalistic and fully customizable. I am not an emacs guy, I have tried using it but I always find myself coming back to Vim. Maybe it’s just a sheer joy I find with Vim, with using its command and how my fingers are now fully accustomed to it that I can’t adjust myself to emacs.

I had a hard time when I started using Vim. The biggest challenge was to completely forget the mouse and get used to navigating only with the keyboard. When you start navigating with the keyboard (either you use the arrow keys, or like me, you use H-J-K-L) the challenge you then face is how to skip multiple lines or how to jump to a particular line. When you finally learn this and your muscle memory starts to manifest, you face another challenge - how to skip words, how to jump to a particular word, searching, replacing, etc. Vim definitely requires a lot of work to get used to and that is what makes Vim just amazing!

As of now, I use VSCode and Vim as my primary text editors and Jupyter Notebook for data engineering and data science. I primarily use VSCode while developing projects because I like to use mouse and many features of VSCode like code formatting, pep8, jump to function definition, etc come quite handy. I use Vim when I have to write a quick script, work on a server, write scripts quickly, do text searching from log files or just whenever I feel like using Vim (because using Vim is quite fun).

Here are some of the essential Vim commands you should know to get work done

1. `:/word` - search for a string in the current file
2. `:%s/old_word/new_word/g` - replace all occurrences of `old_word` with `new_word`
3. `Ctrl-A` - jump to end of the line in Insert mode
4. `Ctrl-I` - jump to the beginning of the line in Insert mode
5. `gg` - to jump to the beginning of the file
6. `Shift-g` - to jump to the end of the file
7. `dd` - to cut a line
8. `v` - to enter select mode. Used for selecting multiple lines for cut/copy
9. `Shift-i` - to enter visual insert mode
10. To comment multiple lines - Go to the beginning of the line, press `Ctrl-v`, then select the lines using up or down arrow, press `Shift-i` to enter visual insert mode and then enter the comment symbol in the first line and press `Esc`. Wait for a second to see the magic happen
11. `b` - move one word backward
12. `w` - move one word forward
13. `y` - yank i.e copy
14. `p` or `Shift-p` - paste
15. `:!unix_command` - to run shell commands
16. `u` - undo changes
17. `Ctrl-r` - redo changes
18. `X` - delete character in backspace mode

That will suffice if you just want to use Vim only in particular cases or when there is no other option (like in a server). I use `badwolf` as my Vim theme.

Besides these commands, I love using Tmux to help manage multiple terminals.

Here are my custom [vimrc](https://github.com/prabhupant/my-vimrc) and [tmux-conf](https://github.com/prabhupant/my-tmux-conf)
