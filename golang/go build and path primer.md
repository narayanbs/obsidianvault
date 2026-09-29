 The easiest way to understand Go is to separate **your source code**, **module dependencies**, **caches**, and **compiled binaries**. Once those paths make sense, most Go commands become much less mysterious.

 # Go on Linux: paths, environment variables, modules & builds

 We'll use this example project throughout:

```
myapp/
├── go.mod
├── go.sum
├── main.go
└── internal/
    └── ...
```

---

 ## 1\. The most important Go paths

 Run:

```
go env
```

 You'll see a lot of variables. These are the ones worth knowing.

 ### `GOROOT`

```
go env GOROOT
```

 Example:

```
/usr/local/go
```

 This is where the **Go installation itself** lives.

 You might find:

```
/usr/local/go/
├── bin/
├── src/
├── pkg/
└── ...
```

 For example:

```
$GOROOT/bin/go
```

 is the Go compiler/command itself.

 You generally **don't put your own libraries or projects inside `GOROOT`**.

 Think:

```
GOROOT = Go itself
```

---

 ### `GOPATH`

```
go env GOPATH
```

 Typically:

```
/home/youruser/go
```

 Historically, `GOPATH` was extremely important because Go projects lived under:

```
$GOPATH/src/
```

 Modern Go uses **modules**, so you don't need to keep projects there.

 But `GOPATH` is still important because Go uses it for things such as:

```
~/go/
├── bin/
└── pkg/
    └── mod/
```

 Think:

```
GOPATH = your Go workspace/cache area
```

---

 ## 2\. `GOBIN`

```
go env GOBIN
```

 This determines where Go installs executables when you use:

```
go install
```

 If `GOBIN` is empty, Go normally uses:

```
$GOPATH/bin
```

 For example:

```
go install github.com/some/tool@latest
```

 might produce:

```
~/go/bin/tool
```

 So:

```
GOBIN
  ↓
where `go install` puts executables
```

 You may want:

```
export PATH="$PATH:$(go env GOPATH)/bin"
```

 in your shell configuration so installed Go tools can be executed directly.

---

 # 3\. `GOMODCACHE`

 This is one of the most useful ones to understand.

```
go env GOMODCACHE
```

 Usually:

```
~/go/pkg/mod
```

 This is where Go downloads **module dependencies**.

 Suppose your project imports:

```
import "github.com/gin-gonic/gin"
```

 and `go.mod` requires:

```
github.com/gin-gonic/gin v1.10.0
```

 Go may download it into something like:

```
~/go/pkg/mod/github.com/gin-gonic/gin@v1.10.0/
```

 So:

```
GOMODCACHE
    ↓
downloaded Go module source
```

 You can inspect it:

```
ls "$(go env GOMODCACHE)"
```

---

 # 4\. `GOCACHE`

```
go env GOCACHE
```

 Typically something like:

```
~/.cache/go-build
```

 This is the **build cache**.

 Suppose you run:

```
go build .
```

 Go compiles your source code and dependencies.

 It doesn't necessarily recompile everything from scratch every time.

 Go can reuse previously compiled artifacts from:

```
~/.cache/go-build/
```

 So you can think:

```
GOMODCACHE
    = downloaded dependency source

GOCACHE
    = cached compilation results
```

 They're different things.

---

 # 5\. `GOENV`

 You may encounter:

```
go env GOENV
```

 It tells you where Go's persistent environment configuration is stored.

 For example:

```
~/.config/go/env
```

 You can set persistent Go configuration with:

```
go env -w GOPROXY=https://proxy.golang.org,direct
```

 This doesn't necessarily modify your shell's environment variables. Go stores the setting in its own environment configuration.

 You can inspect it with:

```
go env -w
```

 Actually, to see the current values, the simpler command is:

```
go env
```

---

 # 6\. `GOOS` and `GOARCH`

 These are particularly important for building programs for another operating system or CPU architecture.

 Check yours:

```
go env GOOS
go env GOARCH
```

 On a typical 64-bit Linux machine:

```
linux
amd64
```

 For example:

```
GOOS=linux GOARCH=amd64 go build .
```

 Build for ARM64:

```
GOOS=linux GOARCH=arm64 go build .
```

 Build for Windows:

```
GOOS=windows GOARCH=amd64 go build .
```

 Build for macOS ARM:

```
GOOS=darwin GOARCH=arm64 go build .
```

 This is one of Go's really nice features: **cross-compilation is built into the toolchain.**

---

 # 7\. The project itself: `go.mod`

 Modern Go projects normally start with:

```
go mod init example.com/myapp
```

 This creates:

```
go.mod
```

 For example:

```
module example.com/myapp

go 1.24

require github.com/gin-gonic/gin v1.10.0
```

 The important concept is:

```
go.mod
   ↓
defines your module
   ↓
declares its dependencies
```

 Your project does **not** need to live under `GOPATH`.

 You can have:

```
~/projects/myapp/
```

 or:

```
/opt/myapp/
```

 or wherever you want.

---

 # 8\. `go.sum`

 You'll often see:

```
go.mod
go.sum
```

 `go.sum` contains cryptographic checksums for module dependencies.

 For example:

```
github.com/foo/bar v1.2.3 h1:...
```

 It's used to help Go verify that downloaded modules correspond to expected content.

 Generally:

 > **Commit `go.mod` and `go.sum` to version control.**

 Don't manually edit `go.sum` unless you have a very specific reason.

---

 # 9\. What happens when you run `go build`?

 Suppose:

```
myapp/
├── go.mod
├── go.sum
└── main.go
```

 You execute:

```
go build .
```

 Conceptually:

```
                 go build .
                      │
                      ▼
                 read go.mod
                      │
                      ▼
             determine dependencies
                      │
             ┌────────┴────────┐
             │                 │
     dependency exists     missing dependency
     in module cache?      from module cache?
             │                 │
             │                 ▼
             │          download module
             │                 │
             └────────┬────────┘
                      ▼
                   compile
                      │
                      ▼
                    link
                      │
                      ▼
               executable
```

 If your package is a command (`package main`), you'll normally get an executable in the current directory:

```
myapp/
├── go.mod
├── go.sum
├── main.go
└── myapp      ← executable
```

---

 # 10\. `go run`

 Now:

```
go run .
```

 Conceptually:

```
source
  ↓
build
  ↓
temporary executable
  ↓
execute it
```

 You don't normally see the executable because Go puts it in a temporary build location.

 So:

```
go run .
    = build + execute temporary binary
```

 Whereas:

```
go build .
    = build + keep binary
```

---

 # 11\. `go install`

 This is different from `go build`.

 For example:

```
go install ./cmd/mytool
```

 builds the executable and installs it into the Go binary installation directory.

 Typically:

```
~/go/bin/mytool
```

 You can find the location with:

```
go env GOBIN
go env GOPATH
```

 If `GOBIN` is empty:

```
$GOPATH/bin
```

 is generally used.
 
 
 `go install` needs a **package path**, and that package must be an executable package:

```
package main
```

 For example:

```
go install github.com/alice/myapp/cmd/server@latest
```

 Go resolves that package path:

```
github.com/alice/myapp/cmd/server
                │
                ▼
            cmd/server/
            ├── main.go
            └── config.go
                │
                ▼
           package main
                │
                ▼
            func main()
                │
                ▼
          $GOBIN/server
```

 ### Contrast with a library

 Suppose you have:

```
github.com/alice/myapp/database
```

 whose files say:

```
package database
```

 That's a **library package**, not an executable.

 So:

```
go install github.com/alice/myapp/database
```

 isn't useful for producing a binary, because there's no `package main` executable there.

 A common modern Go layout is therefore:

```
myapp/
├── go.mod
├── internal/
│   └── database/       ← library packages
│
└── cmd/
    ├── server/         ← package main → binary "server"
    └── worker/         ← package main → binary "worker"
```

 Then:

```
go install ./cmd/server
go install ./cmd/worker
```

 produces two executables:

```
$GOBIN/
├── server
└── worker
```

 So the short mental model is:

 > **`go install` takes a package path  to  package `main`, Go builds an executable → installs the executable into `GOBIN`.**

If you use go install on a package that is not package main, it fails because there is no executable to install.

if we run 
```go
go install github.com/alice/myapp/database
```
we get
```go
package github.com/alice/myapp/database is not a main package

```

---

 # 12\. `go get`

 This one causes a lot of confusion.

 Modern Go:

```
go get github.com/gin-gonic/gin
```

 is primarily about **changing dependency requirements in your module**.

 For example:

```
go get github.com/gin-gonic/gin@v1.10.0
```

 can update:

```
go.mod
go.sum
```

 You generally don't do:

```
go get
go build
```

 every time.

 If `go.mod` already specifies the dependencies, simply:

```
go build
```

 is enough.

---

 # 13\. `go mod tidy`

 Very useful:

```
go mod tidy
```

 It analyzes your project's imports and adjusts `go.mod` and `go.sum`.

 For example, imagine:

```
go.mod
```

 contains an old dependency that your code no longer uses.

 `go mod tidy` can remove unnecessary requirements.

 Conversely, if your code imports something that isn't properly represented in your module requirements, it can add the appropriate dependency.

 A common workflow is:

```
go mod tidy
go test ./...
go build .
```

---

 # 14\. `go mod download`

 This specifically downloads modules:

```
go mod download
```

 It can be useful in CI/CD or when you want to populate the module cache without actually building.

 So:

```
go mod download
    ↓
download dependencies

go build
    ↓
download dependencies if needed
    +
compile/link program
```

 You normally don't need to manually run `go mod download` before `go build`.

---

 # 15\. `go test`

 Run tests:

```
go test ./...
```

 The:

```
./...
```

 means roughly:

 > this package and all packages underneath it

 For example:

```
myapp/
├── main.go
├── user/
│   ├── user.go
│   └── user_test.go
└── database/
    ├── db.go
    └── db_test.go
```

 Then:

```
go test ./...
```

 tests all relevant packages.

---

 # 16\. `go test` also uses the build cache

 Go doesn't treat testing as completely separate from compilation.

 It can cache test-related build artifacts/results as well.

 You can clear the build cache:

```
go clean -cache
```

---

 # 17\. Useful `go clean` commands

 ### Clear build cache

```
go clean -cache
```

 ### Clear downloaded module cache

```
go clean -modcache
```

 Be careful with the latter: it deletes your downloaded modules, so the next build may need to download them again.

 ### Clear installed binaries

```
go clean -i
```

 This has more limited usefulness with modern Go workflows.

---

 # 18\. `GOPROXY`

 This controls where Go gets modules from.

 Check:

```
go env GOPROXY
```

 A common value is:

```
https://proxy.golang.org,direct
```

 Conceptually:

```
go build
   ↓
need github.com/foo/bar?
   ↓
GOPROXY
   ↓
download module
```

 If the proxy can't provide it, `direct` can tell Go to fetch directly from the source, depending on configuration and module settings.

---

 # 19\. `GOSUMDB`

 You'll also encounter:

```
go env GOSUMDB
```

 This relates to Go's **module checksum database**, which helps verify module content.

 A typical configuration is:

```
sum.golang.org
```

 For normal development, you generally leave this alone.

---

 # 20\. `GOPRIVATE`

 This becomes important at work.

 Suppose your company has:

```
git.company.com/backend/auth
```

 You don't want Go treating that as a public module that should go through the public module proxy/checksum infrastructure.

 You can configure:

```
go env -w GOPRIVATE=git.company.com
```

 Then Go knows those modules are private.

---

 # 21\. `CGO_ENABLED`

 Check:

```
go env CGO_ENABLED
```

 Usually:

```
1
```

 or:

```
0
```

 This controls whether **cgo** is enabled.

 Most pure-Go applications don't need to worry about it.

 But if a dependency uses C libraries, things become more interesting.

 For example:

```
CGO_ENABLED=0 go build .
```

 means:

 > build with cgo disabled.

 This is frequently used when creating portable/containerized Go binaries, provided all dependencies support it.

---

 # 22\. `go env` is your friend

 Instead of guessing where Go stores things:

```
go env GOROOT
go env GOPATH
go env GOBIN
go env GOMODCACHE
go env GOCACHE
go env GOENV
go env GOPROXY
go env GOOS
go env GOARCH
go env CGO_ENABLED
```

 Or all at once:

```
go env
```

 You can also inspect a particular setting:

```
go env GOPATH
```

 This is much better than assuming:

```
~/go
```

 because Go's configuration can be customized.

---

 # 23\. The big picture

 This is probably the most useful mental model:

```
                         YOUR MACHINE
┌─────────────────────────────────────────────────────────┐
│                                                         │
│  GOROOT                                                 │
│  /usr/local/go                                          │
│  ┌───────────────────────────────────────────────────┐  │
│  │ Go compiler / standard library / tools             │  │
│  └───────────────────────────────────────────────────┘  │
│                                                         │
│  GOPATH                                                 │
│  ~/go                                                   │
│  ┌───────────────────────────────────────────────────┐  │
│  │                                                   │  │
│  │  pkg/mod/     ← downloaded module source          │  │
│  │                                                   │  │
│  │  bin/         ← go install executables            │  │
│  │                                                   │  │
│  └───────────────────────────────────────────────────┘  │
│                                                         │
│  GOCACHE                                                │
│  ~/.cache/go-build                                      │
│  ┌───────────────────────────────────────────────────┐  │
│  │ compiled/intermediate build artifacts              │  │
│  └───────────────────────────────────────────────────┘  │
│                                                         │
│  YOUR PROJECT                                            │
│  ~/projects/myapp                                       │
│  ┌───────────────────────────────────────────────────┐  │
│  │ go.mod                                             │  │
│  │ go.sum                                             │  │
│  │ main.go                                            │  │
│  │ ...                                                │  │
│  │ myapp       ← output of go build                   │  │
│  └───────────────────────────────────────────────────┘  │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

 # 24\. The commands I would memorize

 You don't need to memorize every `go env` variable. These are the ones I'd get comfortable with:

```
# See Go version
go version

# See all environment settings
go env

# See important paths
go env GOROOT GOPATH GOBIN GOMODCACHE GOCACHE

# Create a module
go mod init example.com/myapp

# Download/manage a dependency
go get github.com/foo/bar

# Clean dependency metadata
go mod tidy

# Download dependencies
go mod download

# Run program
go run .

# Build executable
go build .

# Install executable
go install ./cmd/myapp

# Run all tests
go test ./...

# Format code
go fmt ./...

# Remove build cache
go clean -cache

# Remove module cache
go clean -modcache
```

 And perhaps the most important conceptual distinction:

```
                 Go project
                     │
                     ▼
                  go.mod
                     │
              dependencies
                     │
                     ▼
              GOMODCACHE
              ~/go/pkg/mod
                     │
                     ▼
                  compile
                     │
                     ▼
                GOCACHE
             ~/.cache/go-build
                     │
                     ▼
               final binary
                     │
             ┌───────┴────────┐
             ▼                ▼
         go build          go install
             │                │
       current dir        GOBIN / GOPATH/bin
```

 Once you understand **`GOROOT` = Go itself, `GOPATH` = Go workspace/cache area, `GOMODCACHE` = downloaded modules, `GOCACHE` = compilation cache, and `GOBIN` = installed Go executables**, the rest of the Go toolchain becomes much easier to reason about.