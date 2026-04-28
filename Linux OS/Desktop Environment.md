## Display Servers

You can think of a Display Server as the traffic controller for your desktop: it sits between your applications (like your browser) and your hardware (your graphics card and input devices), telling the computer where to draw windows and how to register your mouse clicks and keystrokes.

1. X window system(X11)
   X11 has been the standard for Linux display servers for decades (since the 1980s). Its architecture was designed for a world where computing was networked, and multiple terminals might be connected to a central server.
   
   In the early days of X11, Applications were essentially given a "canvas" (the screen) and were told, "Draw your window directly onto the final output."
   
   * **The Problem:** If the application was drawing a large window, the screen might refresh in the middle of the drawing process. You would see half of the window finished and half of it unfinished. This resulted in **Screen Tearing** (the jarring horizontal line effect) and visible flickering during resizing.
     
   *  **The "Direct Draw" flaw:** Because applications drew directly to the front buffer (the part of video memory actually displayed on the monitor), there was no "safety net."
     
   *  **Compositor**:  The fix was the compositor.  A compositor introduces an intermediary step using Back Buffers. Instead of the application drawing to the screen directly, it draws to a hidden memory space (a back buffer) that the user cannot see.
     * The Application draws its window content to an off-screen buffer.
     * The Compositor takes these off-screen buffers from all running applications.
     * The Compositor stacks them up (like physical sheets of paper), applies effects like drop shadows, transparency, or animations, and then "flips" the final completed image to the screen at once.
     *  Some of the compositors in X11 include compiz, kwin and picom. 
		
    By forcing applications to draw to a hidden buffer, compositor eliminated tearing entirely 
    because the screen only ever displayed a "complete" frame.
    
    **The Problem with "Bolt-on" Compositing (The X11 Architecture)** 
    While it made the desktop look and perform better, it was an "add-on." The X11 
    protocol was never designed to have a compositor.
    In an X11 environment with a compositor (like Compiz or Compton/Picom):
    
    * The Path: App $\rightarrow$ X Server $\rightarrow$ Compositor $\rightarrow$ X Server $\rightarrow$ Screen.
    * The Inefficiency: The data has to pass through the X Server multiple times. It is a "middleman" architecture that introduces unnecessary complexity and latency.
     
2. Wayland
   
   Wayland was created to modernize the Linux desktop. 
   
   **The Architecture: How Wayland Works**
   
   In the X11 world, the "Display Server" was a massive, separate entity that tried to do everything. Wayland flips this upside down by making the **Compositor** the boss.
   
   1. **The Client (App):** Your browser or text editor does not talk to a "server" anymore. It talks to the **Compositor**.
   2.  **The Compositor:** This is the heart of your desktop (like Mutter for GNOME or KWin for KDE). It handles input events, window management, and screen composition. It essentially _is_ your display server.
   3.  **Buffer Sharing:** Instead of copying pixel data back and forth through a "middleman" server (which X11 does), the application renders its own frame into a memory buffer and tells the compositor, "I have a new frame ready." The compositor then picks that frame up and puts it on the screen. This "Zero-Copy" approach is why Wayland is so much more efficient and tear-free.
      
      
   **The "Bridge": XWayland**
   
   You might be wondering: "If Wayland is the new standard, what happens to my old apps that only support X11?"
   
   This is where XWayland comes in. It is a compatibility layer that runs a mini X Server inside the Wayland session. When you run an old application, it thinks it is talking to a full X11 server, but XWayland translates those commands for the Wayland compositor. It allows for a seamless transition where almost all legacy Linux software continues to work perfectly on a Wayland desktop.


## Window Managers 

Window managers (WMs) are where the conceptual shift between X11 and Wayland becomes most apparent. In X11, the window manager is just one piece of a modular puzzle. In Wayland, the "Window Manager" role is effectively folded into the Compositor itself.

##### The Conceptual Shift

To understand the difference, you have to realize that "Window Manager" means something fundamentally different in the two ecosystems.

##### 1. In the X11 World (Modular)

In X11, the architecture is **decoupled**. You have three distinct entities that work together:

- **X Server:** Manages hardware communication and input routing.
    
- **Window Manager (e.g., i3):** A specialized program that tells the X Server how to size and position windows. It is just another "client" program, similar to your browser or terminal, but with special privileges to "talk" to the X Server about window geometry.
    
- **Compositor (e.g., Picom):** A separate program that fetches window buffers from the X Server to add blur, shadows, or anti-tearing (vsync).
    

**The Result:** You can mix and match. You can run `i3` as your window manager and `picom` as your compositor. They don't have to be from the same developer; they just both happen to be X11 clients.

#### 2. In the Wayland World (Integrated)

Wayland does not have a "Window Manager" in the traditional X11 sense. Instead, it has a **Compositor**.

- **The Compositor (e.g., Sway):** As discussed, this _is_ the display server. Because it is the direct gatekeeper to the GPU, it doesn't just manage the "look" of the windows (compositing); it also manages the **logic** of where those windows go (window management).
    

**The Result:** The window manager and compositor are the same process. You cannot "swap out" the compositor part of Sway for a different one because they are architecturally fused.

---

To use a simple analogy: **i3 is a tenant** in a building, whereas **Sway is the building manager** and the building itself.

### 1. i3 (The Tenant)

In the X11 world, `i3` is just a program running _on top_ of the X Server.

- **The X Server** is the landlord; it owns the hardware, handles the input, and dictates the rules.
    
- **i3** asks the landlord for permission to move windows around.
    
- **Picom** (the compositor) is another tenant; it talks to the landlord to get permission to apply visual effects.
    
- _The consequence:_ Everything is separate. They communicate through the X11 protocol, which adds latency and complexity.
    

### 2. Sway (The Building Manager)

In the Wayland world, `Sway` _is_ the Display Server.

- It handles everything: reading your mouse and keyboard inputs, talking to your graphics card, placing the windows, drawing the title bars, applying the shadows/animations, and sending the final image to your monitor.
    
- _The consequence:_ Because it’s all one process, it's incredibly fast. There is no "protocol overhead" or "middleman" to negotiate with.
    

---

### The Comparison Stack

This table illustrates why your intuition is correct:

| **Task**                            | **X11 Setup (i3 + Picom)** | **Wayland Setup (Sway)** |
| ----------------------------------- | -------------------------- | ------------------------ |
| **Talking to Hardware/GPU**         | X Server                   | Sway                     |
| **Managing Window Positions**       | **i3**                     | **Sway**                 |
| **Compositing (Shadows/VSync)**     | **Picom**                  | **Sway**                 |
| **Handling Keyboard/Mouse**         | X Server                   | Sway                     |
| **Input Methods (e.g., Emoji/IME)** | Separate Daemon            | Sway                     |

### Why this changes everything

You hit on the most important part—the "etc." in your sentence. Because Sway does everything, it has to handle things that `i3` never had to worry about:

1. **Input Handling:** In X11, `i3` didn't care how the keyboard worked; the X Server handled that. In Sway, `Sway` has to talk directly to `libinput` (the Linux kernel's input handler) to understand your keyboard and mouse.
    
2. **Display Output:** In X11, you used a tool like `xrandr` to set your resolution. In Sway, `Sway` handles the display configuration directly (often through the `wlr-randr` utility).
    
3. **Security:** Because `Sway` is the "Display Server," it is the only thing that can see your screen or record your input. This makes your system inherently more secure than `i3`, where _any_ program can technically "snoop" on your X11 session if it has the right permissions.
    

## GTK and QT

If the Display Server/Compositor is the "engine" of your desktop, then **GTK and Qt are the "bodywork and dashboard"** that developers use to build the car.

Frameworks like **GTK** (used by GNOME, Xfce) and **Qt** (used by KDE, VLC, OBS) are **Application Toolkits**. Their job is to provide a consistent way for developers to create buttons, menus, windows, and text boxes without having to manually code how pixels are drawn to the screen.

### How they fit into the stack

You can think of your computer's graphics stack as a layer cake. Here is where the application frameworks sit:

|**Layer**|**Component**|**Function**|
|---|---|---|
|**Top**|**Application**|The software (e.g., Firefox, GIMP).|
|**Middle**|**Toolkit (GTK/Qt)**|Provides the UI elements (Buttons, Menus).|
|**Bridge**|**Backend (GDK/QPA)**|Determines if it should "talk" to X11 or Wayland.|
|**Bottom**|**Display Server/Compositor**|Sway, KWin, or X Server (The "Manager").|

### The "Backend" Bridge (The Magic)

This is where the transition to Wayland becomes really interesting.

GTK (specifically GDK, its backend layer) and Qt (specifically QPA, its platform abstraction layer) have to be "bilingual." When you open an app, the toolkit detects your environment:

1. **If it detects Wayland:** It loads the Wayland backend. It then sends its drawing commands using the Wayland protocol directly to the compositor.
    
2. **If it detects X11:** It loads the X11 backend. It sends its drawing commands to the X Server, which acts as the middleman to the GPU.
    

**This is why most apps don't care if you use Sway or i3.** The application code just says "Draw a button here." The toolkit handles the translation to the specific language that the Display Server speaks.

### Why this changes things for Developers

Before Wayland, developers were often forced to write code that made assumptions about the X Server. They could rely on X11-specific behaviors (like asking the X Server for the position of a window or global coordinates).

In the Wayland world, those behaviors are blocked for security reasons. This forced the toolkits (GTK/Qt) to modernize:

- **Client-Side Decorations (CSD):** On X11, the Window Manager usually drew the "title bar" and "close button." In the Wayland world, the toolkit (GTK/Qt) draws the title bar itself inside the application window. This is why apps often look more "integrated" in modern Linux environments—the app is in control of its own aesthetics.
    
- **Input Handling:** Toolkits now handle complex input methods (like emoji pickers, language IMEs, or touchscreen gestures) far better because they aren't relying on the outdated X11 input protocols.
    

### So, what happens when you run a GTK app on Sway?

1. **The App** asks **GTK** to draw a window.
    
2. **GTK** looks at the OS, sees it is running on **Wayland**, and prepares a buffer.
    
3. **GTK** talks to **Sway** (using the Wayland protocol).
    
4. **Sway** (the Compositor) receives the buffer, positions the window, and draws it to your screen.
    

If that same app ran on **i3 (X11)**:

1. **GTK** would realize it is on **X11**.
    
2. **GTK** would send the instructions to the **X Server**.
    
3. **The X Server** would tell the Window Manager (**i3**) where to put it.
    
4. **The X Server** would also send the data to a compositor like **Picom** to add a shadow.
    

### Summary

Frameworks like GTK and Qt are the reason you can switch between a Wayland desktop (like GNOME) and an X11 desktop (like i3) without having to re-install all your applications. They abstract the "Display Server" away from the application code.

Does this help connect the dots between the code developers write and the "Manager" (Compositor/Server) that ultimately draws the pixels?