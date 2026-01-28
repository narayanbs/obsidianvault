

---

## 1. Object (Model) Space — ✅ Correct

> _“We start with vertices of a primitive shape or model in object space… each object will have its own object space with its origin at (0,0,0).”_

Yes.  
Each mesh is authored in its **own local coordinate system** (object/model space). The origin and orientation are arbitrary and chosen by the artist or procedural generator.

Example:

- A cube centered at `(0,0,0)`
    
- A character mesh with feet at `y = 0`
    

So far, perfect.

---

## 2. World Space — ✅ Correct

> _“Now these objects need to be part of a scene, so they are scaled, translated or transformed into a world space…”_

Exactly.

You apply a **Model Matrix** to each object:

```
WorldPosition = ModelMatrix × ObjectPosition
```

This places all objects into a **shared world coordinate system**, letting them exist relative to each other.

Important detail:

- Every object has its **own model matrix**
    
- After this step, _all_ objects are in the same space
    

---

## 3. View (Camera / Eye) Space — Mostly correct, one clarification ⚠️

> _“Now we have a camera at the origin through which we see the scene…”_

Conceptually yes — **mathematically** we pretend the camera is at the origin, looking down a canonical direction (usually −Z).

But what really happens is:

- The **world is transformed**, not the camera
    
- You apply the **View Matrix**, which is the _inverse of the camera’s transform_
    

```
ViewPosition = ViewMatrix × WorldPosition
```

After this:

- Camera is at `(0,0,0)`
    
- Camera faces forward
    
- Everything is expressed **relative to the camera**
    

> _“…that will make sure that objects beyond the span of the camera are not shown or clipped.”_

Small correction here:

- **The view transform alone does NOT clip**
    
- It just re-expresses coordinates relative to the camera
    

Clipping happens in the **next stage**, after projection.

---

## 4. Clip Space → Projection — ⚠️ This is where precision matters

> _“From view space we move to projection on viewport…”_

There’s an important intermediate space you skipped:

### View Space → Clip Space

You apply the **Projection Matrix** (perspective or orthographic):

```
ClipPosition = ProjectionMatrix × ViewPosition
```

This does two key things:

- Introduces **perspective** (objects farther away shrink)
    
- Encodes the **view frustum** (near plane, far plane, FOV)
    

At this stage:

- Coordinates are still homogeneous `(x, y, z, w)`
    
- Clipping happens **here**, against `-w ≤ x,y,z ≤ w`
    

---

## 5. Normalized Device Coordinates (NDC)

After clipping, the GPU performs **perspective division**:

```
NDC = (x/w, y/w, z/w)
```

Now everything is in:

- X, Y, Z ∈ `[-1, 1]` (API-dependent for Z)
    

---

## 6. Viewport Transform → Screen Space — ✅

> _“…and finally onto the screen.”_

Yes.

The **viewport transform** maps NDC to pixel coordinates:

- `[-1, 1] → [0, width]`
    
- `[-1, 1] → [0, height]`
    

This produces **screen space** coordinates used for rasterization.

---

## The Full Pipeline (Clean Version)

Here’s the canonical flow:

```
Object Space
   ↓  (Model Matrix)
World Space
   ↓  (View Matrix)
View / Eye Space
   ↓  (Projection Matrix)
Clip Space
   ↓  (Clipping + Perspective Divide)
Normalized Device Coordinates
   ↓  (Viewport Transform)
Screen Space
```

---

## Key Corrections to Your Description

You were right about:

- Object → World → View → Projection → Screen
    
- Each object having its own local space
    
- Transforming relative to the camera
    

Small fixes:

- Camera is **not literally at the origin**; we transform the world instead
    
- **Clipping does not happen in view space**
    
- Projection → Clip Space → NDC is an important distinction
    

---

