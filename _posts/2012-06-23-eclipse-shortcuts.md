---
layout: post
title: "Eclipse shortcuts"
category : Programming
---

## Viewing

`Ctrl+Space`  
Content assist  
Context sensitive content completion suggestions while editing Java code.

`Alt+/`  
Word completion  
This is excellent for code editing or writing plain help files with variables and other words having no English language equivalents. The word completion is based on the set of words already present in the current file.

`Ctrl+Shift+Space`  
Context information  
If typing a method call with several parameters use this to show the applicable parameter types. The current parameter where the cursor is will be shown in bold.

`Ctrl+M`  
Maximize active view/editor  
Toggles maximize/restore state of active view or editor

`Ctrl+Shift+L`  
Show key assist  
Show popup list of all currently applicable shortcut keys and key sequences.

`Alt+Shift+W`  
Show in...  
Show in another view (chosen from popup)

`F2`  
Show tooltip description  
Show Quick Javadoc in scrollable, copy-enabled window

`Shift+F2`  
Open attached javadoc  
Show Javadoc in a separate browser window

`Ctrl+F7`  
Go to next view  
Move between views (shows list and toggles between 2 most recent). When in editor, press Ctrl+F7 to switch to the Package Explorer, or hold Ctrl and press F7 multiple times to switch to other views.

`Ctrl+F8`  
Go to next perspective  
Move between perspectives (shows list and toggles between 2 most recent). The same as previous.

`Ctrl+Shift+W` / `Ctrl+W`  
Close editors  
Close all files / Close editor window

## Editing

`Ctrl+1`  
Quick Fix (after finding compiler error)

`Ctrl+2`  
Quick Assist (fast renaming, etc.)

`Ctrl+Shift+F`  
Formats code. You can make a beautiful looking code out of a mess with this. It requires a bit of setup, but it is well worth it. You can find its settings under Window+>Preferences+>Java+>Code style+>Formatter

`Ctrl+Shift+O`  
Organize Imports  
After typing a class name use this shortcut to insert an import statement. This works if multiple class names haven't been imported too.

`Alt+Up/Down Arrow`  
Move the row (or the entire selection) up or down.  
Very useful when rearranging code. You can even select more rows and move them all. Notice, that it will be always correctly indented.

`Ctrl+I`  
Corrects indentation.

`Ctrl+D`  
Delete row.  
You no more need to grab the mouse and select the line, no more Home, Shift+End, Delete. Quick and clean.

`Alt+Shift+J`  
to add javadoc at any place in java source file.

`Alt+Shift+Up`  
Select Enclosing Element

`Alt+Shift+Down`  
Restore Last Selection

`Alt+Shift+Left`  
Select Previous Element

`Alt+Shift+Right`  
Select Next Element  
Useful for selecting context-sensitive blocks (e.g. surrounding loop, method, class, etc.)

`Shift+Enter` / `Ctrl+Shift+Enter`  
Insert Line Below/Above  
Insert a line above or below the current line.

`Ctrl+Alt+Up`  
Duplicate Lines

`Ctrl+Alt+Down`  
Copy Lines  
Doesn't seem like it at first but a great shortcut once you learn to use it. Instead of select, copy and paste simply select and duplicate without affecting the clipboard.

`Alt+Shift+Z`  
Undo refactoring operation

`Alt+Shift+Y`  
Redo refactoring operation

`Ctrl+Shift+Y`  
To lower case

`Ctrl+Shift+X`  
To upper case

`Tab` / `Shift+Tab`  
Indent/Undent

`Ctrl+Slash` / `Ctrl+Backslash`  
Comment/Uncomment

`Alt+Shift+L`  
Extract Local Variable  
Use this to create a local variable from the selected expression. This is useful for breaking up larger expressions to avoid long lines.


`Alt+Shift+M`  
Extract Method  
Use this to extract a new method from existing code. The parameter list and return type will be automatically created.

`Alt+Shift+R`  
Rename+Refactoring  
Use this to rename type, method, or field. All existing references will be refactored as well.

## Navigation

`Ctrl+3`  
Quick Access (fast way to find an editor command)

`Ctrl+Comma / Ctrl+Period`  
Search next/previous, also next/previous Error, next/previous Diff  (only in the appropriate views e.g. search, problems, compare, etc.)

`F3`  
Open Declaration  
Drills down to the declaration of the type, method, or variable the cursor is on. Alternatively, you can hold Ctrl down and click the hyperlinked variable or class or whatever it is the declaration of which you want to see.

`F4`  
Open Type Hierarchy  
Show the type hierarchy (downward tree) or the supertype hierarchy (upward tree).

`Ctrl+Shift+T`  
Find Java Type  
Selection dialog for types (classes, etc.), where you can type the beginning of the short name (w/o packages) and quickly go to the right source file.  Wildcards * (any sequence of chars) and ? (any single char) are supported.
Nested tip: In this dialog box, you can type just the capitalized letters of your class name if you prefer.  For example, I can type FTI to get to my FileTreeIterator class.

`Ctrl+F6`  
Go to next editor (shows list and toggles between 2 most recent)  
Use to move between open editors. This is an slower alternative to Ctrl+E. Comes handy in a situation when you want to periodically switch between two editors, something, what is nearly impossible with Ctrl+E as it sorts entries quite randomly. Or you might just use Alt+Arrows..

`Ctrl+Shift+R`  
Find Resource  
Open any file quickly without browsing for it in the Package Explorer: Ctrl+Shift+R. Wildcards * (any sequence of chars) and ? (any single char) are supported.
Nested tip: In this dialog box, you can type just the capitalized letters of resource name if you prefer.  For example, I can type CBW to get to my CloseBrowserWindow.jsp file.

`Ctrl+O`  
Quick Outline  
Show current context (method, class, package, etc.), and allow quick navigation to other methods, members, inherited methods, etc. Press Ctrl+O a second time to include inherited methods.

`Ctrl+T`  
Show Quick Class Hierarchy  
Go to a supertype/subtype

`Ctrl+F3`  
Selection dialog for members of the current class.  
You can type the beginning of the name and quickly go to the right place in the source file.  Wildcards * (any sequence of chars) and ? (any single char) are supported.

`Ctrl+Shift+H`  
Show context of identifier at cursor  
Selection dialog to show a type in the class hierarchy window. Like Ctrl+Shift+T, but shows class hierarchy instead of Java source.  Wildcards * (any sequence of chars) and ? (any single char) are supported.
Nested tip: In this dialog box, you can type just the capitalized letters of your class name if you prefer.  For example, I can type FTI to get to my FileTreeIterator class.

`Ctrl+Alt+H`  
Show call hierarchy  
Show where a method is called from. In the Call Hierarchy view keep expanding the tree to continue tracing the call chain.

`Ctrl+E`  
Shows you a list of all open editors.  
Switch to another editor in current tab set. Use the arrow buttons, or type the name of the file youre editing

`Alt+Left` / `Right Arrow`  
Move to the last location you edited.  
Go back or forward through recent editing locations

`Ctrl+Q`  
Go to the last edit location  
This shortcut brings you right to where you last edited a file (This even works when youre already looking at a different file.)

`Ctrl+Up` / `Ctrl+Down`  
Scroll Line Up/Down  
Very handy if you want to scroll by 1 line without changing your cursor position or using the mouse.

`Ctrl+L`  
Go to Line  
Go to a specific line number.

`Ctrl+Shift+P`  
Go to matching bracket

`Ctrl+Shift+Down`  
Go to next object (method, variable, class, etc.) in source file.

`Ctrl+Shift+Up`  
Go to previous object (method, variable, class, etc.) in source file.

`Ctrl+PageDn`  
Go to next editor window in current tab set

`Ctrl+PageUp`  
Go to previous editor window in current tab set


## Search

`Ctrl+Shift+G`  
Find all references to selected identifier in workspace (showing them in Search pane)

`Ctrl+J` / `Ctrl+Shift+J`  
Incremental Search forward/back  
Incremental search. Similar to the search in firefox. It shows you results as you type. Don't be surprised, if you hit this combination, nothing happens+at the first glance. Just start typing and eclipse will move your cursor to the first ocurence.

`Ctrl+F`  
Search within a file

`Ctrl+K`  
Search Next within a file

`Ctrl+Shift+K`  
Search Previous within a file

`Ctrl+H`  
Search in project, workspace, file system, etc.

`Ctrl+Shift+U`  
When an identifier is selected: Find all occurrences of identifier in the file (showing them in Search pane)
When a class or interface name that follows one of the keyword "extends" or "implements" is selected: Find all methods in the file that implement methods defined in the class or interface (showing them in Search pane)

`Ctrl+Alt+G`  
Find all occurrences of selected text in workspace (showing them in Search pane)

`Ctrl+H`  
File Search  
Open the Search dialog's file search page

`Ctrl+Shift+G`  
Bind this to "Generate getters and setters". This is a "must have".

`Alt+C`  
Bind this to SVN/CVS "Commit".

`Alt+U`  
Bind this to SVN/CVS "Update".

