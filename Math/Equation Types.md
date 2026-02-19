Yeah, this is a really good thing to get straight early—it clears up a lot of confusion later.

There are **two different ways people “name” equations**, and they often get mixed together:

1. by **number of variables / dimension**
    
2. by **type of equation (linear, quadratic, etc.)**
    

Let’s separate them cleanly.

---

## 1. Names based on number of variables

This is about **how many unknowns** are involved.

|# of variables|Common name|Example|
|---|---|---|
|1|**Univariate**|(2x + 1 = 0)|
|2|**Bivariate**|(x + y = 0)|
|3|**Trivariate**|(x + y + z = 0)|
|4|**Quadrivariate** (rarely said)|(x_1+x_2+x_3+x_4=0)|
|(n)|**Multivariate**|(a_1x_1+\dots+a_nx_n=0)|

Most people stop explicitly naming things after 3 and just say **multivariate**.

---

## 2. Names based on the _type_ of equation

This is about **how variables appear** in the equation.

### Linear

Variables only to the first power.

- (x = 3)
    
- (x + y = 0)
    
- (x + y + z = 0)
    

Geometrically: lines, planes, hyperplanes.

---

### Quadratic

At least one variable is squared, but no higher powers.

- (x^2 = 4)
    
- (x^2 + y^2 = 1)
    
- (x^2 + y + z = 0)
    

Geometrically: parabolas, circles, ellipses, paraboloids.

---

### Cubic

At least one variable has power 3.

- (x^3 + x = 0)
    
- (x^3 + y^3 = 1)
    

---

### Polynomial (general)

Any finite sum of powers of variables.

- (x^5 - 3x^2 + 7 = 0)
    
- (x^2y + y^3 = 1)
    

Linear, quadratic, and cubic are all **polynomial** equations.

---

### Nonlinear (catch-all term)

Anything that’s _not_ linear.

Includes:

- polynomial equations of degree ≥ 2
    
- exponential: (e^x = y)
    
- trigonometric: (\sin x = y)
    
- rational: (\frac{x}{y} = 1)
    

---

## 3. Combining both naming ideas

You’ll often see both used together:

- **Linear bivariate equation** → (x + y = 0)
    
- **Linear trivariate equation** → (x + y + z = 0)
    
- **Quadratic bivariate equation** → (x^2 + y^2 = 1)
    
- **Nonlinear multivariate equation** → basically anything messy 😄
    

---

## 4. Systems of equations

Once you have more than one equation:

- **Linear system** → all equations are linear
    
- **Nonlinear system** → at least one equation is nonlinear
    

Example:  
[  
\begin{cases}  
x + y = 0 \  
x - y = 2  
\end{cases}  
]  
→ linear system in two variables.

---

### One subtle but important point

> “Linear” describes the **equation**, not the **shape**.

The shape (line, plane, surface) depends on:

- number of variables
    
- number of equations
    

If you want, I can also give you a **one-page mental map** that connects  
variables → equation type → geometric object.

Perfect—here’s a clean **mental map** you can keep in your head. Think of it as _equations → space → shape_.

---

## Step 1: Count variables = the space you’re in

The number of variables tells you the **dimension of the space**.

|Variables|Space|Name|
|---|---|---|
|1|number line|(\mathbb{R})|
|2|plane|(\mathbb{R}^2)|
|3|3D space|(\mathbb{R}^3)|
|(n)|(n)-dimensional space|(\mathbb{R}^n)|

---

## Step 2: Look at the equation type = flat or curved

This tells you **what kind of object** lives in that space.

### Linear equations (flat)

General form:  
[  
a_1x_1 + a_2x_2 + \dots + a_nx_n = b  
]

- no powers
    
- no products of variables
    

They always give **flat objects**.

---

## Step 3: One equation → codimension 1

A single linear equation in (n) variables always removes **one dimension**.

|Variables|1 linear equation gives|
|---|---|
|1|a point|
|2|a line|
|3|a plane|
|(n)|an ((n-1))-dimensional hyperplane|

So:

- (x = 2) → point on a line
    
- (x + y = 0) → line in the plane
    
- (x + y + z = 0) → plane in space
    

Same rule every time.

---

## Step 4: More equations → fewer dimensions

Each **independent** linear equation removes another dimension.

### Linear systems cheat table

|Variables|Independent linear equations|Result|
|---|---|---|
|2|1|line|
|2|2|point (or no solution)|
|3|1|plane|
|3|2|line|
|3|3|point|
|(n)|(k)|((n-k))-dimensional object|

This is why:

- two planes intersect in a line
    
- three planes (usually) intersect in a point
    

---

## Step 5: Nonlinear = curved versions

If the equation isn’t linear, the same dimension rules apply—but the shapes bend.

|Equation|Shape|
|---|---|
|(x^2 + y^2 = 1)|circle|
|(x^2 + y^2 + z^2 = 1)|sphere|
|(x^2 + y = 0)|parabola|
|(x^2 + y^2 - z = 0)|paraboloid|

Still:

- 1 equation → surface
    
- 2 equations → curve
    
- 3 equations → points
    

---

## Step 6: The “one-sentence rule” (memorize this)

> **Number of variables = dimension of space**  
> **Number of independent equations = how many dimensions you lose**  
> **Linear = flat, nonlinear = curved**

If you keep just that sentence in mind, almost everything clicks.

---

If you want next:

- a visual analogy (slicing space)
    
- how this connects to **rank and null space**
    
- or why linear algebra people obsess over hyperplanes
    

Just say the word 😄