

The **Intel 8086** had:

* **16-bit registers**: Its general-purpose registers (AX, BX, CX, DX), index registers (SI, DI), pointer registers (SP, BP), and instruction pointer (IP) were all **16 bits** wide.
* **20 address lines (A0–A19)**: This allowed it to generate **20-bit physical addresses**.

Here's the interesting part:

* A 16-bit register can directly hold values only up to **16 bits**.
* But the 8086 needed to access **1 MB (2²⁰ bytes)** of memory, which requires **20-bit addresses**.

To solve this, Intel introduced **segmented memory**. A physical address is formed by combining a **16-bit segment register** with a **16-bit offset**:
$$
[
\text{Physical Address} = (\text{Segment} \times 16) + \text{Offset}
]
$$

or equivalently:

$$
[
\text{Physical Address} = (\text{Segment} \ll 4) + \text{Offset}
]
$$
For example:

* Segment = **1234H**
* Offset = **5678H**

Physical address:

```
12340H
+5678H
-------
179B8H
```

So:

* **Registers:** 16 bits
* **Address bus:** 20 bits
* **Maximum addressable memory:** (2^{20} = 1,048,576) bytes = **1 MB**
* **Data bus:** 16 bits (it could transfer 16 bits at a time)

This segmented addressing scheme is one of the defining architectural features of the 8086.

### Why do segments start every 16 bytes?

A segment register is **16 bits**, but a physical address is **20 bits**.

Instead of storing the full 20-bit base address, the CPU assumes the lowest 4 bits are always zero. It computes:

```text
Physical Address = (Segment << 4) + Offset
```

Shifting left by 4 bits is the same as multiplying by 16, so segment bases are always multiples of **16 bytes**.

For example:

|Segment|Base Address|
|---|--:|
|1000H|10000H|
|1001H|10010H|
|1002H|10020H|
|1003H|10030H|

Notice that each new segment starts only **16 bytes** after the previous one.

---

### But each segment is 64 KB long

Each segment allows offsets from:

```text
0000H to FFFFH
```

So the segment `1000H` covers:

```text
10000H → 1FFFFH
```

The next segment, `1001H`, covers:

```text
10010H → 2000FH
```

Let's visualize them:

```text
Memory

10000 ------------------------------ 1FFFF   Segment 1000
       10010 ------------------------------ 2000F   Segment 1001
              10020 ------------------------------ 2001F   Segment 1002
```

They overlap almost completely.

---

### How much do they overlap?

Segment 1000:

```text
10000H → 1FFFFH
```

Segment 1001:

```text
10010H → 2000FH
```

The second starts only **16 bytes later**, so the overlap is:

```text
64 KB - 16 bytes
= 65,536 - 16
= 65,520 bytes
```

That's why people often say the segments "slide" across memory in **16-byte increments**.

---

### Multiple ways to refer to the same memory

Because of the overlap, many physical addresses have several valid segment:offset pairs.

Suppose we want to access physical address:

```text
10020H
```

These all work:

|Segment|Offset|Physical Address|
|---|---|---|
|1000H|0020H|10020H|
|1001H|0010H|10020H|
|1002H|0000H|10020H|

Even more:

```text
0FFF:0030
1000:0020
1001:0010
1002:0000
```

All refer to **exactly the same byte in RAM**.

---

### How many representations can one address have?

For most addresses (away from the very beginning and end of the 1 MB space), there are up to **4,096** different segment:offset combinations that map to the same physical address.

That's because there are 4,096 possible ways to adjust the segment and offset by 16-byte increments while keeping the sum the same.

For example:

```text
10000:1234
10001:1224
10002:1214
...
```

Each time:

- increase the segment by 1 (base moves +16 bytes)
    
- decrease the offset by 16
    

The physical address stays unchanged.

---

### Why did Intel choose this design?

It was a compromise:

- ✅ Keep registers at **16 bits**, making the CPU cheaper and compatible with existing 16-bit software concepts.
    
- ✅ Reach **1 MB** of memory using a 20-bit address bus.
    
- ✅ Generate the address with simple hardware: a **4-bit left shift** followed by a **20-bit addition**.
    
- ❌ Accept that memory addresses are **not unique**—many segment:offset pairs can refer to the same physical location.
    


