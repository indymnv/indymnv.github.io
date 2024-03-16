+++
title = "set up NeoVim + Tmux for a Data Science Workflow with Julia" 
tags = ["python", "neovim", "tmux", "julia", "rstats", "tooling"] 
description = "Notes to start walking with Neovim and Tmux in the Data Science World" 
rss_title = "set up NeoVim + Tmux for a Data Science Workflow with Julia"
rss_description = "Notes to start walking with Neovim and Tmux in the Data Science World"
rss_pubdate = Date(2023, 07, 4) 
+++

# set up NeoVim + Tmux for a Data Science Workflow with Julia


**Date:** 2023-07-04

**Summary:** Notes to start walking with Neovim and Tmux in the Data Science World

**tags:** #Python #Julia #rstats #tmux #neovim #tooling 

~~~
<head>
 <img class="center" src="/assets/love-neovim.jpeg" width="900" height="400">
<meta property="og:image" content="/assets/love-neovim.jpeg" />
</head>
~~~

# Table of Contents
\toc

## Introduction
In this post, I will provide some notes on getting started with Neovim for a Data Science Workflow. This setup is not strictly related to Julia and can also be used with Python and R. The idea is to work with a double panel structure, where one side contains your code and the other side has the REPL, which receives the snippets of code you send from the code side.

In this blog, I will mention the things you can add to make it comfortable for data analysis or more serious development. With this typical kick starter in Neovim and Tmux, I will explain some changes and new packages that are important for this purpose. Finally, I will dive into the details that still need improvement.

## Why start using Neovim and Tmux? My motivations

I must say, I like notebooks. I used them extensively in my first job in analytics, and they really helped me dive into the problem and experiment with different use cases. I have also worked with VSCode, although I am not a big fan of it, it has helped me in some specific use cases where a more "software engineering" perspective is needed.

With that in mind, when I read the book "Approaching Almost Any Machine Learning Problem" by Abhishek Thakur, and then watched the controversial and yet funny conference by Joel Grus on why he [dislikes notebooks](https://www.youtube.com/watch?v=7jiPeIFXb6U&ab_channel=O%27Reilly), I started to think deeply about the perspective of writing software that follows good practices, is expressive, and still easy to prototype. Unfortunately, I think Grus is right about it. Working with notebooks can lead to some weird behaviors, like running cells in different positions, populating your analysis with too much unnecessary information and plots, not creating abstractions when needed, and having issues with reproducibility. On the other hand, the script perspective didn't help me with fast iteration when I needed quick answers to simple questions.

When I moved to Julia, I realized that the REPL is in another level, and I understood that an important part of this community uses (Neo)Vim or Emacs for development. I was curious about using these tools for data science projects. Although there are not many articles about it, and the community using Vim/Emacs is quite small compared to other options, I found it pretty cool because it is minimalistic (though you can customize it extensively), fast, and promises to increase productivity after mastering the Vim keybindings (in 10 years).

What I realized is that you can still prototype like notebooks (but with a perspective closer to RStudio) with one pane for your code and another pane with your REPL open. Then, you can start transforming your code to make it look like serious software, all within one window without moving all the .ipynb file contents to another .py or .jl file. I found this workflow more enjoyable.

## Instalation process required

First of all, make sure to install Neovim and Tmux. There are plenty of tutorials out there on this topic, so I won't go into the details here.

The important thing is to create an init.lua file. If you don't want to install everything one by one, I recommend following the  [kick starter](https://github.com/nvim-lua/kickstart.nvim), provided by nvim-lua/kickstart.nvim. It provides the basic tools for working with Neovim, including a package manager, Treesitter, LSP integration, etc. This kickstarter uses LazyVim, which should be faster and doesn't require frequent updates like PackerNvim. Just create the init.lua file, copy and paste all the content into that file, save it, and quit. When you open it again, it should start installing or upgrading everything.

Also you want to create a Tmux config, the default config in Tmux is already ok for working with Data Science or the Julia experience, but anyway you would want to edit a file for setting a colorscheme or edit some shortcuts, to create just add `~/.tmux.conf` 

You also want to create a Tmux config file. The default config in Tmux is already suitable for working with Data Science or the Julia experience, but you may want to edit it to set a colorscheme or modify some shortcuts. Just add `~/.tmux.conf` to create the file.

## Editing the init.lua and tmux.config

Here are the things required for editing the init.lua file:

1. Add Julia in the init.lua for [treesitter](https://github.com/nvim-lua/kickstart.nvim/blob/master/init.lua#L309) and also consider to add the julials = {} for [local servers](https://github.com/nvim-lua/kickstart.nvim/blob/master/init.lua#L427)
2. Make sure to add the sysimage for the languageserver. in this [discussion](https://discourse.julialang.org/t/neovim-languageserver-jl/37286/83?page=5), they summarize the procedures well, Follow the instructions and add the snippet of code to your init.lua 

```julia
-- Run Julia LSP
require'lspconfig'.julials.setup{
    on_new_config = function(new_config, _)
        local julia = vim.fn.expand("~/.julia/environments/nvim-lspconfig/bin/julia")
        if require'lspconfig'.util.path.is_file(julia) then
	    -- vim.notify("Hello!")
            new_config.cmd[1] = julia
        end
    end
}
``` 
3. This should be enough for setting up Julia. If you open a Julia file, Neovim should be able to detect the LSP and work with other properties like jump to definition, etc. For R and Python, this should be a bit more straightforward for now (not need step 2).

4. There are other things you want to add, for a data science project, one is [vim-slime](https://github.com/jpalardy/vim-slime), This package is great for sending snippets of code from your file to a Julia REPL. Make sure to install it and add the following code to your init.lua. This will allow you to open a Tmux pane and start interacting with the file. You can modify the snippet below to change your target_pane (if you prefer the REPL on your left side or above, you can change this). The actual shortcut is Ctrl-c + Ctrl-c, which you can modify if you prefer.

```julia
vim.g.slime_target = 'tmux'
-- vim.g.slime_default_config = {"socket_name" = "default", "target_pane" = "{last}"}
vim.g.slime_default_config = {
  -- Lua doesn't have a string split function!
  socket_name = vim.api.nvim_eval('get(split($TMUX, ","), 0)'),
  target_pane = '{top-right}',
}
``` 

5. For Tmux, there are some things you can add. Here is my config, which is really simple. However, I encourage you to find your own taste with Tmux.

```julia

set -g mouse on
set -g history-limit 102400
set -g base-index 1
set -g pane-base-index 1
set -g renumber-windows on

unbind C-b
set -g prefix C-x

# vim key movements between panes
# smart pane switching with awareness of vim splits
bind h select-pane -L
bind j select-pane -D
bind k select-pane -U
bind l select-pane -R
# reloading for now:w

unbind r 
bind r source-file ~/.tmux.conf \; display "Reloaded ~/.tmux.conf"
# plugin
# Initialize TMUX plugin manager (keep this line at the very bottom of tmux.conf)

set -g @plugin 'egel/tmux-gruvbox'
set -g @tmux-gruvbox 'dark' # or 'light'
run '~/.tmux/plugins/tpm/tpm'

``` 
For this purpose, I have considered the following in my Tmux config: activate the mouse, increase the history limit in the panes (this is necessary because the default limit in Tmux is quite constrained), count from 1 with panes, change the prefix (I found it easier to use Ctrl-x rather than Ctrl-b), and add the keybindings to move between paneslike vim. The r shortcut is used to restart the config file when you add or modify features, so you can use prefix + r to apply your changes. Finally, in Tmux, you can use a package manager called TPM. Make sure to added in your config file.

## Some challenges for improving the workflow

So far, the workflow with Neovim and Tmux has been set up nicely. However, there are some areas that can be improved. One of them is the visualization aspect. As a data scientist, you need to constantly iterate and visualize your data. If you want to have a deep understanding of your dataset and generate plenty of visualizations, the current setup may not be the best. However, in Julia, you can easily switch to Pluto to display all the figures you want. One thing I have tried is to constantly display those plots you are working on inside the terminal. One way to do this is by using [unicodeplots](https://github.com/JuliaPlots/UnicodePlots.jl). If you like working with Plots.jl, you can change your backend from gr() to unicodeplots(). In my opinion, the quality of the visualization may not be the best, but it allows for instant plots in your terminal without the need for third-party software. For fast iteration, it is good enough.

Another important point to consider is maintaining consistency in the workflow between Julia code and the Julia REPL. Currently, I have a workflow with Vim and my code, but the REPL follows a different logic. This is where the aforementioned repository could potentially help, as it aims to bridge the gap and maintain homogeneity between the two panels. Integrating Vim keybindings into the [REPL](https://github.com/caleb-allen/VimBindings.jl) would provide a seamless experience, allowing for a smoother transition and enhancing the overall workflow. It is definitely an area I look forward to exploring in the future to further improve my development process.

## Conclusions

In this blog, I have explained how to set up Neovim and Tmux with Julia (or any other data science programming language). This setup provides a minimalist perspective. For people who like to have a variety of tools at hand, it may feel a bit lacking. However, if you are someone who is looking for a lightweight tool, minimalistic design, and enjoys working within the terminal, I highly recommend giving it a try.



~~~
<div>
{{insert webring.html}}
</div>
~~~
