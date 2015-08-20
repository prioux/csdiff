
## csdiff : Colorize The Output Of The 'sdiff' Command

csdiff is a simple perl wrapper that produces a colorized version
of the output of the UNIX 'sdiff' command.

![Screenshot](https://raw.githubusercontent.com/prioux/csdiff/master/screenshots/csdiff_screenshot.png)

#### Invokation

The program is invoked much like 'sdiff' itself. Most options and
arguments are passed on directly to sdiff, and the output is 
intercepted and colorized using ANSI color sequences. The
exception is for a select few options that csdiff will interpret
for itself when placed in front of any other options that 'sdiff'
understands. The csdiff-specific options are -C (context
mode), -F (filter mode) and -S (summary mode).


```bash
  csdiff        file1 file2  # compare two files
  csdiff     -i file1 file2  # use -i option of sdiff
  csdiff -C5 -i file1 file2  # here, -C is an option of csdiff
```

#### csdiff Options

* The -C option selects how many lines of context to show around changes.

* The -F option selects which parts of the report to show. One or several
  of these letters can be specified:

  - 's' or 'u'   Same or unchanged lines
  - 'a'          Added lines
  - 'd'          Deleted lines
  - 'm' or 'c'   Modified/changed lines
  - 'o'          Omitted (only when -C is specified)

  Several of these letter can be supplied to a single -F option:

```bash
  csdiff -F adm file1 file2   # only shows added, deleted and modified lines
```

* Note that when using -C and -F at the same time, unselected reports
  in -F become part of the definition of 'context' for -C.

* The -S option triggers a 'summary' mode (no diff is shown, and -F and -C are ignored)

#### History

This is an old wrapper script, written probably around 2006 by Pierre Rioux
and privately maintained until its release on GitHub in August 2015. The
code stinks a bit, but it's quite stable and has been working flawlessly
on Mac OS X and Linux platforms ever since it was created.


