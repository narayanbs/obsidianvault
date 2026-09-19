### what is 752 in binary

To convert the decimal number **752** to binary, we find the largest powers of 2 that fit into the number and subtract them as we go.

### The Calculation

- $512$ is the largest power of 2 in 752: $752 - 512 = 240$
    
- $128$ is the next: $240 - 128 = 112$
    
- $64$ is the next: $112 - 64 = 48$
    
- $32$ is the next: $48 - 32 = 16$
    
- $16$ is the next: $16 - 16 = 0$
    

---

### The Binary Representation

By placing a **1** in the positions of the powers we used and a **0** in the others:

|**Power of 2**|**Value**|**Used?**|**Bit**|
|---|---|---|---|
|$2^9$|512|Yes|**1**|
|$2^8$|256|No|**0**|
|$2^7$|128|Yes|**1**|
|$2^6$|64|Yes|**1**|
|$2^5$|32|Yes|**1**|
|$2^4$|16|Yes|**1**|
|$2^3$|8|No|**0**|
|$2^2$|4|No|**0**|
|$2^1$|2|No|**0**|
|$2^0$|1|No|**0**|

**Decimal 752 in binary is:**

> **1011110000**

*The Above is called `subtraction method`. The most common way to find binary is by using  a method called `successive dvision method` where we repeatedly divide the number by 2 and make note of the remainder. 


### but isnt it positive, so shouldnt the sign bit be 0

There is a small distinction between **"pure binary"** and **"signed binary"** (like Two's Complement) that usually causes this confusion.

### 1. Pure (Unsigned) Binary

When we just convert a number like 752, we are typically looking at the magnitude. In this case, the number starts with the first "1" required to reach that value:

- **752** = `1011110000`
    

### 2. Signed Binary (Computer Memory)

When a computer stores this number in a fixed-size container (like a 16-bit signed integer), it **must** account for the sign bit. Since 752 is positive, the most significant bit (the far left) is indeed **0**.

If we look at 752 in a **16-bit signed format**, it looks like this:

> **0**000001011110000
> > _(The bolded 0 is the sign bit)_

If we look at -752 in 16-bit signed format (twos'complement), it looks like this:
> **1**111110100010000
> > _(The bolded 1 is the sign bit)_


### 3. Why the "1" appeared first in my last answer

In general math and logic, we usually omit "leading zeros" just like we don't write "0752" in decimal unless we are filling out a specific form.

### Summary of Rules

- **Positive Numbers:** The sign bit is always **0**.
    
- **Negative Numbers:** The sign bit is always **1** (usually using Two's Complement).
    
- **Standard Notation:** If no bit-length is specified (like "8-bit" or "16-bit"), we just write the number starting from the first "1".
    

So, if you were writing this for a computer science exam or a specific architecture, you are 100% correct: to denote it as positive, you would prefix the magnitude with a **0**.

