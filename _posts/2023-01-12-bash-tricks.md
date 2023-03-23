---
layout: post
title: 'Bash Scripting Tricks'
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

## Special variables

| Syntax  | Description                         |
| ------- | ----------------------------------- |
| \$0     | Filename of current script          |
| \$1-\$9 | The n^th^ argument                  |
| \$\$    | Process ID                          |
| \$#     | Number of arguments                 |
| \$\*    | The list of arguments               |
| \$@     | The list of arguments               |
| \$?     | The exit status of previous command |

- Difference between `$*` and `$@`

```bash
for arg in $*; do
  echo "$arg "
done

for arg in $@; do
  echo "$arg "
done

for arg in "$*"; do
  echo "$arg "
done

for arg in "$@"; do
  echo "$arg "
done
```

> Test with arguments `"a 1" "b 2" "c 3"`

```
a
1
b
2
c
3
a
1
b
2
c
3
a 1 b 2 c 3
a 1
b 2
c 3
```

When unquoted, `$*` and `$@` are identical, both breaking up arguments whether
quoted or not. However, when quoted, `$*` is all arguments joined together with
spaces; `$@` is an array of arguments matching exactly what was given to shell.

## Parameter expansion

`${#var}` expands to length in char of `var`

### Conditional

| Syntax             | Unset                 | Empty     | Has content |
| ------------------ | --------------------- | --------- | ----------- |
| \${var-default}    | default               |           | content     |
| \${var:-default}   | default               | default   | content     |
| \${var+alternate}  |                       | alternate | alternate   |
| \${var:+alternate} |                       |           | alternate   |
| \${var?error}      | print error to stderr |           | content     |
| \${var:?error}     | print error to stderr | error     | content     |

> Example script

```bash
#!/bin/bash
in="${1:?Please provide input file.}"
out="${2:-out.gif}"
png="$(mktemp --suffix '.png')"
[[ ! -f "$in" ]] && {
  echo "File not found."
  exit 1
}

ffmpeg -y -i ...
```

### Substring

> Pattern is same as globbing

| Syntax                | Description                                |
| --------------------- | ------------------------------------------ |
| \${var#pattern}       | Remove characters from left edge           |
| \${var##pattern}      | Remove characters from left edge greedily  |
| \${var%pattern}       | Remove characters from right edge          |
| \${var%%pattern}      | Remove characters from right edge greedily |
| \${var:offset}        |                                            |
| \${var:offset:length} |                                            |
| \${@:offset}          | Slice argument array                       |
| \${@:offset:length}   |                                            |
| \${\*:offset}         | Slice argument array                       |
| \${\*:offset:length}  |                                            |

- PATH priority

```bash
echo "Lowest priority in PATH: ${PATH##*:}"
echo "Everything except lowest priority: ${PATH%:*}"
echo "Highest priority in PATH: ${PATH%%:*}"
echo "Everything except highest priority: ${PATH#*:}"
```

- Batch convert img

```bash
for f in *.png; do ffmpeg -i "$f" "${f%.png}.jpg"; done
```

### Substitution

| Syntax                  | Description                     |
| ----------------------- | ------------------------------- |
| \${var/pattern/string}  | Replace only the leftmost match |
| \${var//pattern/string} | Replace all matches             |
| \${var/#pattern/string} | Match at the beginning          |
| \${var/%pattern/string} | Match at the end                |

> Pattern matching is always greedy

### Bash array

| Syntax      | Description              |
| ----------- | ------------------------ |
| \${arr[*]}  | All elements of an array |
| \${arr[@]}  | All elements of an array |
| \${!arr[*]} | All indices of an array  |
| \${!arr[@]} | All indices of an array  |
| \${#arr[*]} | The length of an array   |
| \${#arr[@]} | The length of an array   |

Similar to `$*` and `$@`, `${arr[*]}` and `${arr[@]}` behaves differently when
quoted.

```bash
for el in ${arr[*]}; do
  echo $el
done
for el in ${arr[@]}; do
  echo $el
done
for el in "${arr[*]}"; do
  echo $el
done
for el in "${arr[@]}"; do
  echo $el
done
```

> Test with `arr=("a 1" "b 2" "c 3")`

```
a
1
b
2
c
3
a
1
b
2
c
3
a 1 b 2 c 3
a 1
b 2
c 3
```

[This answer](https://stackoverflow.com/a/39690101) gives another example where
quoted array parameter expansion is used along with `printf`.

## Redirection

File descriptors for stdin, stdout, and stderr are 0, 1, and 2, respectively.

| Syntax                            | Description                               |
| --------------------------------- | ----------------------------------------- |
| `<command> < <input_file>`        | Accept input from a file                  |
| `<command> > <output_file>`       | Redirect stdout to a file                 |
| `<command> >> <output_file>`      | Append stdout to a file                   |
| `> <output_file>`                 | Empty a file                              |
| `<command> 2> <output_file>`      | Redirect stderr to a file                 |
| `<command> 2>> <output_file>`     | Append stdout to a file                   |
| `<command> &> <output_file>`      | Redirect both stdout and stderr to a file |
| `<command> >> <output_file> 2>&1` | Append both stdout and stderr to file     |
| `<command> 2>&1 <output_file>`    | Redirect stderr to where stdout goes      |
| `<command> >&2`                   | Redirect stdout to where stderr goes      |
| <code>&#124;</code>               | Pipe                                      |

Multiple instances of input and output redirection and/or pipes can be combined
in a single command line.

```bash
<command1> | <command2> | <command3> < <input_file> > <output_file>
```

[This answer](https://stackoverflow.com/a/876242) explains why
`<command> >> <output_file> 2>&1` works as expected and
`<command> 2>&1 >> <output_file>` does not.

- Suppress stderr

```bash
<command> 2> /dev/null
```

- Suppress both stdout and stderr

```bash
<command> &> /dev/null
```

```bash
<command> > /dev/null 2>&1
```

## Here document

Here document is a special form of redirection, which uses a pair of limit
string to feed a multiline string into the stdin of a command.

- Print help message

```bash
#!/bin/bash

print_help() {
  cat <<EOF
Usage: ...

Options: ...
  -h, --help               Show help message
EOF
  exit 1
}

print_help
```

- Write to file

```bash
#!/bin/bash

config="$HOME/myconf"
write_config() {
  cat <<EOF >"$config"
opt1 = 1
opt2 = 2
EOF
}

write_config
```

- Pipe to another command

```bash
#!/bin/bash

pipe_command() {
  cat <<EOF | mysql
SELECT * FROM users;
EOF
}

pipe_command
```

Here string is another form of here document.

- Prepend a line to a file

```bash
cat - $file <<< $line > $newfile
```

## Pattern matching

Bash itself does not recognize regular expressions, but it does carry out
filename expansion, also known as globbing. The re-matching operator `=~`
accepts regular expressions, as of Bash version 3.

### Regular expression

Certain commands and utilities, such as `grep`, `sed` and `awk`, interpret
regular expressions.

Any special symbol may become literal by preceding it with a `\`.

- Basic re

| Symbol             | Match                                 |
| ------------------ | ------------------------------------- |
| `*`                | zero or more of re before             |
| `.`                | one of any char except newline        |
| `^`                | beginning of line                     |
| `$`                | end of line                           |
| Bracket expression |                                       |
| `\`                | escapes special character             |
| `\<\>`             | marks word boundaries                 |
| `\b`               | marks word boundaries                 |
| `\B`               | marks the opposite of word boundaries |

- Extended re

| Symbol              | Match                              |
| ------------------- | ---------------------------------- |
| `?`                 | zero or one of re before           |
| `+`                 | one or more of re before           |
| `{}`                | number of occurrences of re before |
| `()`                | re group                           |
| <code>&#124;</code> | or operator                        |

### Globbing

| Symbol             | Match                  |
| ------------------ | ---------------------- |
| `*`                | any number of any char |
| `?`                | one of any char        |
| Bracket expression |                        |

> Wildcards should be escaped to match their literals

Bash performs filename expansion on _unquoted_ command-line arguments like
`echo *`.

```bash
str="a * b"
for w in $str; do
  echo $w
done
```

```
a
<every_file_in_current_folder>
b
```

Wildcards will not expand to a dot in globbing, it has to be explicitly
included. For example, to match `.bashrc` and `.zshrc`, `.*shrc` instead of
`*shrc` should be used.

## Word splitting

- When does word splitting happen?

> The shell scans the results of parameter expansion, command substitution, and
> arithmetic expansion that did not occur within double quotes for word
> splitting.

- Some examples

```bash
str="a b c"
for w in $str; do
  echo $w
done
for w in "$str"; do
  echo $w
done
```

```
a
b
c
a b c
```

```bash
str="a b c"
printf "%s\n" $str
printf "%s\n" "$str"
```

```
a
b
c
a b c
```

## Parse command-line arguments

[Here](https://github.com/MusicFreak456/Uniwroc/blob/master/SemestrVI/Linux/lista3/zadanie4/hwb)
is a good practice of parsing command-line arguments.

```bash
./parse_cmdline_args.sh --help
./parse_cmdline_args.sh --output file
./parse_cmdline_args.sh --output=file
./parse_cmdline_args.sh -o file
./parse_cmdline_args.sh -ofile
./parse_cmdline_args.sh -o file --verbose foo bar
```

```bash
#!/bin/bash

print_help() {
  cat <<EOF
Usage: ...

Options: ...
  -h, --help               Show help message
EOF
  exit 1
}

args=$(getopt --options "ho:v" --longoptions "help,output:,verbose" -- "$@" 2>/dev/null)
if [[ $? -ne 0 ]]; then
  print_help
  exit 1
fi

eval set -- "$args"

OUTPUT="default_name"
VERBOSE=0

while [ : ]; do
  case "$1" in
    -h | --help)
      print_help
      exit 1
      ;;
    -o | --output)
      OUTPUT="$2"
      shift
      ;;
    -v | --verbose)
      VERBOSE=1
      ;;
    --)
      shift
      break
      ;;
    *)
      echo "Invalid option: $1" >&2
      exit 1
      ;;
  esac
  shift
done

# handle positional arguments
printf "%s\n" "$@"
```

## Quotes, Brackets, Ampersand, Bar

Remember to always use double quotes around parameter expansions because it has
fewer surprises and most usually does what we want.
[This answer](https://stackoverflow.com/a/55023462) is a simple illustration.

Remember to always use double brackets in test constructs because it is
generally safer to use. [This answer](https://serverfault.com/a/52050) explains
the pros.

[Ampersands & on the command line](https://bashitout.com/2013/05/18/Ampersands-on-the-command-line.html)

[Bash command chaining](https://askubuntu.com/a/539293)

```bash
[[ -d "$name" ]] || mkdir -p "$name"
```

```bash
selected=$(cat "$file1" "$file2" | fzf)
```

```bash
newstr=$(echo "$str" | tr ' ' '_')
```

## Command groups

- Parentheses

```bash
(<command1>; <command2>; <command3>)
```

This command group creates a subshell and executes in a separate environment.

- Braces

```bash
{ <command1>; <command2>; <command3>; }
```

This command group executes in the current context and _allows redirection_.

> Notice the trailing semicolon and whitespaces on both sides

[Here](https://github.com/ohmyzsh/ohmyzsh/blob/master/oh-my-zsh.sh) is a good
practice of command grouping.

## Error handling & Debugging

- Stop on error

```bash
command || { echo "command error" >&2; exit 1; }
```

```bash
if ! command; then echo "command error" >&2; exit 1; fi
```

```bash
command
if [[ $? -ne 0 ]]; then echo "command error" >&2; exit 1; fi
```

- Useful when debugging

```bash
set -Eeux
set -o pipefail
```

## Exit status

- Reserved codes

| Code # | Meaning                          |
| ------ | -------------------------------- |
| 2      | Misuse of shell builtins         |
| 126    | Command found but not executable |
| 127    | Command not found                |
| 130    | Keyboard interruption            |

Exit statuses fall between 0 and 255. A 0 status means a command has executed
successfully. A 1 status means a general error during execution.

`$?` gives the exit status of previous command.

## References

- [Bash Test Operators](https://kapeli.com/cheat_sheets/Bash_Test_Operators.docset/Contents/Resources/Documents/index)
- [Brace expansion](https://www.howtogeek.com/725657/how-to-use-brace-expansion-in-linuxs-bash-shell/)
- [Character classes and bracket expression](https://www.gnu.org/software/grep/manual/html_node/Character-Classes-and-Bracket-Expressions.html)
- [Cheatsheet](https://tldp.org/LDP/abs/html/refcards.html)
- [Exit Status](https://www.gnu.org/software/bash/manual/html_node/Exit-Status.html)
- [Filename expansion](https://www.gnu.org/software/bash/manual/html_node/Filename-Expansion.html)
- [Globbing](https://tldp.org/LDP/abs/html/globbingref.html)
- [Parameter expansion](https://opensource.com/article/17/6/bash-parameter-expansion)
- [Regular expression](https://tldp.org/LDP/abs/html/x17129.html)
- [Special Characters](https://tldp.org/LDP/abs/html/special-chars.html)
- [The Set Builtin](https://www.gnu.org/software/bash/manual/html_node/The-Set-Builtin.html)
- [The Shopt Builtin](https://www.gnu.org/software/bash/manual/html_node/The-Shopt-Builtin.html)
- [Word splitting](https://www.gnu.org/software/bash/manual/html_node/Word-Splitting.html)

## Great sites

- [Advanced Bash-Scripting Guide](https://tldp.org/LDP/abs/html/)
- [Dash Cheat Sheets](https://kapeli.com/cheatsheets)
- [Find command-line tools](https://command-not-found.com/)
- [TLDR](https://tldr.finzzz.net/)
