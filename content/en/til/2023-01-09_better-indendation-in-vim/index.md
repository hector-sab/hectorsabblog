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

Once the text has been indented, the visual mode is disabled and all the selection is lost. This is particularly annoying for occasions when trying to indent several times one same block of code. To overcome this, you can add the following lines to your init.lua ( `~/.confings/nvim/init.lua`). In the video below we can see that the visual mode stays active after indenting, giving us the possibility to keep indenting at our will.

```lua
vim.keymap.set("v", "<", "<gv")
vim.keymap.set("v", ">", ">gv")
```

{{< video src="vim-better-indent.webm" type="video/webm" preload="auto" >}}

# Alternatives

As correctly pointed out by [Jamie](https://mastodon.online/@suprjami@fosstodon.org) and [Marcel Bischoff](https://mastodon.online/@hrbf@mastodon.social) on [mastodon](https://mastodon.online/@hector_sab/109664369612200712), there are two other options for indenting that are more vimistic than this remap. Choose whatever fits you best :)

## Specify number of indentations

Whenever you know exactly how many indentations your block needs you can specify it right before pressing `>` or `<`. In the video below you can appreciate how `3>` will indent 3 times the selected block.

{{< video src="vim-indent-motion-repeat.webm" type="video/webm" preload="auto" >}}

I personally don't tend to use this one that often as it takes me more time to think about how many indentations I need that just indent until it makes sense visually.

## Repeat previous motion

You can also use the `.` (dot) to repeat the last change. Which in our case is the indention of a selected block of text. Then you can indent first with `>`, continue indenting with `.`, and in case you need to unindent you can use `u` to undo the indentation.

{{< video src="vim-indent-dot-repeat.webm" type="video/webm" preload="auto" >}}

If I had to put the effor on replacing my muscle memory for something more vimistic, this would be it.
