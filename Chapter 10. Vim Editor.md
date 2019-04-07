# Chapter 10. Vim Editor
## 1. Why Vim?

There are many editors, like __emacs__, __nano__, __pico__...

Why Vim?

- All *nix system have built-in Vim editor.
- Other program will call Vim's interface, like __crontab__, __visudo__, __edquota__. 
- Progamming-friendly?
- Simple and fast.

Vim is an advanced version of **Vi**, which is equipped with grammar checking and color highlight.

## 2. Vi

### Edit-Mode, Normal-Mode, Command-Mode.

- Edit-Mode: Typing and modifying the file.
- Normal-Mode: Reading, deleting, copying and pasting the contents of the file.
- Command-Mode: Loading, storing, replacing, quiting and so on.

**Edit-Mode <--( [i,o,a,R] | [Esc] )--> Normal Mode <--( [Esc] | [/,?,:] )--> Command-Mode**

### Normal-Mode

1. Move

| Key         | Description |
| ----------- | ----------- |
| **h** | :arrow_left:  |
| **j** | :arrow_down:  |
| **k** | :arrow_up:    |
| **l** | :arrow_right: |
| [30j] | Jump 30 lines |
| **Ctrl + f** | Next Page |
| **Ctrl + b** | Previous Page |
| 20<Space> | Jump 20 Chars |
| 20<Enter> | Jump 20 Lines |
| 20G | Jump to the 20th Line |
| **0** | Head of the Line |
| **$** | Tail of the Line |
| **G** | End of File |
| **gg** | Jump tp the first Line |

2. Copy and Paste

| Key         | Description |
| ----------- | ----------- |
| **x** | [Del] |
| **X** | [Backspace] |
| 20x | [Del]*20 |
| **dd** | delete Line |
| 20dd | delete 20 Lines |
| **yy** | copy Line |
| 20yy | copy 20 Lines |
| **p** | paste from next line |
| P | paste from previous line |

3. Undo, Redo and Repeat

| Key | Description |
| --- | ----------- |
| **_u_** | Undo |
| **_Ctrl + r_** | Redo |
| **_._** | Repeat |

4. Search and Replace

| Key | Description |
| --- | ----------- |
| **_/word_** | search **word** after cursor |
| **_?word_** | search **word** before cursor |
| **_n_** | next search result |
| **_N_** | last search result |
| **_:100,200s/word1/word2/gc_** | search from 100th line to 200th line for **word1** and replace to **word2** |
| :1,$s/word1/word2/g | search from first line to last line ... |

### Edit-Mode

From Normal-Mode to Edit-Mode

| Key | Description |
| --- | ----------- |
| **i** | insert from current cursor |
| I | insert from the first Non-space char of current line |
| a | insert from the next char |
| **A** | insert from the tail of current line |
| **o** | insert from next new Line |
| **O** | insert from last new Line |
| r | replace one Char |
| R | replace until you press [Esc] |

### Command-Mode

From Normal-Mode to Command-Mode

| Key | Description | 
| --- | ----------- |
| **:wq** | Write to Harddisk and quit |
| :q! | Quit and not Store |
| :w [Filename] | Store the file as a new file |
| :! [Command] | Leave vi temporarily and display the result of execution [Command]. |
| :set nu | display Line Number |
| :set nonu | not display Line Number |

## 3. Vim

### Block operation

| Key | Description |
| --- | ----------- |
| **v** | Char selection |
| **V** | Line selection |
| **_Ctrl + v_** | Block selection(Rectangle) |
| **y** | Copy selected part |
| **d** | Delete selected part |

### Multpile files editing

| Key | Description |
| --- | ----------- |
| :n | Edit next file |
| :N | Edit last file |
| :files | List all opened files |

### Multiple Windows

| Key | Description |
| --- | ----------- |
| :sp | Same file in two windows |
| :sp [Filename] | New file in splited window |
| Ctrl + w, j | Next window |
| Ctrl + w, k | Last window |

### Vim Configuration
| Config | Description |
| ------ | ----------- |
| :set nu | display Line Number |
| :set nonu | not display Line Number |
| :set autoindent | Automatic indent |
| :set ruler | right-down side status |
| :syntax on | Grammar highlight |
| :syntax off | no highlight(Text file) |
| :set bg=dark | dark style |
| :set bg= light | light style |

## 4. Note

### Chinese character code
```
LANG=.....
```
### Code of [Enter/Return]
```
dos2unix [-kn] file [newfile]
unix2dos [-kn] file [newfile]
# -k keep [mtime] unmodified 
# -n keep original file and output to [newfile]
```
