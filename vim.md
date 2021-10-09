# vimrc
```""""""""""""""""""""""""""""""
" vim color & format
""""""""""""""""""""""""""""""
set number
set showmatch
set shiftwidth=4
set softtabstop=4
syntax enable
colorscheme spring
""""""""""""""""""""""""""""""

""""""""""""""""""""""""""""""
" key link
""""""""""""""""""""""""""""""
inoremap <C-]>             <C-X><C-]>
inoremap <C-F>             <C-X><C-F>
inoremap <C-D>             <C-X><C-D>
inoremap <C-L>             <C-X><C-L>
""""""""""""""""""""""""""""""

""""""""""""""""""""""""""""""
" cscope
""""""""""""""""""""""""""""""
if has("cscope")
	set csprg=~/.linkPath/cscope
	set csto=0
	set cst
	set nocsverb
	" add any database in current directory
	if filereadable("cscope.out")
	    cs add cscope.out
	" else add database pointed to by environment
	elseif $CSCOPE_DB != ""
	    cs add $CSCOPE_DB
	endif
	set csverb
endif
nmap <C-_>c :cs find c <C-R>=expand("<cword>")<CR><CR>	"search fun use
nmap <C-_>d :cs find d <C-R>=expand("<cword>")<CR><CR>	"search fun has fun
nmap <C-_>s :cs find s <C-R>=expand("<cword>")<CR><CR>	"search fun appear
nmap <C-_>g :cs find g <C-R>=expand("<cword>")<CR><CR>	"search fun define
nmap <C-_>e :cs find e <C-R>=expand("<cword>")<CR><CR>	"search str egrep
nmap <C-_>f :cs find f <C-R>=expand("<cfile>")<CR><CR>	"search str file
nmap <C-_>t :cs find t <C-R>=expand("<cword>")<CR><CR>	"search str
nmap <C-_>i :cs find i ^<C-R>=expand("<cfile>")<CR>$<CR>	"search file in file

""""""""""""""""""""""""""""""
" Tag list (ctags)
""""""""""""""""""""""""""""""
map <silent> <F3> :TlistToggle<cr>	"打开关闭快捷映射
let Tlist_Show_One_File = 1            "不同时显示多个文件的tag，只显示当前文件的
let Tlist_Exit_OnlyWindow = 1          "如果taglist窗口是最后一个窗口，则退出vim
let Tlist_Use_Right_Window = 1         "在右侧窗口中显示taglist窗口 
let Tlist_GainFocus_On_ToggleOpen = 1	"focus after :TlistToggl
""""""""""""""""""""""""""""""

""""""""""""""""""""""""""""""
" plugin Vundle.vim
""""""""""""""""""""""""""""""
set nocompatible              " be iMproved, required
filetype off                  " required

" set the runtime path to include Vundle and initialize
set rtp+=~/.vim/bundle/Vundle.vim
call vundle#begin()
" alternatively, pass a path where Vundle should install plugins
"call vundle#begin('~/some/path/here')

" let Vundle manage Vundle, required
Plugin 'VundleVim/Vundle.vim'

" The following are examples of different formats supported.
" Keep Plugin commands between vundle#begin/end.
" plugin on GitHub repo
Plugin 'tpope/vim-fugitive'
" plugin from http://vim-scripts.org/vim/scripts.html
" Plugin 'L9'
" Git plugin not hosted on GitHub
Plugin 'git://git.wincent.com/command-t.git'
" git repos on your local machine (i.e. when working on your own plugin)
Plugin 'file:///home/gmarik/path/to/plugin'
" The sparkup vim script is in a subdirectory of this repo called vim.
" Pass the path to set the runtimepath properly.
Plugin 'rstacruz/sparkup', {'rtp': 'vim/'}
" Install L9 and avoid a Naming conflict if you've already installed a
" different version somewhere else.
" Plugin 'ascenator/L9', {'name': 'newL9'}
Plugin 'taglist.vim'
Plugin 'genutils'
Plugin 'jlanzarotta/bufexplorer'

" All of your Plugins must be added before the following line
call vundle#end()            " required
filetype plugin indent on    " required
" To ignore plugin indent changes, instead use:
"filetype plugin on
"
" Brief help
" :PluginList       - lists configured plugins
" :PluginInstall    - installs plugins; append `!` to update or just :PluginUpdate
" :PluginSearch foo - searches for foo; append `!` to refresh local cache
" :PluginClean      - confirms removal of unused plugins; append `!` to auto-approve removal
"
" see :h vundle for more details or wiki for FAQ
" Put your non-Plugin stuff after this line
""""""""""""""""""""""""""""""

""""""""""""""""""""""""""""""
" execute project related configuration in current directory
""""""""""""""""""""""""""""""
if filereadable("wp.vim")
    source wp.vim
endif
""""""""""""""""""""""""""""""

""""""""""""""""""""""""""""""
"default
""""""""""""""""""""""""""""""
" When started as "evim", evim.vim will already have done these settings.
if v:progname =~? "evim"
  finish
endif

" Get the defaults that most users want.
source $VIMRUNTIME/defaults.vim

if has("vms")
  set nobackup		" do not keep a backup file, use versions instead
else
  set backup		" keep a backup file (restore to previous version)
  if has('persistent_undo')
    set undofile	" keep an undo file (undo changes after closing)
  endif
endif

if &t_Co > 2 || has("gui_running")
  " Switch on highlighting the last used search pattern.
  set hlsearch
endif

" Only do this part when compiled with support for autocommands.
if has("autocmd")

  " Put these in an autocmd group, so that we can delete them easily.
  augroup vimrcEx
  au!

  " For all text files set 'textwidth' to 78 characters.
  autocmd FileType text setlocal textwidth=78

  augroup END

else

  set autoindent		" always set autoindenting on

endif " has("autocmd")

" Add optional packages.
"
" The matchit plugin makes the % command work better, but it is not backwards
" compatible.
" The ! means the package won't be loaded right away but when plugins are
" loaded during initialization.
if has('syntax') && has('eval')
  packadd! matchit
endif
""""""""""""""""""""""""""""""

""""""""""""""""""""""""""""""
"mapleader , easy option 
""""""""""""""""""""""""""""""
"Set mapleader
let mapleader = ","

function! SwitchToBuf(filename)
    "let fullfn = substitute(a:filename, "^\\~/", $HOME . "/", "")
    " find in current tab
    let bufwinnr = bufwinnr(a:filename)
    if bufwinnr != -1
        exec bufwinnr . "wincmd w"
        return
    else
        " find in each tab
        tabfirst
        let tab = 1
        while tab <= tabpagenr("$")
            let bufwinnr = bufwinnr(a:filename)
            if bufwinnr != -1
                exec "normal " . tab . "gt"
                exec bufwinnr . "wincmd w"
                return
            endif
            tabnext
            let tab = tab + 1
        endwhile
        " not exist, new tab
        exec "tabnew " . a:filename
    endif
endfunction

"Fast reloading of the .vimrc
map <silent> <leader>ss :source $MYVIMRC<cr>
"Fast editing of .vimrc
map <silent> <leader>ee :call SwitchToBuf($MYVIMRC)<cr>
"When .vimrc is edited, reload it
autocmd! bufwritepost .vimrc source ~/.vimrc

" netrw setting
let g:netrw_winsize = 50
nmap <silent> <leader>fe :Sexplore!<cr>
```