---
layout: post-with-comments
title: Smarter Chromium GN in Vim with gn-language-server
category:
- dyk
tags:
- chromium
---
GN Language Server for Chromium development was announced on [chromium-dev][1].
It's very easy to install in VSCode, NeoVim or Emacs. But how can you configure
it with classic Vim + YCM?

## Setup

First, install the language server with Cargo.
```bash
cargo install --locked gn-language-server
```
Then, add this to your vimrc.
```bash
let g:ycm_language_server = [
      \ {
      \   'name': 'gn',
      \   'cmdline': [ 'gn-language-server' ],
      \   'filetypes': [ 'gn' ],
      \ }
  \ ]
```

That easy, right?

## What's Working

### Hover Documentation
![hover]({% asset_path hover.gif %})
### Go To Imports
![jump_import]({% asset_path jump_import.gif %})
### Go To Dependencies
![jump_deps]({% asset_path jump_deps.gif %})

## Current Limitations
The following features are not working yet. They may need more configuration or
further work:

### Code Folding
Classic Vim and YCM don't support LSP-based folding, and I'm not a big fan of
that feature anyway. But you can configure another plugin that supports
LSP-based folding, or simply rely on indent-based folding.

### Go To Definition
When I try to go to the definition of `template`, I get an error `KeyError:
'uri'`. I'm not sure whether this is caused by my local configuration, but it
needs further investigation.
![go_def_error]({% asset_path go_def_error.gif %})

[1]: https://groups.google.com/a/chromium.org/g/chromium-dev/c/uTa5mrlvbvw
