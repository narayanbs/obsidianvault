C _does_ have encapsulation and “access control” — just not with keywords like `private` or `public`, but through headers, source files, and opaque types.**

Let’s unpack it in plain terms.

---

## What the original complaint says

The complaint goes like this:

> “C doesn’t have access modifiers, so everything is public.”

That seems true if you only look at this:

```c
struct StringBuilder {
    char *payload;
};

void Append(struct StringBuilder *obj, const char *snippet);
```

Anyone who includes that definition can touch `payload` directly. No privacy. No encapsulation.

---


Instead, C uses **file-level encapsulation**.

---

## The key idea: headers are interfaces, `.c` files are implementations

In C, a “module” is usually:

- a **header file (`.h`)** → what users are allowed to see
    
- a **source file (`.c`)** → what users are _not_ allowed to see
    

You only expose what you put in the header.

---

## The trick: opaque structs

### In the header (`string_builder.h`)

```c
typedef struct StringBuilder StringBuilder;

void Append(StringBuilder *obj, const char *snippet);
```

Important points:

- `StringBuilder` is mentioned
    
- **but its contents are hidden**
    
- Users know it exists, but not what’s inside
    

This is called an **opaque type**.

---

### In the implementation (`string_builder.c`)

```c
struct StringBuilder {
    char *payload;
};

void Append(StringBuilder *obj, const char *snippet)
{
    ...
}
```

Only this file knows:

- that `payload` exists
    
- how the struct is laid out
    

---

## What this achieves (the “Ta-Da!” moment)

From the caller’s point of view:

```c
StringBuilder *sb;
sb->payload = ...;   // ❌ ILLEGAL — compiler error
```

They _cannot_ access `payload`, because the compiler doesn’t know the struct’s layout.

So effectively:

- `payload` is **private**
    
- `Append` is **public**
    
- enforced **at compile time**
    

No keywords required.

---

So **the client only ever uses `StringBuilder` via pointers and functions provided in the header.** They never touch its fields directly.

Let’s walk through it concretely.

---

## 1️⃣ What the client *sees* (the header)

```c
/* string_builder.h */

typedef struct StringBuilder StringBuilder;

StringBuilder *StringBuilder_create(void);
void StringBuilder_append(StringBuilder *sb, const char *text);
void StringBuilder_destroy(StringBuilder *sb);
```

From the client’s point of view:

* `StringBuilder` is a *type*
* but it’s an **incomplete type**
* they have **no idea what’s inside**

That’s intentional.

---

## 2️⃣ What the client code looks like

```c
#include "string_builder.h"

int main(void)
{
    StringBuilder *sb = StringBuilder_create();

    StringBuilder_append(sb, "Hello, ");
    StringBuilder_append(sb, "world!");

    StringBuilder_destroy(sb);
}
```

That’s it.

The client:

* can declare pointers to `StringBuilder`
* can pass them to functions
* **cannot** access fields
* **cannot** allocate it on the stack
* **cannot** do `sizeof(StringBuilder)`

---

## 3️⃣ What the client is *not allowed* to do

All of these fail to compile:

```c
StringBuilder sb;          // ❌ size unknown
sb.payload = NULL;         // ❌ incomplete type
sizeof(StringBuilder);     // ❌ incomplete type
```

Because the compiler doesn’t know the struct layout.

That’s the enforcement.

---

## 4️⃣ What the implementation does

```c
/* string_builder.c */

#include "string_builder.h"
#include <stdlib.h>
#include <string.h>

struct StringBuilder {
    char *payload;
    size_t length;
};

StringBuilder *StringBuilder_create(void)
{
    StringBuilder *sb = malloc(sizeof *sb);
    sb->payload = NULL;
    sb->length = 0;
    return sb;
}

void StringBuilder_append(StringBuilder *sb, const char *text)
{
    /* manipulate sb->payload internally */
}

void StringBuilder_destroy(StringBuilder *sb)
{
    free(sb->payload);
    free(sb);
}
```

Only this file knows:

* how big the struct is
* what fields exist
* how memory is managed

---

## 5️⃣ Why pointers are required

Because the type is incomplete in the header:

* you **can** have `StringBuilder *`
* you **cannot** have `StringBuilder` by value

This naturally pushes you toward:

* heap allocation
* controlled creation/destruction
* consistent lifetime rules

Which is exactly what you want for an encapsulated object.

---

## 6️⃣ Mental model (important)

Think of it like this:

* The header says:
  **“Trust me, this type exists. Use these functions.”**
* The source file says:
  **“Here’s how it actually works — but that’s none of your business.”**

That’s C-style OO.

---

## 7️⃣ One-line summary

> The client uses `StringBuilder` only as an opaque handle passed to API functions; all data access and memory management are hidden inside the `.c` file.

If you want, next we can:

* compare this to C++ `private` members
* show how `const` works with opaque types
* or talk about when *not* to use this pattern (there are tradeoffs)
