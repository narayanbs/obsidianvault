
---

# Why is ( Ax + By + Cz + D = 0 ) the Equation of a Plane in 3D?

The equation
[
Ax + By + Cz + D = 0
]
is the standard form of a plane in 3D because it represents **all points ((x, y, z)) that satisfy a linear condition based on a normal vector**.

## 1. Normal Vector to the Plane

* The vector
  [
  \mathbf{n} = \langle A, B, C \rangle
  ]
  is called the **normal vector** of the plane.
* This vector is **perpendicular** (normal) to every vector lying *in* the plane.

## 2. Geometric Interpretation

* Let ( \mathbf{P_0} = (x_0, y_0, z_0) ) be a known point on the plane.
* For any other point ( \mathbf{P} = (x, y, z) ) on the plane, the vector
  [
  \overrightarrow{P_0P} = \langle x - x_0, y - y_0, z - z_0 \rangle
  ]
  lies **in** the plane.
* Since ( \mathbf{n} ) is normal to the plane, it must be **perpendicular** to ( \overrightarrow{P_0P} ). Thus, their dot product is zero:
  [
  \mathbf{n} \cdot \overrightarrow{P_0P} = 0
  ]
* Writing this out explicitly:
  [
  A(x - x_0) + B(y - y_0) + C(z - z_0) = 0
  ]

## 3. Rearranging to Standard Form

* Expand the expression:
  [
  Ax + By + Cz - (A x_0 + B y_0 + C z_0) = 0
  ]
* Let
  [
  D = - (A x_0 + B y_0 + C z_0)
  ]
* The equation becomes:
  [
  Ax + By + Cz + D = 0
  ]

This equation represents **all points ((x, y, z)) on the plane** with normal vector ( \mathbf{n} ).

---

## Summary

* ( A, B, C ) define the **normal vector** perpendicular to the plane.
* ( D ) shifts the plane along the direction of the normal vector.
* Every point ( (x, y, z) ) satisfying
  [
  Ax + By + Cz + D = 0
  ]
  lies on the plane.

---

If you want, I can also prepare an example or visualization!

