In Go, **moving from `v1` to `v2` is not just a Git tag change**. Starting with `v2`, the **module path itself must include `/v2`**. This is called **semantic import versioning**. ([Go][1])

Let's use your sample module:

```
narayanbs.com/modlib
```

## Version 1

Repository layout:

```
modlib/
├── go.mod
├── math.go
└── strings.go
```

### go.mod

```go
module narayanbs.com/modlib

go 1.24
```

### math.go

```go
package modlib

func Add(a, b int) int {
    return a + b
}
```

Consumers use it like this:

```go
import "narayanbs.com/modlib"

func main() {
    fmt.Println(modlib.Add(1, 2))
}
```

You release:

```
git tag v1.0.0
git push origin v1.0.0
```

---

# Now you want to introduce breaking changes

Suppose you decide:

```go
func Add(a, b int) int
```

becomes

```go
func Add(values ...int) int
```

Since this breaks compatibility, it must become **v2**.

---

# Option 1 (Recommended): Create a v2 directory

Your repository becomes:

```
modlib/
│
├── go.mod
├── math.go
│
└── v2/
    ├── go.mod
    ├── math.go
    └── strings.go
```

The root stays as **v1**.

---

## v2/go.mod

```go
module narayanbs.com/modlib/v2

go 1.24
```

Notice the module path.

It is **not**

```
module narayanbs.com/modlib
```

It **must** be

```
module narayanbs.com/modlib/v2
```

---

## v2/math.go

```go
package modlib

func Add(values ...int) int {
    sum := 0

    for _, v := range values {
        sum += v
    }

    return sum
}
```

---

## Release it

```
git tag v2.0.0
git push origin v2.0.0
```

No special tag like

```
v2/v2.0.0
```

Just

```
v2.0.0
```

is correct. ([Go][1])

---

# Client using v1

```go
import "narayanbs.com/modlib"

func main() {
    modlib.Add(1, 2)
}
```

Nothing changes.

---

# Client upgrading to v2

They change the import:

```go
import "narayanbs.com/modlib/v2"

func main() {
    modlib.Add(1, 2, 3, 4)
}
```

Notice the package name is still `modlib`; only the **import path** changes.

---

# Updating your own imports

If your module has multiple packages, every internal import must also use `/v2`.

Suppose v1 has:

```go
import "narayanbs.com/modlib/internal/math"
```

In v2 it becomes:

```go
import "narayanbs.com/modlib/v2/internal/math"
```

This is a step people often overlook. ([Go][2])

---

# How users install v2

```bash
go get narayanbs.com/modlib/v2@latest
```

or

```bash
go get narayanbs.com/modlib/v2@v2.1.3
```

---

# Can v1 and v2 coexist?

Yes, and this is one of the reasons Go requires the `/v2` suffix.

A program can legally use both versions:

```go
import (
    v1 "narayanbs.com/modlib"
    v2 "narayanbs.com/modlib/v2"
)

func main() {
    fmt.Println(v1.Add(1, 2))
    fmt.Println(v2.Add(1, 2, 3))
}
```

The Go toolchain treats them as **different modules** because they have different module paths. ([Go][3])

---

## Summary

| Version | `go.mod` module path             | Import path               | Git tag  |
| ------- | -------------------------------- | ------------------------- | -------- |
| v1      | `module narayanbs.com/modlib`    | `narayanbs.com/modlib`    | `v1.0.0` |
| v2      | `module narayanbs.com/modlib/v2` | `narayanbs.com/modlib/v2` | `v2.0.0` |

This "major version in the import path" rule is unique to Go's module system and is designed to let incompatible major versions coexist safely in the same build. ([Go][1])

[1]: https://go.dev/blog/v2-go-modules?utm_source=chatgpt.com "Go Modules: v2 and Beyond - The Go Programming Language"
[2]: https://go.dev/doc/modules/major-version?utm_source=chatgpt.com "Developing a major version update - The Go Programming Language"
[3]: https://go.dev/blog/using-go-modules?utm_source=chatgpt.com "Using Go Modules - The Go Programming Language"

# Practical example

Sure. Let's assume you're developing on GitHub and your repository is:

```text
https://github.com/narayanbs/modlib
```

but your module path is

```text
narayanbs.com/modlib
```

(That works as long as `narayanbs.com/modlib` redirects to the GitHub repo.)

I'll show the complete lifecycle.

---

# 1. Create the repository

```bash
mkdir modlib
cd modlib

git init
go mod init narayanbs.com/modlib
```

Your tree looks like:

```text
modlib/
    go.mod
```

Create a file.

```go
// math.go

package modlib

func Add(a, b int) int {
    return a + b
}
```

Commit it.

```bash
git add .
git commit -m "Initial version"
```

Connect to GitHub.

```bash
git remote add origin git@github.com:narayanbs/modlib.git
```

Push.

```bash
git branch -M main
git push -u origin main
```

---

# 2. Release v1.0.0

You're still in the root of the repository.

```bash
pwd
```

```
.../modlib
```

Create the tag.

```bash
git tag v1.0.0
```

Verify it.

```bash
git tag
```

```
v1.0.0
```

Push the tag.

```bash
git push origin v1.0.0
```

Notice this is **not**

```bash
cd v1
git tag ...
```

There is no `v1` directory.

Tags always belong to the **repository**, not to a directory.

---

# 3. Continue development

Later you decide to make breaking changes.

Create the v2 directory.

```bash
mkdir v2
```

Inside it:

```bash
cd v2
```

Initialize another module.

```bash
go mod init narayanbs.com/modlib/v2
```

Create the new code.

```go
package modlib

func Add(values ...int) int {
    sum := 0
    for _, v := range values {
        sum += v
    }
    return sum
}
```

Now your tree is

```text
modlib/
    go.mod
    math.go

    v2/
        go.mod
        math.go
```

---

Go back to the repository root.

```bash
cd ..
```

You're back in

```
modlib/
```

Commit everything.

```bash
git add .
git commit -m "Add v2 module"
```

Push.

```bash
git push
```

---

# 4. Release v2.0.0

Again, you're at the repository root.

```bash
pwd
```

```
.../modlib
```

Create the tag.

```bash
git tag v2.0.0
```

Verify.

```bash
git tag
```

```
v1.0.0
v2.0.0
```

Push the tag.

```bash
git push origin v2.0.0
```

That's it.

Notice you **did not**

```bash
cd v2
git tag v2.0.0
```

Git doesn't work that way.

The tag points to a **commit**, and that commit contains the entire repository—including the `v2` directory.

---

# 5. What happens when a user runs

```bash
go get narayanbs.com/modlib/v2
```

Go:

1. Clones the repository.
    
2. Finds the `v2.0.0` tag.
    
3. Checks out that commit.
    
4. Looks inside the `v2/` directory.
    
5. Reads `v2/go.mod`.
    
6. Confirms it says:
    

```go
module narayanbs.com/modlib/v2
```

Then it builds from there.

---

# 6. Releasing v2.1.0

Suppose you only modify `v2/math.go`.

```bash
cd v2
vim math.go
cd ..
```

Commit.

```bash
git add .
git commit -m "Improve Add implementation"
```

Push.

```bash
git push
```

Create the new tag.

```bash
git tag v2.1.0
git push origin v2.1.0
```

---

# 7. Releasing a v1 bug fix

Suppose someone reports a bug in the original API.

Edit the root module:

```text
modlib/
    math.go
```

Do **not** touch `v2/`.

Commit.

```bash
git add .
git commit -m "Fix v1 bug"
git push
```

Tag it.

```bash
git tag v1.0.1
git push origin v1.0.1
```

Now users can choose:

```bash
go get narayanbs.com/modlib@v1.0.1
```

or

```bash
go get narayanbs.com/modlib/v2@v2.1.0
```

Both versions coexist in the same repository.

---

The key idea that often clicks for people is this:

- **Git tags apply to the entire repository at a specific commit.** They are created from the repository root (or any directory within the repo—it doesn't matter, because Git tracks the whole repository, not your current working directory).
    
- **Go modules determine which part of the repository to use.** For `v1`, Go uses the root module (`go.mod` in the repository root). For `v2`, Go follows the `/v2` suffix in the import path, enters the `v2/` directory, and reads the `go.mod` there.
    

So the Git tag and the Go module directory serve different purposes: the tag identifies the repository snapshot, while the module path tells Go which module within that snapshot to build.