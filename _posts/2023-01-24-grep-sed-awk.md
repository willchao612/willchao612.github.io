---
layout: post
title: 'The power of grep, sed, and awk'
subtitle: ''
author: 'Will'
header-style: text
lang: en
tags:
  - Bash
  - Tricks
  - 编程语言
  - 笔记
---

![](https://i.imgur.com/01BllaC.jpg)

### Grep

<small>
<code>g/re/p</code>: globally search for a regular expression and print matching lines
</small>

#### Usage

```bash
grep [OPTION]... PATTERNS [FILE]...
```

<small>
instructions from running <code>grep --help</code>
</small>

PATTERNS can contain multiple patterns separated by newlines. When FILE is '-',
read standard input. With no FILE, read '.' if recursive, '-' otherwise. With
fewer than two FILEs, assume -h. Exit status is 0 if any line is selected, 1
otherwise; if any error occurs and -q is not given, the exit status is 2.

Grep supports basic regular expressions by default.

<small>
most used switches
</small>

- `--color={always|never|auto}`
- `--exclude-dir=GLOB`: skip directories that match GLOB
- `--exclude=GLOB`: skip files that match GLOB
- `--include=GLOB`: search only files that match GLOB (a file pattern)
- `-C`, `--context=NUM`: print NUM lines of output context
- `-E`, `--extended-regexp`: PATTERNS are extended regular expressions
- `-F`, `--fixed-strings`: PATTERNS are strings
- `-L`, `--files-without-match`: print only names of FILEs with no selected
  lines
- `-c`, `--count`: print only a count of selected lines per FILE
- `-i`, `--ignore-case`: ignore case distinctions in patterns and data
- `-l`, `--files-with-matches`: print only names of FILEs with selected lines
- `-n`, `--line-number`: print line number with output lines
- `-q`, `--quiet`, `--silent`: suppress all normal output
- `-r`, `--recursive`: search all files recursively in current directory
- `-v`, `--invert-match`: select non-matching lines

<small>
a nice alias for <code>grep</code>
</small>

```bash
alias grep="grep --color=auto --exclude-dir={.bzr,CVS,.git,.hg,.svn,.idea,.tox}"
```

#### Alternatives

[ripgrep](https://github.com/BurntSushi/ripgrep)

### Sed

<small>
<ins>s</ins>tream <ins>ed</ins>itor
</small>

#### Usage

```bash
sed [OPTION]... {script-only-if-no-other-script} [input-file]...
```

<small>
instructions from running <code>sed --help</code>
</small>

If no -e, --expression, -f, or --file option is given, then the first non-option
argument is taken as the sed script to interpret. All remaining arguments are
names of input files; if no input files are specified, then the standard input
is read.

Sed supports basic regular expressions by default.

<small>
most used switches
</small>

- `--debug`: annotate program execution
- `-E`, `-r`, `--regexp-extended`: use extended regular expressions in the
  script (for portability use POSIX -E).
- `-e script`, `--expression=script`: add the script to the commands to be
  executed
- `-i[SUFFIX]`, `--in-place[=SUFFIX]`: edit files in place (makes backup if
  SUFFIX supplied)
- `-n`, `--quiet`, `--silent`: suppress automatic printing of pattern space

<small>
most used commands
</small>

##### `d`

DELETE pattern space

`<address>d`

```bash
seq 10 | sed 1,5d # 6 7 8 9 10
echo "hello\n\nworld" | sed '/^$/d' # hello world
```

##### `p`

PRINT pattern space to stdout. Usually used with -n

`<address>p`

```bash
seq 10 | sed -n 1~3p # 1 4 7 10
```

##### `q`

QUIT

`<address>q`

```bash
seq 10 | sed 3q # 1 2 3
```

##### `s`

SUBSTITUTE

`<address>s/regexp/replacement/flags`

```bash
command | sed -E 's/(apple)/\U\1/g'
```

`/` may be replaced by any other single character. This is particularly useful
if `regexp` itself contains `/`.

```bash
sed 's#/#_#g' <<< "path/to/file" # path_to_file
```

##### `{ Commands }`

`<address>{ command1; command2; command3 }`

```bash
seq 10 | sed '{3,7s/[[:digit:]]/x/; /x/d}' # 1 2 8 9 10
```

###### Special sequences

- `\E`: stop case conversion
- `\L`: lowercase all characters after it
- `\U`: uppercase all characters after it
- `\l`: lowercase next character
- `\u`: uppercase next character

###### Flags

- _number_: replace the _number_^th^ match
- `g`: replace all matches (first match by default)
- `I`: case-insensitive mode

#### Address

- _number_: the _number_^th^ line
- `/regexp/`: lines matching a pattern
- `/regexp/I`: lines matching a pattern in case-insensitive mode
- `start,end`: lines within a range
- `start~step`: every _step_^th^ line from the _start_^th^ line
- `$`: the last line

```bash
printf "%s\n" a b c | sed '1d' # b c
printf "%s\n" a B c | sed '/b/Id' # a c
printf "%s\n" a b c | sed '$d' # a b
```

> Appending an `!` after an address negates the selection

```bash
printf "%s\n" a b B c | sed '/b/I!d' # b B
```

#### Snippet

- capitalize

  ```bash
  function capitalize {
    sed -E "s/^(.)/\U\1/"
  }
  ```

- dequote

  ```bash
  function dequote {
    sed -e "s/^'//" -e "s/'$//"
  }
  ```

### Awk

![](https://i.imgur.com/yILlhCF.jpg)

<small>
[Designers] Alfred V. <ins>A</ins>ho, Peter J. <ins>W</ins>einberger, and Brian W. <ins>K</ins>ernighan
@ AT&T Bell Laboratories
</small>

#### Usage

```bash
awk [POSIX or GNU style options] [--] 'program' file ...
```

Awk supports most extended regular expressions (ERE) by default.

<small>
most used switches
</small>

- `-F fs`, `--field-separator=fs`
- `-v var=val`

#### Program structure

`<pattern> { <action1>; <action2> }; <pattern> { <action1>; <action2> }`

##### Patterns

- `BEGIN`
- `END`

  > BEGIN and END are two special kinds of patterns which are not tested against
  > the input. The action parts of all BEGIN patterns are merged as if all the
  > statements had been written in a single BEGIN rule. They are ex‐ ecuted
  > before any of the input is read. Similarly, all the END rules are merged,
  > and executed when all the input is exhausted (or when an exit statement is
  > executed). BEGIN and END patterns cannot be combined with other patterns in
  > pattern expressions. BEGIN and END patterns cannot have missing action
  > parts.

- `<test_expression>`

  Tests whether certain fields match certain regular expressions.

- `/<regexp>/`

  See _regular expressions_ section in `man awk`

- `&&`, `||`, `!`

  AND, OR, NOT

- `/<regexp1>/, /<regexp2>/`

  Range

- null (empty pattern)

  Matches all records

##### Actions

> Action statements are enclosed in braces, { and }. Action statements consist
> of the usual assignment, conditional, and looping statements found in most
> languages. The operators, control statements, and input/output statements
> available are patterned after those in C.

For a detailed list, see _actions_ section in `man awk`.

<small>
most used control statement
</small>

- `if (<cond>) <stmt> else <stmt>`

<small>
most used i/o statements
</small>

- `next`

  Stop processing current record

- `print <expr_list>`

  Print a comma or space separated expression list

- `printf <fmt> <expr_list>`

  Similar to that in bash

Both `print` and `printf` can be followed by redirection `>` and `>>`.

#### Builtins

##### Variable

- `FS`: Field separator

- `OFS`: Output field separator

  Print username and default shell

  ```bash
  awk 'BEGIN {FS=":"; OFS="-";} {print $1, $NF}' /etc/passwd
  ```

  Same as

  ```bash
  awk -F':' -v OFS='-' '{print $1, $NF}' /etc/passwd
  ```

- `NR`: Record number

  Print file with line number

  ```bash
  awk '{print NR, $0}' <filename>
  ```

  Count line number

  ```bash
  awk 'END {print NR}' <filename>
  ```

- `NF`: Field number

  Print the last field of each record

  ```bash
  awk '{print $NF}' <filename>
  ```

##### Function

- `gsub(re, sub [, str])`: Replace every substring matching _re_ with _sub_ in
  _str_ (default \$0)
- `sub(re, sub [, str])`: Same as `gsub`, but only replace the first match
- `length(str)`: Length of _str_ (default \$0)
- `substr(str, idx [, len])`: Substring of _str_ at index _idx_ of length _len_,
  use the rest of _str_ if _len_ is not provided

### Acknowledgements

- [GNU grep's manual](https://www.gnu.org/software/grep/manual/html_node/index.html)
- [GNU sed's manual](https://www.gnu.org/software/sed/manual/html_node/index.html)
- [GNU awk's manual](https://www.gnu.org/software/gawk/manual/html_node/index.html)
- [When to use grep, sed, and awk](https://unix.stackexchange.com/a/303049)
- [Awk - A useful little language](https://dev.to/rrampage/awk---a-useful-little-language-2fhf)
