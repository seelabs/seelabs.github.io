---
layout: post
title: "Single Key Mode Debugging In Visual Studio"
description: "Emulating GDB's single key debugging in visual studio"
category: articles
tags: [debugging]
comments: true  
---

# Introduction

This is a quick hack to let you type a single 's', 'n', or 'f' in visual studio
to step into, over, or out of a function - just like [single key mode in
GDB](https://sourceware.org/gdb/onlinedocs/gdb/TUI-Single-Key-Mode.html). It will
be most useful for people who use vim to edit code and only use visual studio for
debugging.

The first release of the See Labs GUI will only runs on windows. However, in
order to use excellent tools like valgrind and the clang sanitizer tools, I do
have a linux port (there is also a client port for the raspberry pi).

When I'm debugging on Linux, I'm usually in GDB's "Single Key Mode". When in
single key mode, typing a single 's' (no CR) will step into a function, 'n' will
step over a function, 'f' will step out of a function (finish). In visual studio
the equivalent keys are F11, F10, and Shift-F11. There are two reasons I want
single key mode instead of the visual studio defaults:

1. I like to keep my windows and linux environments as similar as possible. For
   example, on windows I use cygwin and emacs (in evil mode).

2. I can't always use my (noisy) full keyboard. My quiet keyboard requires me to
   hit a "Fn" modifier key to get at F7-F12. So to hit shift-F11 to step out of a
   function requires me to hit three keys (Fn-S-F5).

# Installation

The hack is very simple:

* Install
   [VsVim](http://visualstudiogallery.msdn.microsoft.com/59ca71b3-a4a3-46ca-8fe1-0e90e3f79329). This
   will install a vim emulator as your text editor. If you don't like vim this
   hack is not for you.

* VsVim will read your .vimrc file. Put the folowing .vimrc in your "windows"
   home directory (i.e. /User/swd for me). Since I run cygwin, this won't conflict
   with a .vimrc in my cygwin home directory (i.e. /cygwin/home/swd). It should be
   self-explanatory:

{% highlight html %}
set incsearch
map <C-[> <ESC>
nnoremap B :vsc Build.BuildSolution<CR>
nnoremap <CR> n

nnoremap <C-d> :source /Users/swd/single_key_debug.vimrc<CR>
nnoremap <C-u> :nunmap s<CR>:nunmap n<CR>:nunmap f<CR>:nunmap b<CR>:nunmap c<CR>:nunmap K<CR>:nunmap p<CR>
{% endhighlight %}

* Create a file in your "windows" home directory called single_key_debug.vimrc and put the following commands in it:

{% highlight html %}
nnoremap s :vsc Debug.StepInto<CR>
nnoremap n :vsc Debug.StepOver<CR>
nnoremap f :vsc Debug.StepOut<CR>
nnoremap b :vsc Debug.ToggleBreakpoint<CR>
nnoremap c :vsc Debug.Start<CR>
nnoremap K :vsc Debug.StopDebugging<CR>
nnoremap p :vsc Debug.QuickWatch<CR>
{% endhighlight %}


Notice that the 'n' command for single key debugging conflicts with the usual vim
binding of 'next match'. I have remapped 'carriage return' to work as n usually
does so I can still search while in debug mode.

# Usage

Bring up a project in visual studio. Type Control-d to enter single key debug
mode. Type a single 'b' to set a breakpoint. 'n' to go to the next line
(step-over), 's' to step-into, f to step out (finish), etc. Refer to the
keybinding in single_key_debug.vimrc for reference.

Type Control-u to exit single key debug mode.

# Using u and d to navidate the stack

Unfortunately, there is no easy to get the current stack frame, so implementing
'u' to move up the frame and 'd' to move down the frame is difficult to
implement. However, I implemented this hacky way to do it. The idea is to:

* Create a set to files named 'stack_commands_n.vimrc' where 'n' is the current stack frame.

* Each file binds 'u' to set to the next stack from and source
  'stack_commands_n+1.vimrc' and 'd' to move down and source
  'stack_commands_n-1.vimrc

For example, here are the contents for stack_commands_3.vimrc:

{% highlight html %}
nnoremap u :vsc Debug.SetCurrentStackFrame 4<CR>:source /Users/swd/vsvim_single_key_debug/stack_commands_4.vimrc<CR>
nnoremap d :vsc Debug.SetCurrentStackFrame 2<CR>:source /Users/swd/vsvim_single_key_debug/stack_commands_2.vimrc<CR>
{% endhighlight %}

* I used the following simple python script to auto generate these files for me:

{% highlight python %}
file_template=\
'''
nnoremap u :vsc Debug.SetCurrentStackFrame %(next)d<CR>:source /Users/swd/vsvim_single_key_debug/stack_commands_%(next)d.vimrc<CR>
nnoremap d :vsc Debug.SetCurrentStackFrame %(prev)d<CR>:source /Users/swd/vsvim_single_key_debug/stack_commands_%(prev)d.vimrc<CR>
'''
file_name_template='stack_commands_%(frame)d.vimrc'
for i in range(1,50):
    file_name=file_name_template % {'frame': i}
    with open(file_name,'w') as f:
        if i==1:
            f.write(file_template % {'next':i+1,'prev':i})
        else:
            f.write(file_template % {'next':i+1,'prev':i-1})
{% endhighlight %}

* I also modified to 'c' command to source stack_commands_1.vimrc (i.e. reset the current stack frame). 
{% highlight html %}
nnoremap c :vsc Debug.Start<CR>:source /Users/swd/vsvim_single_key_debug/stack_commands_1.vimrc<CR>
{% endhighlight %}

* I also added the 'r' command to 'reset' the stack commands in case it gets out of sync
{% highlight html %}
nnoremap r :source /Users/swd/vsvim_single_key_debug/stack_commands_1.vimrc<CR>
{% endhighlight %}

* Of course, all the above assumes these files live in /Users/swd/vsvim_single_key_debug 
