---
layout: post
title: Tips for using Vim when developing Chromium
category:
- Chromium
- Tips
tags:
- chromium
- vim
- igalia-planet
date: 2023-01-30 11:14 +0900
---
There are lots of powerful IDEs which are broadly used by developers. Vim is a
text editor, but it can be turned into a good IDE with awesome plugins and a
little bit of configuration.

Many people already prefer using Vim to develop software because of its
lightness and availability. I am one of them, and always use Vim to develop
Chromium. However, someone would think that it’s hard to get Vim to reach the
same performance as other IDEs, since Chromium is a very huge project.

For those potential users, this post introduces some tips for using Vim as an
IDE when developing Chromium.

> The context of this document is for Linux users who are used to using Vim.
{: .prompt-info }

## Code Formatting
I strongly recommend using [vim-codefmt][3] for code formatting. It supports
most file types in Chromium including GN (See [GN docs][4]) and autoformatting.
However, I don't like to use autoformatting and just bind `:FormatLines` to
`ctrl-I` for formating a selected visual block.

```sh
map <C-I> :FormatLines<CR>
```

![codefmt]({% asset_path codefmt.gif %})

Another option is using `clang-format` in `depot_tools` by `clang-format.vim` in
[tools/vim][1]. Please check about [tools/vim][1] at the following section.

## tools/vim

You can easily find a bunch of Vim files in [tools/vim][1]. I create a *.vimrc*
file locally in a working directory and load only what I need.

```sh
let chtool_path=getcwd().'/tools'

filetype off
let &rtp.=','.chtool_path.'/vim/mojom'
exec 'source' chtool_path.'/vim/filetypes.vim'
exec 'source' chtool_path.'/vim/ninja-build.vim'
filetype plugin indent on
```

Chromium provides vimscript files for syntax highliting and file detection of
Mojom, which is the IDL for Mojo interfaces (IPC) among Chromium services. And
`ninja-build.vim` allows you compile a file with `ctrl-O` or build the specific
target with `:CrBuild` command. See each Vim files for details.

### Running Tests

[tools/autotest.py][11] is very useful when you run tests of Chromium. As the
description of the script, `autotest.py` finds the appropriate test suits and
builds it, then runs it. `!` is an option for running `autotest.py` inside Vim,
but sometimes it's a hassle to type all parameters. What about to write simple
commands (or functions) with scripts under `tools/vim`?

This is an example script for running a test for the current line. Some codes
are copied from `ninja-build.vim` and imported from `ninja-output.py`.

```sh
pythonx << endpython
import os, vim

def path_to_build_dir():
  # Codes from tools/vim/ninja-build.vim.
  chrome_root = os.path.dirname(vim.current.buffer.name)
  fingerprints = ['chrome', 'net', 'v8', 'build', 'skia']
  while chrome_root  and not all(
      [os.path.isdir(os.path.join(chrome_root , fp)) for fp in fingerprints]):
    chrome_root = os.path.dirname(chrome_root)
  sys.path.append(os.path.join(chrome_root, 'tools', 'vim'))
  # Import GetNinjaOutputDirectory from tools/vim/ninja_output.py.
  from ninja_output import GetNinjaOutputDirectory
  return GetNinjaOutputDirectory(chrome_root)

def run_test_for_line(linenumb):
  run_cmd = ' '.join(['!tools/autotest.py', '-C', path_to_build_dir(), '%',
                      '--line', linenumb] )
  vim.command(run_cmd)
endpython

fun! RunTestForCurrentLine()
  let l:current_line = shellescape(line('.'))
  pythonx run_test_for_line(vim.eval('l:current_line'))
endfun

map <C-T> :call RunTestForCurrentLine()<CR>
```

Place the cursor on what we want to test and `ctrl-t` ... Here you go.
![autotest]({% asset_path autotest.gif %})

### YouCompleteMe

[tools/vim][1] has an example configuration for [YouCompleteMe][8] (a code
completion engine for Vim). See [tools/vim/chromium.ycm_extra_conf.py][10].

```sh
let g:ycm_extra_conf_globlist = ['../.ycm_extra_conf.py']
let g:ycm_goto_buffer_command = 'split-or-existing-window'
```

As you already know, YouCompleteMe requires clangd. Very fortunately, Chromium
already supports clangd and remote index server to get daily index snapshots.

Do not skip documents about [using Clangd to build chromium][2] and [chromium
remote index server][9].

Here are short demos for completion suggestions,
![ycm_auto]({% asset_path ycm_auto.gif %})


and code jumping(`:YcmCompleter GoTo`, `:YcmCompleter GoToReferences`) during
Chromium development.
![ycm_jump]({% asset_path ycm_jump.gif %})

## Commenter

A good developer writes good comments, so being a good commenter makes you a
good developer (joke). In my humble opinion, [NERD commenter][5] will make you a
good developer (joke again). You can get all details from the docs of NERD
commenter, and Chromium needs only an additional option for mojom.

```sh
let g:NERDCustomDelimiters = { 'mojom': {'left': '//'} }
```

## File Navigation

Chromium is really huge, so we benefit from efficient file navigation tools.
There are lots of awesome fuzzy finder plugins for this purpose and I have been
using [command-t][6] for a long time. But command-t requires a Vim executable
with ruby/lua support or Neovim. If you need a simpler yet still powerful
navigator, [fzf.vim][7] is an altenative great option.

Since Chromium a git project, search commands based by `git ls-files` will
provide results very quickly. Use the option `let g:CommandTFileScanner = 'git'`
for `command-t`, and use the command `:GFiles` for `fzf.vim`.

![filenav]({% asset_path filenav.gif %})

In case you can't depend on `git ls-files` based commands, please make sure your
plugin use `rg` or `fd` which are the fastest command line search tools. For
example, [fzf.vim][7] has an option `FZF_DEFAULT_COMMAND` and the following
suggestion in the manual: `export FZF_DEFAULT_COMMAND='fd --type f'`.

## Conclusion

Thanks for reading and I hope this blog post can be some help to your
development enviroment for Chromium. Any comment for sharing some other cool
tips always be welcomed, so that we can make our Vim more productive. (Right,
this is the real purpose of this blog post.)

.... I will be back someday with debugging Chromium in Vim.
![sneak peek]({% asset_path back.png %})

[1]: https://source.chromium.org/chromium/chromium/src/+/main:tools/vim
[2]: https://chromium.googlesource.com/chromium/src.git/+/HEAD/docs/clangd.md
[3]: https://github.com/google/vim-codefmt
[4]: https://source.chromium.org/gn/gn/+/main:misc/vim/README.md
[5]: https://github.com/preservim/nerdcommenter
[6]: https://github.com/wincent/command-t
[7]: https://github.com/junegunn/fzf
[8]: https://github.com/ycm-core/YouCompleteMe
[9]: https://linux.clangd-index.chromium.org/
[10]: https://source.chromium.org/chromium/chromium/src/+/main:tools/vim/chromium.ycm_extra_conf.py
[11]: https://source.chromium.org/chromium/chromium/src/+/main:tools/autotest.py
