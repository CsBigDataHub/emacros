Emacros: A Package for Organizing and Handling Keyboard Macros in GNU Emacs
===========================================================================

Emacros: A Package for Organizing, Saving, Loading, and Conveniently Executing Your Keyboard Macros in GNU Emacs

<!-- markdown-toc start - Don't edit this section. Run M-x markdown-toc-refresh-toc -->
**Table of Contents**

- [Emacros: A Package for Organizing and Handling Keyboard Macros in GNU Emacs](#emacros-a-package-for-organizing-and-handling-keyboard-macros-in-gnu-emacs)
    - [Overview](#overview)
    - [Download and Installation](#download-and-installation)
        - [global macro dir](#global-macro-dir)
    - [Feedback and Bug Reports](#feedback-and-bug-reports)
    - [Emacros Manual](#emacros-manual)
        - [Naming, Saving, and Executing Keyboard Macros](#naming-saving-and-executing-keyboard-macros)
            - [emacros-name-last-kbd-macro-add](#emacros-name-last-kbd-macro-add)
            - [emacros-execute-named-macro](#emacros-execute-named-macro)
            - [emacros-auto-execute-named-macro](#emacros-auto-execute-named-macro)
        - [Loading Macro Definitions](#loading-macro-definitions)
            - [Automatic Macro Loading](#automatic-macro-loading)
            - [Manually Loading and Unloading Macros](#manually-loading-and-unloading-macros)
        - [Manipulating Macro Files](#manipulating-macro-files)
            - [emacros-rename-macro](#emacros-rename-macro)
            - [emacros-move-macro](#emacros-move-macro)
            - [emacros-remove-macro](#emacros-remove-macro)
        - [Getting Help with Macros](#getting-help-with-macros)
            - [emacros-show-macro-names](#emacros-show-macro-names)
            - [emacros-show-macros](#emacros-show-macros)
        - [Subtleties](#subtleties)
            - [Getting the Right Major Mode](#getting-the-right-major-mode)
            - [Slashes in Mode Names](#slashes-in-mode-names)
            - [Compatibility with Emacs' Macro Saving](#compatibility-with-emacs-macro-saving)
            - [Visiting Macro Files](#visiting-macro-files)
            - [Completing Macro Names](#completing-macro-names)
            - [Emacros vs. Kmacro](#emacros-vs-kmacro)
            - [Acknowledgment](#acknowledgment)
    - [Copying Information](#copying-information)

<!-- markdown-toc end -->

* * *

Overview
--------

The purpose of keyboard macros in an editor is to expedite the entering of key sequences that occur frequently and are lengthy or inconvenient to type. GNU Emacs provides keyboard macros. Beginning with Version 22, GNU Emacs contains the kmacro package, which provides support for dealing with keyboard macros that have been defined and named during the current Emacs session. Support for saving, organizing, and reloading keyboard macros remains weak. The Emacros package facilitates these tasks.

Emacros' way of saving macro definitions to files is based on the idea that macro definitions should be separated by major modes to which they pertain. The macros used when editing a TeX file, for example, will not be needed when working on a C++ program. Moreover, within each mode, there will be macros that should be available whenever Emacs is in that mode, and others that are relevant for specific projects only. Consequently, each mode should allow one global macro file and several local ones in different directories as needed. This arrangement makes it easy to keep track of existing macro definitions.

Download and Installation
-------------------------

**Method 1**

Install from `MELPA`

**Method 2**

clone the repository and add it to `load-path`

### global macro dir
The directory where your global macro files will be kept defaults to your `.emacs.d` directory. You may wish to choose a different directory. In that case, you must also place the line

(setq emacros-global-dir "DIR")

in your Emacs initialization file. DIR can be specified in unexpanded form, e.g., as `"~/emacs"`, and it may be given with or without trailing slash.

Feedback and Bug Reports
------------------------

Please open a issue in this repository.

Emacros Manual
--------------

### Naming, Saving, and Executing Keyboard Macros

A keyboard macro really consists of two components: the (complicated) key sequence which is to be entered, and the (short) command which invokes the entering of the sequence. Here, we will refer to the key sequence as the _macro_, and to the command as its _name_.

When you use Emacros, defining a keyboard macro is done as usual with `C-x (` and `C-x )`, or with `F3` and `F4`, whichever you prefer. Emacros comes in when you name the macro: instead of giving the macro a name with Emacs' `name-last-kbd-macro` or with kmacro's `kmacro-name-last-macro`, you use the Emacros function `emacros-name-last-kbd-macro-add`.

#### emacros-name-last-kbd-macro-add

This function first prompts the user for a name to be given to the most recently defined keyboard macro. The name must neither be the empty string nor an integer, and it must only use letters, digits, and the characters `_` and `-`. The name of an existing Emacs Lisp function which is not a keyboard macro will not be accepted as input.

Next, the function saves the macro definition to a file named `MODE-mac.el`, where `MODE` is the name of the current major mode. (See Section [Slashes in Mode Names](#slashes-in-mode-names) for a qualification of this statement.) This file can be in the [directory for global macros](#global-macro-dir), in which case the macro will be available whenever `MODE` is the major mode, or it can be in the current directory, in which case the macro will be locally available whenever `MODE` is the major mode and the file that is being visited is from this directory. The function will ask you to choose between `l` for local and `g` for global.

When `emacros-name-last-kbd-macro-add` is called with prefix argument, as in

``C-u M-x emacros-name-last-kbd-macro-add RET`

then you will be prompted to explicitly enter the name of a file for saving the macro.

If a macro with the same name already exists in the file that you are saving to, you will be prompted for overwriting the existing definition.

It is possible to have several macro definitions with the same name, one in the global macro file `MODE-mac.el` and the others in local files `MODE-mac.el`. When macros are being loaded, the last local defintion of a multiply defined macro name becomes the active one. Functions that manipulate macro files, such as [emacros-remove-macro](#emacros-remove-macro), will always look in the local and global macro file to do their work.

Note that the function `emacros-name-last-kbd-macro-add` does not add the newly defined macro to the keyboard macro ring that is maintained by the kmacro package. See Section [Emacros vs. Kmacro](#emacros-vs-kmacro) below for a more detailed explanation.

You may or may not want to bind the function `emacros-name-last-kbd-macro-add` to a key. You could, for example, bind it to `Ctrl-a` by placing the line

`(global-set-key "\\C-ca" 'emacros-name-last-kbd-macro-add)`

in your Emacs initialization file.

#### emacros-execute-named-macro

Once a macro MACRO has a name MACRONAME, this name is in fact a command which causes the macro to be inserted before the cursor: typing

``M-x MACRONAME RET`

inserts MACRO. This has the disadvantage that completion and history take into account all command names rather than just macro names. The problem is resolved by the function `emacros-execute-named-macro`.This function should probably be bound to a key sequence such as `Ctrl-c e`. The line

`(global-set-key "\\C-ce" 'emacros-execute-named-macro)`

in your Emacs initialization file does the job. Typing `Ctrl-c e` is then similar to `M-x`, but for macros only.

#### emacros-auto-execute-named-macro

This function is provided as a further convenience for the impatient. `emacros-auto-execute-named-macro` should definitely be bound to a key if you intend to use it at all. For example, `Ctrl-c x` would be a good choice. Typing `Ctrl-c x` will then prompt for the name of a macro. The cursor will stay at its position in the current buffer. As soon as the sequence entered matches the name of a macro, the macro is inserted and regular editing is resumed without the need to type a `return`.

When entering the name of a macro for auto-execution, the `return` key provides a rudimentary completion: the input is completed as far as possible, and execution occurs automatically as soon as a match has been found, even if it is "complete but not unique" in the sense of Emacs.

It is clear that you can not in this way insert a macro whose name contains the name of another macro as an initial substring. You may want to simply avoid this situation; however, if it occurs, you can always use [`emacros-execute-named-macro`](#emacros-execute-named-macro) to insert such problem macros.

### Loading Macro Definitions

One of the most important features of Emacros is that macro definitions are loaded automatically when a file is visited. There are also ways to load and unload macros manually.

#### Automatic Macro Loading

Everytime you read a file into Emacs with `Ctrl-x Ctrl-f`, `Ctrl-x Ctrl-v`, or `Ctrl-x 4 f`, the function

emacros-load-macros

will be invoked automatically. It will load those macros that have been saved to files named `MODE-mac.el` in the current directory and in the directory for global macros. Here, `MODE` is name of the major mode that Emacs has chosen for the visited file. (See Section [Slashes in Mode Names](#slashes-in-mode-names) for a qualification of this statement.) The function can also be used interactively; the next section describes a situation where you may want to do this.

#### Manually Loading and Unloading Macros

If you have been editing a file and then visit another one with a different mode and/or from a different directory, then the macros pertaining to the new file will be loaded, and all others that were loaded previously will remain active as well. If there are not too many macros around, this is probably what you want. In the long run, however, especially when you are one of those users that never leave Emacs, you would end up with all macros being loaded, thus rendering the separation by modes pointless. The function

`emacros-refresh-macros`

takes care of this problem. Typing

`M-x emacros-refresh-macros RET`

will unload all previously loaded macros and load the ones pertaining to the current buffer, thus creating the same situation as if you had just started Emacs and found the file that the current buffer is visiting. Another situation that necessitates similar action arises when you do not work "mode-consciously." If, for example, you edit a file named `test` and then turn it into a `.tex` file by typing

`C-x C-w test.tex RET`

then Emacros will not automatically load the macros for TeX mode. To get things right, you must explicitly refresh your loaded macros:

`M-x emacros-refresh-macros RET`

If you want to keep previously loaded macros active rather than refreshing, then you want to replace `emacros-refresh-macros` with [`emacros-load-macros`](#emacros-load-macros) in the above. Since the macro files `MODE-mac.el` contain ordinary Emacs Lisp code, you may also make macros available by loading macro files explicitly. This is done using the GNU Emacs function for loading Lisp code, as in

```
M-x load-file RET
Load file: ~/thesis/TeX-mac.el RET
```

This is also the way to load macros when you have used the function [`emacros-name-last-kbd-macro-add`](#function-emacros-name-last-kbd-macro-add) with a prefix argument, thus choosing your own file for saving your macros.

### Manipulating Macro Files

There are three functions that allow you to manipulate macro definitions that have already been saved. You can rename macros, move them betweeen local and global macro files, and remove them entirely.

#### emacros-rename-macro

This function assigns a new name to a previously named macro, making the change effective in the current session and in the local or global macro file pertaining to the current buffer, as appropriate. The old name is cleared of its meaning and may be used for a later macro. Typing

`M-x emacros-rename-macro RET`

will prompt for the old name that is to be changed and then for the new name, subject to the same restrictions that apply when naming with [`emacros-name-last-kbd-macro-add`](#emacros-name-last-kbd-macro-add). Completion and history are available when entering the old name. If a macro with the new name already exists in the file where the change takes place, you will be prompted for overwriting the existing defintion.

_Note:_ The function `emacros-rename-macro` can access for renaming all those macros that pertain to the current buffer. You cannot access just every macro that happens to have been loaded during the current session. For example, if you have been working on a TeX file, then change to a buffer visiting a `.c` file, and then try to rename a macro named `NAME` that is kept in `TeX-mac.el`, you will get the message

Macro named `NAME` not found in current file(s) C-mac.el: no action
taken.

You must change back to the buffer visiting the TeX file before you can access your macro for renaming. All this applies to moving and removing macros as well.

#### emacros-move-macro

This function moves macro definitions between the local and global macro file pertaining to the current buffer. Typing

`M-x emacros-move-macro RET`

will prompt for the name of a macro to be moved and for a choice between "from local" and "from global" before performing the desired task.

#### emacros-remove-macro

This function deletes macros from current macro files and disables them in the current session. Typing

`M-x emacros-remove-macro RET`

will prompt for the name of the macro to be deleted. A macro that has been deleted is irretrievably lost. As with [`emacros-rename-macro`](#emacros-rename-macro), the macro's name is made available for use with a different macro.

### Getting Help with Macros

One way in which Emacros assists you with recalling macro names is by offering completion and command line history whenever the name of a macro is to be entered. Beyond that, there are two functions that provide help with keyboard macros. You can list the names of all currently defined macros, and you can obtain a list of the names of the macros together with a textual representation of each macro's definition. There is also the possibility to help yourself by inspecting and editing the files where the macros are stored.

#### emacros-show-macro-names

The function `emacros-show-macro-names` displays a list of the names of all currently defined macros in the Emacs `*Help*` buffer, much like completion lists are displayed in the `*Completion*` buffer. With prefix argument, as in

`C-u M-x emacros-show-macro-names RET`

the function displays the macro names in a single column rather than in the two column format that Emacs uses for completion lists.

#### emacros-show-macros

The function `emacros-show-macros` displays a list of the names of all currently defined macros together with a textual representation of each macro's definition in the Emacs `*Help*` buffer.

Those parts of a macro definition that are plain character insertions are shown as strings in the textual representation. Function keys are shown using Emacs' standard textual representation. Control and alt keys are currently shown as "funny characters" in strings. For example, if you were to define a macro named `bm` with definition

`M-x buffer-menu RET`

then the output of the function `emacros-show-macros` would display the following line for this macro:

bm  "buffer-menu" <return>

This is certainly less than perfect. It wouldn't be hard to fix, if someone has the patience to do it.

### Subtleties

This chapter discusses a few subtleties about Emacros. Except perhaps for the first section, these are things that the user should not really have to worry about.

#### Getting the Right Major Mode

There is a subtlety about Emacs' way of determining the major mode for a visited file that needs attention when using Emacros. The problem occurs when Emacs uses not only the file extension to determine the mode, but actually looks into the file. An example for this is LaTeX mode. The proper extension for LaTeX files is `.tex`. If you visit a non-empty file with extension `tex`, and there is nothing LaTeX-specific in it, then Emacs will put you in TeX mode. If you then add the line

\\documentstyle\[...\]{...}

to the top of the file and visit the file again later, you will find yourself in LaTeX mode. The fact that the major
mode is different depending on what's in the file can of course cause a problem when using Emacros. However, this can be
fixed by choosing your mode explicitly and loading macros manually: to be in LaTeX mode and have your LaTex macros
loaded regardless of what's in the file, you must say

```
M-x latex-mode RET
M-x emacros-refresh-macros RET

or

M-x latex-mode RET
M-x emacros-load-macros RET
```

The first version will unload all currently defined macros and load the ones for the LaTex buffer only. The second one loads the macros for the LaTeX buffer on top of what may already be loaded.

#### Slashes in Mode Names

There are major modes for GNU Emacs whose name is not a constant string. The name of C++ mode, for example, is "`C++`". However, when you turn on `auto-newline` mode and `electric-mode`, the mode name changes to "`C++/la`". The slash in the mode name is of course a problem for Emacros, since it wants to use the mode name as part of a file name. Fortunately, however, it is also reasonable to ignore the qualifiations following the slash when it comes to saving macros. Therefore, Emacros uses only the part of the major mode name up to and not including the first forward slash when it builds its macro file names.

#### Compatibility with Emacs' Macro Saving

Suppose you have been saving kbd-macros using the Emacs function `insert-kbd-macro`, and you have loaded them by loading the file in which they were saved. The Emacros functions for executing and displaying macros will work for these macros.

To be able to rename, move, and remove your old macros by means of Emacros functions, you must edit your macro files. First, replace `fset` with `emacros-new-macro` in each macro definition. Then move the macro definitions to the appropriate files `MODE-mac.el`, placing each macro definition on a single line.

#### Visiting Macro Files

The functions for saving, renaming, moving, or removing macro definitions make changes to one or more of the files `MODE-mac.el`. If you are visiting such a file when using one of these functions and the respective buffer is modified, you will be notified and asked if it is ok to continue and possibly save. Answer `y` only if you are sure that the changes that you have made are meaningful. If your answer is `n`, then no action is taken at all.

#### Completing Macro Names

When a function requires the input of an existing macro name, completion works just as it normally does in GNU Emacs. The only exception is the function [`emacros-auto-execute-named-macro`](#emacros-auto-execute-named-macro). Here, no completion lists are provided. The reason is that such a list would necessarily be misleading: suppose you have three macros named `abc`, `abd`, and `abcdefg`, and you have enetered `ab`. Listing all three as completions would be wrong because you cannot get to `abcdefg` with auto-execution. On the other hand, listing just `abc` and `abd` would be misleading as well because these are not the only macro names that start with `ab`. Auto-execution is really meant for frequently used macros whose names you just don't have trouble with.

#### Emacros vs. Kmacro

Beginnig with GNU Emacs Version 22, the kmacro package is part of the GNU Emacs distribution. The kmacro package provides support for dealing with macros that have been defined and named in the current Emacs session. For example, it places all keyboard macros that have been defined and named with the function `kmacro-name-last-macro` on the keyboard macro ring for easy retrieval.

The Emacros package, by contrast, provides support for saving keyboard macros to files and reloading them in future Emacs sessions. Keyboard macros that have been named and saved with the Emacros function `emacros-name-last-kbd-macro-add` are completely separate from the kmacro world. In particular, they are not placed on kmacro's macro ring. The main reason for this design decision is that in the Emacros world, where keyboard macros are saved and reloaded, there will often be a large number of macros. Holding them in a ring is not a very good way to keep track of them. The functions `emacros-execute-named-macro` and `emacros-auto-execute-named-macro` with their completion features and the help functions `emacros-show-macros` and `emacros-show-macro-names` are better suited for this purpose.

#### Acknowledgment

This is a package created by **Thomas Becker**. I found this package useful and wanted to submit to `MELPA`.

Original source code can be found at - http://thbecker.net/free_software_utilities/emacs_lisp/emacros/emacros.html.
However it is quite outdated. I have resolved all the flycheck warning in his code before submitting it to `MELPA`.

Copying Information
-------------------

Emacros is an extension to GNU Emacs. Everyone is granted permission to copy, modify and redistribute Emacros, but only under the conditions described in the GNU Emacs General Public License. A copy of this license is supposed to have been given to you along with GNU Emacs so you can know your rights and responsibilities. It should be in a file named `COPYING`.

Emacros is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY. No author or distributor accepts responsibility to anyone for the consequences of using it or for whether it serves any particular purpose or works at all, unless he says so in writing. Refer to the GNU Emacs General Public License for full details.

* * *

[Back to top](#top)
