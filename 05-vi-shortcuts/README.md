# VI / Vim Editor Absolute Essentials

Vim (Vi Improved) is a terminal-based text editor built into almost every Linux machine. Unlike standard text editors where you type text immediately, Vim is a **modal editor**. It operates in distinct modes, meaning the keys on your keyboard change functionality entirely depending on your current mode.

---

### 🔄 The Three Core Modes of Vim

1.  **Normal Mode (The Control Center):** The default mode when you open a file. You cannot type text here. Instead, your keyboard keys act as commands for lightning-fast navigation, copying, pasting, and deleting. Press `Esc` at any time to return to this mode.
2.  **Insert Mode (The Drafting Table):** The standard mode where you type text exactly like a regular word processor. You enter this mode from Normal Mode by pressing `i` (or other insertion shortcuts). 
3.  **Command Mode (The File Manager):** Used for administrative actions like saving, exiting, searching, and configuring editor properties. You enter this mode from Normal Mode by typing a colon (`:`).


---

### Basic Navigation
- `h` – Move **left**  
- `l` – Move **right**  
- `j` – Move **down**  
- `k` – Move **up**  
- `0` – Move to the **beginning** of the line  
- `^` – Move to the **first non-blank** character of the line  
- `$` – Move to the **end** of the line  
- `w` – Move to the **next word**  
- `b` – Move to the **previous word**  
- `gg` – Move to the **start** of the file  
- `G` – Move to the **end** of the file  
- `:n` – Move to **line number `n`**  

---

### Insert Mode Shortcuts
- `i` – Insert before cursor  
- `I` – Insert at the beginning of the line  
- `a` – Append after cursor  
- `A` – Append at the end of the line  
- `o` – Open a new line below  
- `O` – Open a new line above  
- `Esc` – Exit insert mode  

---

### ✂️ 3. Editing, Yanking (Copying), & Pasting

> 💡 **Vim Concept:** In Vim, "deleting" text actually cuts it into a hidden clipboard (register). This means any delete command can also double as a cut-and-paste action.

*   `x` – Delete (cut) the single character sitting **directly under** the cursor.
*   `X` – Delete (cut) the character located **directly before** the cursor.
*   `dw` – Delete from the cursor position to the start of the **next word**.
*   `dd` – Delete (cut) the **entire current line**.
*   `D` (or `d$`) – Delete everything from your **current cursor position to the end** of the line.
*   `d0` – Delete everything from your **current cursor position back to the start** of the line.
*   `u` – **Undo** your last editing action (can be pressed repeatedly).
*   `Ctrl + r` – **Redo** an action that you accidentally undid.
*   `yy` – **Yank (Copy)** the entire current line into memory.
*   `yw` – **Yank (Copy)** the specific word ahead of the cursor.
*   `p` – **Paste** your copied or cut text **after** the cursor (or on the line below).
*   `P` – **Paste** your copied or cut text **before** the cursor (or on the line above).


---

### Search and Replace
- `/pattern` – Search **forward** for a pattern  
- `?pattern` – Search **backward** for a pattern  
- `n` – Jump to the **next occurrence** of your active search pattern.
- `N` – **previous occurrence**
- `:%s/old/new/g` – Replace **all occurrences** of "old" with "new"  
- `:s/old/new/g` – Replace **all occurrences** in the current line  

---

### Working with Multiple Files
- `:e filename` – Open a **new file**  
- `:w` – Save file  
- `:wq` – Save and exit  
- `:q!` – Quit **without saving**  
- `:split filename` – Split screen **horizontally** and open another file  
- `:vsplit filename` – Split screen **vertically**  
- `Ctrl + w + w` – Switch between split screens  

---

## 🖱️ Advanced Vim Mouse Control & Clipboard Sharing

By default, Vim runs in a strict terminal mode where clicking or dragging your mouse either does nothing or selects text using your main terminal emulator's window selection instead of Vim's internal engine. You can change this behavior instantly.

---

### 1. Enabling and Disabling Mouse Controls

You can instruct Vim to intercept your mouse actions directly for smooth clicking, scrolling, and visual block selections.

* **Enable full mouse support:**
  ```vim
  :set mouse=a
  ```
  * **What this does:** The `a` stands for **all** modes (Normal, Insert, Visual, and Command). You can now click anywhere to position your cursor, use your mouse wheel to scroll through long code files, and drag your mouse to highlight blocks of text natively in Vim's *Visual Mode*.

* **Disable mouse support (Return to default terminal selection):**
  ```vim
  :set mouse=
  ```
  * **What this does:** Clears the mouse setting completely. Your mouse actions return to being handled entirely by your terminal window (Putty, iTerm2, Windows Terminal, or PowerShell) instead of Vim.

---

## ✂️ Quick Guide: Visual Mode & Copy-Pasting in Vim

Vim has a special highlighting mode called **Visual Mode**. This is the equivalent of clicking and dragging your mouse across text in a standard text editor.

---

### 1. How to Select (Highlight) Text

To select text, you must start from **Normal Mode** (press `Esc` first to be sure), position your cursor at the start of your text, and press one of these keys:

*   **`v` (Lowercase) – Character-by-Character Selection:** Highlights text character-by-character from your starting point. Perfect for selecting words or parts of a sentence.
*   **`V` (Uppercase) – Line-by-Line Selection:** Instantly highlights the **entire line** your cursor is on. Moving up or down highlights entire rows at a time. 

Once you press `v` or `V`, use your standard navigation keys (`h`, `j`, `k`, `l`, or arrow keys) to stretch the highlight over your target text.

---

### 2. How to Copy, Cut, and Paste Your Selection

Once your text is highlighted in Visual Mode, press a single key to take action:

*   **`y` (Yank / Copy):** Copies the highlighted text into Vim's temporary memory buffer and immediately drops you back into Normal Mode.
*   **`d` or `x` (Delete / Cut):** Cuts the highlighted text out of the document and saves it into memory.

#### To Paste Your Text:
1. Move your cursor to the exact spot where you want the text to go.
2. Press **`p`** (lowercase) to paste the text **after** your cursor (or on the line below).
3. Press **`P`** (uppercase) to paste the text **before** your cursor (or on the line above).

