This idea sounds abstract at first, but it’s actually very practical. 

---

## The core idea in one sentence

> **Don’t pass “maybe-valid” data around your system.  
> Parse it once at the boundary, turn it into a _type that guarantees correctness_, and only pass that around.**

That’s what **“parse, don’t validate”** means.

---

## The problem with “validate and keep the string”

### What usually happens

You get some input:

```c
char *untrustedInput;  // e.g. email address from user
```

You do this:

```c
if (!validateEmail(untrustedInput)) {
    return error;
}
```

And then… you keep passing `untrustedInput` (a plain string) deeper into the system.

### Why this is bad

Because **nothing in the type system remembers that it was validated**.

Deep inside your program, another function sees:

```c
void sendWelcomeEmail(const char *email);
```

That function has no way to know:

- Was this validated?
    
- Using which rules?
    
- Is it still valid after transformations?
    

So developers will:

- re-validate (possibly differently)
    
- forget to validate
    
- assume it’s valid when it isn’t
    

This leads to:

- inconsistent rules
    
- duplicated logic
    
- subtle bugs
    
- security issues
    

---

## What “parse, don’t validate” changes

Instead of asking:

> “Is this string valid?”

You ask:

> **“Can I turn this string into a _valid email value_?”**

---

## Parsing produces a _new type_

```c
email_t theEmail = parseEmail(untrustedInput);
if (theEmail == PARSE_ERROR) {
    return error;
}
```

Key difference:

- `email_t` is **not a string**
    
- It _represents_ “a valid email address”
    
- If you have an `email_t`, it is **by definition valid**
    

Now your system looks like this:

```c
void sendWelcomeEmail(email_t email);
```

That function:

- does **zero validation**
    
- does **not care where the email came from**
    
- can **trust the type**
    

---

## Why this eliminates repeated validation

Because invalid data **cannot exist** in the rest of the system.

You physically cannot call:

```c
sendWelcomeEmail("not-an-email");
```

The compiler won’t allow it.

So instead of relying on:

- discipline
    
- comments
    
- code reviews
    

You rely on:

- **types**
    
- **compiler guarantees**
    

---

## “Parse” vs “Validate” (important distinction)

### Validate

- returns `true` / `false`
    
- keeps the original data
    
- does not _change the type_
    

```c
bool validateEmail(const char *);
```

After this call, the data is still:

```c
const char *
```

### Parse

- returns a **different type**
    
- fails if input is invalid
    
- _creates a value that is guaranteed correct_
    

```c
email_t parseEmail(const char *);
```

---

## Why validation deep in the system is a smell

If a function does this:

```c
void process(email_t email) {
    if (!validateEmailAgain(email)) { ... }
}
```

That tells you:

- the type doesn’t guarantee correctness
    
- the boundary was leaky
    
- the design is lying to you
    

In a good type-driven design:

- validation happens **once**
    
- parsing happens **at the boundary**
    
- internal code assumes correctness
    

---

## Real-world analogy

Think airport security ✈️

- **Validation approach**:  
    Every gate re-checks your passport.
    
- **Parse approach**:  
    You pass security once and get a wristband.  
    After that, no one checks again.
    

`email_t` is the wristband.

---

## How this ties to type-driven design

Type-driven design says:

- illegal states should be **unrepresentable**
    
- correctness should be encoded in the type system
    
- functions should accept _only valid data_
    

So instead of:

```c
void send(const char *email);
```

You design:

```c
void send(email_t email);
```

Now mistakes are caught:

- at compile time
    
- instead of at runtime
    
- or worse, in production
    

---

## One-line takeaway

> “Parse, don’t validate” means: **convert untrusted input into trusted, well-typed values at the boundary, and never pass raw data around internally.**

Let’s make this _very_ concrete with a small C example that shows **parse, don’t validate** in action.

We’ll build:

- an `email_t` **opaque type**
    
- a `parse_email()` function
    
- code that **cannot misuse** emails once parsed
    

---

## 1️⃣ The goal

We want this to be **impossible**:

```c
send_welcome_email("not-an-email");   // ❌
```

And this to be the _only_ legal path:

```c
email_t email = parse_email(input);
send_welcome_email(email);            // ✅ guaranteed valid
```

---

## 2️⃣ The public interface (email.h)

This is what the rest of the system sees.

```c
/* email.h */

#ifndef EMAIL_H
#define EMAIL_H

typedef struct email email_t;

/* Try to parse a string into a valid email.
   Returns NULL on failure. */
email_t *email_parse(const char *input);

/* Accessors (no raw string mutation) */
const char *email_c_str(const email_t *email);

/* Cleanup */
void email_destroy(email_t *email);

#endif
```

Key points:

- `email_t` is **opaque**
    
- callers cannot create or modify it directly
    
- validity is guaranteed by construction
    

---

## 3️⃣ The implementation (email.c)

```c
/* email.c */

#include "email.h"
#include <stdlib.h>
#include <string.h>
#include <ctype.h>

struct email {
    char *value;
};

/* Very naive validation for demo purposes */
static int is_valid_email(const char *s)
{
    const char *at = strchr(s, '@');
    return at && at != s && strchr(at, '.');
}

email_t *email_parse(const char *input)
{
    if (!input || !is_valid_email(input)) {
        return NULL;
    }

    email_t *e = malloc(sizeof *e);
    if (!e) return NULL;

    e->value = strdup(input);
    return e;
}

const char *email_c_str(const email_t *email)
{
    return email->value;
}

void email_destroy(email_t *email)
{
    free(email->value);
    free(email);
}
```

The **only** place where validity is checked:

- `email_parse`
    
- once
    
- at the boundary
    

---

## 4️⃣ Client code (main.c)

```c
#include "email.h"
#include <stdio.h>

void send_welcome_email(email_t *email)
{
    printf("Sending welcome email to %s\n",
           email_c_str(email));
}

int main(void)
{
    const char *input = "user@example.com";

    email_t *email = email_parse(input);
    if (!email) {
        printf("Invalid email\n");
        return 1;
    }

    send_welcome_email(email);

    email_destroy(email);
}
```

Notice:

- `send_welcome_email` does **no validation**
    
- it accepts only `email_t *`
    
- misuse is structurally impossible
    

---

## 5️⃣ What the compiler prevents for you

These all fail to compile:

```c
email_t email;                  // ❌ incomplete type
email.value = "x";              // ❌ no access
send_welcome_email("x@y");      // ❌ wrong type
```

That’s the _type system enforcing correctness_.

---

## 6️⃣ Why this scales in large systems

### Without parse-don’t-validate

- every function re-checks strings
    
- rules drift over time
    
- bugs creep in at boundaries
    

### With parse-don’t-validate

- **one choke point**
    
- invariant: _“email_t is always valid”_
    
- easier refactors
    
- easier reasoning
    
- fewer security bugs
    

---

## 7️⃣ When this is especially valuable

This pattern shines for:

- email addresses
    
- usernames
    
- IDs
    
- file paths
    
- money / currency
    
- dates
    
- permissions
    
- protocol messages
    

Basically: **anything with rules**.

---

## 8️⃣ When it might be overkill

It _can_ be too heavy for:

- tiny scripts
    
- throwaway code
    
- performance-critical hot loops
    
- trivially constrained values
    

But for:

- libraries
    
- infrastructure
    
- security boundaries
    
- large teams
    

…it’s gold.

---

## 9️⃣ Final intuition to keep

> Validation asks _“is this okay?”_  
> Parsing says _“I refuse to create a value unless it’s okay.”_

That shift is the entire philosophy.

If you want next:

- a zero-allocation version
    
- how Rust encodes this in the type system
    
- or how this ties back to the opaque-struct C discussion