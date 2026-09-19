 The important thing is that there are **two separate layers** involved:
 
1. **Kitty creates a graphical window** using the OS/window system.
2. **Kitty creates a pseudo-terminal (PTY)** so that `bash` behaves as though it is attached to a real terminal.

 A simplified flow looks like this:

```
You launch kitty
      │
      ▼
kitty process starts
      │
      ├── initializes graphics/window system
      │
      ├── creates a window
      │
      ├── allocates a PTY
      │       │
      │       ├── master  ─────── kitty
      │       │
      │       └── slave   ─────── shell (bash)
      │
      ├── forks
      │
      └── child:
              ├── creates session
              ├── makes PTY slave controlling terminal
              ├── connects stdin/stdout/stderr → slave
              └── execve("/bin/bash", ...)
                         │
                         ▼
                       bash
                         │
                         ├── reads configuration
                         ├── initializes itself
                         └── prints prompt
                                  │
                                  ▼
                             "$ "
```

 Let's walk through it more precisely.

## 1\. You execute `kitty`

 Suppose you type:

```
$ kitty
```

 Your existing shell does something approximately like:

```
fork()
  |
  +-- child: execve("kitty", ...)
```

 Now the new process is the **kitty process**.

 It inherits things like:

 - environment variables
- current working directory
- file descriptors
- permissions
- terminal-related state

 At this point, kitty is itself still attached to the terminal from which you launched it.

---

## 2\. Kitty initializes its graphical environment

 Kitty needs to create a graphical window.

 Depending on the platform, it talks to the windowing system, e.g. on Linux:

```
kitty
  │
  └── Wayland / X11
          │
          └── compositor / X server
```

 Kitty asks the graphical system for a window.

 Conceptually:

```
kitty
   │
   │ "Create a window of size 120×40"
   ▼
window system
   │
   ▼
graphical window appears
```

 The window itself isn't the terminal.

 It's just the **visual surface** on which kitty will eventually render characters.

---

## 3\. Kitty creates a PTY

 Now comes the interesting part.

 Kitty wants to run something like:

```
bash
```

 But it doesn't want bash's stdin/stdout to simply be pipes.

 For example, a pipe would give:

```
kitty ──pipe──> bash
kitty <─pipe─── bash
```

 That isn't sufficient for a real terminal because programs need terminal-specific functionality:

 - cursor movement
- window size
- raw mode
- Ctrl-C handling
- foreground process groups
- terminal escape sequences
- job control
- `tcsetattr()`
- `ioctl()` operations
- etc.

 So kitty creates a **pseudo-terminal**.

 On Linux, conceptually this involves:

```
int master = posix_openpt(O_RDWR | O_NOCTTY);
grantpt(master);
unlockpt(master);

char *slave_name = ptsname(master);
```

 This produces a pair:

```
             PTY
        ┌─────────────┐
        │             │
        │   master    │
        │      │      │
        │      │      │
        │      │      │
        │    slave    │
        │             │
        └─────────────┘
```

 The master and slave are two ends of the same virtual terminal.

---

## 4\. Who owns the master and slave?

 Typically:

```
kitty process
     │
     └── master FD

shell process
     │
     └── slave FD
```

 So you were essentially correct.

 But there's an important detail:

 **Kitty doesn't normally "give the slave" to bash just by passing an FD across some IPC mechanism.**

 Instead, kitty normally creates the PTY and then forks/creates the child process. The child sets up the slave as its controlling terminal and connects it to fd 0/1/2.

 Conceptually:

```
                 kitty
                   │
             PTY master
                   │
                   │
             PTY subsystem
                   │
             PTY slave
                   │
                bash
```

---

## 5\. Kitty creates the shell process

 Kitty now needs to create the child that will become bash.

 Conceptually:

```
pid = fork();
```

 After the fork:

```
             kitty
              │
              │ fork()
              │
       ┌──────┴──────┐
       │             │
     parent         child
       │             │
     kitty          soon bash
```

 The child is initially another copy of the kitty process.

 But the child is about to completely change its identity.

---

## 6\. Child sets up the PTY

 The child needs to turn the PTY slave into its terminal.

 This is one of the most important parts.

 Conceptually, the child does something along the lines of:

```
setsid()
```

 This creates a new session.

 Then:

```
ioctl(slave, TIOCSCTTY, ...)
```

 makes the PTY slave the **controlling terminal** of that session.

 Now you have something like:

```
        Session
           │
           │
      bash process
           │
           │
    controlling terminal
           │
           ▼
       PTY slave
```

 This controlling-terminal relationship is crucial for Unix job control.

---

## 7\. stdin/stdout/stderr are connected to the slave

 The child then makes:

```
stdin  (fd 0) ──┐
stdout (fd 1) ──┼──> PTY slave
stderr (fd 2) ──┘
```

 Usually this is done using `dup2()`.

 Conceptually:

```
dup2(slave, STDIN_FILENO);
dup2(slave, STDOUT_FILENO);
dup2(slave, STDERR_FILENO);
```

 Now if the shell executes:

```
write(1, "$ ", 2);
```

 the bytes don't go directly to the graphical window.

 They travel through the PTY.

---

## 8\. The child executes bash

 Now the child does something like:

```
execve("/bin/bash", ...);
```

 The process that used to be a kitty child is replaced by bash.

 So:

```
before exec:

kitty child
    │
    └── PTY slave

after exec:

bash
    │
    └── PTY slave
```

 The PID can remain the same.

 `execve()` replaces the process's program image; it doesn't create a new PID.

---

## 9\. Bash starts

 Bash now sees:

```
stdin  → PTY slave
stdout → PTY slave
stderr → PTY slave
```

 It can also discover that its terminal is a terminal:

```
isatty(0)
```

 returns true.

 Compare this with:

```
echo hello | bash
```

 where stdin may be a pipe rather than a terminal.

 That distinction is extremely important to shells.

---

## 10\. Bash initializes its terminal state

 Bash performs its initialization.

 Depending on how it was invoked, it may read things such as:

```
/etc/profile
~/.bash_profile
~/.bashrc
```

 It also sets up shell state, environment variables, job control, signal handling, etc.

 For an interactive shell, bash needs to establish itself as the appropriate foreground process group.

 Conceptually:

```
controlling terminal
        │
        ▼
foreground process group
        │
        ▼
       bash
```

 This is what allows things like:

```
Ctrl-C
Ctrl-Z
Ctrl-\
```

 to have their normal terminal semantics.

---

## 11\. Bash writes the prompt

 Eventually bash decides:

 > I'm interactive, I'm ready to accept a command.

 It constructs the prompt, perhaps:

```
user@machine:~$
```

 and writes it to fd 1:

```
write(1, "user@machine:~$ ", ...);
```

 Remember:

```
bash
 │
 │ write()
 ▼
fd 1
 │
 ▼
PTY slave
 │
 ▼
PTY subsystem
 │
 ▼
PTY master
 │
 ▼
kitty
```

 This is the key data path.

---

## 12\. Kitty reads from the master

 Kitty has the **master** side of the PTY.

 It waits for data from it.

 Conceptually:

```
read(master_fd, buffer, ...);
```

 Bash's prompt:

```
"user@machine:~$ "
```

 therefore arrives at kitty through the PTY master.

 Kitty receives bytes such as:

```
u s e r @ m a c h i n e : ~ $
```

 along with potentially terminal control sequences.

 For example, bash may output ANSI escape sequences to change:

 - color
- cursor position
- text attributes
- etc.

---

## 13\. Kitty interprets the terminal output

 This is another important distinction.

 Kitty doesn't simply display every byte literally.

 It has a **terminal emulator**.

 For example, if bash sends:

```
"\033[31m"
```

 that's essentially:

```
ESC [ 31 m
```

 which means something like:

 > switch foreground color to red

 Kitty's terminal emulator interprets the escape sequence and updates its internal terminal screen state.

 Conceptually:

```
PTY master
    │
    ▼
bytes from bash
    │
    ▼
kitty terminal emulator
    │
    ├── update cursor
    ├── update colors
    ├── update characters
    └── update screen buffer
             │
             ▼
       graphics renderer
             │
             ▼
          GPU/display
```

---

## 14\. Kitty renders the prompt

 Kitty now has an internal representation roughly like:

```
┌────────────────────────────────────────────┐
│                                            │
│                                            │
│ user@machine:~$                            │
│                                            │
│                                            │
└────────────────────────────────────────────┘
```

 It renders that into the graphical window.

 The compositor ultimately puts that window on your screen.

 So the complete path for the prompt is:

```
bash
 │
 │ write("user@machine:~$")
 ▼
PTY slave
 │
 ▼
PTY kernel subsystem
 │
 ▼
PTY master
 │
 ▼
kitty read()
 │
 ▼
terminal emulator
 │
 ▼
kitty rendering
 │
 ▼
Wayland/X11
 │
 ▼
compositor
 │
 ▼
GPU/display
 │
 ▼
your eyes
```

---

## 15\. What happens when you type a command?

 This is the reverse direction.

 Suppose you type:

```
ls
```

 Your keyboard event first reaches the graphical system:

```
keyboard
   │
   ▼
Wayland/X11
   │
   ▼
kitty
```

 Kitty receives the keyboard events.

 It translates them into terminal input bytes.

 For example:

```
l
s
Enter
```

 becomes approximately:

```
'l'
's'
'\n'
```

 Kitty writes those bytes to the **PTY master**:

```
keyboard
   │
   ▼
kitty
   │
   │ write(master, "ls\n", 3)
   ▼
PTY master
   │
   ▼
PTY subsystem
   │
   ▼
PTY slave
   │
   ▼
bash
```

 Bash is blocked waiting for input, roughly:

```
read(0, buffer, ...);
```

 The PTY wakes it up.

 Bash receives:

```
ls\n
```

 It parses the command and may fork:

```
bash
 │
 │ fork()
 ▼
child
 │
 └── execve("ls")
```

 Then `ls` writes its output:

```
ls
 │
 │ write(stdout, ...)
 ▼
PTY slave
 │
 ▼
PTY master
 │
 ▼
kitty
 │
 ▼
terminal emulator
 │
 ▼
screen
```

 And once `ls` finishes, bash prints another prompt.

---

 ## The whole lifecycle

 Putting everything together:

```
                  YOU
                   │
                   │ launch kitty
                   ▼
             ┌───────────┐
             │   kitty   │
             └─────┬─────┘
                   │
             create GUI window
                   │
                   ▼
             ┌───────────┐
             │  Window   │
             └───────────┘

                   kitty
                     │
                     │ create PTY
                     ▼
              ┌─────────────┐
              │     PTY     │
              │             │
              │ master      │
              │    │        │
              │    │        │
              │   slave     │
              └────┼────────┘
                   │
                   │ fork
                   ▼
                 child
                   │
             setsid()
                   │
          set controlling tty
                   │
            dup2(slave, 0)
            dup2(slave, 1)
            dup2(slave, 2)
                   │
                   ▼
             execve(bash)
                   │
                   ▼
                 bash
                   │
             initialize
                   │
             read .bashrc
                   │
             setup job control
                   │
                   ▼
          write("$ ", stdout)
                   │
                   ▼
              PTY slave
                   │
                   ▼
              PTY master
                   │
                   ▼
                kitty
                   │
          terminal emulator
                   │
             render screen
                   │
                   ▼
                DISPLAY
```

 ### One subtle but important correction

 It's useful to think of the PTY as **not being a pipe owned by kitty and bash**.

 The kernel is in the middle:

```
                 USER SPACE
       ┌───────────────────────────┐
       │                           │
       │   kitty            bash   │
       │     │                │    │
       └─────┼────────────────┼────┘
             │                │
             │                │
       master FD          slave FD
             │                │
       ┌─────┴────────────────┴────┐
       │          KERNEL            │
       │                            │
       │       PTY subsystem        │
       │                            │
       └────────────────────────────┘
```

 The **master and slave are file descriptors referring to two sides of a kernel-managed PTY**.

 And this is why kitty can read what bash writes, while bash can read what you type through kitty.

 The most important conceptual split is therefore:

```
               GRAPHICS SIDE

keyboard → window system → kitty → renderer → display
                         │
                         │
                         ▼
                    PTY master
                         │
                    KERNEL PTY
                         │
                    PTY slave
                         │
                         ▼
                      bash
```

 If you want to go one level deeper, the next really interesting part is **exactly what `fork()`, `setsid()`, `TIOCSCTTY`, `dup2()`, `execve()`, and the controlling-terminal/foreground-process-group machinery do at the kernel level**. That's where the seemingly magical "bash is connected to kitty" relationship actually gets established.