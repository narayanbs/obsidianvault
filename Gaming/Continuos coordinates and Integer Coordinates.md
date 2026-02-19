This distinction is **very important** in graphics and math.

We’ll break it down clearly.

---

# 1️⃣ Continuous Coordinates (Mathematical Plane)

This is the normal **Cartesian coordinate system** you learn in math.

If your canvas width = 10 and height = 10, centered at (0,0):

$$ 
x \in [-5, 5]  
$$  
$$
y \in [-5, 5]  
$$

### Key properties:

- Coordinates can be **any real number**
    
- Examples:
    
    - (1.2, 3.7)
        
    - (-4.999, 0.001)
        
- Infinite possible positions
    
- Used in:
    
    - Geometry
        
    - Physics simulations
        
    - Vector graphics
        
    - Transformations (rotation, scaling)
        

👉 Think of this as a smooth sheet of paper — you can put a point anywhere.

---

# 2️⃣ Integer Coordinates (Pixel Grid / Discrete Space)

Screens are **not continuous** — they are made of pixels.

If width = 10 pixels:

You do NOT actually have continuous space from -5 to +5.

You have **10 discrete pixel positions**.

Each pixel has a fixed location.

---

## 🔎 Important Concept: Pixel Centers

Pixels are tiny squares.

Coordinates typically refer to the **center of each pixel**, not the edges.

If we center a 10×10 pixel grid at (0,0), the integer coordinates would usually be:

$$ 
x \in {-5, -4, -3, -2, -1, 0, 1, 2, 3, 4}  
$$

Notice:

- There are 10 integers
    
- It goes from -5 to +4
    
- Not -5 to +5 (that would be 11 values)
    

Same for y.

---

## Why not -5 to +5?

Because:

$$ 
-5, -4, -3, -2, -1, 0, 1, 2, 3, 4  
$$

That’s 10 values.

If you included +5, you'd get 11 pixels.

---

# 3️⃣ Another Way to View It (Half-Pixel View)

Sometimes people define pixel edges like this:

$$  
x \in [-5, 5)  
$$

That means:

- -5 is included
    
- 5 is excluded
    
- Width = 10 units
    

This is very common in graphics APIs.

---

# 4️⃣ Why This Matters in Graphics

When you:

- Draw a line
    
- Rotate a shape
    
- Rasterize a triangle
    

You compute everything in **continuous coordinates**, then convert to **integer pixel coordinates**.

That conversion step is called:

> Rasterization

It decides which pixels get colored.

---

# 5️⃣ Real-World Example

Suppose you compute a point at:

[  
(2.3, 1.7)  
]

On a real screen:

It must become something like:

[  
(2, 2)  
]

(or maybe (2,1) depending on rounding rules)

Because pixels only exist at integer positions.

---

# 6️⃣ Quick Summary

|Continuous|Integer (Discrete)|
|---|---|
|Real numbers|Whole numbers only|
|Infinite positions|Fixed number of pixels|
|Used for math|Used for actual display|
|Smooth|Grid-based|

---

# 🎯 Intuition

Continuous space = geometry world  
Integer space = pixel world

Graphics is basically:

> Continuous math → Discrete pixels

---

If you'd like, I can next explain:

- Why even-sized canvases behave slightly asymmetrically
    
- Or how subpixel rendering works
    
- Or how OpenGL / DirectX treat coordinates internally