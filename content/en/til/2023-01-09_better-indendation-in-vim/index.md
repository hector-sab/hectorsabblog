---
title: Better indentation in vim
date: 2023-01-09T22:37:33Z
draft: false
categories:
- vim
tags:
- vim
- better-vim
---

The are times when you need to indent a few lines at the same time, or just one. (neo)vim comes with key bindings for indenting the lines highlighted in visual mode to the right and left (`>` and `<`). However, they are useful just for one motion as shown in the first video below.

{{< video src="vim-regular-indent.webm" type="video/webm" preload="auto" >}}

Once the text has been indented, the visual mode is disabled all the selection is lost. This is particularly annoying for occasions when trying to indent several times one same block of code. To overcome this, you can add the following lines to your init.lua ( `~/.confings/nvim/init.lua`). In the video below we can see that the visual mode stays active after indenting, giving us the possibility to keep indenting at our will.

```lua
vim.keymap.set("v", "<", "<gv")
vim.keymap.set("v", ">", ">gv")
```

{{< video src="vim-better-indent.webm" type="video/webm" preload="auto" >}}
