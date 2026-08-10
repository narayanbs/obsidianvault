
# Protected Mode 

### 1. GDTR LDTR

The CPU already has special registers:

* **GDTR** = Global Descriptor Table Register
* **LDTR** = Local Descriptor Table Register

**GDTR contains**

* Base address of the GDT (Global descriptor table)
* Limit (size)

**LDTR contains**

* A selector that points to the LDT descriptor (Local descriptor table) in the GDT.

So:

```
GDTR
 ├── Base = address of GDT
 └── Limit

LDTR
 └── Selector to the current LDT
```

### 2. Segment registers 

The segment registers (`CS`, `DS`, `SS`, `ES`, `FS`, `GS`) contain a **segment selector**, which is a 16-bit value.

The selector is divided into:

```
15                3 2 1 0
+-----------------+-+-+-+
|   Index         |T|RPL|
+-----------------+-+-+-+
```

* **Index (13 bits):** Entry number in the descriptor table.
* **TI (Table Indicator):**

  * 0 = GDT
  * 1 = LDT
* **RPL (Requested Privilege Level):** 0–3.

So the selector is **not just an index**. It also specifies **which table** to use and the privilege level.

The selector has a **13-bit index**, so

```
2^13 = 8192 entries
```
So each GDT or LDT can contain up to **8192 descriptors**.

---

### 3. GDT and LDT entries

Each descriptor is **8 bytes**.  Since there are 8192 entries 

Maximum size:

```
8192 × 8 bytes = 65536 bytes = 64 KB
```

---

### 4. What does each descriptor contain?

A segment descriptor contains fields such as:

* Base address (where the segment starts)
* Segment limit (size)
* Type (code/data/system)
* DPL (Descriptor Privilege Level)
* Present bit
* Read/write/execute permissions
* Granularity (byte or 4 KB pages)
* Default operand size (16/32-bit)
* Other control flags

Conceptually:

```
Descriptor
+----------------------+
| Base Address         |
| Segment Limit        |
| Type                 |
| DPL                  |
| Present              |
| Granularity          |
| Default Size         |
| Access Rights        |
+----------------------+
```

---

### 5. How address translation works

Suppose:

```
DS = 0x0023
```

The CPU interprets it as:

```
Selector
  Index = 4
  TI = 0  → use GDT
  RPL = 3
```

Then the CPU:

1. Uses **GDTR** to find the GDT.
2. Looks up descriptor **#4**.
3. Reads its base, limit, and protection information.
4. Checks permissions.
5. Computes the linear address:

```
Linear Address = Segment Base + Offset
```

---

