
```
Session
   │
   ├── Process Group
   │      ├── Process
   │      └── Process
   │
   ├── Process Group
   │      ├── Process
   │      └── Process
   │
   └── Process Group
          └── Process
```

 And a **controlling terminal** is associated with the session, while the terminal tracks which process group is currently in the **foreground**.

 Let's build this from the ground up.

---

 # 1\. Start with processes

 Suppose you open Kitty and it starts Bash:

```
kitty
   │
   └── bash
```

 There are two processes:

```
PID 1000  kitty
PID 1001  bash
```

 But Unix needs more structure than individual processes.

 Why?

 Because when you type:

```
vim
```

 Bash doesn't just create one isolated process. It needs to manage the entire **job**.

 For example:

```
cat file | grep hello | sort
```

 This involves three processes:

```
cat ──pipe──> grep ──pipe──> sort
```

 Bash wants to treat all three as **one job**.

 That's where **process groups** come in.

---

 # 2\. Process groups

 A **process group** is a collection of processes that are treated as a unit for job control.

 Every process belongs to exactly one process group.

 A process group has a **PGID** (process group ID).

 For example:

```
PID     PGID
1001    1001     bash
```

 Bash might initially be both:

```
PID  = 1001
PGID = 1001
```

 So:

```
Process Group 1001
└── bash (PID 1001)
```

 Now you run:

```
sleep 1000
```

 Bash creates another process.

 Suppose:

```
PID     PGID
1001    1001     bash
1050    1050     sleep
```

 Now there are two process groups:

```
PGID 1001                 PGID 1050
┌─────────────┐           ┌─────────────┐
│ bash        │           │ sleep       │
│ PID 1001    │           │ PID 1050    │
└─────────────┘           └─────────────┘
```

 This allows the shell to say:

 > "Give the terminal to process group 1050."

 rather than having to manage individual processes.

---

 # 3\. Why process groups matter

 Consider:

```
cat file | grep foo | less
```

 You actually have:

```
       PGID 2000
    ┌───────────────┐
    │               │
   cat             grep            less
  PID 2000        PID 2001        PID 2002
    │               │               │
    └───────pipe────┴──────pipe─────┘
```

 All three processes belong to the same process group:

```
PGID = 2000
```

 So Bash can treat the pipeline as one unit.

 For example, if you press:

```
Ctrl-C
```

 the terminal doesn't merely send SIGINT to `less`.

 It sends SIGINT to the **foreground process group**.

 Therefore:

```
SIGINT
  │
  ├──> cat
  ├──> grep
  └──> less
```

 All processes in that foreground job receive it.

 That's one of the major purposes of process groups.

---

 # 4\. Jobs

 A **job** is primarily a shell concept.

 For example:

```
sleep 100
```

 Bash considers this a job.

 You might see:

```
[1] 1050
```

 The `[1]` is the shell's **job number**.

 It is not the PID.

 You could have:

```
Job       PID       PGID
-------------------------
[1]       1050      1050
```

 If you then run:

```
sleep 200
```

 you might get:

```
[2] 1051
```

 So Bash maintains something like:

```
Bash's job table

Job 1 → PGID 1050 → sleep 100
Job 2 → PGID 1051 → sleep 200
```

 The kernel knows about:

 - processes
- process groups
- sessions
- terminals

 But **"job 1" is Bash's bookkeeping concept**.

 That's an important distinction.

```
Kernel                    Bash
------                    ----
process                   job
process group             job number
session                   job table
controlling terminal
```

 A job normally corresponds to one process group, although the exact shell bookkeeping can be more nuanced.

---

 # 5\. Sessions

 Now we move one level higher.

 A **session** is a collection of process groups.

 For example:

```
Session 500
│
├── Process Group 500
│     └── bash
│
├── Process Group 550
│     └── sleep
│
└── Process Group 600
      ├── cat
      ├── grep
      └── less
```

 Every process belongs to:

```
one process group
one session
```

 And a process group belongs to exactly one session.

 So conceptually:

```
Session
   │
   ├── Process Group
   │       ├── Process
   │       └── Process
   │
   └── Process Group
           ├── Process
           └── Process
```

---

 # 6\. Why do sessions exist?

 Sessions are particularly important for things like:

 - terminal login sessions
- shells
- job control
- controlling terminals
- daemons

 For your Kitty example, think:

```
Kitty
  │
  └── Bash session
       │
       ├── Bash
       ├── foreground job
       ├── background job
       └── another background job
```

 The session provides the larger container within which terminal job control operates.

---

 # 7\. Controlling terminal

 Now we get to the central concept.

 A session can have a **controlling terminal**.

 For your Kitty/Bash setup:

```
             Session
                │
                │
          controlling terminal
                │
                ▼
              PTY
```

 Remember that Kitty created the PTY:

```
Kitty
  │
  │ master
  ▼
┌─────────────────┐
│   kernel PTY    │
└─────────────────┘
  ▲
  │ slave
  │
 Bash
```

 The PTY slave becomes the controlling terminal of Bash's session.

 So conceptually:

```
Session 500
    │
    ├── controlling terminal
    │       │
    │       ▼
    │      PTY
    │
    └── process groups
```

 This is what makes Bash a proper interactive shell.

---

 # 8\. The terminal remembers a foreground process group

 This is perhaps the most important detail.

 The terminal doesn't merely know:

 > "This is Bash's terminal."

 It also tracks:

 > "Which process group is currently allowed to interact with me?"

 There is a **foreground process group ID** associated with the terminal.

 Suppose Bash is sitting at the prompt:

```
Session 500

Controlling terminal
    │
    │ foreground PGID = 500
    ▼
Process Group 500
    │
    └── bash
```

 Bash is foreground.

---

 # 9\. What happens when you run `vim`?

 You type:

```
vim file.txt
```

 Bash creates a new process group:

```
Session 500
│
├── PGID 500
│     └── bash
│
└── PGID 600
      └── vim
```

 Bash then tells the terminal:

 > Make PGID 600 the foreground process group.

 Conceptually:

```
tcsetpgrp(terminal_fd, 600);
```

 Now:

```
Controlling terminal
        │
        │ foreground PGID = 600
        ▼
   Process Group 600
        │
        └── vim
```

 Bash waits for the job.

---

 # 10\. Now you press Ctrl-C

 This is where everything comes together.

 You press:

```
Ctrl-C
```

 Kitty receives the keyboard event.

 Kitty sends the appropriate byte to the PTY:

```
^C
```

 The **kernel's terminal subsystem** interprets that as the terminal interrupt character.

 It finds:

```
foreground process group = 600
```

 and sends:

```
SIGINT
```

 to **every process in process group 600**.

 So:

```
Ctrl-C
   │
   ▼
PTY
   │
   ▼
foreground PGID = 600
   │
   ├── SIGINT → vim
   └── SIGINT → any other process in that group
```

 Bash doesn't need to manually catch Ctrl-C and forward it to Vim.

 The terminal subsystem handles the delivery.

 That's a beautiful part of Unix terminal design.

---

 # 11\. Foreground job

 A **foreground job** is a job whose process group is currently the terminal's foreground process group.

 For example:

```
Session
│
├── PGID 1000
│     └── bash
│
└── PGID 1100
      └── vim   ← foreground
```

 The terminal says:

```
foreground PGID = 1100
```

 Therefore Vim is the foreground job.

 Bash is waiting.

---

 # 12\. Background job

 Now suppose you run:

```
sleep 100 &
```

 The `&` tells Bash:

 > Start this job, but don't wait for it to finish.

 Bash creates:

```
Session
│
├── PGID 1000
│     └── bash
│
└── PGID 1100
      └── sleep 100
```

 But the terminal remains:

```
foreground PGID = 1000
```

 Therefore:

```
bash        ← foreground
sleep       ← background
```

 Bash immediately gives you another prompt:

```
$ sleep 100 &
[1] 1100
$ _
```

---

 # 13\. Why can't a background process simply read from the terminal?

 This is another clever piece of Unix job control.

 Suppose:

```
cat &
```

 `cat` is now a background process.

 But `cat` tries:

```
read(0, buffer, ...);
```

 Its stdin is the controlling terminal.

 However:

```
foreground PGID = bash's PGID
background PGID = cat's PGID
```

 The terminal subsystem notices that `cat` is not in the foreground process group.

 The read is therefore stopped, typically with:

```
SIGTTIN
```

 So you may see something like:

```
[1]+  Stopped    cat
```

 The background job cannot freely read from the terminal.

---

 # 14\. Background processes can write, though

 Writing is slightly different.

 A background process can often write to the terminal:

```
echo hello &
```

 You may see:

```
hello
```

 even though `echo` is technically a background job.

 There is a terminal setting controlling whether background terminal writes are restricted:

```
TOSTOP
```

 If `TOSTOP` is enabled, background writes can result in:

```
SIGTTOU
```

 So terminal job control has mechanisms for both reading and writing.

---

 # 15\. `fg`

 Now suppose you have:

```
$ sleep 100 &
[1] 1100
$
```

 Bash considers it job 1.

 You type:

```
fg %1
```

 Bash essentially does two important things.

 First:

```
terminal foreground PGID
        ↓
    PGID of sleep
```

 Then Bash waits for that job.

 Conceptually:

```
Before:

terminal
   │
   └── foreground → Bash

After fg:

terminal
   │
   └── foreground → sleep
```

---

 # 16\. `bg`

 Suppose you press:

```
Ctrl-Z
```

 while a foreground job is running.

 The terminal driver sends:

```
SIGTSTP
```

 to the foreground process group.

 For example:

```
terminal
    │
    │ SIGTSTP
    ▼
PGID 1200
 ├── vim
 └── helper
```

 The processes stop.

 Bash gets notified that the job stopped.

 It then takes the terminal back:

```
foreground PGID → Bash
```

 and prints:

```
[1]+  Stopped    vim
$
```

 Now you can type:

```
bg %1
```

 Bash sends the stopped job:

```
SIGCONT
```

 and lets it continue in the background.

 So:

```
Stopped
   │
   │ bg
   ▼
Running in background
```

---

 # 17\. The complete job-control state machine

 A job can essentially move through states like:

```
                 start
                   │
                   ▼
              FOREGROUND
                   │
             Ctrl-Z │
                   ▼
                STOPPED
                /      \
              bg        fg
              /          \
             ▼            ▼
      BACKGROUND      FOREGROUND
             │
             │ finishes
             ▼
          DONE
```

 Or:

```
FOREGROUND
    │
    ├── Ctrl-C ──> TERMINATED
    │
    ├── Ctrl-Z ──> STOPPED
    │                 │
    │                 ├── bg ──> BACKGROUND
    │                 │
    │                 └── fg ──> FOREGROUND
    │
    └── exits ───> DONE
```

---

 # 18\. Pipelines make process groups especially useful

 Consider:

```
cat file | grep foo | sort | less
```

 There are four processes:

```
cat → grep → sort → less
```

 Bash makes them one process group:

```
              PGID 2000
                  │
       ┌──────────┼──────────┐
       │          │          │
      cat        grep       sort       less
```

 More accurately:

```
cat ──pipe──> grep ──pipe──> sort ──pipe──> less
 │              │              │              │
 └──────────────┴──────────────┴──────────────┘
                       │
                     PGID
```

 If the pipeline is foreground:

```
terminal foreground PGID = 2000
```

 Press Ctrl-C:

```
SIGINT
 │
 └──> PGID 2000
       ├── cat
       ├── grep
       ├── sort
       └── less
```

 All four can receive the signal.

 That's why a pipeline behaves like a single interactive job.

---

 # 19\. How Bash itself fits into this

 There is a subtle point here.

 When Bash starts, it may initially be in the same process group as the process that launched it.

 But an interactive shell doing job control wants to organize things so that:

```
Session
│
├── Bash's process group
│       └── bash
│
├── Job 1's process group
│       └── ...
│
├── Job 2's process group
│       └── ...
│
└── Job 3's process group
        └── ...
```

 The terminal's foreground PGID gets switched between these groups.

 So when Bash is waiting for you:

```
terminal
    │
    └── foreground → bash's PGID
```

 When you run Vim:

```
terminal
    │
    └── foreground → vim's PGID
```

 When Vim exits:

```
terminal
    │
    └── foreground → bash's PGID
```

 This switching is the essence of shell job control.

---

 # 20\. `ps` lets you see this

 On Linux, try:

```
ps -o pid,ppid,pgid,sid,tpgid,stat,cmd
```

 You'll see columns like:

```
PID   PPID  PGID  SID   TPGID  STAT CMD
1000     1  1000  1000   1000  Ss   bash
1100  1000  1100  1000   1000  S    sleep 100
```

 The important fields are:

```
PID
    Process ID

PPID
    Parent Process ID

PGID
    Process Group ID

SID
    Session ID

TPGID
    Terminal's foreground Process Group ID
```

 Suppose Bash is running and you launch:

```
sleep 100 &
```

 you might conceptually see:

```
PID   PPID  PGID  SID   TPGID
1000     1  1000  1000   1000   bash
1100  1000  1100  1000   1000   sleep
```

 Notice:

```
Bash:
PGID  = 1000
SID   = 1000
TPGID = 1000

sleep:
PGID  = 1100
SID   = 1000
```

 Both are in the same **session**.

 But they belong to different **process groups**.

 And the terminal's foreground group is Bash's group.

---

 # 21\. Here's the hierarchy to remember

 If you remember only one diagram, make it this one:

```
                    SESSION
                       │
             ┌─────────┴─────────┐
             │                   │
       PROCESS GROUP        PROCESS GROUP
             │                   │
        ┌────┼────┐          ┌────┴────┐
        │    │    │          │         │
      proc proc proc        proc      proc
```

 Then attach a terminal:

```
                    SESSION
                       │
                       │
               controlling terminal
                       │
                       │
                foreground PGID
                       │
             ┌─────────┴─────────┐
             │                   │
       PROCESS GROUP        PROCESS GROUP
        (foreground)         (background)
             │                   │
        ┌────┼────┐          ┌────┴────┐
        │    │    │          │         │
      proc proc proc        proc      proc
```

 And Bash's concept of **jobs** sits on top:

```
                    Bash
                     │
                 job table
                     │
          ┌──────────┼──────────┐
          │          │          │
        Job 1      Job 2      Job 3
          │          │          │
        PGID       PGID       PGID
```

---

 # 22\. The really important distinction

 These four terms answer four different questions:

 | Concept | Question it answers |
| --- | --- |
| **Process** | What program instance is running? |
| **Process group** | Which processes should receive job-control signals together? |
| **Session** | Which collection of process groups belongs to this terminal/login context? |
| **Job** | Which process group is Bash tracking as one command/job? |

 And the controlling terminal answers:

 > **Which session's terminal is this, and which process group is currently in the foreground?**

---

 ## Put this back into your Kitty example

 Now the earlier picture becomes much richer:

```
                         Kitty
                           │
                           │ owns PTY master
                           ▼
                    ┌─────────────┐
                    │  PTY master │
                    └──────┬──────┘
                           │
                     kernel PTY
                           │
                    ┌──────┴──────┐
                    │  PTY slave  │
                    └──────┬──────┘
                           │
                   controlling terminal
                           │
                        Session
                           │
              ┌────────────┼────────────┐
              │            │            │
              ▼            ▼            ▼
           PGID 1       PGID 2       PGID 3
              │            │            │
            bash         vim          sleep
              │
           Job table
              │
          ┌───┴───┐
          │       │
        Job 1    Job 2
```

 When Bash is waiting for input:

```
TPGID = Bash's PGID
```

 When you run `vim`:

```
TPGID = Vim's PGID
```

 When you press Ctrl-C:

```
keyboard
   ↓
Kitty
   ↓
PTY master
   ↓
kernel terminal driver
   ↓
foreground PGID
   ↓
SIGINT to every process in that group
```

 And when Vim exits:

```
vim exits
   ↓
kernel notifies Bash
   ↓
Bash takes terminal back
   ↓
TPGID = Bash's PGID
   ↓
Bash prints "$ "
```

 **That last `TPGID` transition is one of the keys to understanding Unix shells.** Once you understand `PGID + SID + TPGID + controlling terminal`, commands like `fg`, `bg`, `Ctrl-C`, and `Ctrl-Z` stop seeming like special shell magic—they become fairly straightforward kernel mechanisms.