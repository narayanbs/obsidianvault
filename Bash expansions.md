In Bash, **substitutions happen before word splitting**.

The order is roughly:

1. **Brace expansion**
2. **Tilde expansion**
3. **Parameter expansion** (`$var`)
4. **Command substitution** (`$(...)` or `` `...` ``)
5. **Arithmetic expansion** (`$((...))`)
6. **Process substitution** (`<(...)`, `>(...)`) (where supported)
7. **Word splitting**
8. **Filename expansion (globbing)** (`*`, `?`, `[abc]`)
9. **Quote removal**

The important point is that **word splitting operates on the results of expansions**, not on the original command text.

### Example

```bash
var="hello world"

echo $var
```

Steps:

* Parameter expansion: `$var` → `hello world`
* Word splitting: `hello world` → `hello`, `world`
* `echo` receives two arguments.

Output:

```
hello world
```

Although the output looks the same, `echo` was actually passed two separate arguments.

### Quoted expansion

```bash
echo "$var"
```

Steps:

* Parameter expansion: `$var` → `hello world`
* Because it's quoted, **word splitting is skipped**.
* `echo` receives one argument: `hello world`.

### Command substitution

```bash
files=$(printf "a b\nc")
echo $files
```

Steps:

1. Command substitution:

   ```
   a b
   c
   ```
2. Word splitting (using `$IFS`, which defaults to spaces, tabs, and newlines):

   ```
   a
   b
   c
   ```
3. Globbing is then performed on each resulting word unless prevented by quoting.

### A subtle consequence

```bash
touch foo
var="*"

echo $var
```

Processing:

* Parameter expansion: `*`
* Word splitting: still one word (`*`)
* Filename expansion: `*` expands to matching filenames (including `foo`)

If you write:

```bash
echo "$var"
```

then the `*` remains a literal `*` because quoting suppresses both word splitting and filename expansion.

So, the short answer is: **Bash performs substitutions first, then word splitting, and then pathname expansion.**


## Tilde expansion

**Tilde expansion** is the process where Bash replaces a leading `~` with a home directory.

It happens **before** parameter expansion, command substitution, and word splitting.

Here are the common forms:

| Input    | Expands to                             |
| -------- | -------------------------------------- |
| `~`      | Your home directory                    |
| `~/docs` | `your_home/docs`                       |
| `~alice` | Alice's home directory                 |
| `~+`     | Current working directory (`$PWD`)     |
| `~-`     | Previous working directory (`$OLDPWD`) |

### Examples

```bash
echo ~
```

might become

```text
/home/sam
```

before `echo` is executed.

```bash
cd ~/projects
```

becomes something like

```bash
cd /home/sam/projects
```

### Another user's home

```bash
echo ~root
```

might produce

```text
/root
```

if the user `root` exists.

### `~+` and `~-`

```bash
pwd
# /tmp

echo ~+
# /tmp
```

If you then do:

```bash
cd /etc
cd -
```

then:

```bash
echo ~-
```

prints the previous directory.

### Quoting matters

Tilde expansion only occurs when the `~` is **unquoted** and appears at the **beginning of a word**.

```bash
echo ~
```

→ expands.

```bash
echo "~"
```

→ prints

```text
~
```

because the quotes prevent tilde expansion.

Similarly,

```bash
x=~
echo "$x"
```

works because the `~` in the assignment is expanded **when the assignment is parsed**, so `x` stores the actual home directory.

But:

```bash
x="~"
echo "$x"
```

prints

```text
~
```

because the `~` was quoted and never expanded.

### Important limitation

Tilde expansion is **not recursive**. Bash only looks for `~` in the original command text, not in the result of later expansions.

For example:

```bash
x="~"
echo $x
```

prints

```text
~
```

not your home directory. The `~` came from parameter expansion, and tilde expansion had already happened earlier in the expansion order.
