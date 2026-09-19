---
tags: [coding, bash, shell]
type: cheatsheet
source: compiled reference (Bash)
last-verified: 2026-08-27
---

# Bash

## Up
- [[Coding]]

A practical Bash scripting cheatsheet — script structure, variables, expansions, control flow, functions, text processing, and safe-scripting habits.

---

## Script Basics

```bash
#!/usr/bin/env bash          # shebang — first line
set -euo pipefail            # fail fast: error, unset var, pipe failure
IFS=$'\n\t'                  # safer word splitting

# run:  chmod +x script.sh && ./script.sh
# or:   bash script.sh
```

| `set` flag | Effect |
|---|---|
| `-e` | Exit on any command returning non-zero |
| `-u` | Error on use of an unset variable |
| `-o pipefail` | A pipeline fails if any stage fails (not just the last) |
| `-x` | Print each command before running (debug/trace) |

---

## Variables

```bash
name="Ada"                   # no spaces around =
greeting="Hello, $name"
readonly PI=3.14             # constant
unset name                   # delete

echo "$name"                 # always quote expansions
echo "${name}_suffix"        # braces to delimit

# command substitution
today=$(date +%Y-%m-%d)
files=$(ls *.txt)

# arithmetic
count=$((2 + 3))
(( count++ ))
result=$(( (a + b) * 2 ))

# environment
export PATH="$HOME/bin:$PATH"
```

### Special variables

| Var | Meaning |
|---|---|
| `$0` | Script name |
| `$1`, `$2`, … | Positional arguments |
| `$#` | Number of arguments |
| `$@` | All args as separate words (use `"$@"`) |
| `$*` | All args as one word |
| `$?` | Exit status of last command |
| `$$` | PID of the script |
| `$!` | PID of last background job |

---

## Parameter Expansion

```bash
${var:-default}     # use default if var unset/empty
${var:=default}     # assign default if unset/empty
${var:?error msg}   # error and exit if unset/empty
${var:+alt}         # use alt if var IS set

${#var}             # length
${var:2}            # substring from index 2
${var:2:3}          # substring, offset 2, length 3

${var#pattern}      # remove shortest leading match
${var##pattern}     # remove longest leading match  (e.g. basename)
${var%pattern}      # remove shortest trailing match
${var%%pattern}     # remove longest trailing match (e.g. strip ext)

${var/foo/bar}      # replace first foo with bar
${var//foo/bar}     # replace all
${var^^}            # uppercase    ${var,,}  lowercase

# examples
path="/a/b/c.txt"
echo "${path##*/}"  # c.txt      (basename)
echo "${path%/*}"   # /a/b       (dirname)
echo "${path%.*}"   # /a/b/c     (strip extension)
```

---

## Quoting & Expansion

```bash
echo "$var"          # double quotes: expand $ but keep as one word
echo '$var'          # single quotes: literal, no expansion
echo "path: ${HOME}/bin"
echo \$literal       # backslash escapes one char

# brace expansion
echo {1..5}          # 1 2 3 4 5
echo {a..e}          # a b c d e
echo file{1,2,3}.txt # file1.txt file2.txt file3.txt
mkdir -p proj/{src,test,docs}

# globbing
*.txt   ?          # any chars / single char
[abc]   [0-9]      # char class / range
shopt -s globstar  # enable ** for recursive match
**/*.md
```

---

## Arrays

```bash
arr=(one two three)
arr+=(four)                  # append
echo "${arr[0]}"             # one
echo "${arr[@]}"             # all elements
echo "${#arr[@]}"            # length
echo "${arr[@]:1:2}"         # slice: 2 elements from index 1

for x in "${arr[@]}"; do echo "$x"; done

# associative array (Bash 4+)
declare -A ages
ages[alice]=30
echo "${ages[alice]}"
for k in "${!ages[@]}"; do echo "$k=${ages[$k]}"; done
```

---

## Control Flow

```bash
# if — note the [[ ]] test brackets
if [[ "$x" -gt 10 ]]; then
    echo "big"
elif [[ "$x" -eq 10 ]]; then
    echo "ten"
else
    echo "small"
fi

# one-liners
[[ -f file.txt ]] && echo "exists"
[[ -d dir ]] || mkdir dir

# case
case "$1" in
    start)      echo "starting" ;;
    stop|halt)  echo "stopping" ;;
    *)          echo "unknown" ;;
esac

# loops
for i in {1..5}; do echo "$i"; done
for f in *.txt; do echo "$f"; done
for ((i=0; i<5; i++)); do echo "$i"; done

while [[ "$n" -lt 5 ]]; do (( n++ )); done
until [[ "$done" == "yes" ]]; do ...; done

while IFS= read -r line; do   # read a file line by line
    echo "$line"
done < input.txt

break ; continue
```

### Test operators

| String | Numeric | File |
|---|---|---|
| `-z str` empty | `-eq` equal | `-e` exists |
| `-n str` non-empty | `-ne` not equal | `-f` regular file |
| `s1 == s2` equal | `-lt` less than | `-d` directory |
| `s1 != s2` | `-le` ≤ | `-r`/`-w`/`-x` readable/writable/executable |
| `s1 =~ regex` match | `-gt` `-ge` | `-s` non-empty file |

Use `[[ ]]` (Bash) over `[ ]` (POSIX) — it handles `&&`, `||`, `<`, and `=~` safely and doesn't need quoting inside.

---

## Functions

```bash
greet() {
    local name="$1"           # local scope
    echo "Hello, $name"
    return 0                  # exit status (0-255), not a value
}

greet "Ada"                   # call
result=$(greet "Ada")         # capture output as "return value"

# all args
log() { echo "[$(date +%T)] $*"; }
```

Functions "return" data via stdout (captured with `$(...)`); `return` sets only the exit status.

---

## Input / Output & Redirection

```bash
command > file          # stdout to file (overwrite)
command >> file         # append
command 2> err.log      # stderr to file
command &> all.log      # both stdout+stderr
command 2>&1            # redirect stderr to wherever stdout goes
command < input.txt     # stdin from file
command1 | command2     # pipe stdout → stdin
command 2>/dev/null     # discard errors

# here-doc
cat <<EOF
multi-line
$var is expanded
EOF

cat <<'EOF'             # quoted delimiter = no expansion
literal $var
EOF

# read input
read -r -p "Name: " name
read -rp "Password: " -s pw    # silent
```

---

## Text Processing (the core toolkit)

```bash
grep "pattern" file            # search lines
grep -r -i -n "todo" .         # recursive, case-insensitive, line numbers
grep -E "a|b" ; grep -v "skip" # extended regex / invert

sed 's/old/new/g' file         # substitute (all on each line)
sed -i 's/old/new/g' file      # edit in place
sed -n '5,10p' file            # print lines 5–10

awk '{print $1, $3}' file      # fields (space/tab separated)
awk -F, '{print $2}' file      # comma-delimited
awk '$3 > 100 {print $1}' file # conditional
awk '{sum+=$1} END {print sum}'# aggregate

cut -d, -f1,3 file             # cut columns by delimiter
sort file | uniq -c | sort -rn # count & rank unique lines
tr 'a-z' 'A-Z' < file          # translate chars
wc -l file                     # line count  (-w words, -c bytes)
head -n 20 file ; tail -n 20 file ; tail -f log
xargs                          # build commands from stdin
```

```bash
# common combo: find + xargs
find . -name "*.log" -mtime +7 -print0 | xargs -0 rm
find . -type f -name "*.py" | xargs grep -l "import os"
```

---

## Exit Codes & Error Handling

```bash
command || { echo "failed" >&2; exit 1; }

if ! command; then
    echo "error" >&2
    exit 1
fi

# trap: run cleanup on exit or signal
cleanup() { rm -f "$tmpfile"; }
trap cleanup EXIT
trap 'echo interrupted; exit 130' INT TERM

tmpfile=$(mktemp)              # safe temp file
```

Exit `0` = success, non-zero = failure. `>&2` sends messages to stderr.

---

## Job Control

```bash
command &            # run in background
jobs                 # list background jobs
fg %1 ; bg %1        # foreground / background job 1
wait                 # wait for all background jobs
wait $!              # wait for the last one
nohup cmd &          # survive terminal close
kill %1 ; kill -9 PID
```

---

## Safe-Scripting Checklist

- Start with `#!/usr/bin/env bash` and `set -euo pipefail`.
- **Always quote** variable expansions: `"$var"`, `"$@"`, `"${arr[@]}"`.
- Use `[[ ]]` for tests, `$(...)` for command substitution (not backticks).
- Prefer `local` variables inside functions.
- Check that required args/commands exist (`command -v tool >/dev/null || exit 1`).
- Use `mktemp` for temp files and `trap` to clean up.
- Run scripts through `shellcheck` — it catches quoting and portability bugs.
