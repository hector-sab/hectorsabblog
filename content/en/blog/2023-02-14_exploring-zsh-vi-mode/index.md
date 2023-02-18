---
title: Exploring zsh vi mode
date: 2023-02-14T19:00:20Z
draft: false
---

# Exploring zsh vi mode

One thing that I always wish I had when using my shell is a way of moving fast wherever I need or deleting words in a breeze. Sure, you can use `Ctrl+<Left/Right Arrow>` to jump back and forth words or keep pressing backspace/delete to get rid of undesired characters. But there must be a better way, right? (Plus, a few weeks ago I got a mac and this didn't work because the binding was already used by the system to switch virtual desktops and I couldn't be bothered to change it).

The answer is yes, there's a better way of navigating your shell. In this blog, I'll try to show you a few of the capabilities that zsh offers in its vi mode and how to set them up.

DISCLAIMER: Zsh comes with the emacs shortcuts by default. If you are not comfortable and not interested in learning those based on vim, feel free to stick with the emac ones. Knowing either should be enough to make your life and your craft a bit smoother.

# What can the vi mode do?

If you are familiar with vim, what you get when you enable the vi mode is access to the insert, normal and visual modes, plus bindings that are close to the motions we know and love from vim.

For those unfamiliar with vim, the text editor allows users to control everything through their keyboard. To enable a user to do all it needs to do the text editor needs to be able to allocate multiple behaviors to the same keys based on the context in which it's pressed. Sometimes we are focused on writing text (insert mode), others we want to move around the text (normal mode), and others we just want to select it (visual mode) to paste it elsewhere.

The zsh vi mode makes it possible to get an experience similar to vim but in our shell where we are capable of changing modes and performing motions as if we were in the text editor without actually needing to open it. You can press `ESC` and `i`/`a` to switch between the normal and insert modes, do `CTRL+h`/`CTRL+w`/`CTRL+u` to delete the character/word/line to the left of the cursor's current position, `F+<character>` to find the `character` backward in the line, and more.

{{< video src="zsh-vim-mode.webm" caption="Using zsh with vi mode enabled." type="video/webm" preload="auto" >}}

# How to set it up?

To enable the vi mode you need to execute the following line or add it to your `$HOME/.zshrc`.

```bash
bindkey -v
```

From this point forward you are all set! Now you are in the insert mode. If you added it to your zshrc do `source $HOME/.zshrc` so it takes effect in your current terminal, or open a new one. Sometimes this is all you need, but there are a few modifications explained next that I like to do to get an even greater experience.

# Improved vi mode

NOTE: If you want to know more about these commands you can [check the documentation (Zsh Line Editor)](https://zsh.sourceforge.io/Doc/Release/Zsh-Line-Editor.html).

## Fast transition to normal mode

When you are in insert mode and press `ESC` to enable normal mode you will notice that it takes a moment to transition into that mode. To speed up things, you need the following line to reduce the timeout.

```bash
export KEYTIMEOUT=1
```

The videos below show the effects of using this env var. It is worth noting that the cursor shown has been modified to show a bean when in insert mode and a block in normal mode; the default behavior is to only show the block. Down below you can find how to customize your cursor if you want.

{{< video src="zsh-keytimeout-slow.webm" caption="No KEYTIMEOUT set. Slow transition between insert and normal mode." type="video/webm" preload="auto" >}}
{{< video src="zsh-keytimeout-fast.webm" caption="KEYTIMEOUT set to 1. Fast transition between insert and normal mode." type="video/webm" preload="auto" >}}

## Backspace

Imagine you are in your terminal in insert mode typing some command and you decide to go into normal mode to move your cursor one word back, you press `i` to change into insert mode and decide to delete a few characters that are at the left of your cursor's current position by pressing the backspace key. In any other situation, this should remove the characters you want, but not here; that will do nothing. `X` (`SHIFT+x`) in normal mode is the default key to delete characters to the left, though this is not the natural behavior (almost) anyone would expect.

{{< video src="zsh-backspace-default.webm" caption="Default behavior of backspace once normal mode has been activated." type="video/webm" preload="auto" >}}

To get backspace working every time that insert mode is enabled, the following line is required.

```bash
bindkey '^?' backward-delete-char
```

Note that the character `^` normally represents the key `CTRL`, but in this case `^?` represents the escape code sequence used by my keyboard backspace key. If you are wondering if pressing `CTRL+?` would also delete the character, the answer is yes.

## Delete

If you move your cursor to the left after writing something (without deleting anything) and try to delete the characters to the right of the cursor's current position, your cursor will move to the right changing a few characters from lower to upper case (or vice versa).

{{< video src="zsh-delete-not-working.webm" caption="Default behaviour of the delete key." type="video/webm" preload="auto" >}}

To fix this we need the following commands.

```bash
bindkey '^[[3~' delete-char # Enabled in insert mode
bindkey -a '^[[3~' delete-char # Enabled in normal mode
```

Note that the flag `-a` specifies that the binding should be available in the normal mode. If you don't feel like having the Delete key enabled in normal mode, just remove the second line from your rc and all should be good.

## Search history

In vim, you can use press `/` or `?` in normal mode to search words in your document either forward or backward relative to your cursor's current position. In zsh, this is the same but it works on your commands history and in reverse order. If you press `/+<command>+ENTER`, zsh will search your history for anything containing `<command>`, and pressing `n` will iterate from newest to oldest. Pressing `N` will iterate in reverse.

{{< video src="zsh-search-default.webm" caption="Default behavior when searching the commands history. Note that pressing / shows ? in the search and vice versa. Also note how searching by pressing the ? key won't result in anything." type="video/webm" preload="auto" >}}

I have found the default behavior of `?` in vi zsh mode a bit useless as it will search forward from the history's newest entry, and since zsh is unable to see the future and know what are our future commands, it will show nothing. For this reason, I have bound an incremental backward search to it for now.

```bash
bindkey -a '?' history-incremental-search-backwards
```

## Deleting characters shortcuts

In vim insert mode it's possible to delete characters in a few ways. For example, you can use `CTRL+h` to delete one character to the left, `CTRL+w` to delete all the characters on a word between the beginning of the word to the cursor's current position, or `CTRL+u` to delete all the characters between the beginning of the line to the cursor's current position. However, this is not enabled by default in zsh vi mode in some circumstances.

{{< video src="zsh-delete-characters-default.webm" caption="Default behavior of different vim character deletion commands." type="video/webm" preload="auto" >}}

The following lines make this behavior available in zsh.

```bash
bindkey '^h' backward-delete-char
bindkey '^w' backward-kill-word
bindkey '^u' backward-kill-line
```

{{< video src="zsh-delete-characters-improved.webm" caption="Improved behavior for character deletion commands." type="video/webm" preload="auto" >}}

## Edit command in vim

There are times when I write commands that are too long (looking at you, curl) and wish vim was open to edit them. Fortunately, we can set zsh to send the current command in the prompt to vim, continue editing there, and then get it back with ease. The command below sets `CTRL+e` on `insert` and `normal` modes to do so.

```bash
autoload edit-command-line; zle -N edit-command-line
bindkey '^e' edit-command-line   # Enabled in insert mode
bindkey -a '^e' edit-command-line  # Enabled in normal mode
```

{{< video src="zsh-edit-in-vim.webm" caption="Continue editing the shell command in vim." type="video/webm" preload="auto" >}}

NOTE: This will use whatever you have set for the env var `EDITOR`. Mine is set to `nvim`.

## Cursor

One last thing that we can do to improve the vi mode is to make the cursor style change based on if we are in `insert` or `normal` mode. That can be achieved by adding the snippet below, and you can specify if the cursor should be solid or blinking.

{{< video src="zsh-cursor-solid.webm" caption="Solid cursor for both the insert and normal mode." type="video/webm" preload="auto" >}}

{{< video src="zsh-cursor-blink.webm" caption="Blink cursor for both the insert and normal mode." type="video/webm" preload="auto" >}}

```bash
# Change cursor shape for different vi modes.
# - Taken from https://github.com/LukeSmithxyz/voidrice/blob/e0331ad0e76dcbcfcc08cb991d9e7f99382517db/.config/zsh/.zshrc
# - Information on how to change the cursor style from https://vim.fandom.com/wiki/Change_cursor_shape_in_different_modes
#
# Cursor styles:
# 1 -> blinking block
# 2 -> solid block
# 3 -> blinking underscore
# 4 -> solid underscore
# 5 -> blinking vertical bar
# 6 -> solid vertical bar
#
# To change the cursor style we need to modify `\e[# q` where `#` is the cursor style.
function zle-keymap-select () {
  case $KEYMAP in
    vicmd) echo -ne '\e[2 q';;   # block
    viins|main) echo -ne '\e[6 q';; # beam
  esac
}
zle -N zle-keymap-select
zle-line-init() {
  zle -K viins # initiate `vi insert` as keymap (can be removed if `bindkey -V` has been set elsewhere)
  echo -ne "\e[6 q"
}
zle -N zle-line-init
echo -ne '\e[6 q' # Use beam shape cursor on startup.
preexec() { echo -ne '\e[6 q' ;} # Use beam shape cursor for each new prompt.
```
