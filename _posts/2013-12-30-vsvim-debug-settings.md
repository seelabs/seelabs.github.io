---
layout: post
title: "Single Key Mode Debugging In Visual Studio"
description: "Emulating GDB's single key debugging in visual studio"
category: articles
tags: [debugging]
comments: true  
---

This is a quick hack to let you type a single 's', 'n', or 'f' in visual studio
to step into, over, or out of a function - just like [single key mode in
GDB](https://sourceware.org/gdb/onlinedocs/gdb/TUI-Single-Key-Mode.html). It will
be most useful for people who use vim to edit code and only use visual studio for
debugging.

The first release of the See Labs GUI will only runs on windows. However, in
order to use excellent tools like valgrind and the clang sanitizer tools, I do
have a linux port (there is also a client port for the raspberry pi).

When I'm debugging on Linux, I'm usually in GDB's "Single Key Mode". You toggle
this mode by typing C-x s. When in single key mode, typing a single 's' (no CR)
will step into a function, 'n' will step over a function, 'f' will step out of a
function (finish). In visual studio the equivalent keys are F11, F10, and
Shift-F11. There are two reasons I want single key mode instead of the visual
studio defaults:

1. I like to keep my windows and linux environments as similar as possible. For
   example, on windows I use cygwin and emacs (in evil mode).

2. I can't always use my (noisy) full keyboard. My quiet keyboard requires me to
   hit a "Fn" modifier key to get at F7-F12. So to hit shift-F11 to step out of a
   function requires me to hit three keys (Fn-S-F5).

The hack is very simple:

1. Install
   [VsVim](http://visualstudiogallery.msdn.microsoft.com/59ca71b3-a4a3-46ca-8fe1-0e90e3f79329). This
   will install a vim emulator as your text editor. If you don't like vim this
   hack is not for you.

2. VsVim will read your .vimrc file. Put the folowing .vimrc in your "windows"
   home directory (i.e. /User/swd for me). Since I run cygwin, this won't conflict
   with a .vimrc in my cygwin home directory (i.e. /cygwin/home/swd). It should be
   self-explanatory:

<pre>
map \<C-[\> \<ESC\>
nmap s :vsc Debug.StepInto\<CR\>
nmap n :vsc Debug.StepOver\<CR\>
nmap f :vsc Debug.StepOut\<CR\>
nmap b :vsc Debug.ToggleBreakpoint\<CR\>
nmap c :vsc Debug.Start\<CR\>
nmap K :vsc Debug.StopDebugging\<CR\>
nmap p :vsc Debug.QuickWatch\<CR\>
nmap B :vsc Build.BuildSolution\<CR\>
</pre>

# Testing

Bring up a project in visual studio. Type a single 'b' to set a breakpoint. 'n'
to go to the next line (step-over), 's' to step-into, f to step out (finish),
etc.
