---
layout: post
title:  "Learning SQL"
date:   2016-03-03
categories: SQL
---

### Installing SQLite in Window 10 OS

First, find out how to check if you have SQLite is installed in your system. Type a simple command to your command-prompt, like this:

```
which sqlite3
```

The above command did not work. So, I need to install precompiled SQLite from [SQLite website](http://www.sqlite.org/download.html).

Once I unzipped the files, below, in my C drive and created a folder name SQLite,

`sqlite-dll-win32-x86-3110100.zip`

`sqlite-tools-win32-x86-3110100.zip`

these files were extracted inside:

sqlite3.def, sqlite3.dll, sqlite3.exe, sqlite3_analyzer.exe, sqldiff.exe

I think you only need the first three.

#### Adding a new PATH

Then added C:\SQLite3 to my PATH variable so that I can invoke the SQLite Command Shell right from the Windows console.

In my command-prompt, I typed: `C:\Users\Heber> setx /m path "%path%;C:\SQLite"`

The setx command adds a directory to the current user’s Path variable. To make variables available at the system, or machine level, I use the setx command in conjunction with the /m flag. However, run the Command Terminal as Administrator for this to work.

To check if the path was created, type this command: `C:\Users\Heber> echo %PATH%`

Make sure it appears in the list.

#### Testing it

Now test if everything works correctly. Go to your command-prompt and type, `sqlite3`

The Output:

`C:\Users\Heber>sqlite3
SQLite version 3.11.1 2016-03-03 16:17:53
Enter ".help" for usage hints.
Connected to a transient in-memory database.
Use ".open FILENAME" to reopen on a persistent database.
sqlite>`

Great! We're ready to use SQLite.
