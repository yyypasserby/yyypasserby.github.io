---
title: My Setup
date: 2017-05-31 11:22:33
categories:
  - setup
tags:
  - zsh
  - tmux
  - vim
---

## Oh My Zsh

https://github.com/robbyrussell/oh-my-zsh

### Install

```sh
sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```

### Configure

```sh
plugins=(git cpp autojump osx)
[[ -s ~/.autojump/etc/profile.d/autojump.sh ]] && . ~/.autojump/etc/profile.d/autojump.sh
```

## Oh My Tmux

https://github.com/gpakosz/.tmux

### Install

```sh
cd
git clone https://github.com/gpakosz/.tmux.git
ln -s -f .tmux/.tmux.conf
cp .tmux/.tmux.conf.local .
tmux source-file ~/.tmux.conf
```

## spf13-vim

https://github.com/spf13/spf13-vim

### Download

```sh
curl https://j.mp/spf13-vim3 -L > spf13-vim.sh && sh spf13-vim.sh
```

### Install

Open vim and run

```vim
:PluginInstall
```

### Configure

Modify `.vimrc.bundles`

```vim
let g:spf13_bundle_groups=['general', 'writing', 'youcompleteme', 'programming', 'python', 'javascript', 'html', 'misc',]
```

Install YouCompleteMe (cpp & javascript)

```sh
cd ~/.vim/bundle/YouCompleteMe
./install.py --clang-completer
./install.py --tern-completer
```

Add settings for YouCompleteMe in `.vimrc.local`

```vim
let g:ycm_global_ycm_extra_conf = "~/.vim/.ycm_extra_conf.py"
let g:ycm_confirm_extra_conf = 0
let g:ycm_seed_identifiers_with_syntax = 1
let g:ycm_collect_identifiers_from_tag_files = 1
let g:ycm_min_num_of_chars_for_completion = 1
set completeopt=longest,menu
```

Get `.ycm_extra_conf.py`

```sh
cd
wget https://raw.githubusercontent.com/Valloric/ycmd/master/cpp/ycm/.ycm_extra_conf.py
```

Add the path under `include <...>` to the `.ycm_extra_conf.py` with `-isystem`

```sh
echo | clang -v -E -x c++ -
```

Open vim and restart YcmServer

```vim
:YcmRestartServer
```

#### [optional] Add google/vim-codefmt

Add plugins to `.vimrc.bundles.local` and run `:PluginInstall`

```vim
Plugin 'google/vim-maktaba'
Plugin 'google/vim-codefmt'
Plugin 'google/vim-glaive'
```

Add settings to `.vimrc.local`

```vim
call glaive#Install()
Glaive codefmt plugin[mappings]
Glaive codefmt google_java_executable="java -jar /path/to/google-java-format-VERSION-all-deps.jar"
Glaive codefmt clang_format_style="google"

augroup autoformat_settings
  autocmd FileType bzl AutoFormatBuffer buildifier
  autocmd FileType c,cpp,proto,javascript AutoFormatBuffer clang-format
  autocmd FileType dart AutoFormatBuffer dartfmt
  autocmd FileType go AutoFormatBuffer gofmt
  autocmd FileType gn AutoFormatBuffer gn
  autocmd FileType html,css,json AutoFormatBuffer js-beautify
  autocmd FileType java AutoFormatBuffer google-java-format
  autocmd FileType python AutoFormatBuffer yapf
  " Alternative: autocmd FileType python AutoFormatBuffer autopep8
augroup END

" vim-javascript
autocmd Filetype javascript setlocal ts=2 sts=2 sw=2
```

Unbind existing key mapping of `<Leader>=` in `.vimrc`
