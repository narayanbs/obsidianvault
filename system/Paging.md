

---

### 1. Virtual Address Structure

On a **32-bit system**, a virtual address is 32 bits. In **two-level paging**:

- **Page Directory** – top-level table
    
- **Page Table** – second-level table
    
- **Page Offset** – offset inside the actual page
    

Typically, the 32-bit virtual address is divided like this:

|Bits|Purpose|
|---|---|
|31–22|Page Directory Index (10 bits)|
|21–12|Page Table Index (10 bits)|
|11–0|Page Offset (12 bits)|

- **Page Directory Index (PDI):** selects an entry in the Page Directory.
    
- **Page Table Index (PTI):** selects an entry in the Page Table pointed to by the Page Directory entry.
    
- **Page Offset:** offset within the 4 KB page (since 12 bits → (2^{12} = 4096) bytes per page).
    

---

### 2. Address Translation Step-by-Step

1. **Start with the Page Directory Base Address (PDBR)**  
    The CPU has a register (CR3 on x86) pointing to the **base of the Page Directory**.
    
2. **Use the Page Directory Index (PDI)**
    
    - Multiply the PDI by the size of a Page Directory Entry (4 bytes on x86)
        
    - Add it to the Page Directory Base Address → this gives the **address of the PDE** (Page Directory Entry).
        
3. **Read the PDE to get the Page Table Base Address**  
    The PDE contains the **base address of the Page Table** for that virtual address’s section.
    
4. **Use the Page Table Index (PTI)**
    
    - Multiply PTI by the size of a Page Table Entry (4 bytes)
        
    - Add it to the Page Table Base Address from the PDE → gives the **address of the Page Table Entry (PTE)**.
        
5. **Read the PTE to get the Page Frame Base Address**  
    The PTE contains the **physical page frame address** in memory.
    
6. **Add the Page Offset**  
    Finally, add the **page offset** (last 12 bits of virtual address) to the page frame address → **physical address**.
    

---

### 3. Visual Summary

```
Virtual Address (32 bits):
| 31-22 | 21-12 | 11-0 |
| PDI   | PTI   | Offset |

Physical Address = [Page Frame from PTE] + Offset
```

---

✅ **Key Points:**

- Each page is usually **4 KB** (4096 bytes).
    
- Two-level paging reduces memory needed for page tables compared to a single-level table.
    
- The translation involves **2 memory lookups** (Page Directory + Page Table) before accessing the final physical memory.
    

---
Here’s a clear **diagram of 32-bit two-level paging** and how a virtual address is translated to a physical address:

```
Virtual Address (32 bits)
+----------------+----------------+--------------+
| Page Directory | Page Table     | Page Offset  |
| Index (10-bit) | Index (10-bit) | Offset (12b)|
+----------------+----------------+--------------+
        |                 |              
        v                 v              
Page Directory Base         \
(CR3 register)               \
        |                    \
        | + (PDI * 4)        \  <-- Each PDE is 4 bytes
        v                      \
    Page Directory Entry (PDE)   \
        |                         \
        | points to base of Page Table
        v
    Page Table Base Address
        | + (PTI * 4)             <-- Each PTE is 4 bytes
        v
    Page Table Entry (PTE)
        | points to Page Frame
        v
    Physical Page Frame Base Address
        + Page Offset
        v
    Physical Address
```

### Step-by-Step Translation:

1. **Take the 10-bit Page Directory Index (PDI)** from the virtual address.
    
2. Multiply by **4 bytes**, add to the **Page Directory Base Address (from CR3)** → get the **PDE**.
    
3. PDE points to a **Page Table**.
    
4. Take the **10-bit Page Table Index (PTI)**, multiply by 4, add to the **Page Table Base** → get the **PTE**.
    
5. PTE gives the **physical page frame base**.
    
6. Add the **12-bit page offset** to reach the **exact physical memory address**.
    

---
Another diagram: 

```
Virtual Address (32 bits)
+----------------+----------------+--------------+
|  PDI (10 bits) |  PTI (10 bits) |  Offset(12) |
+----------------+----------------+--------------+
         |                 |
         v                 v
  +----------------+     Page Offset
  | Page Directory |         |
  | Entry (PDE)   |         v
  +----------------+  +--------------+
         |               |  Add to   |
         v               | Page Frame|
  +----------------+     +--------------+
  | Page Table     |---------------> Physical Address
  | Entry (PTE)   |
  +----------------+
         |
         v
  Physical Page Frame
```

- **PDI → PDE → Page Table Base**
    
- **PTI → PTE → Page Frame Base**
    
- **Offset → Add to Page Frame → Physical Address**
    
---

### 1. Why do we multiply by 4 in paging?

- **Page Directory Entry (PDE)** and **Page Table Entry (PTE)** are stored in memory.
    
- Each entry is **4 bytes** (32 bits) on a 32-bit x86 system.
    

When you have an **index** into an array of entries, you need to convert it into a **byte offset** in memory.

---

### 2. Example

- Suppose your **Page Directory Index (PDI)** = 3.
    
- Each PDE is 4 bytes.
    
- To find the memory address of the 3rd PDE:
    

[  
\text{Address of PDE} = \text{Page Directory Base} + (\text{PDI} \times 4)  
]

- Why 4? Because each entry takes 4 bytes. If you didn’t multiply by 4, you’d just land **4 times too early** in memory — you wouldn’t reach the correct PDE.
    

---

### 3. Same for Page Table Index (PTI)

- PTI selects the entry inside the Page Table.
    
- Each **Page Table Entry (PTE)** is also **4 bytes**, so:
    

[  
\text{Address of PTE} = \text{Page Table Base} + (\text{PTI} \times 4)  
]

---

✅ **Key point:** Multiplying by 4 converts an **array index** (which counts entries) into a **byte offset** (which memory uses).

---

If you want, I can **draw a mini diagram showing the multiplication by 4 in memory**, which makes it really easy to visualize.

Do you want me to do that?