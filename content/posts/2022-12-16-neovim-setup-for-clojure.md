---
title: "Neovim Setup for Clojure"
date: 2022-12-16T17:07:15+01:00
tags: ["clojure", "neovim"]
---

# Neovim Setup for Clojure

I am a big fan of Neovim so I will be altering my setup to make it easy and convenient 
for me to program in Clojure.

## LSP 

For my LSP I am using the built in neovim lsp and [williamboman/nvim-lsp-installer](https://github.com/williamboman/nvim-lsp-installer)
to install them. For Clojure I am using [clojure-lsp](https://clojure-lsp.io/).

```lua
-- In lsp-installer.lua
local status_ok, lsp_installer = pcall(require, "nvim-lsp-installer")
if not status_ok then
	return
end
lsp_installer.setup {}

local o = require('user.lsp.handlers').on_attach
local c = require('user.lsp.handlers').capabilities

local lspconfig = require('lspconfig')

lspconfig.clojure_lsp.setup {
    on_attach = o
    capabilites = c
  }

```

## Plugins

Plugins that I find useful

- [Olical/conjure](https://github.com/Olical/conjure)
- [tpope/vim-dispatch](https://github.com/tpope/vim-dispatch)
- [radenling/vim-dispath-neovim](https://github.com/radenling/vim-dispatch-neovim)
- [clojure-vim/vim-jack-in](https://github.com/clojure-vim/vim-jack-in)

Including that in my Neovim config that uses Packer looks like this
```lua
return packer.startup(function(use)
  -- ...

  use "Olical/conjure"
  use "tpope/vim-dispatch"
  use "radenling/vim-dispatch-neovim"
  use "clojure-vim/vim-jack-in"

  -- ...
)
```

### Conjure

With conjure I can evaluate Clojure code in Neovim by using handy shortcuts.
It works by evaluating Clojure code in spawned nREPL sever. This comes pretty handy
because working with Clojure involves REPL a lot.
Here is the basic overview of conjure keybinds

```
<localleader>ls   -  Open log buffer in horizontal split window. 
<localleader>lv   -  Open log buffer in vertical split window.
<localleader>E    -  Evaluate visual mode selection.
<localleader>ee   -  Evaluate the form under the cursor.
<localleader>ece  -  Evaluate the form under the cursor and display the result as a comment.
<localleader>er   -  Evaluate the root form under the cursor.
<localleader>ecr  -  Evaluate the root form under the cursor and display the result as a comment.

```

### Vim Dispatch and Vim Jack In

Vim Dispatch is a tool used to spawn asynchronous tasks to build or test code in the background.
I will be using it in conjuction with *Vim Jack In* to provide a REPL inside my Neovim setup.
The only commands for working with Clojure are:
```
:Clj [args]  - for working with clj
:Lein [args] - for working with lein
```

I will primarily be using Leiningen for managing my Clojure projects.

## Workflow

Example workflow for working with Clojure projects in Neovim would include:
- setting up Conjure and Vim Jack In by executing :Lein
- using Conjure to execute my code in REPL


