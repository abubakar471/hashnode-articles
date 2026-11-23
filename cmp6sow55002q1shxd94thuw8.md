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

---

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

---

## **Step 2: Creating the vim configuration file**

After neovim is successfully installed, close the terminal and press (win + R) key type

%LocalAppData%

In this folder, create a nvim folder and inside that folder create a init.vim file and paste the following configuration in it.

```plaintext
" ====================================================================
" NEOVIM CONFIG (Windows Compatible & Optimized)
" Everforest Theme, Splits, Telescope, Bufferline, & Live Inline Diagnostics
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

" --- WINDOWS-SPECIFIC SHELL FIX (Complete) ---
if has('win32') || has('win64')
    set shell=powershell
    set shellcmdflag=-NoLogo\ -NoProfile\ -ExecutionPolicy\ RemoteSigned\ -Command\ [Console]::InputEncoding=[Console]::OutputEncoding=[System.Text.Encoding]::UTF8;
    set shellredir=-RedirectStandardOutput\ %s\ -NoNewWindow\ -Wait
    set shellpipe=2>&1\ \|\ Out-File\ -Encoding\ UTF8\ %s;\ exit\ $LastExitCode
    set shellquote=
    set shellxquote=
endif

" UX Improvements
set scrolloff=8
set sidescrolloff=8
set signcolumn=yes
" Blank space past end of buffer (keep trailing space after eob:\)
set fillchars+=eob:\ 
set updatetime=250
set shortmess+=c

" Indentation
set tabstop=4
set shiftwidth=4
set expandtab
set smartindent

" Clipboard (Uses Windows win32yank / native clipboard automatically)
set clipboard=unnamedplus

" =========================
" 🔌 PLUGINS
" =========================
call plug#begin(stdpath('data') . '/plugged')

" UI & Theme (Everforest)
Plug 'sainnhe/everforest'
Plug 'nvim-lualine/lualine.nvim'
Plug 'akinsho/bufferline.nvim'
Plug 'nvim-tree/nvim-tree.lua'
Plug 'nvim-tree/nvim-web-devicons'
Plug 'akinsho/toggleterm.nvim'

" Telescope (File Finder & UI)
Plug 'nvim-lua/plenary.nvim'
Plug 'nvim-telescope/telescope.nvim', { 'tag': '0.1.x' }
Plug 'nvim-telescope/telescope-fzf-native.nvim', { 'do': 'cmake -S. -Bbuild -DCMAKE_BUILD_TYPE=Release && cmake --build build --config Release && cmake --install build --prefix build' }

" Syntax & Smart Suggestions
Plug 'nvim-treesitter/nvim-treesitter', {'do': ':TSUpdate'}
Plug 'neoclide/coc.nvim', {'branch': 'release'}

call plug#end()

" =========================
" 🎨 EVERFOREST THEME SETTINGS
" =========================
set background=dark
let g:everforest_background = 'medium'      " Options: 'soft', 'medium', 'hard'
let g:everforest_transparent_background = 2   " 0 = opaque, 1 = main bg transparent, 2 = include floating/popup bg
let g:everforest_better_performance = 1
let g:everforest_diagnostic_text_highlight = 1
let g:everforest_diagnostic_line_highlight = 0
let g:everforest_diagnostic_virtual_text = 'colored'

colorscheme everforest

" =========================
" 🎨 LUA CONFIG
" =========================
lua << EOF
-- 0. Guard for older/newer Treesitter ft_to_lang lookups used by plugins
if vim.treesitter and not vim.treesitter.ft_to_lang and vim.treesitter.language then
    vim.treesitter.ft_to_lang = function(ft)
        return vim.treesitter.language.get_lang(ft) or ft
    end
end

-- Keep splits, floating windows, and gutters transparent over Windows Terminal
local transparent_groups = {
    "Normal",
    "NormalNC",
    "NormalFloat",
    "FloatBorder",
    "NvimTreeNormal",
    "NvimTreeNormalNC",
    "EndOfBuffer",
    "SignColumn",
}
for _, group in ipairs(transparent_groups) do
    vim.api.nvim_set_hl(0, group, { bg = "none" })
end

-- Dim or hide the vertical border line between NvimTree and split windows
-- To hide it completely instead of dimming, change fg to "#1e2220" or add `set fillchars+=vert:\ `
vim.api.nvim_set_hl(0, "WinSeparator", { fg = "#1d201f", bg = "none", nocombine = true })
vim.api.nvim_set_hl(0, "VertSplit",    { fg = "#1d201f", bg = "none", nocombine = true })

-- Primeagen-style live inline diagnostics (Everforest palette)
vim.api.nvim_set_hl(0, "CocErrorVirtualText",   { fg = "#e67e80", bg = "none", bold = true })
vim.api.nvim_set_hl(0, "CocWarningVirtualText", { fg = "#dbbc7f", bg = "none", italic = true })
vim.api.nvim_set_hl(0, "CocInfoVirtualText",    { fg = "#7fbbb3", bg = "none", italic = true })
vim.api.nvim_set_hl(0, "CocHintVirtualText",    { fg = "#a7c080", bg = "none", italic = true })

vim.api.nvim_set_hl(0, "CocErrorHighlight",     { sp = "#e67e80", undercurl = true })
vim.api.nvim_set_hl(0, "CocWarningHighlight",   { sp = "#dbbc7f", undercurl = true })
vim.api.nvim_set_hl(0, "CocErrorSign",          { fg = "#e67e80", bg = "none", bold = true })
vim.api.nvim_set_hl(0, "CocWarningSign",        { fg = "#dbbc7f", bg = "none", bold = true })

-- 1. Setup Lualine
local status_lualine, lualine = pcall(require, "lualine")
if status_lualine then
    lualine.setup({
        options = {
            theme = "everforest",
            globalstatus = true
        }
    })
end

-- 2. UI Components (Bufferline & Nvim-Tree)
local status_bufferline, bufferline = pcall(require, "bufferline")
if status_bufferline then
    bufferline.setup({
        options = {
            mode = "buffers",
            numbers = "ordinal",
            diagnostics = "coc",
            separator_style = "slant",
            show_buffer_close_icons = true,
            show_close_icon = false,
        }
    })
end

local status_tree, nvim_tree = pcall(require, "nvim-tree")
if status_tree then
    nvim_tree.setup({})
end

-- 3. ToggleTerm (Controlled via Ctrl+/ and Ctrl+_ mappings below)
local status_term, toggleterm = pcall(require, "toggleterm")
if status_term then
    toggleterm.setup({
        shell = "C:/PROGRA~1/Git/bin/bash.exe --login -i",
        direction = "horizontal",
        size = 15,
        insert_mappings = true,
        terminal_mappings = true,
        start_in_insert = true,
        on_open = function(term)
            vim.cmd("startinsert!")
            vim.api.nvim_buf_set_keymap(term.bufnr, "n", "q", "<cmd>close<CR>", { noremap = true, silent = true })
        end,
    })
end

-- 4. Telescope (Disable TS previewer highlight to prevent nvim-treesitter clashes)
local status_telescope, telescope = pcall(require, "telescope")
if status_telescope then
    telescope.setup({
        defaults = {
            layout_strategy = "horizontal",
            layout_config = {
                prompt_position = "top",
                preview_width = 0.55,
            },
            sorting_strategy = "ascending",
            winblend = 15,
            preview = {
                treesitter = false,
            },
        }
    })
    pcall(telescope.load_extension, "fzf")
end

-- 5. Treesitter (Safe setup for both legacy configs and newer nvim-treesitter)
local status_ts_install, ts_install = pcall(require, "nvim-treesitter.install")
if status_ts_install then
    ts_install.prefer_git = false
    ts_install.compilers = { "zig", "clang", "gcc", "cl" }
end

local status_ts_configs, ts_configs = pcall(require, "nvim-treesitter.configs")
if status_ts_configs then
    ts_configs.setup({
        ensure_installed = { "lua", "python", "javascript", "typescript", "cpp", "java" },
        highlight = { enable = true },
    })
elseif vim.treesitter and vim.treesitter.start then
    vim.api.nvim_create_autocmd("FileType", {
        pattern = { "lua", "python", "javascript", "typescript", "cpp", "java" },
        callback = function(args)
            pcall(vim.treesitter.start, args.buf)
        end,
    })
end
EOF

" =========================
" 💾 AUTO-SAVE ON SWITCH
" =========================
augroup AutoSaveOnSwitch
    autocmd!
    autocmd BufLeave,FocusLost * silent! wa
augroup END

" =========================
" 🧠 SMART CODE SUGGESTIONS & DIAGNOSTICS (COC)
" =========================
inoremap <silent><expr> <TAB> coc#pum#visible() ? coc#pum#next(1) : CheckBackspace() ? "\<Tab>" : coc#refresh()
inoremap <silent><expr> <S-TAB> coc#pum#visible() ? coc#pum#prev(1) : "\<C-h>"
function! CheckBackspace() abort
  let col = col('.') - 1
  return !col || getline('.')[col - 1]  =~# '\s'
endfunction
inoremap <silent><expr> <CR> coc#pum#visible() ? coc#pum#confirm() : "\<C-g>u\<CR>\<c-r>=coc#on_enter()\<CR>"

nmap <silent> [g <Plug>(coc-diagnostic-prev)
nmap <silent> ]g <Plug>(coc-diagnostic-next)
nmap <silent> gd <Plug>(coc-definition)
nnoremap <silent> K :call ShowDocumentation()<CR>
nnoremap <silent> <leader>d :CocList diagnostics<CR>

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

" Manual & Auto-format
nnoremap <leader>f :call CocAction('format')<CR>
autocmd BufWritePre * silent! call CocAction('format')

" Sidebar Tree Toggle
nnoremap <leader>e :NvimTreeToggle<CR>

" ==========================================================
" 🗂️ SPLIT VIEW MANAGEMENT
" ==========================================================
nnoremap <leader>sv :vsplit<CR>
nnoremap <leader>sh :split<CR>
nnoremap <leader>sb :vsplit<CR>:bnext<CR>
nnoremap <leader>sc :close<CR>
nnoremap <leader>s= <C-w>=
nnoremap <leader>s> :vertical resize +5<CR>
nnoremap <leader>s< :vertical resize -5<CR>

" ==========================================================
" 📑 PROFESSIONAL TAB & BUFFER NAVIGATION
" ==========================================================
nnoremap <TAB> :bnext<CR>
nnoremap <S-TAB> :bprevious<CR>
nnoremap <S-l> :bnext<CR>
nnoremap <S-h> :bprevious<CR>

" Clean buffer close
nnoremap <leader>x :bdelete<CR>
nnoremap <leader>q :bd<CR>

" Jump directly to tabs 1-9
nnoremap <leader>1 <Cmd>BufferLineGoToBuffer 1<CR>
nnoremap <leader>2 <Cmd>BufferLineGoToBuffer 2<CR>
nnoremap <leader>3 <Cmd>BufferLineGoToBuffer 3<CR>
nnoremap <leader>4 <Cmd>BufferLineGoToBuffer 4<CR>
nnoremap <leader>5 <Cmd>BufferLineGoToBuffer 5<CR>
nnoremap <leader>6 <Cmd>BufferLineGoToBuffer 6<CR>
nnoremap <leader>7 <Cmd>BufferLineGoToBuffer 7<CR>
nnoremap <leader>8 <Cmd>BufferLineGoToBuffer 8<CR>
nnoremap <leader>9 <Cmd>BufferLineGoToBuffer $<CR>

" 💾 SMART SAVE & ABSOLUTE QUIT
nnoremap <C-s> :wa<CR>
inoremap <C-s> <Esc>:wa<CR>a
vnoremap <C-s> <Esc>:wa<CR>gv

nnoremap <leader>w :wa<CR>:qa<CR>
nnoremap <leader>Q :qa!<CR>

cnoreabbrev q qa
cnoreabbrev wq wqa

" 🧭 WINDOW NAVIGATION & TERMINAL MAPPINGS
nnoremap <A-Left> <C-w>h
nnoremap <A-Right> <C-w>l
nnoremap <A-Up> <C-w>k
nnoremap <A-Down> <C-w>j

" Toggle terminal with Ctrl+/ (plus <C-_> fallback for Windows Terminal)
nnoremap <C-/> <cmd>ToggleTerm<CR>
inoremap <C-/> <Esc><cmd>ToggleTerm<CR>
tnoremap <C-/> <C-\><C-n><cmd>ToggleTerm<CR>

nnoremap <C-_> <cmd>ToggleTerm<CR>
inoremap <C-_> <Esc><cmd>ToggleTerm<CR>
tnoremap <C-_> <C-\><C-n><cmd>ToggleTerm<CR>

tnoremap <A-Left> <C-\><C-n><C-w>h
tnoremap <A-Right> <C-\><C-n><C-w>l
tnoremap <A-Up> <C-\><C-n><C-w>k
tnoremap <A-Down> <C-\><C-n><C-w>j
tnoremap <Esc> <C-\><C-n>
```

---

## **Step 3: How to Fix Terminal Errors on Windows**

Now, open the powershell and install the vim-plug this is needed as it is a plugin manager for neovim.

Goto the <https://github.com/junegunn/vim-plug> and see the installation command for windows. After you have installed the vim-plug now it is time to install

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

---

## **Step 4: Add Language support and Autocompletion**

Once you open neovim again type ( : ) and run this command

```plaintext
CocInstall coc-pyright coc-tsserver coc-lua coc-java coc-json coc-html coc-css coc-clangd coc-snippets
```

#### (Optional): if you are using c++, you might need to do this step:

Instatall clangd globally in your windows machine:

### Install LLVM/`clangd` Externally and Point `coc-clangd` to It

**Install LLVM (includes** `clangd.exe`**) via PowerShell / Windows Terminal:** Run:

```plaintext
winget install -e --id LLVM.LLVM
```

*(If you already have MSYS2/MinGW installed, you can install* `clangd` *there instead, e.g.,* `pacman -S mingw-w64-ucrt-x86_64-clang-tools-extra`*.)Verification:* Open a **new** PowerShell window and run `clangd --version` (or check that `C:\Program Files\LLVM\bin\clangd.exe` exists).

This will install language support for popular languages. After that close neovim open it again. and type : and run the below command

```plaintext
CocConfig
```

A config file will open paste the below config there, and save by going into the command mode pressing ( : ) key and type (:wq)

```json
{
  "json.schemaDownload.trustedDomains": {
    "https://www.schemastore.org": true
  },
  "diagnostic.enable": true,
  "diagnostic.refreshOnInsertMode": true,
  "diagnostic.checkCurrentLine": true,
  "diagnostic.virtualText": true,
  "diagnostic.virtualTextCurrentLineOnly": false,
  "diagnostic.virtualTextPrefix": " ■■■ ",
  "diagnostic.virtualTextLimitInOneLine": 80,
  "diagnostic.messageTarget": "float",
  "diagnostic.autoRefresh": true,
  "clangd.path": "C:/Program Files/LLVM/bin/clangd.exe",
  "clangd.arguments": [
    "--background-index",
    "--clang-tidy",
    "--header-insertion=never"
  ],
  "clangd.fallbackFlags": [
    "-std=c++20"
  ]
}
```

After you save the file, close neovim and reopen.

Now, you will see all the issues and errors are gone you can now use neovim with a more optimized features. Your Neovim is now ready to use.

---

## **Looking for more aesthetic look?**

Do this then...

Search the font JetBrainsMono from this page, then download and install the JetBrainsMono font face first : [JetBrainsMon Nerd Font](https://www.nerdfonts.com/font-downloads)

Open your powershell and press (Ctrl + , ) key it will open up the powershell settings.

There choose the profile powershell > Appearance

1. Set the Transparency to 72%
2. Enable acrylic material
3. Set the font face to “JetBrainsMono Nerd Font” (Make sure to install the font first)

You will get a final look like this :

![](https://cdn.hashnode.com/uploads/covers/69031e029c8c3e68582a8acc/c44f6d0e-723d-4ba6-a093-5685bb5ad696.png align="center")

---

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

❗**Important:**\
In this config:

`q` becomes `qa`

So typing `:q` quits all. So, do Ctrl + s before typing :q if you want to save the files opened right now.

### 📂 Buffer Navigation (Open Files)

| Key | What it does |
| --- | --- |
| space + tab | Next file |
| space + (0-9) | switch between files |
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

- Normal mode
- Terminal mode

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

### Split-View Shortcuts Added

- **Vertical split (side-by-side):** `<leader>sv` (`Space` + `s` + `v`) — opens a new vertical split; in `nvim-tree`, press `v` on a file to open it in a vertical split.
- **Horizontal split (top/bottom):** `<leader>sh` (`Space` + `s` + `h`) — opens a new horizontal split; in `nvim-tree`, press `x` on a file to open it in a horizontal split.
- **Open current buffer in a split:** `<leader>sb` (`Space` + `s` + `b`) — splits vertically and switches to the next buffer so you can view two files side by side quickly.
- **Resize splits:** `<leader>s=` (`Space` + `s` + `=`) equalizes split sizes; `<leader>s>` / `<leader>s<` widens/narrows vertical splits (`<C-w>>` / `<C-w><` also work natively).
- **Close current split pane:** `<leader>sc` (`Space` + `s` + `c`) or `<C-w>c`.

### 🖱 Mouse Support

This config has:

`set mouse=a`

So you can:

- Click to move cursor
- Scroll
- Resize splits

---

That was all. Thanks for reading so far. You can also subscribe to my newsletter visiting the following link:

👉 [Abu Bakar's Substack](http://abubakardevs.substack.com/)