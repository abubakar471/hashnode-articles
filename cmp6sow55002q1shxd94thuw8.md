---
title: "How to Set Up Neovim on Windows (2026): From Terminal to Full IDE"
seoTitle: "How to Set Up Neovim on Windows in 2026"
seoDescription: "Learn how to install Neovim on Windows, fix PowerShell errors, and set up a file explorer and dark themes. The complete guide to a pro Neovim IDE"
datePublished: 2026-05-15T10:49:16.223Z
cuid: cmp6sow55002q1shxd94thuw8
slug: how-to-set-up-neovim-on-windows-2026-from-terminal-to-full-ide
cover: https://cdn.hashnode.com/uploads/covers/69031e029c8c3e68582a8acc/7aadde4b-5dc9-484e-ab10-598c08437b52.jpg
ogImage: https://cdn.hashnode.com/uploads/og-images/69031e029c8c3e68582a8acc/c8c84989-5b3c-43f8-a7ce-b7d3df897239.jpg
tags: ides, vim, neovim, lua

---

Neovim is a fork of Vim that offers a modern, faster codebase with first-class Lua scripting support, built-in Language Server Protocol (LSP), and improved asynchronous functionality, allowing it to act as a lightweight IDE.

**Prerequisite**

Install [Node.js LTS version](https://nodejs.org/en/download) as some of the plugins will need this.

* * *

## **Step 1: Install Neovim on Windows**

First you have to install into your windows system, in windows open the powershell and give script execution permission using the command

```plaintext
Set-ExecutionPolicy RemoteSigned
```

Then, install neovim, by running this command. (Note: If you don't have winget, update your windows from settings.)

```plaintext
winget install neovim.neovim
```

### **Optional:**

If you get any error as, “winget : The term ‘winget’ is not recognized as the name of a cmdlet”, then run the following 3 commands

```plaintext
Install-PackageProvider -Name NuGet -Force | Out-Null
```

```plaintext
Install-Module -Name Microsoft.WinGet.Client -Force -Repository PSGallery | Out-Null
```

```plaintext
Repair-WinGetPackageManager
```

* * *

## **Step 2: Creating the vim configuration file**

After neovim is successfully installed, close the terminal and press (win + R) key type

%LocalAppData%

In this folder, create a nvim folder and inside that folder create a init.vim file and paste the following configuration in it.

```plaintext
" ====================================================================
" NEOVIM CONFIG
" Solarized Osaka UI, Telescope Integration, & Absolute Exit Logic
" ====================================================================

set nocompatible
filetype plugin indent on
syntax on

" --- BASIC SETTINGS ---
set number
set relativenumber
set mouse=a
set termguicolors
set cursorline
set hidden
set nobackup
set nowritebackup
set autowrite
set autowriteall 

let mapleader=" "

" Windows-specific Shell Fix
set shell=powershell
set shellcmdflag=-NoLogo\ -NoProfile\ -ExecutionPolicy\ RemoteSigned\ -Command
set shellquote=
set shellxquote=

" UX Improvements
set scrolloff=8
set sidescrolloff=8
set signcolumn=yes 
set updatetime=300
set shortmess+=c

" Indentation
set tabstop=4
set shiftwidth=4
set expandtab
set smartindent

" Clipboard
set clipboard=unnamedplus

" =========================
" 🔌 PLUGINS
" =========================
call plug#begin()

" UI & Theme (Solarized Osaka Vibe)
Plug 'craftzdog/solarized-osaka.nvim'
Plug 'nvim-lualine/lualine.nvim'
Plug 'akinsho/bufferline.nvim'
Plug 'nvim-tree/nvim-tree.lua'
Plug 'nvim-tree/nvim-web-devicons'
Plug 'akinsho/toggleterm.nvim'

" Telescope (File Finder & UI)
Plug 'nvim-lua/plenary.nvim'
Plug 'nvim-telescope/telescope.nvim', { 'tag': '0.1.x' }
Plug 'nvim-telescope/telescope-fzf-native.nvim', { 'do': 'make' }

" Syntax & Smart Suggestions
Plug 'nvim-treesitter/nvim-treesitter', {'do': ':TSUpdate'}
Plug 'neoclide/coc.nvim', {'branch': 'release'}

call plug#end()

" =========================
" 🎨 THEME & LUA CONFIG
" =========================
lua << EOF
-- 1. Setup Solarized Osaka
local status_osaka, osaka = pcall(require, "solarized-osaka")
if status_osaka then
    osaka.setup({
        transparent = true, 
        terminal_colors = true,
        styles = {
            comments = { italic = true },
            keywords = { italic = true },
            functions = { bold = true },
            variables = { italic = true },
        },
        sidebars = { "qf", "help", "nvim-tree" },
        day_brightness = 0.3,
    })
end
vim.cmd('colorscheme solarized-osaka')

-- 2. Setup Lualine
require('lualine').setup({ 
    options = { 
        theme = "solarized-osaka", 
        globalstatus = true 
    }
})

-- 3. UI Components
require("bufferline").setup{}
require("nvim-tree").setup{}

-- 4. ToggleTerm
require("toggleterm").setup({
    open_mapping = [[<c-\>]],
    direction = "horizontal",
    size = 15,
    insert_mappings = true,
    terminal_mappings = true,
    start_in_insert = true,
    on_open = function(term)
        vim.cmd("startinsert!")
        vim.api.nvim_buf_set_keymap(term.bufnr, "n", "q", "<cmd>close<CR>", {noremap = true, silent = true})
    end,
})

-- 5. Telescope (Beautiful Floating UI)
require('telescope').setup({
  defaults = {
    layout_strategy = "horizontal",
    layout_config = {
      prompt_position = "top",
      preview_width = 0.55,
    },
    sorting_strategy = "ascending",
    winblend = 10,
    borderchars = { "─", "│", "─", "│", "┌", "┐", "┘", "└" },
  }
})
pcall(require('telescope').load_extension, 'fzf')

-- 6. Treesitter
local status_ts, ts = pcall(require, "nvim-treesitter.configs")
if status_ts then
    ts.setup({
        ensure_installed = { "lua", "python", "javascript", "typescript", "cpp", "java" },
        highlight = { enable = true },
    })
end
EOF

" Transparency Fix
function! AdaptColorscheme()
   highlight Normal guibg=NONE ctermbg=NONE
   highlight NonText guibg=NONE ctermbg=NONE
   highlight SignColumn guibg=NONE ctermbg=NONE
   highlight EndOfBuffer guibg=NONE ctermbg=NONE
endfunction
autocmd ColorScheme * call AdaptColorscheme()
doautocmd ColorScheme solarized-osaka

" =========================
" 🧠 SMART CODE SUGGESTIONS (COC)
" =========================
inoremap <silent><expr> <TAB> coc#pum#visible() ? coc#pum#next(1) : CheckBackspace() ? "\<Tab>" : coc#refresh()
inoremap <expr><S-TAB> coc#pum#visible() ? coc#pum#prev(1) : "\<C-h>"
function! CheckBackspace() abort
  let col = col('.') - 1
  return !col || getline('.')[col - 1]  =~# '\s'
endfunction
inoremap <silent><expr> <CR> coc#pum#visible() ? coc#pum#confirm() : "\<C-g>u\<CR>\<c-r>=coc#on_enter()\<CR>"

nmap <silent> [g <Plug>(coc-diagnostic-prev)
nmap <silent> ]g <Plug>(coc-diagnostic-next)
nmap <silent> gd <Plug>(coc-definition)
nnoremap <silent> K :call ShowDocumentation()<CR>

function! ShowDocumentation()
  if CocAction('hasProvider', 'hover') | call CocActionAsync('doHover')
  else | call feedkeys('K', 'in') | endif
endfunction

let g:coc_global_extensions = ['coc-pyright', 'coc-tsserver', 'coc-lua', 'coc-clangd', 'coc-json']

" =========================
" 🛠️ KEYMAPS & NAV FIXES
" =========================

" Telescope Fuzzy Finder
nnoremap <leader>ff <cmd>Telescope find_files<cr>
nnoremap <leader>fg <cmd>Telescope live_grep<cr>
nnoremap <leader>fb <cmd>Telescope buffers<cr>

" Auto-format on save
nnoremap <leader>f :call CocAction('format')<CR>
autocmd BufWritePre * silent! call CocAction('format')

" Sidebar
nnoremap <leader>e :NvimTreeToggle<CR>

" Buffer Nav
nnoremap <A-j> :bnext<CR>
nnoremap <A-k> :bprevious<CR>
nnoremap <leader>q :bd<CR>

" 💾 SMART SAVE & ABSOLUTE QUIT
nnoremap <C-s> :wa<CR>
inoremap <C-s> <Esc>:wa<CR>a
vnoremap <C-s> <Esc>:wa<CR>gv

nnoremap <leader>w :wa<CR>:qa<CR>
nnoremap <leader>Q :qa!<CR>

" Command Abbreviations
cnoreabbrev q qa
cnoreabbrev wq wqa

" 🧭 WINDOW NAVIGATION
nnoremap <A-Left> <C-w>h
nnoremap <A-Right> <C-w>l
nnoremap <A-Up> <C-w>k
nnoremap <A-Down> <C-w>j

tnoremap <A-Left> <C-\><C-n><C-w>h
tnoremap <A-Right> <C-\><C-n><C-w>l
tnoremap <A-Up> <C-\><C-n><C-w>k
tnoremap <A-Down> <C-\><C-n><C-w>j
tnoremap <Esc> <C-\><C-n>
```

* * *

## **Step 3: How to Fix Terminal Errors on Windows**

Now, open the powershell and install the vim-plug this is needed as it is a plugin manager for neovim.

Goto the [https://github.com/junegunn/vim-plug](https://github.com/junegunn/vim-plug) and see the installation command for windows. After you have installed the vim-plug now it is time to install

Now, open the powershell and type

```plaintext
nvim
```

this will open neovim and you will see some errors but ignore them just press ( : ) key to go into the command mode and type the command

```plaintext
PlugInstall
```

and then again type (:) and in the command mode type

```plaintext
TSUpdate
```

This will install all the necessary plugins mentioned in your inti.vim configuration file. Once installed you can close the powershell and open again to just restart neovim in a fresh state.

* * *

## **Step 4: Add Language support and Autocompletion**

Once you open neovim again type ( : ) and run this command

```plaintext
CocInstall coc-pyright coc-tsserver coc-lua coc-java coc-json coc-html coc-css coc-clangd coc-snippets
```

This will install language support for popular languages. After that close neovim open it again. and type : and run the below command

```plaintext
CocConfig
```

A config file will open paste the below config there, and save by going into the command mode pressing ( : ) key and type (:wq)

```json
{
    "suggest.noselect": false,
    "suggest.enablePreview": true,
    "suggest.triggerAfterInsertEnter": true,
    "suggest.minTriggerInputLength": 1,
    "languageserver": {
        "clangd": {
            "command": "clangd",
            "rootPatterns": [
                "compile_flags.txt",
                "compile_commands.json",
                ".git/",
                ".clangd"
            ],
            "filetypes": [
                "c",
                "cc",
                "cpp",
                "objc",
                "objcpp"
            ],
            "args": [
                "--query-driver=C:/msys64/ucrt64/bin/g++.exe",
                "--background-index",
                "--clang-tidy",
                "--header-insertion=never"
            ]
        }
    },
    "clangd.arguments": [
        "--query-driver=C:/msys64/ucrt64/bin/g++.exe"
    ]
}
```

After you save the file, close neovim and reopen.

Now, you will see all the issues and errors are gone you can now use neovim with a more optimized features. Your Neovim is now ready to use.

* * *

## **Looking for more aesthetic look?**

Do this then...

Search the font JetBrainsMono from this page, then download and install the JetBrainsMono font face first : [JetBrainsMon Nerd Font](https://www.nerdfonts.com/font-downloads)

Open your powershell and press (Ctrl + , ) key it will open up the powershell settings.

There choose the profile powershell > Appearance

1.  Set the Transparency to 72%
    
2.  Enable acrylic material
    
3.  Set the font face to “JetBrainsMono Nerd Font” (Make sure to install the font first)
    

You will get a final look like this :

![](https://cdn.hashnode.com/uploads/covers/69031e029c8c3e68582a8acc/c44f6d0e-723d-4ba6-a093-5685bb5ad696.png align="center")

* * *

## Additional (Neovim Keymap Guide)

### 🔍 File Search & Navigation

| Key | What it does | Simple explanation |
| --- | --- | --- |
| `Space + f + f` | Find files | Search and open any file quickly |
| `Space + f + g` | Live grep | Search text inside all project files |
| `Space + f + b` | Buffers | See all open files and switch between them |
| `Space + e` | Toggle file explorer | Open/close project sidebar |

### 📁 File Operations (inside file explorer = space + e)

| Key | Action | What it does |
| --- | --- | --- |
| `a` | Create | Create new file/folder |
| `d` | Delete | Delete selected file/folder |
| `r` | Rename | Rename file/folder |
| `x` | Cut | Mark file to move |
| `c` | Copy | Mark file to copy |
| `p` | Paste | Paste copied/cut file |
| `R` | Refresh | Reload tree |
| `Enter` | Open file | Open selected file |
| `Ctrl + ]` | Create folder recursively | Make nested folders |

### 💾 Save / Quit

| Key | What it does | Simple explanation |
| --- | --- | --- |
| `Ctrl + S` | Save all files | Save everything instantly |
| `Space + w` | Save all and quit Neovim | Save then completely exit |
| `Space + Q` | Force quit | Exit without saving |
| `:q` | Quit | Actually quits **all windows** (because of abbreviation) |
| `:wq` | Save and quit | Saves everything then exits |

❗**Important:**  
In this config:

`q` becomes `qa`

So typing `:q` quits all. So, do Ctrl + s before typing :q if you want to save the files opened right now.

### 📂 Buffer Navigation (Open Files)

| Key | What it does |
| --- | --- |
| `Alt + J` | Next file |
| `Alt + K` | Previous file |
| `Space + q` | Close current file |

### 🔳 Window Navigation (Split Screens)

Move between split windows.

| Key | Move to |
| --- | --- |
| `Alt + Left` | Left split |
| `Alt + Right` | Right split |
| `Alt + Up` | Upper split |
| `Alt + Down` | Lower split |

Works in:

*   Normal mode
    
*   Terminal mode
    

### 💻 Terminal

| Key | What it does |
| --- | --- |
| `Ctrl + \` | Open terminal |
| `Esc` | Exit terminal insert mode |
| `q` | Close terminal window |

### 🧠 Code Intelligence (CoC)

Autocomplete features

| Key | What it does |
| --- | --- |
| `Tab` | Next suggestion |
| `Shift + Tab` | Previous suggestion |
| `Enter` | Confirm suggestion |

### 🚩 Code Navigation

| Key | What it does |
| --- | --- |
| `g + d` | Go to definition |
| `K` | Show documentation / hover info |

Example:

Cursor on a function name → press `gd`

It jumps to where that function is defined.

### ❌Diagnostics (Errors / Warnings)

| Key | What it does |
| --- | --- |
| `[g` | Previous error |
| `]g` | Next error |

### 🎨 Formatting

| Key | What it does |
| --- | --- |
| `Space + f` | Format current file |

Also, Formatting happens automatically when saving.

### 💬 Insert Mode Shortcuts

These work while typing.

| Key | What it does |
| --- | --- |
| `Ctrl + S` | Save without leaving insert mode |
| `Tab` | Autocomplete / indent |
| `Shift + Tab` | Previous suggestion |

### 👁‍🗨 Visual Mode

| Key | What it does |
| --- | --- |
| v | To go into visual mode press v and you can use the arrows to select multiple lines |
| `Ctrl + S` | Save while keeping selection |

### 🖱 Mouse Support

This config has:

`set mouse=a`

So you can:

*   Click to move cursor
    
*   Scroll
    
*   Resize splits
    

* * *

That was all. Thanks for reading so far. You can also subscribe to my newsletter visiting the following link:

[Abu Bakar's Substack](http://abubakardevs.substack.com/)