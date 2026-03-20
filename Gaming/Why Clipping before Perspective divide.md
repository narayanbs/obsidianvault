Great question — this is one of the most important subtleties in 3D graphics.

The short answer:

> **Clipping must happen before perspective divide because after the divide, the math is no longer linear.**

Let’s unpack that carefully.

---

# 1️⃣ What is the perspective divide?

After projection, vertices are in **clip space**:

```
(x, y, z, w)
```

Then we do the perspective divide:

```
x' = x / w
y' = y / w
z' = z / w
```

This converts clip space → **Normalized Device Coordinates (NDC)**.

The divide by `w` is what creates perspective:

- Objects farther away (larger |w|) shrink.
    
- Objects closer appear larger.
    

---

# 2️⃣ Why clipping must happen BEFORE dividing by w

### ✅ Reason 1: Division is non-linear

Before divide:

- Edges between vertices are linear in 4D space.
    

After divide:

- Edges become curved in screen space.
    
- Interpolation is no longer linear.
    

Clipping relies on finding **intersection points on triangle edges**.

That requires linear interpolation.

You can safely do this in:

- View space
    
- Or clip space (before divide)
    

You **cannot safely do this after dividing**, because the relationship is no longer linear.

---

### ✅ Reason 2: w can be zero or negative

If a vertex is behind the camera:

```
w ≤ 0
```

Then:

```
x / w
```

- May explode toward infinity
    
- May flip sign
    
- May produce nonsense coordinates
    

If you divide first, you can get:

- Huge coordinates
    
- Geometry that wraps across the screen
    
- Broken triangles
    

Clipping removes those problematic vertices _before_ they cause trouble.

---

# 3️⃣ What clipping actually checks in clip space

In clip space, the visible region is:

```
-w ≤ x ≤ w
-w ≤ y ≤ w
-w ≤ z ≤ w
```

These are linear inequalities.

That’s perfect for clipping.

Once you divide by w, the condition becomes:

```
-1 ≤ x' ≤ 1
```

But now x' came from x/w — not linear anymore.

---

# 4️⃣ What goes wrong if you divide first?

Imagine a triangle where:

- One vertex is in front of the camera
    
- One vertex is behind the camera
    

If you divide first:

- The vertex behind the camera might map to a huge flipped coordinate
    
- The triangle may stretch across the entire screen
    
- Rasterization becomes garbage
    

This is why near-plane clipping is especially important.

---

# 5️⃣ Pipeline Summary (Correct Order)

The proper order is:

1. Model transform
    
2. View transform
    
3. Projection transform
    
4. **Clip in clip space (before divide)**
    
5. Perspective divide
    
6. Viewport transform
    
7. Rasterization
    

Clipping must occur at step 4.

---

# 6️⃣ Intuition Version

Think of it like this:

Perspective divide bends space.

Clipping must be done while space is still straight.

---

If you'd like, I can next explain:

- Why interpolation of texture coordinates must also use 1/w
    
- Or show a numeric example of a triangle crossing the near plane
    

Both are really important concepts.