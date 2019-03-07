# 1. https://sanctum.geek.nz/arabesque/vi-mode-in-bash/

# 2. https://opensource.com/article/17/3/fun-vi-mode-your-shell

# 3. https://catonmat.net/bash-vi-editing-mode-cheat-sheet

# Ad 1.
Because the Bash shell uses GNU Readline, you’re able to set options in .bashrc and .inputrc to extensively customise how you
enter your command lines, including specifying whether you want the default Emacs-like keybindings (e.g. Ctrl+W to erase a word)
or bindings using a modal interface familiar to vi users.

To use vi mode in Bash and any other tool that uses GNU Readline, such as the MySQL command line, you need only put this into
your .inputrc file, then log out and in again:

set editing-mode vi

If you only want to use this mode in Bash, an alternative is to use the following in your .bashrc, again with a login/logout or
resourcing of the file:

set -o vi

You can check this has applied correctly with the following, which will spit out a list of the currently available bindings for a
   set of fixed possible actions for text:

$ bind -P

The other possible value for editing-mode is emacs. Both options simply change settings in the output of bind -P above.

This done, you should find that you open a command line in insert mode, but when you press Escape or Ctrl+[, you will enter an
emulation of Vi’s normal mode. Enter will run the command in its present state while in either mode. The bindings of most use in
this mode are the classic keys for moving to positions on a line:

     * ^ — Move to start of line
     * $ — Move to end of line
     * b — Move back a word
     * w — Move forward a word
     * e — Move to the end of the next word

Deleting, yanking, and pasting all work too. Also note that k and j in normal mode allow you to iterate through your history in
the same way that Ctrl+P and Ctrl+N do in Emacs mode. Searching with ? and / doesn’t work by default, but the Ctrl+R and Ctrl+S
bindings still seem to work.

If you are reasonably used to working with the modeless Emacs for most of your shell work but do occasionally find yourself in a
situation where being able to edit a long command line in vi would be handy, you may prefer to press Ctrl+X Ctrl+E to bring up

the command line in your $EDITOR instead, affording you the complete power of Vim to edit rather than the somewhat sparse vi
emulation provided by Readline. If you want to edit a command you just submitted, the fc (fix command) Bash builtin works too.

The shell is a very different environment for editing text, being inherently interactive and fixed to one line rather than a
static multi-line buffer, which leads some otherwise diehard vi users such as myself to prefer the Emacs mode for editing
commands. For one thing, vi mode in Bash trips on the vi anti-pattern of putting you in insert mode by default, and at the start
of every command line. But if the basic vi commands are etched indelibly into your memory and have become automatic, you may
appreciate being able to edit your command line using these keys instead. It’s certainly worth a try at any rate, and I do still
occasionally see set -o vi in the .bashrc files of a few of the pros on GitHub.

# Ad 2.

   Last updated 5 weeks ago

   bash readline vi editing mode default keyboard shortcut cheat sheet Bash provides two modes for command line editing - emacs and
   vi. Emacs editing mode is the default and I already wrote an article and created a cheat sheet for this mode.

   This time I am going to introduce you to bash's vi editing mode and give out a detailed cheat sheet with the default keyboard
   mappings for this mode.

   The difference between the two modes is what command each key combination (or key) gets bound to. You may inspect your current
   keyboard mappings with bash's built in bind command:

 $ bind -P

 abort can be found on "\C-g", "\C-x\C-g", "\M-\C-g".
 accept-line can be found on "\C-j", "\C-m".
 alias-expand-line is not bound to any keys
 ...

   To get into the vi editing mode type


 $ set -o vi

   in your bash shell (to switch back to emacs editing mode, type set -o emacs).

   If you are used to a vi text editor you will feel yourself at home.

   The editing happens in two modes - command mode and insert mode. In insert mode everything you type gets output to the terminal,
   but in the command mode the keys are used for various commands.

   Here are a few examples with screenshots to illustrate the vi editing mode.

   Let '[i]' be the position of cursor in insert mode in all the examples and '[c]' be the position of cursor in command mode.

   Examples:

   Once you have changed the readline editing mode to vi (by typing set -o vi), you will be working in insert mode.

   The example will be performed on this command:

 $ echo arg1 arg2 arg3 arg4<strong>[i]</strong>

   Example 1:

   Suppose you have typed a command with a few arguments and want to insert another argument before an argument which is three words
   backward.

 $ echo arg1 <em>(want to insert arg5 here)</em> arg2 arg3 arg4<strong>[i]</strong>

   Hit 'ESC' to switch to command mode and press '3' followed by 'B':

 $ echo arg1 <strong>[c]</strong>arg2 arg3 arg4

   Alternatively you could have hit 'B' three times: 'BBB'.

   Now, enter insert mode by hitting 'i' and type 'arg5 '

 $ echo arg1 arg5 <strong>[i]</strong>arg2 arg3 arg4

 Example 2:
 * Avoiding the arrow keys
   Suppose you wanted to change arg2 to arg5:

 $ echo arg1 <strong>[c]</strong>arg2 arg3 arg4

   To do this, you can type 'cw' which means 'change word' and just type out 'arg5':

 $ echo arg1 arg5<strong>[c]</strong> arg3 arg4

   Or even quicker, you can type 'f2r5', where 'f2' moves the cursor right to next occurrence of character '2' and 'r5' replaces the
   character under the cursor with character '5'.

   Example 3:

   Suppose you typed a longer command and you noticed that you had made several mistakes, and wanted to do the correction in the vi
   editor itself. You can type 'v' to edit the command in the editor and not on the command line!

   Example 4:

   Suppose you typed a long command and remembered that you had to execute another one before it. No need to erase the current
   command! You can switch to command mode by hitting ESC and then type '#' which will send the current command as a comment in the
   command history. After you type the command you had forgotten, you may go two commands back in history by typing 'kk' (or '2k'),
   erase the '#' character which was appended as a comment and execute the command, this makes the whole command look like 'ESC 2k0x
   ENTER'.

   These are really basic examples, and it doesn't get much more complex than this. You should check out the cheat sheet for other
   tips and examples, and try them out!

   To create the cheat sheet, I downloaded bash-2.05b source code and scanned through lib/readline/vi_keymap.c source code file and
   lib/readline/vi_mode.c to find all the default key bindings.

   It turned out that the commands documented in vi_keymap.c were all documented in man 3 readline and I didn't find anything new.

   After that I checked bashline.c source file function initialize_readline to find how the default keyboard shortcuts were changed.
   I found that 'CTRL-e' (which switched from vi mode to emacs) got undefined, 'v' got defined which opens the existing command in
   the editor, and 'strong>@</strong' which replaces a macro key (char) with the corresponding string.

.---------------------------------------------------------------------------.
|                                                                           |
|                          Readline VI Editing Mode                         |
|                     Default Keyboard Shortcuts for Bash                   |
|                               Cheat Sheet                                 |
|                                                                           |
'---------------------------------------------------------------------v1.21-'
| Created by Peter Krumins (peter@catonmat.net, @pkrumins on twitter)       |
| www.catonmat.net -- good coders code, great reuse                         |
|                                                                           |
| Released under the GNU Free Document License                              |
'---------------------------------------------------------------------------'

 ======================== Keyboard Shortcut Summary ========================

.--------------.------------------------------------------------------------.
|              |                                                            |
| Shortcut     | Description                                                |
|              |                                                            |
'--------------'------------------------------------------------------------'
| Switching to COMMAND Mode:                                                |
'--------------.------------------------------------------------------------'
| ESC          | Switch to command mode.                                    |
'--------------'------------------------------------------------------------'
| Commands for Entering INPUT Mode:                                         |
'--------------.------------------------------------------------------------'
| i            | Insert before cursor.                                      |
'--------------+------------------------------------------------------------'
| a            | Insert after cursor.                                       |
'--------------+------------------------------------------------------------'
| I            | Insert at the beginning of line.                           |
'--------------+------------------------------------------------------------'
| A            | Insert at the end of line.                                 |
'--------------+------------------------------------------------------------'
| c<mov. comm> | Change text of a movement command <mov. comm> (see below). |
'--------------+------------------------------------------------------------'
| C            | Change text to the end of line (equivalent to c$).         |
'--------------+------------------------------------------------------------'
| cc or S      | Change current line (equivalent to 0c$).                   |
'--------------+------------------------------------------------------------'
| s            | Delete a single character under the cursor and enter input |
|              | mode (equivalent to c[SPACE]).                             |
'--------------+------------------------------------------------------------'
| r            | Replaces a single character under the cursor (without      |
|              | leaving command mode).                                     |
'--------------+------------------------------------------------------------'
| R            | Replaces characters under cursor.                          |
'--------------+------------------------------------------------------------'
| v            | Edit (and execute) the current command in the text editor. |
|              | (an editor defined in $VISUAL or $EDITOR variables, or vi  |
'--------------'------------------------------------------------------------'
| Basic Movement Commands (in command mode):                                |
'--------------.------------------------------------------------------------'
| h            | Move one character right.                                  |
'--------------+------------------------------------------------------------'
| l            | Move one character left.                                   |
'--------------+------------------------------------------------------------'
| w            | Move one word or token right.                              |
'--------------+------------------------------------------------------------'
| b            | Move one word or token left.                               |
'--------------+------------------------------------------------------------'
| W            | Move one non-blank word right.                             |
'--------------+------------------------------------------------------------'
| B            | Move one non-blank word left.                              |
'--------------+------------------------------------------------------------'
| e            | Move to the end of the current word.                       |
'--------------+------------------------------------------------------------'
| E            | Move to the end of the current non-blank word.             |
'--------------+------------------------------------------------------------'
| 0            | Move to the beginning of line                              |
'--------------+------------------------------------------------------------'
| ^            | Move to the first non-blank character of line.             |
'--------------+------------------------------------------------------------'
| $            | Move to the end of line.                                   |
'--------------+------------------------------------------------------------'
| %            | Move to the corresponding opening/closing bracket.         |
'--------------'------------------------------------------------------------'
| Character Finding Commands (these are also Movement Commands):            |
'--------------.------------------------------------------------------------'
| fc           | Move right to the next occurance of char c.                |
'--------------+------------------------------------------------------------'
| Fc           | Move left to the previous occurance of c.                  |
'--------------+------------------------------------------------------------'
| tc           | Move right to the next occurance of c, then one char       |
|              | backward.                                                  |
'--------------+------------------------------------------------------------'
| Tc           | Move left to the previous occurance of c, then one char    |
|              | forward.                                                   |
'--------------+------------------------------------------------------------'
| ;            | Redo the last character finding command.                   |
'--------------+------------------------------------------------------------'
| ,            | Redo the last character finding command in opposite        |
|              | direction.                                                 |
'--------------+------------------------------------------------------------'
| |            | Move to the n-th column (you may specify the argument n by |
|              | typing it on number keys, for example, 20|)                |
'--------------'------------------------------------------------------------'
| Deletion Commands:                                                        |
'--------------.------------------------------------------------------------'
| x            | Delete a single character under the cursor.                |
'--------------+------------------------------------------------------------'
| X            | Delete a character before the cursor.                      |
'--------------+------------------------------------------------------------'
| d<mov. comm> | Delete text of a movement command <mov. comm> (see above). |
'--------------+------------------------------------------------------------'
| D            | Delete to the end of the line (equivalent to d$).          |
'--------------+------------------------------------------------------------'
| dd           | Delete current line (equivalent to 0d$).                   |
'--------------+------------------------------------------------------------'
| CTRL-w       | Delete the previous word.                                  |
'--------------+------------------------------------------------------------'
| CTRL-u       | Delete from the cursor to the beginning of line.           |
'--------------'------------------------------------------------------------'
| Undo, Redo and Copy/Paste Commands:                                       |
'--------------.------------------------------------------------------------'
| u            | Undo previous text modification.                           |
'--------------+------------------------------------------------------------'
| U            | Undo all previous text modifications.                      |
'--------------+------------------------------------------------------------'
| .            | Redo the last text modification.                           |
'--------------+------------------------------------------------------------'
| y<mov. comm> | Yank a movement into buffer (copy).                        |
'--------------+------------------------------------------------------------'
| yy           | Yank the whole line.                                       |
'--------------+------------------------------------------------------------'
| p            | Insert the yanked text at the cursor.                      |
'--------------+------------------------------------------------------------'
| P            | Insert the yanked text before the cursor.                  |
'--------------'------------------------------------------------------------'
| Commands for Command History:                                             |
'--------------.------------------------------------------------------------'
| k            | Insert the yanked text before the cursor.                  |
'--------------+------------------------------------------------------------'
| j            | Insert the yanked text before the cursor.                  |
'--------------+------------------------------------------------------------'
| G            | Insert the yanked text before the cursor.                  |
'--------------+------------------------------------------------------------'
| /string or   | Search history backward for a command matching string.     |
| CTRL-r       |                                                            |
'--------------+------------------------------------------------------------'
| ?string or   | Search history forward for a command matching string.      |
| CTRL-s       | (Note that on most machines Ctrl-s STOPS the terminal      |
|              | output, change it with `stty' (Ctrl-q to resume)).         |
'--------------+------------------------------------------------------------'
| n            | Repeat search in the same direction as previous.           |
'--------------+------------------------------------------------------------'
| N            | Repeat search in the opposite direction as previous.       |
'--------------'------------------------------------------------------------'
| Completion commands:                                                      |
'--------------.------------------------------------------------------------'
| TAB or = or  | List all possible completions.                             |
| CTRL-i       |                                                            |
'--------------+------------------------------------------------------------'
| *            | Insert all possible completions.                           |
'--------------'------------------------------------------------------------'
| Miscellaneous commands:                                                   |
'--------------.------------------------------------------------------------'
| ~            | Invert case of the character under cursor and move a       |
|              | character right.                                           |
'--------------+------------------------------------------------------------'
| #            | Prepend '#' (comment character) to the line and send it to |
|              | the history.                                               |
'--------------+------------------------------------------------------------'
| _            | Inserts the n-th word of the previous command in the       |
|              | current line.                                              |
'--------------+------------------------------------------------------------'
| 0, 1, 2, ... | Sets the numeric argument.                                 |
'--------------+------------------------------------------------------------'
| CTRL-v       | Insert a character literally (quoted insert).              |
'--------------+------------------------------------------------------------'
| CTRL-r       | Transpose (exchange) two characters.                       |
'--------------'------------------------------------------------------------'


 ===========================================================================

.---------------------------------------------------------------------------.
| Created by Peter Krumins (peter@catonmat.net, @pkrumins on twitter)       |
| www.catonmat.net -- good coders code, great coders reuse                  |
'---------------------------------------------------------------------------'

