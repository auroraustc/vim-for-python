# vim-for-python
Set vim for python development

让vim适合python开发

# 目录
- [功能](#功能)
- [参考](#参考)
- [开始操作](#开始操作)
  - [需要提前准备的环境](#需要提前准备的环境)
  - [Python 3.8](#python-38)
  - [ncurses](#ncurses)
  - [vim 8.2](#vim-82)
  - [ctags](#ctags)
  - [安装一部分vim插件](#安装一部分vim插件)
  - [自动补全插件YouCompleteMe](#自动补全插件youcompleteme)
- [一些功能的快捷键举例](#一些功能的快捷键举例)
- [使用ipdb单步调试](#使用ipdb单步调试)
  - [安装ipdb](#安装ipdb)
  - [常用指令](#常用指令)

# 功能
- 代码高亮、跳转等
- 自动补全

# 参考
* https://cloud.tencent.com/developer/article/1745640
* https://zhuanlan.zhihu.com/p/30022074

# 开始操作
## 需要提前准备的环境
* Python 3.8
  * YouCompleteMe插件需要较高版本的Python 3.6+。考虑到可能会使用multiprocessing，建议使用Python 3.8+
* ncurses
  * 编译vim需要的依赖
* vim 8.2及以上
  * YouCompleteMe的需求
* ctags
  * vim一些插件需要的依赖
* libclang
  * YouCompleteMe需要的依赖

## Python 3.8
可自行编译，也可以用系统自带的Python。如果使用系统自带的Python，确保python3-dev也已经安装过了（一般都有）。

## ncurses
在这里下载ncurses：https://invisible-island.net/ncurses/ncurses.html#download_ncurses ，选择第四个via http，下载得到```ncurses.tar.gz```。
```bash
$ tar -zxvf ncurses.tar.gz
$ cd ncurses-6.2/
$ mkdir install
$ cd install
$ NCURSES_INSTALL_DIR=$(pwd)
$ cd ../
$ ./configure --prefix=$NCURSES_INSTALL_DIR
$ make -j 8
$ make install
```
接下来设置一些环境变量：
```bash
$ export PATH=$NCURSES_INSTALL_DIR/bin/:$PATH
$ export LIBRARY_PATH=$NCURSES_INSTALL_DIR/lib/:$LIBRARY_PATH
$ export LD_LIBRARY_PATH=$NCURSES_INSTALL_DIR/lib/:$LD_LIBRARY_PATH
$ export C_INCLUDE_PATH=$NCURSES_INSTALL_DIR/include/:$C_INCLUDE_PATH
$ export CPLUS_INCLUDE_PATH=$NCURSES_INSTALL_DIR/include/:$CPLUS_INCLUDE_PATH
```
至此ncurses安装完成。也可将这些环境变量添加到```~/.bashrc```中：
```bash
# 以上五个export也可以添加到~/.bashrc中，记得把$NCURSES_INSTALL_DIR改成其代表的绝对路径，比如：
$ echo $NCURSES_INSTALL_DIR
/home/user1/soft/ncurses-6.2/install
# 那么就可以通过以下命令添加到~/.bashrc（注意最后的是两个箭头>>）：
$ echo "
export PATH=/home/user1/soft/ncurses-6.2/install/bin/:$PATH
export LIBRARY_PATH=/home/user1/soft/ncurses-6.2/install/lib/:$LIBRARY_PATH
export LD_LIBRARY_PATH=/home/user1/soft/ncurses-6.2/install/lib/:$LD_LIBRARY_PATH
export C_INCLUDE_PATH=/home/user1/soft/ncurses-6.2/install/include/:$C_INCLUDE_PATH
export CPLUS_INCLUDE_PATH=/home/user1/soft/ncurses-6.2/install/include/:$CPLUS_INCLUDE_PATH
" >> ~/.bashrc
$ source ~/.bashrc
```

## vim 8.2 
```bash
$ git clone https://github.com/vim/vim
$ cd vim
$ mkdir install
$ cd install
$ VIM_INSTALL_DIR=$(pwd)
$ cd ../
$ ./configure --prefix=$VIM_INSTALL_DIR --enable-luainterp=yes \ 
  --enable-mzschemeinterp --enable-perlinterp=yes  --enable-python3interp=yes \ 
  --enable-tclinterp=yes --enable-rubyinterp=yes --enable-cscope --enable-terminal \ 
  --enable-autoservername --enable-multibyte --enable-xim --enable-fontset \ 
  --with-modified-by=shlian --with-compiledby=shlian  \ 
  --with-python3-command=python3
# 不要在anaconda的base环境下configure，不然可能会报类似的错误：
# lto1: fatal error: bytecode stream in file ‘/home/user1/soft/anaconda3/lib/python3.8/config-3.8-x86_64-linux-gnu/libpython3.8.a’ generated with LTO version 6.0 instead of the expected 8.1。先conda deactivate再configure
$ make install -j8
# 如果出现以下提示：make: 'install' is up to date.那么可以如下操作：
$ cd src
$ make install -j8
# 可以添加一个软链接，让vi命令也打开vim：
$ cd $VIM_INSTALL_DIR/bin
$ ln -s vim vi
```
然后设置一下环境变量：
```bash
$ export PATH=$VIM_INSTALL_DIR/bin/:$PATH
# 或者如下，注意修改成自己的vim安装路径
$ echo "
export PATH=/home/user1/soft/vim/install/bin/:$PATH
" >> ~/.bashrc
$ source ~/.bashrc
```
查看vim是否启用Python支持：
```bash
$ vim --version | grep python3
# 应该看到+python3的字样
```

## ctags
```bash
$ git clone https://github.com/universal-ctags/ctags.git
$ cd ctags
$ mkdir install
$ cd install/
$ CTAGS_INSTALL_DIR=$(pwd)
$ cd ../
$ ./autogen.sh
$ ./configure --prefix=$CTAGS_INSTALL_DIR
$ make install -j8
```
同样地，设置环境变量：
```bash
$ export PATH=$CTAGS_INSTALL_DIR/bin/:$PATH
# 或者：
echo "
export PATH=/home/user1/soft/ctags/install/bin/:$PATH
" >> ~/.bashrc
```

## 安装一部分vim插件
首先安装vim的插件管理插件
```bash
$ mkdir -p  ~/.vim/bundle
$ git clone https://github.com/VundleVim/Vundle.vim.git  ~/.vim/bundle/Vundle.vim
# 打开~/.vimrc进行初步的设置
$ vim ~/.vimrc
```
先设置成paste模式，按键shift+;进入命令模式，然后输入set paste，回车。之后把下面这些粘贴进去
```bash
set nocompatible              " be iMproved, required
filetype off                  " required
 
" set the runtime path to include Vundle and initialize
set rtp+=~/.vim/bundle/Vundle.vim
call vundle#begin()
 
" let Vundle manage Vundle, required
Plugin 'VundleVim/Vundle.vim'
 
" All of your Plugins must be added before the following line
call vundle#end()            " required
filetype plugin indent on    " required
```
保存退出vim，然后重新打开~/.vimrc
```bash
$ vim ~/.vimrc
```
按键shift+;进入命令模式，输入PluginInstall，回车等待即可。

然后用vim打开~/.vimrc启用一些有用的插件。此处我是直接从网上抄来的，稍做了几行修改。可以直接把以下内容在paste模式下粘贴进~/.vimrc中：
```bash
"/**
"* @file .vimrc
"* @brief vimrc
"* @author shlian
"* @version 1.1.0
"* @date 2020-09-01
"*/

set nocompatible " 关闭 vi 兼容模式
set smartindent "当在大括号中间回车的时候，他会智能缩进，因为他知道括号中间要缩进
set tabstop=4
set shiftwidth=4
set expandtab
syntax on " 自动语法高亮
set number " 显示行号

"设置代码参考线
highlight ColorColumn ctermbg=darkgray
set colorcolumn=140

" 高亮显示当前行
set cursorline 
  "red（红），white（白），black（黑），green（绿），yellow（黄），blue（蓝），purple（紫），gray（灰），brown（棕），tan(褐色)，syan(青色)
hi CursorLine   cterm=NONE ctermbg=darkgray ctermfg=NONE
"hi CursorColumn cterm=NONE ctermbg=darkred ctermfg=white 

set ruler " 打开状态栏标尺
set shiftwidth=4 " 设定 << 和 >> 命令移动时的宽度为 4
set softtabstop=4 " 使得按退格键时可以一次删掉 4 个空格
set tabstop=4 " 设定 tab 长度为 4
set nobackup " 覆盖文件时不备份
"set autochdir " 自动切换当前目录为当前文件所在的目录
set backupcopy=yes " 设置备份时的行为为覆盖
set ignorecase smartcase " 搜索时忽略大小写，但在有一个或以上大写字母时仍保持对大小写敏感
"set nowrapscan " 禁止在搜索到文件两端时重新搜索
set incsearch " 输入搜索内容时就显示搜索结果
set hlsearch " 搜索时高亮显示被找到的文本
set noerrorbells " 关闭错误信息响铃
set novisualbell " 关闭使用可视响铃代替呼叫
set t_vb= " 置空错误铃声的终端代码
set showmatch " 插入括号时，短暂地跳转到匹配的对应括号
" set matchtime=2 " 短暂跳转到匹配括号的时间
set magic " 设置魔术
set hidden " 允许在有未保存的修改时切换缓冲区，此时的修改由 vim 负责保存
set guioptions-=T " 隐藏工具栏
set guioptions-=m " 隐藏菜单栏
set smartindent " 开启新行时使用智能自动缩进
set backspace=indent,eol,start
" 不设定在插入状态无法用退格键和 Delete 键删除回车符
set cmdheight=1 " 设定命令行的行数为 1
set laststatus=2 " 显示状态栏 (默认值为 1, 无法显示状态栏)
set statusline=\ %<%F[%1*%M%*%n%R%H]%=\ %y\ %0(%{&fileformat}\ %{&encoding}\ %c:%l/%L%)\

" 设置在状态行显示的信息
set foldenable " 开启折叠
"set foldmethod=syntax " 设置语法折叠************************
set foldcolumn=0 " 设置折叠区域的宽度
setlocal foldlevel=1 " 设置折叠层数为
"set foldclose=all " 设置为自动关闭折叠 
nnoremap <space> @=((foldclosed(line('.')) < 0) ? 'zc' : 'zo')<CR>
" 用空格键来开关折叠

" return OS type, eg: windows, or linux, mac, et.st..
function! MySys()
    if has("win16") || has("win32") || has("win64") || has("win95")
        return "windows"
    elseif has("unix")
        return "linux"
    endif
endfunction

" 用户目录变量$VIMFILES
if MySys() == "windows"
    let $VIMFILES = $VIM.'/vimfiles'
elseif MySys() == "linux"
    let $VIMFILES = $HOME.'/.vim'
endif

" 设定doc文档目录
let helptags=$VIMFILES.'/doc'

" 设置字体 以及中文支持
if has("win32")
    set guifont=Inconsolata:h12:cANSI
endif

" 配置多语言环境
if has("multi_byte")
" UTF-8 编码
set encoding=utf-8
set termencoding=utf-8
set formatoptions+=mM
set fencs=utf-8,gbk

if v:lang =~? '^\(zh\)\|\(ja\)\|\(ko\)'
    set ambiwidth=double
endif

if has("win32")
    source $VIMRUNTIME/delmenu.vim
    source $VIMRUNTIME/menu.vim
    language messages zh_CN.utf-8
endif
else
    echoerr "Sorry, this version of (g)vim was not compiled with +multi_byte"
endif

" Buffers操作快捷方式!
"nnoremap <C-RETURN> :bnext<CR>
"nnoremap <C-S-RETURN> :bprevious<CR>

" Tab操作快捷方式!
"nnoremap <C-TAB> :tabnext<CR>
"nnoremap <C-S-TAB> :tabprev<CR>

"关于tab的快捷键
" map tn :tabnext<cr>
" map tp :tabprevious<cr>
" map td :tabnew .<cr>
" map te :tabedit
" map tc :tabclose<cr>

"窗口分割时,进行切换的按键热键需要连接两次,比如从下方窗口移动
"光标到上方窗口,需要<c-w><c-w>k,非常麻烦,现在重映射为<c-k>,切换的
"时候会变得非常方便.
nnoremap <C-h> <C-w>h
nnoremap <C-j> <C-w>j
nnoremap <C-k> <C-w>k
nnoremap <C-l> <C-w>l

" set fileformats=unix,dos,mac
 nmap <leader>fd :se fileformat=dos<CR>
 nmap <leader>fu :se fileformat=unix<CR>

" use Ctrl+[l|n|p|cc] to list|next|previous|jump to count the result
" map <C-x>l <ESC>:cl<CR>
" map <C-x>n <ESC>:cn<CR>
" map <C-x>p <ESC>:cp<CR>
" map <C-x>c <ESC>:cc<CR>


" Python 文件的一般设置，比如不要 tab 等
autocmd FileType python set tabstop=4 shiftwidth=4 expandtab
autocmd FileType python map <F12> :!python %<CR>

" 选中状态下 Ctrl+c 复制
"vmap <C-c> "+y

"十六进制显示文件 
nmap <leader>H :%!xxd<CR>     
"二进制显示文件
nmap <leader>B :%!xxd -r<CR>

"******************************************************************************************************************

set rtp+=~/.vim/bundle/Vundle.vim

call vundle#begin()
	filetype plugin indent on    " 必须加载vim自带和插件相应的语法和文件类型相关脚本

	"vim包管理工具
	Plugin 'gmarik/Vundle.vim'

	"文件目录增加git 状态
	Plugin 'Xuyuanp/nerdtree-git-plugin'
	
	"tab智能补全
	Plugin 'ervandew/supertab'

	"代码可视化缩进块
    Plugin 'yggdroot/indentline'
        let g:indentLine_enabled = 1
        let g:indentLine_color_term = 230
        "let g:indentLine_char_list = ['|', '¦', '┆', '┊','|'] 会出现光标的错行，慎用
        let g:indentLine_char_list = ['|', '¦', '¦', '¦']


	"彩虹括号
	Plugin 'kien/rainbow_parentheses.vim'

	"真彩色
	Plugin 'tpope/vim-sensible'

	"git左边栏增删改提示
	Plugin 'airblade/vim-gitgutter'

    Plugin 'altercation/solarized'
	Plugin 'altercation/vim-colors-solarized'  "solarized
	    let g:solarized_termtrans  = 1         " 使用 termnal 背景
	    let g:solarized_visibility = "high"    " 使用 :set list 显示特殊字符时的高亮级别

	    " GUI 模式浅色背景，终端模式深色背景
	    if has('gui_running')
			set background=light
	    else
			set background=dark
	    endif
	    " 主题设置为 solarized
		"colorscheme solarized

	"文件目录分屏
	Plugin 'scrooloose/nerdtree'
		let NERDTreeHighlightCursorline = 1       " 高亮当前行
		let NERDTreeShowLineNumbers     = 1       " 显示行号
		" 忽略列表中的文件
		let NERDTreeIgnore = [ '\.pyc$', '\.pyo$', '\.obj$', '\.o$', '\.egg$', '^\.git$', '^\.repo$', '^\.svn$', '^\.hg$' ]
		" 启动 vim 时打开 NERDTree
		"autocmd vimenter * NERDTree
		" 当打开 VIM，没有指定文件时和打开一个目录时，打开 NERDTree
		"autocmd StdinReadPre * let s:std_in = 1
		"autocmd VimEnter * if argc() == 0 && !exists("s:std_in") | NERDTree | endif
		"autocmd VimEnter * if argc() == 1 && isdirectory(argv()[0]) && !exists("s:std_in") | exe 'NERDTree' argv()[0] | wincmd p | ene | exe 'cd '.argv()[0] | endif
		" 关闭 NERDTree，当没有文件打开的时候
		"autocmd bufenter * if (winnr("$") == 1 && exists("b:NERDTreeType") && b:NERDTreeType == "primary") | q | end
        
		" <leader>nt 打开 nerdtree 窗口，在左侧栏显示
		map <leader>nt :NERDTreeToggle<CR>
		" <leader>tc 关闭当前的 tab
		map <leader>tc :tabc<CR>
		" <leader>to 关闭所有其他的 tab
		map <leader>to :tabo<CR>
		" <leader>ts 查看所有打开的 tab
		map <leader>ts :tabs<CR>
		" <leader>tp 前一个 tab
		map <leader>tp :tabp<CR>
		" <leader>tn 后一个 tab
		map <leader>tn :tabn<CR

    "tagbar
    Plugin 'majutsushi/tagbar'
        let g:tagbar_ctags_bin = 'ctags' " tagbar 依赖 ctags 插件
        let g:tagbar_width     = 30      " 设置 tagbar 的宽度为 30 列，默认 40 列
        let g:tagbar_autofocus = 1       " 打开 tagbar 时光标在 tagbar 页面内，默认在 vim 打开的文件内
        let g:tagbar_left      = 1       " 让 tagbar 在页面左侧显示，默认右边
        "let g:tagbar_sort      = 0       " 标签不排序，默认排序
         
        " <leader>tb 打开 tagbar 窗口，在左侧栏显示
        map <leader>tb :TagbarToggle<CR>

    "taglist
    Plugin 'vim-scripts/taglist.vim'

        let Tlist_Show_One_File           = 1    " 只显示当前文件的tags
        let Tlist_GainFocus_On_ToggleOpen = 1    " 打开 Tlist 窗口时，光标跳到 Tlist 窗口
        let Tlist_Exit_OnlyWindow         = 1    " 如果 Tlist 窗口是最后一个窗口则退出 Vim
        let Tlist_Use_Left_Window         = 1    " 在左侧窗口中显示
        let Tlist_File_Fold_Auto_Close    = 1    " 自动折叠
        let Tlist_Auto_Update             = 1    " 自动更新
         
        " <leader>tl 打开 Tlist 窗口，在左侧栏显示
         map <leader>tl :TlistToggle<CR>

    "winmanager
    Plugin 'vim-scripts/winmanager'
        let g:NERDTree_title="[NERDTree]"
        "let g:winManagerWindowLayout="TagList|NERDTree"
        let g:winManagerWindowLayout='FileExplorer|TagList'

        function! NERDTree_Start()
            exec 'NERDTree'
        endfunction
        function! NERDTree_IsValid()
            return 1
        endfunction

        nmap wm :WMToggle<CR>

    "powerline fonts
    Plugin 'powerline/powerline-fonts'

    "状态栏
    Plugin 'vim-airline/vim-airline'
    Plugin 'vim-airline/vim-airline-themes'

        let g:airline_powerline_fonts = 1   " 使用powerline打过补丁的字体
        if !exists('g:airline_symbols')
            let g:airline_symbols = {}
        endif

        " 关闭当前 buffer
        "noremap <C-x> :w<CR>:bd<CR>
        "<leader>1~9 切到 buffer1~9
        map <leader>1 :b 1<CR>
        map <leader>2 :b 2<CR>
        map <leader>3 :b 3<CR>
        map <leader>4 :b 4<CR>
        map <leader>5 :b 5<CR>
        map <leader>6 :b 6<CR>
        map <leader>7 :b 7<CR>
        map <leader>8 :b 8<CR>
        map <leader>9 :b 9<CR>

        "Vim 在与屏幕/键盘交互时使用的编码(取决于实际的终端的设定)        
        set encoding=utf-8
        set langmenu=zh_CN.UTF-8
        " 设置打开文件的编码格式  
        set fileencodings=ucs-bom,utf-8,cp936,gb18030,big5,euc-jp,euc-kr,latin1 
        set fileencoding=utf-8
        " 解决菜单乱码
        source $VIMRUNTIME/delmenu.vim
        source $VIMRUNTIME/menu.vim
        " 解决consle输出乱码
        "set termencoding = cp936  
        " 设置中文提示
        " language messages zh_CN.utf-8 
        " 设置中文帮助
        set helplang=cn
        " 设置为双字宽显示，否则无法完整显示如:☆
        set ambiwidth=double
        " 总是显示状态栏 
        let laststatus = 2
        let g:airline_theme='solarized'      " 设置主题,simple、dark、solarized、bubblegum 详见.vim/bundle/vim-airline-themes

        " 开启tabline
        let g:airline#extensions#tabline#enabled = 1      "tabline中当前
        let g:airline#extensions#tabline#left_sep = ' '   "buffer两端的分隔字符
        let g:airline#extensions#tabline#left_alt_sep = '|'      "tabline中
        let g:airline#extensions#tabline#buffer_nr_show = 1      "buffer显示编号

        let g:airline_left_sep = ' '
        let g:airline_left_alt_sep = '▶'
        let g:airline_right_sep = ' '
        let g:airline_right_alt_sep = '◀'
        let g:airline_symbols.crypt = '?'
        let g:airline_symbols.linenr = '¶'
        let g:airline_symbols.maxlinenr = '㏑'
        let g:airline_symbols.branch = '⎇'
        let g:airline_symbols.paste = 'ρ'
        let g:airline_symbols.spell = 'Ꞩ'
        let g:airline_symbols.notexists = 'Ɇ'
        let g:airline_symbols.whitespace = 'Ξ'

        " 映射切换buffer的键位
        nnoremap [b :bp<CR>
        nnoremap ]b :bn<CR>
        " 设置字体 
        set guifont=Powerline_Consolas:h14:cANSI

    Plugin 'enricobacis/vim-airline-clock'
        "let g:airline#extensions#clock#auto = 0  关闭
        let g:airline#extensions#clock#format = '%H:%M:%S'

    
    ""代码动态检查(使用YCM自带的即可)
    "Plugin 'w0rp/ale'
    "    let g:ale_lint_on_text_changed       = 'normal'                     " 代码更改后启动检查 
    "    let g:ale_lint_on_insert_leave       = 1                            " 退出插入模式即检查
    "    let g:ale_sign_column_always         = 1                            " 总是显示动态检查结果
    "    "let g:ale_statusline_format = ['✗ %d', '⚡%d','✔  OK']
    "    let g:ale_sign_error                 = '>>'                         " error 告警符号
    "    let g:ale_sign_warning               = '--'                         " warning 告警符号
    "    let g:ale_echo_msg_error_str         = 'E'                          " 错误显示字符
    "    let g:ale_echo_msg_warning_str       = 'W'                          " 警告显示字符
    "    let g:ale_echo_msg_format            = '[%linter%] %s [%severity%]' " 告警显示格式
    "     
    "    " C 语言配置检查参数
    "    let g:ale_c_gcc_options              = '-Wall -Werror -O2 -std=c11'
    "    let g:ale_c_clang_options            = '-Wall -Werror -O2 -std=c11'
    "    let g:ale_c_cppcheck_options         = ''
    "    " C++ 配置检查参数
    "    let g:ale_cpp_gcc_options            = '-Wall -Werror -O2 -std=c++14'
    "    let g:ale_cpp_clang_options          = '-Wall -Werror -O2 -std=c++14'
    "    let g:ale_cpp_cppcheck_options       = ''
    "     
    "    "使用clang对c和c++进行语法检查，对python使用pylint进行语法检查
    "    let g:ale_linters = {  'c++': ['clang', 'gcc'] }
    "    " <F9> 触发/关闭代码动态检查
    "    map <F9> :ALEToggle<CR>
    "    "普通模式下，ak 前往上一个错误或警告，aj 前往下一个错误或警告                                                                                                                                                    
    "    nmap ak <Plug>(ale_previous_wrap)
    "    nmap aj <Plug>(ale_next_wrap)
    "    " ad 查看错误或警告的详细信息
    "    nmap ad :ALEDetail<CR>

    "模糊查找文件，ctrl+p
    Plugin 'ctrlpvim/ctrlp.vim'

    "git log与code对应
    "Plugin 'vim-fugitive'

    "显示文件修改痕迹
    Plugin 'chrisbra/changesPlugin'
    
     
    "Plugin 'bufexplorer'

    "浏览最近打开的文件
    Plugin 'vim-startify'

    "撤销
    Plugin 'Gundo'
    nnoremap <leader>u :GundoToggle<CR>

    "恢复上次关闭时所打开的文件
    "Plugin 'Sessionman'
    "set sessionoptions=blank,buffers,curdir,folds,tabpages,winsize
    "nmap <leader>sl :SessionList<CR>
    "nmap <leader>ss :SessionSave<CR>
    "nmap <leader>sc :SessionClose<CR>

    "Plugin 'Powerline'
    "Plugin 'rainbow_parentheses'

    Plugin 'ludovicchabant/vim-gutentags' 
        "gutentags搜索工程目录的标志，碰到这些文件/目录名就停止向上一级目录递归
        
        let g:gutentags_project_root = ['.root', '.svn', '.git', '.project']
        
        " 所生成的数据文件的名称 "
        let g:gutentags_ctags_tagfile = '.tags'

        " 将自动生成的 tags 文件全部放入 ~/.cache/tags 目录中，避免污染工程目录 "
         let s:vim_tags = expand('~/.cache/tags')
         let g:gutentags_cache_dir = s:vim_tags
         " 检测 ~/.cache/tags 不存在就新建 "
         if !isdirectory(s:vim_tags)
            silent! call mkdir(s:vim_tags, 'p')
         endif
        
            " 配置 ctags 的参数 "
        let g:gutentags_ctags_extra_args = ['--fields=+niazS', '--extra=+q']
        let g:gutentags_ctags_extra_args += ['--c++-kinds=+pxI']
        let g:gutentags_ctags_extra_args += ['--c-kinds=+px']

    Plugin 'skywind3000/gutentags_plus'
        " gutentags搜索工程目录的标志，碰到这些文件/目录名就停止向上一级目录递归
        " let g:gutentags_project_root = ['.root', '.svn', '.git', '.project']
        
        " 所生成的数据文件的名称 "
        let g:gutentags_ctags_tagfile = '.tags'
        
        " 将自动生成的 tags 文件全部放入 ~/.cache/tags 目录中，避免污染工程目录
        let s:vim_tags = expand('~/.cache/tags')
        let g:gutentags_cache_dir = s:vim_tags
        " 检测 ~/.cache/tags 不存在就新建 "
        if !isdirectory(s:vim_tags)
           silent! call mkdir(s:vim_tags, 'p')
           endif
        
           " 配置 ctags 的参数 "
           let g:gutentags_ctags_extra_args = ['--fields=+niazS','--extra=+q']
           let g:gutentags_ctags_extra_args += ['--c++-kinds=+pxI']
           "let g:gutentags_ctags_extra_args += ['--c-kinds=+px']

    "json viewer
    Plugin 'elzr/vim-json'

    " vim script library 用法详见src
    Plugin 'L9'


    "在needtree中显示文件图标
    Plugin 'ryanoasis/vim-devicons'

    "c++ 语法高亮
    Plugin 'octol/vim-cpp-enhanced-highlight'
        "let g:cpp_class_scope_highlight = 1
        "let g:cpp_member_variable_highlight = 1
        "let g:cpp_class_decl_highlight = 1
        "let g:cpp_posix_standard = 1
        "let g:cpp_experimental_simple_template_highlight = 1
        "let g:cpp_experimental_template_highlight = 1
        "let g:cpp_concepts_highlight = 1
        "let g:cpp_no_function_highlight = 1

    "switch .h <->.cpp
    "Plugin 'a.vim'

    "查找文件
    Plugin 'junegunn/fzf'
        ":FZF 从当前目录查找
        ":FZF ~ 从home目录查找
        nmap <leader>f :FZF ~<CR>
        nmap <leader>F :FZF<CR>

    "查找
    Plugin 'FuzzyFinder'
        ":FufBuffer|       - Buffer mode (|fuf-buffer-mode|)
        ":FufFile|         - File mode (|fuf-file-mode|)
        ":FufCoverageFile| - Coverage-File mode (|fuf-coveragefile-mode|)
        ":FufDir|          - Directory mode (|fuf-dir-mode|)
        ":FufMruFile|      - MRU-File mode (|fuf-mrufile-mode|)
        ":FufMruCmd|       - MRU-Command mode (|fuf-mrucmd-mode|)
        ":FufBookmarkFile| - Bookmark-File mode (|fuf-bookmarkfile-mode|)
        ":FufBookmarkDir|  - Bookmark-Dir mode (|fuf-bookmarkdir-mode|)
        ":FufTag|          - Tag mode (|fuf-tag-mode|)
        ":FufBufferTag|    - Buffer-Tag mode (|fuf-buffertag-mode|)
        ":FufTaggedFile|   - Tagged-File mode (|fuf-taggedfile-mode|)
        ":FufJumpList|     - Jump-List mode (|fuf-jumplist-mode|)
        ":FufChangeList|   - Change-List mode (|fuf-changelist-mode|)
        ":FufQuickfix|     - Quickfix mode (|fuf-quickfix-mode|)
        ":FufLine|         - Line mode (|fuf-line-mode|)
        ":FufHelp|         - Help mode (|fuf-help-mode|)
        
    "doc    
    Plugin 'DoxygenToolkit.vim'
        ":Dox
        ":DoxAuthor
        ":DoxBlock
        ":DoxLic

    "draw txt graph
    Plugin 'DrawIt'
        "\di start
        "\ds stop

    ""bookmarks
    "Plugin 'mattesgroeger/vim-bookmarks'
    "    highlight BookmarkSign ctermbg=NONE ctermfg=160
    "    highlight BookmarkLine ctermbg=NONE ctermfg=NONE
    "    let g:bookmark_sign = '♥'
    "    let g:bookmark_highlight_lines = 1
    "    nmap <leader><leader> <Plug>BookmarkToggle
    "    nmap <leader>i <Plug>BookmarkAnnotate
    "    nmap <leader>a <Plug>BookmarkShowAll
    "    nmap <leader>j <Plug>BookmarkNext
    "    nmap <leader>k <Plug>BookmarkPrev
    "    nmap <leader>c <Plug>BookmarkClear
    "    nmap <leader>x <Plug>BookmarkClearAll
    "    nmap <leader>kk <Plug>BookmarkMoveUp
    "    nmap <leader>jj <Plug>BookmarkMoveDown
    "    nmap <leader>g <Plug>BookmarkMoveToLine


    "highlighting for Google's Protocol Buffers
    "Plugin 'uarun/vim-protobuf'

    "显示所有的leader映射
    Plugin 'hecal3/vim-leader-guide'
       ":LeaderGuide '\' 
    
    "cmake 
    Plugin 'jansenm/vim-cmake'   

    "Plugin 'ihacklog/hicursorwords'
    "    let g:HiCursorWords_delay = 200
    "    let g:HiCursorWords_hiGroupRegexp = ''
    "    let g:HiCursorWords_debugEchoHiName = 0

    ""语法检查(使用YCM自带的即可)
    "Plugin 'scrooloose/syntastic'
    "    set statusline+=%#warningmsg#
    "    set statusline+=%{SyntasticStatuslineFlag()}
    "    set statusline+=%*
    "    let g:syntastic_always_populate_loc_list = 1
    "    let g:syntastic_auto_loc_list = 0
    "    let g:syntastic_check_on_open = 1
    "    let g:syntastic_check_on_wq = 0
    "    let g:syntastic_cpp_checkers = ['gcc']
    "    let g:syntastic_cpp_compiler = 'g++'
    "    let g:syntastic_cpp_compiler_options = '-std=c++11 -stdlib=libc++'
    "    "if !exists('g:syntastic_cpp_compiler_options')
    "    "    let g:syntastic_cpp_compiler_options = ' -std=c++11 -lstdc++ '
    "    "endif

    "注释
    Plugin 'scrooloose/nerdcommenter'
        "\cc 注释
        " Add spaces after comment delimiters by default
        let g:NERDSpaceDelims = 1
        
        " Use compact syntax for prettified multi-line comments
        let g:NERDCompactSexyComs = 1
        
        " Align line-wise comment delimiters flush left instead of following code indentation
        let g:NERDDefaultAlign = 'left'
        
        " Set a language to use its alternate delimiters by default
        let g:NERDAltDelims_java = 1
        
        " Add your own custom formats or override the defaults
        let g:NERDCustomDelimiters = { 'c': { 'left': '/**','right': '*/' } }
        
        " Allow commenting and inverting empty lines (useful when commenting a region)
        let g:NERDCommentEmptyLines = 1
        
        " Enable trimming of trailing whitespace when uncommenting
        let g:NERDTrimTrailingWhitespace = 1
        
        " Enable NERDCommenterToggle to check all selected lines is commented or not 
        let g:NERDToggleCheckAllLines = 1
        

    "Plugin 'tpope/surround-vim'

    "Plugin 'Valloric/YouCompleteMe'
        "往前跳和往后跳的快捷键为Ctrl+O以及Ctrl+I
        let g:ycm_key_list_select_completion=['<c-n>']
        let g:ycm_key_list_previous_completion=['<c-p>']

        set completeopt=menu  "关闭preview window
        "let g:ycm_add_preview_to_completeopt =0
        "let g:ycm_autoclose_preview_window_after_completion=1
        "let g:ycm_autoclose_preview_window_after_insertion=1
        "let g:ycm_always_populate_location_list = 0

        let g:ycm_confirm_extra_conf=0 "关闭加载.ycm_extra_conf.py提示
        let g:ycm_collect_identifiers_from_tags_files=1 " 开启 YCM 基于标签引擎
        let g:ycm_min_num_of_chars_for_completion=1 " 从第1个键入字符就开始罗列匹配项
        let g:ycm_cache_omnifunc=0 " 禁止缓存匹配项,每次都重新生成匹配项

        let g:ycm_seed_identifiers_with_syntax=1 " 语法关键字补全
         nnoremap <F5> :YcmForceCompileAndDiagnostics<CR> "force recomile with syntastic
        "nnoremap <leader>lo :lopen<CR> "open locationlist
        "nnoremap <leader>lc :lclose<CR>    "close locationlist

        "inoremap <leader><leader> <C-x><C-o>
        let g:ycm_complete_in_comments = 1 "在注释输入中也能补全
        let g:ycm_complete_in_strings = 1 "在字符串输入中也能补全
        let g:ycm_collect_identifiers_from_comments_and_strings = 0 "注释和字符串中的文字也会被收入补全

        let g:ycm_max_num_identifier_candidates = 50
        let g:ycm_auto_trigger = 1

        let g:ycm_error_symbol = '>>'
        let g:ycm_warning_symbol = '>'

        "sub commands
        "YcmCompleter RefactorRename :重命名
        "YcmCompleter GoToSymbol  
        
        nnoremap <leader>go :YcmCompleter GoTo<CR> "跳转
        nnoremap <leader>gd :YcmCompleter GoToDefinitionElseDeclaration<CR> "跳转到定义或声明
        nnoremap <leader>gt :YcmCompleter GetType<CR> "get类型

        nmap gi :YcmCompleter GoToInclude<CR>   "跳转到include、声明或定义
        nmap gm :YcmCompleter GoToImprecise<CR> "跳转到实现
        nmap gr :YcmCompleter GoToReferences<CR> "跳转到引用
        nmap fi :YcmCompleter FixIt<CR> "根据Ycm的建议修复错误

        nnoremap <F6> :YcmForceCompileAndDiagnostics<CR> "重新编译和诊断
        "nmap <F4> :YcmDiags<CR>  "F4进行诊断
        
        "nnoremap <leader>gl :YcmCompleter GoToDeclaration<CR> "跳转到声明
        "nnoremap <leader>gf :YcmCompleter GoToDefinition<CR>  "跳转到定义



        "highlight Pmenu ctermfg=4 ctermbg=0 guifg=#ffffff guibg=#000000  "提示不再是粉红色(pink)
        highlight Pmenu ctermfg=4 ctermbg=8 guifg=#ffffff guibg=#000000  "提示不再是粉红色(pink)


call vundle#end()
```
退出vim，然后重新打开vim，shift+;进入命令模式，输入PluginInstall等待插件安装完成。

启用F5运行的方法是在~/.vimrc中加入以下：
```bash
map <F5> :call CompileRunGcc()<CR>
func! CompileRunGcc()
        exec "w"
        if &filetype == 'c'
                exec "!g++ % -o %<"
                exec "!time ./%<"
        elseif &filetype == 'cpp'
                exec "!g++ % -o %<"
                exec "!time ./%<"
        elseif &filetype == 'java'
                exec "!javac %"
                exec "!time java %<"
        elseif &filetype == 'sh'
                :!time bash %
        elseif &filetype == 'python'
                exec "!clear"
                exec "!time python3 %"
        elseif &filetype == 'html'
                exec "!firefox % &"
        elseif &filetype == 'go'
                " exec "!go build %<"
                exec "!time go run %"
        elseif &filetype == 'mkd'
                exec "!~/.vim/markdown.pl % > %.html &"
                exec "!firefox %.html &"
        endif
endfunc
```

## 自动补全插件YouCompleteMe
首先把YouCompleteMe从GitHub上clone下来：
```bash
$ cd ~/.vim/bundle
$ git clone https://github.com/ycm-core/YouCompleteMe.git
$ cd YouCompleteMe
$ git submodule update --init --recursive
```
首先执行一下安装，直到屏幕上出现libclang的下载地址：
```bash
 $ ./install.py --all
 # ......一些信息
 Downloading libclang 11.0.0 from xxx
 # 上面的xxx就是我们需要下载的，一般是https://dl.bintray.com/ycm-core/libclang/libclang-11.0.0-x86_64-unknown-linux-gnu.tar.bz2
 # 按ctrl+C终止。自动下载要么连接超时要么下载下来的东西hash mismatch，我们得手动下载，然后改一下某个CMakeList.txt里面的东西。
 $ ^C
```
把libclang-11.0.0-x86_64-unknown-linux-gnu.tar.bz2下载下来，传到服务器上，然后解压：
```bash
$ tar -xjf libclang-11.0.0-x86_64-unknown-linux-gnu.tar.bz2
```
此时会得到一个lib文件夹，里面有```libclang.so.11```和```libclang.so```。假设这两个动态库文件在```/home/user1/soft/libclang/lib```目录下，记住这个目录。

然后我们需要修改```third_party/ycmd/cpp/ycm/CMakeLists.txt```这个文件：
```bash
$ cd ~/.vim/bundle/YouCompleteMe
$ vim ./third_party/ycmd/cpp/ycm/CMakeLists.txt
```
把打开的CMakeLists.txt从93行到117行全部加＃号注释掉，变成类似如下：
```bash
   # if( EXISTS "${LIBCLANG_LOCAL_FILE}" )
   #   file( SHA256 "${LIBCLANG_LOCAL_FILE}" LIBCLANG_LOCAL_SHA256 )

   #   if( "${LIBCLANG_LOCAL_SHA256}" STREQUAL "${LIBCLANG_SHA256}" )
   #     set( LIBCLANG_DOWNLOAD OFF )
   #   else()
   #     file( REMOVE "${LIBCLANG_LOCAL_FILE}" )
   #   endif()
   # endif()

   # if( LIBCLANG_DOWNLOAD )
   #   message( STATUS
   #            "Downloading libclang ${CLANG_VERSION} from ${LIBCLANG_URL}" )

   #   file(
   #     DOWNLOAD "${LIBCLANG_URL}" "${LIBCLANG_LOCAL_FILE}"
   #     SHOW_PROGRESS EXPECTED_HASH SHA256=${LIBCLANG_SHA256}
   #   )
   # else()
   #   message( STATUS "Using libclang archive: ${LIBCLANG_LOCAL_FILE}" )
   # endif()

   # # Copy and extract the Clang archive in the building directory.
   # file( COPY "${LIBCLANG_LOCAL_FILE}" DESTINATION "${CMAKE_CURRENT_BINARY_DIR}/../" )
   # execute_process( COMMAND ${CMAKE_COMMAND} -E tar -xjf ${LIBCLANG_FILENAME} )
```
然后对121行进行修改，把原来的
```bash
  file( GLOB_RECURSE PATH_TO_LIBCLANG ${CMAKE_BINARY_DIR}/libclang.* )
```
修改成```libclang.so.11```和```libclang.so```的目录：
```bash
  file( GLOB_RECURSE PATH_TO_LIBCLANG /home/user1/soft/libclang/lib/libclang.* )
```
保存退出。此时应当仍在```~/.vim/bundle/YouCompleteMe```目录下。执行安装：
```bash
./install.py --clang-completer
```
等它安装完，打开~/.vimrc，把```”Plugin ‘Valloric/YouCompleteMe```这一行最开头的英文双引号```"```去掉，保存退出。

最后重新进入vim，在命令模式下输入PluginInstall，把YouCompleteMe插件也安装上。

# 一些功能的快捷键举例
* 代码跳转：光标放到函数上，然后依次按下```\gd```
* 窗口切换：依次按下```]b```或者```[b```
* ...

# 使用ipdb单步调试
## 安装ipdb
```bash
python3 -m pip install ipdb --user
```
或者也可以在virtualenv中安装，可以避免一些冲突，也不需要--user了：
```bash
python3 -m virtualenv env_name
source env_name/bin/activate
python3 -m pip install ipdb
# 使用deactivate退出virtualenv
```
## 常用指令

指令| 功能
--- | :---:
h(elp) [指令] | 显示指定指令的帮助
run [参数] | 开始运行，可以指定运行参数
r(eturn) | 继续运行，直到当前函数返回
c(ont(inue)) | 继续运行，直到断点
n(ext) | 逐行执行
s(tep) | 逐行执行，碰到函数会进入
a(rgs) | 显示当前所处函数的参数
p 表达式 | 打印表达式的值
l(ist) [n[,m]] | 打印源代码第n到m行，不指定参数默认打印10行
w(here)  | 显示运行到哪儿
b(ack)t(race) | 显示call stack
u(p) [n] | 从当前所处frame上移n级
d(own) [n] | 从当前所处frame下移n级
b(reak) 文件名:行数 | 在文件的指定行设置断点
cl(ear) [断点序号] | 删除指定（或所有）的断点

中括号代表指令后可以接参数```X```，也可以不接，比如：
```bash
# 显示源代码中指定的行
# 当前行上下5行
$ list 
# 第10行的上下5行
$ list 10
# 第10行到第20行
$ list 10,20
```
没有中括号代表必须接参数```X```，比如：
```bash
# 打印sys.argv的值
$ p sys.argv
```
