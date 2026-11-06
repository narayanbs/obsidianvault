# Go Templates — Complete Tutorial

 Go templates are used to generate text or HTML dynamically. They’re commonly used with **`html/template`** for web pages and **`text/template`** for plain text, configuration files, emails, etc.

 The core idea is:

```
template.Parse(...)
template.Execute(...)
```

 Let's build this from the basics to advanced usage.
## 1. Your first Go template

```go
package main

import (
	"os"
	"text/template"
)

func main() {
	tmpl := template.Must(template.New("hello").Parse("Hello, {{.Name}}!"))

	data := struct {
		Name string
	}{
		Name: "Alice",
	}

	tmpl.Execute(os.Stdout, data)
}
```

 Output:

```
Hello, Alice!
```

 ### What's `{{.Name}}`?

 `{{ ... }}` marks a **template action**.

 `.` means the current data.

 So:

```go
{{.Name}}
```

 means:

 > Get the `Name` field from the current data.

---
## 2\. Structs

 Templates work very naturally with structs.

```go
type User struct {
	Name  string
	Email string
	Age   int
}
```

```go
tmpl := template.Must(template.New("user").Parse(`
Name: {{.Name}}
Email: {{.Email}}
Age: {{.Age}}
`))

user := User{
	Name:  "Alice",
	Email: "alice@example.com",
	Age:   25,
}

tmpl.Execute(os.Stdout, user)
```

 Output:

```
Name: Alice
Email: alice@example.com
Age: 25
```

 Exported fields work:

```
.Name
.Email
.Age
```

 Unexported fields don't:

```
.name
```

---

## 3\. Maps

 You can also pass maps.

```go
data := map[string]string{
	"Name":  "Alice",
	"Email": "alice@example.com",
}
```

 Template:

```
Name: {{.Name}}
Email: {{.Email}}
```

 You can also explicitly use:

```
{{index . "Name"}}
```

---
## 4\. Nested data

 Suppose:

```go
type Address struct {
	City    string
	Country string
}

type User struct {
	Name    string
	Address Address
}
```

 You can access nested fields:

```go
{{.Name}}
{{.Address.City}}
{{.Address.Country}}
```

 For example:

```
Alice
Bengaluru
India
```

---
## 5\. Passing pointers

 This works too:

```go
user := &User{
	Name: "Alice",
}

tmpl.Execute(os.Stdout, user)
```

 The template automatically dereferences the pointer when accessing fields.

---
## 6\. `if`

 Templates support conditional rendering.

```go
{{if .LoggedIn}}
    Welcome back, {{.Name}}!
{{end}}
```
 

```go
data := struct {
	LoggedIn bool
	Name     string
}{
	LoggedIn: true,
	Name:     "Alice",
}
```

 Output:

```
Welcome back, Alice!
```

---
## 7\. `if` with `else`

```go
{{if .LoggedIn}}
    Welcome, {{.Name}}!
{{else}}
    Please log in.
{{end}}
```

---
## 8\. `else if`

 You can write:

```go
{{if eq .Role "admin"}}
    Admin
{{else if eq .Role "user"}}
    User
{{else}}
    Guest
{{end}}
```

---
## 9\. Truthiness

 Go templates consider several values false:

 - `false`
- `0`
- `""`
- `nil`
- empty arrays/slices/maps
- empty strings

 Therefore:

```go
{{if .Name}}
    Name: {{.Name}}
{{end}}
```

 only displays the name if it's non-empty.

---
## 10. Comparison functions

 Templates provide comparison functions.

 ### Equal

```go
{{if eq .Role "admin"}}
    Admin
{{end}}
```

 ### Not equal

```go
{{if ne .Role "admin"}}
    Not admin
{{end}}
```

 ### Greater than

```go
{{if gt .Age 18}}
    Adult
{{end}}
```

 ### Less than

```go
{{if lt .Age 18}}
    Minor
{{end}}
```

 Also available:

```
eq
ne
lt
le
gt
ge
```

---
## 11. Logical operators

 ### AND

```go
{{if and .LoggedIn .IsAdmin}}
    Admin dashboard
{{end}}
```

 ### OR

```go
{{if or .IsAdmin .IsOwner}}
    You have access
{{end}}
```

 ### NOT

```go
{{if not .Disabled}}
    Enabled
{{end}}
```

---
## 12. `range`

 `range` is one of the most important template features.

 Suppose:

```go
type User struct {
	Name string
}

users := []User{
	{Name: "Alice"},
	{Name: "Bob"},
	{Name: "Charlie"},
}
```

 Template:

```go
<ul>
{{range .}}
    <li>{{.Name}}</li>
{{end}}
</ul>
```

 Output:

```go
<ul>
    <li>Alice</li>
    <li>Bob</li>
    <li>Charlie</li>
</ul>
```

---

 # 13\. `range` with `else`

 You can handle an empty collection:

```go
{{range .Users}}
    <p>{{.Name}}</p>
{{else}}
    <p>No users found.</p>
{{end}}
```

 If `.Users` is empty, the `else` section executes.

---

 # 14\. Index and value

 For slices:

```go
{{range $index, $user := .Users}}
    {{$index}}: {{$user.Name}}
{{end}}
```

 Output:

```
0: Alice
1: Bob
2: Charlie
```

 The syntax is:

```
{{range $index, $value := .}}
```

---

 # 15\. Variables

 Templates support variables.

```go
{{$name := .Name}}

Hello {{$name}}
```

 You can assign a value:

```go
{{$x := 10}}
```

 Then:

```go
{{$x}}
```

 Variables are especially useful inside `range`.

```go
{{range $i, $user := .Users}}
    {{$i}} - {{$user.Name}}
{{end}}
```

---

 # 16\. The `$` variable

 `$` usually represents the root data.

 This becomes very useful inside nested `range` blocks.

 For example:

```go
type Page struct {
	Title string
	Users []User
}
```

 Template:

```go
<h1>{{.Title}}</h1>

{{range .Users}}
    <p>{{.Name}}</p>
    <p>Page: {{$ .Title}}</p>
{{end}}
```

 The spacing above is wrong. The correct syntax is:

```go
<h1>{{.Title}}</h1>

{{range .Users}}
    <p>{{.Name}}</p>
    <p>Page: {{$ .Title}}</p>
{{end}}
```

 Actually, root access is written without the space:

```go
{{$ .Title}}
```

 is **not valid**.

 Use:

```go
{{$ .Title}}
```

 Wait—Go template syntax for root field access is:

```
{{$.Title}}
```

 So:

```go
<h1>{{.Title}}</h1>

{{range .Users}}
    <p>{{.Name}}</p>
    <p>Page: {{$.Title}}</p>
{{end}}
```

 Here:

```
.
```

 means current context, while:

```
$
```

 means root context.

---

 # 17\. Pipelines

 Pipelines are a fundamental Go-template feature.

```go
{{.Name | printf "Hello, %s"}}
```

 A pipeline passes the result of one operation into another.

 For example:

```go
{{.Name | printf "%q"}}
```

 You can chain operations:

```go
{{.Name | printf "%s" | printf "%q"}}
```

---

 # 18\. Built-in functions

 Go templates provide several useful functions.

 Common ones include:

```go
and
or
not
eq
ne
lt
le
gt
ge
index
slice
len
printf
print
println
html
js
urlquery
```

 Example:

```
Users: {{len .Users}}
```

---

 # 19\. `printf`

 `printf` is extremely useful.

```go
{{printf "Hello %s!" .Name}}
```

 You can format numbers:

```go
{{printf "%.2f" .Price}}
```

 Or multiple values:

```go
{{printf "%s (%d)" .Name .Age}}
```

---

 # 20\. `index`

 Access an element by index:

```go
{{index .Users 0}}
```

 For a map:

```go
{{index .Prices "apple"}}
```

---

 # 21\. Methods

 Templates can call exported methods.

 Go:

```go
type User struct {
	Name string
}

func (u User) Greeting() string {
	return "Hello, " + u.Name
}
```

 Template:

```go
{{.Greeting}}
```

 Output:

```
Hello, Alice
```

 You can also call methods with arguments if the method is permitted by template execution rules:

```go
{{.GetName}}
```

 Methods that return a single value are straightforward.

 Methods can also return:

```
(value, error)
```

 If the error is non-nil, template execution fails.

---

 # 22\. Functions you define yourself

 This is where Go templates become extremely powerful.

 Suppose you want:

```
{{upper .Name}}
```

 Define a Go function:

```go
func upper(s string) string {
	return strings.ToUpper(s)
}
```

 Register it with `Funcs`:

```go
tmpl := template.Must(
	template.New("test").
		Funcs(template.FuncMap{
			"upper": upper,
		}).
		Parse(`{{upper .Name}}`),
)
```

 Now:

```go
tmpl.Execute(os.Stdout, struct {
	Name string
}{
	Name: "alice",
})
```

 produces:

```
ALICE
```

---

 # 23\. Functions with multiple arguments

 
```go
func add(a, b int) int {
	return a + b
}
```

 Register:

```go
template.FuncMap{
	"add": add,
}
```

 Template:

```
{{add 10 20}}
```

 Output:

```
30
```

---

 # 24\. Function pipelines

 Functions work beautifully with pipelines.

 Suppose:

```go
func upper(s string) string {
	return strings.ToUpper(s)
}
```

 Then:

```
{{.Name | upper}}
```

 The output of `.Name` becomes the function argument.

 Conceptually:

```
.Name | upper
```

 means:

```
upper(.Name)
```

---

 # 25\. Function argument position in pipelines

 This is an important detail.

 Suppose:

```go
func repeat(n int, s string) string
```

 You can write:

```
{{"hello" | repeat 3}}
```

 The pipeline value becomes the **last argument**.

 Conceptually:

```go
repeat(3, "hello")
```

 This behavior is extremely useful for template composition.

---

 # 26\. `with`

 `with` changes the current context.

 Suppose:

```go
type Page struct {
	User *User
}
```

 Instead of:

```go
{{if .User}}
    {{.User.Name}}
    {{.User.Email}}
{{end}}
```

 you can write:

```go
{{with .User}}
    {{.Name}}
    {{.Email}}
{{end}}
```

 Inside `with`:

```
.
```

 now refers to `.User`.

---

 # 27\. `with` with `else`

```go
{{with .User}}
    Hello, {{.Name}}
{{else}}
    No user logged in.
{{end}}
```

 This is useful for optional values.

---

 # 28\. Nested contexts

 Consider:

```go
type Page struct {
	Title string
	User  User
}
```

 Template:

```go
<h1>{{.Title}}</h1>

{{with .User}}
    <p>Name: {{.Name}}</p>
{{end}}
```

 Inside `with .User`, the dot is the `User`.

 To access the root:

```
{{$.Title}}
```

---

 # 29\. `define`

 `define` lets you create reusable template blocks.

```go
{{define "header"}}
<header>
    <h1>My Website</h1>
</header>
{{end}}
```

 Then invoke it:

```
{{template "header"}}
```

---

 # 30\. Passing data to templates

 You can pass the current context:

```go
{{template "user" .}}
```

 Define:

```go
{{define "user"}}
    <p>{{.Name}}</p>
{{end}}
```

 This means:

 > Execute the `user` template using the current `.` as its data.

---

 # 31\. Passing a different value

 You can pass a nested value:

```go
{{template "user" .User}}
```

 Then inside:

```go
{{define "user"}}
    {{.Name}}
{{end}}
```

 Here `.` is the `User`.

---

 # 32\. `block`

 `block` combines defining and invoking a template.

```go
{{block "content" .}}
    Default content
{{end}}
```

 This is especially useful with template inheritance patterns.

---

 # 33\. `template` vs `block`

 `template`:

```
{{template "content" .}}
```

 executes an already-defined template.

 `block`:

```go
{{block "content" .}}
    Default content
{{end}}
```

 provides default content that another template can override.

---

 # 34\. Template files

 Instead of putting templates inside Go strings, create:

```go
templates/
    home.html
    user.html
```

 `home.html`:

```html
<!DOCTYPE html>
<html>
<head>
    <title>{{.Title}}</title>
</head>
<body>
    <h1>Hello {{.Name}}</h1>
</body>
</html>
```

 
```go
tmpl := template.Must(
	template.ParseFiles("templates/home.html"),
)

tmpl.ExecuteTemplate(os.Stdout, "home.html", data)
```

---

 # 35\. `ParseGlob`

 If you have multiple template files:

```
templates/
    home.html
    about.html
    contact.html
```

 You can load them:

```go
tmpl := template.Must(
	template.ParseGlob("templates/*.html"),
)
```

 Then:

```go
tmpl.ExecuteTemplate(
	os.Stdout,
	"home.html",
	data,
)
```

---

 # 36\. `ParseFiles`

 You can explicitly load multiple files:

```go
template.ParseFiles(
	"templates/header.html",
	"templates/home.html",
	"templates/footer.html",
)
```

---

 # 37\. Complete HTML example

 For web applications, use:

```
html/template
```

 rather than:

```
text/template
```

 Example:

```go
package main

import (
	"html/template"
	"net/http"
)

type User struct {
	Name string
}

var tmpl = template.Must(
	template.ParseFiles("templates/index.html"),
)

func home(w http.ResponseWriter, r *http.Request) {
	data := User{
		Name: "Alice",
	}

	err := tmpl.ExecuteTemplate(w, "index.html", data)
	if err != nil {
		http.Error(w, err.Error(), http.StatusInternalServerError)
	}
}

func main() {
	http.HandleFunc("/", home)

	http.ListenAndServe(":8080", nil)
}
```

 `templates/index.html`:

```html
<!DOCTYPE html>
<html>
<head>
    <title>Go Templates</title>
</head>
<body>
    <h1>Hello {{.Name}}</h1>
</body>
</html>
```

---

 # 38\. Why `html/template` instead of `text/template`?

 For HTML, use:

```
html/template
```

 because it performs contextual HTML escaping.

 For example, if a user supplies:

```
<script>alert("x")</script>
```

 and you render:

```
{{.Name}}
```

 `html/template` escapes the content instead of blindly inserting executable HTML.

 This is an important security feature.

 Use:

```
import "html/template"
```

 for HTML pages.

 Use:

```
import "text/template"
```

 for non-HTML text.

---

 # 39\. Template comments

 You can write comments:

```go
{{/* This is a template comment */}}
```

 They won't appear in the generated output.

---

 # 40\. Whitespace control

 Go templates support trimming whitespace.

```go
{{- .Name -}}
```

 The `-` tells the template engine to trim surrounding whitespace.

 For example:

```go
Hello
    {{- .Name -}}
!
```

 can remove whitespace around the action.

 Be careful with this because it can make HTML/templates harder to read.

---

 # 41\. `else` and `else if` syntax

 Go templates allow compact forms:

```go
{{if .Admin}}
Admin
{{else if .User}}
User
{{else}}
Guest
{{end}}
```

 You can also put actions on the same line:

```go
{{if .Admin}}Admin{{else}}User{{end}}
```

---

 # 42\. Comparisons with multiple values

 You can do:

```go
{{if eq .Status "active"}}
    Active
{{end}}
```

 For multiple conditions:

```go
{{if and (eq .Status "active") .Verified}}
    Active and verified
{{end}}
```

 Parentheses allow functions to be nested.

---

 # 43\. Nested function calls

 Example:

```go
{{if eq (len .Users) 0}}
    No users
{{end}}
```

 Another:

```
{{if gt (len .Users) 10}}
    Many users
{{end}}
```

---

 # 44\. `range` over maps

 Suppose:

```go
data := map[string]string{
	"language": "Go",
	"version":  "1.26",
}
```

 Template:

```go
{{range $key, $value := .}}
    {{$key}} = {{$value}}
{{end}}
```

 Map iteration has deterministic behavior for basic key types when keys are sorted by their defined order, but you should not generally build application logic that depends on map iteration order.

---

 # 45\. `range` over strings

 You can range over some iterable values.

 For example, a slice:

```go
{{range .Tags}}
    {{.}}
{{end}}
```

 is much more common than ranging over strings directly.

---

 # 46\. `len`

```go
{{len .Users}}
```

 For example:

```
3
```

 You can combine it:

```go
{{if gt (len .Users) 0}}
    Users exist
{{end}}
```

---

 # 47\. `slice`

 You can slice certain values:

```
{{slice .Users 0 3}}
```

 This corresponds conceptually to:

```
users[0:3]
```

---

 # 48\. `print`, `printf`, `println`

 ### `print`

```
{{print .First .Last}}
```

 ### `printf`

```
{{printf "%s %s" .First .Last}}
```

 ### `println`

```
{{println .Name}}
```

 `printf` is usually the most useful of the three.

---

 # 49\. Template inheritance pattern

 Go doesn't have Django-style inheritance built directly into the language, but `define`, `template`, and `block` can implement a layout pattern.

 For example:

```
templates/
    base.html
    home.html
```

 `base.html`:

```html
{{define "base"}}
<!DOCTYPE html>
<html>
<head>
    <title>{{.Title}}</title>
</head>
<body>

{{block "content" .}}
Default content
{{end}}

</body>
</html>
{{end}}
```

 `home.html`:

```html
{{define "content"}}
<h1>Welcome!</h1>
<p>This is the home page.</p>
{{end}}
```

 Then parse both together:

```go
tmpl := template.Must(
	template.ParseFiles(
		"templates/base.html",
		"templates/home.html",
	),
)
```

 The exact organization of named templates matters, so for production applications it's worth establishing a consistent layout convention.

---

 # 50\. Template composition

 A practical application might have:

```
templates/
├── layouts/
│   └── base.html
├── partials/
│   ├── header.html
│   └── footer.html
└── pages/
    ├── home.html
    └── users.html
```

 Partials can be defined:

```
{{define "header"}}
<header>
    <h1>{{.Title}}</h1>
</header>
{{end}}
```

 Then:

```
{{template "header" .}}
```

 This allows you to keep large HTML applications manageable.

---

 # 51\. Handling template errors

 Avoid ignoring errors in production.

 Instead of:

```
tmpl.Execute(w, data)
```

 prefer:

```go
if err := tmpl.Execute(w, data); err != nil {
	http.Error(
		w,
		"Template error",
		http.StatusInternalServerError,
	)
	return
}
```

 For startup parsing:

```go
tmpl, err := template.ParseFiles("index.html")
if err != nil {
	log.Fatal(err)
}
```

 Or:

```go
tmpl := template.Must(
	template.ParseFiles("index.html"),
)
```

 `template.Must` is particularly convenient when failure means the application cannot start correctly.

---

 # 52\. A complete practical example

 Let's build a small user page.

 Go:

```go
package main

import (
	"html/template"
	"net/http"
)

type User struct {
	Name   string
	Email  string
	Admin  bool
	Active bool
}

type PageData struct {
	Title string
	User  User
	Tags  []string
}

var tmpl = template.Must(
	template.ParseFiles("templates/user.html"),
)

func userPage(w http.ResponseWriter, r *http.Request) {
	data := PageData{
		Title: "User Profile",
		User: User{
			Name:   "Alice",
			Email:  "alice@example.com",
			Admin:  true,
			Active: true,
		},
		Tags: []string{
			"Go",
			"Backend",
			"Web",
		},
	}

	if err := tmpl.ExecuteTemplate(
		w,
		"user.html",
		data,
	); err != nil {
		http.Error(
			w,
			"Template error",
			http.StatusInternalServerError,
		)
	}
}

func main() {
	http.HandleFunc("/", userPage)

	http.ListenAndServe(":8080", nil)
}
```

 Template:

```html
<!DOCTYPE html>
<html>
<head>
    <title>{{.Title}}</title>
</head>

<body>

<h1>{{.User.Name}}</h1>

<p>Email: {{.User.Email}}</p>

{{if .User.Active}}
    <p>Status: Active</p>
{{else}}
    <p>Status: Inactive</p>
{{end}}

{{if .User.Admin}}
    <strong>Administrator</strong>
{{end}}

<h2>Skills</h2>

<ul>
{{range .Tags}}
    <li>{{.}}</li>
{{else}}
    <li>No tags.</li>
{{end}}
</ul>

</body>
</html>
```

---

 # 53\. The mental model

 The easiest way to understand Go templates is:

```
DATA
  ↓
.
  ↓
FIELDS / METHODS
  ↓
ACTIONS
  ↓
PIPELINES
  ↓
OUTPUT
```

 For example:

```
{{.User.Name | printf "%q"}}
```

 means roughly:

```
Get User
   ↓
Get Name
   ↓
Pass Name to printf
   ↓
Write result
```

---

 # 54\. The most important syntax to memorize

 If you're learning Go templates, memorize these first:

```
{{.Name}}
```

 Field access.

```
{{if .Active}}
    Active
{{else}}
    Inactive
{{end}}
```

 Conditionals.

```
{{range .Users}}
    {{.Name}}
{{end}}
```

 Loops.

```
{{with .User}}
    {{.Name}}
{{end}}
```

 Change context.

```
{{$.Title}}
```

 Access root context.

```
{{$x := .Name}}
```

 Create variable.

```
{{template "header" .}}
```

 Execute another template.

```
{{define "header"}}
...
{{end}}
```

 Define reusable template.

```
{{.Name | upper}}
```

 Pipeline.

```
{{printf "%s (%d)" .Name .Age}}
```

 Formatting.

---

 # 55\. `text/template` vs `html/template`

 | Feature | `text/template` | `html/template` |
| --- | --- | --- |
| Plain text | ✅ | ✅ |
| HTML | ⚠️ | ✅ |
| Automatic HTML escaping | ❌ | ✅ |
| Emails/text files | ✅ | Possible |
| Web pages | Possible | ✅ |
| Security-conscious HTML rendering | ❌ | ✅ |

For a Go web server, the usual choice is:

```
import "html/template"
```

---

 # 56\. Common mistakes

 ### Mistake 1: Forgetting the dot

 Wrong:

```
{{Name}}
```

 Usually:

```
{{.Name}}
```

 ### Mistake 2: Using `text/template` for untrusted HTML

 For HTML responses:

```
import "html/template"
```

 ### Mistake 3: Forgetting `end`

 Wrong:

```
{{if .Active}}
    Active
```

 Correct:

```
{{if .Active}}
    Active
{{end}}
```

 ### Mistake 4: Losing the root context

 Inside:

```
{{range .Users}}
```

 `.` is now a user.

 To access the original page:

```
{{$.Title}}
```

 ### Mistake 5: Not checking execution errors

 Use:

```
if err := tmpl.Execute(w, data); err != nil {
    // handle error
}
```

---

 # 57\. Cheat sheet

```
{{.}}
```

 Current value.

```
{{.Name}}
```

 Field.

```
{{.User.Name}}
```

 Nested field.

```
{{$.Title}}
```

 Root field.

```
{{if .Active}}...{{end}}
```

 Conditional.

```
{{if .Active}}...{{else}}...{{end}}
```

 Conditional + else.

```
{{range .Items}}...{{end}}
```

 Loop.

```
{{range .Items}}...{{else}}...{{end}}
```

 Loop + empty case.

```
{{with .User}}...{{end}}
```

 Change context.

```
{{$x := .Name}}
```

 Variable.

```
{{len .Items}}
```

 Length.

```
{{index .Items 0}}
```

 Index.

```
{{eq .Status "ok"}}
```

 Comparison.

```
{{and .A .B}}
```

 AND.

```
{{or .A .B}}
```

 OR.

```
{{not .A}}
```

 NOT.

```
{{printf "%s" .Name}}
```

 Formatting.

```
{{.Name | upper}}
```

 Pipeline.

```
{{define "header"}}...{{end}}
```

 Define template.

```
{{template "header" .}}
```

 Execute template.

```
{{block "content" .}}...{{end}}
```

 Block/default content.

---

 ## Final learning path

 If you're learning this for **Go backend/web development**, learn it in this order:

 1. `{{.Field}}`
2. Structs and nested fields
3. `if / else`
4. `range`
5. `with`
6. `$` root context
7. Variables
8. Built-in functions
9. Pipelines
10. Custom functions with `FuncMap`
11. `define`
12. `template`
13. `block`
14. `ParseFiles` / `ParseGlob`
15. `html/template` escaping
16. Layouts and partials
17. Error handling
18. Building a complete server-rendered application

 Once these are comfortable, Go templates are relatively small—the important part is understanding **how the current `.` context changes inside `range`, `with`, and nested templates**.