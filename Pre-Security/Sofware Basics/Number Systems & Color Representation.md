Number Systems & Color Representation
 
### Why Computers Use Binary
 
Unlike humans (base-10, owing to having ten fingers), computers operate on physical states with only two distinguishable conditions:
 
| Physical Phenomenon | Two States | Example System |
|---|---|---|
| Voltage level | Low / High | Transistor logic |
| Magnetic polarity | North / South | Hard disk drives |
| Light presence | Off / On | Fiber optics |
 
A transistor either passes or blocks current — there is no practical "in-between" state for digital logic to rely on. This binary (on/off) behavior is represented numerically as `0` and `1`, with each single digit referred to as a **bit**.
 
| Term | Definition |
|---|---|
| **Bit** | A single binary digit (`0` or `1`) — represents one of 2 states |
| **Byte (Octet)** | A group of 8 bits — represents one of 256 states (`2⁸`) |
 
---
 
### Color Representation: A Practical Application of Binary
 
#### Eight Colors from Three Switches
 
Computer color is built from three channels — **Red, Green, Blue (RGB)** — each acting like a switch.
 
If each channel has only two states (on/off), the total number of representable colors is:
 
```
2 × 2 × 2 = 8 colors
```
 
| Binary (R G B) | Meaning | Color |
|---|---|---|
| `000` | All off | Black |
| `001` | Blue only | Blue |
| `010` | Green only | Green |
| `100` | Red only | Red |
| `011` | Green + Blue | Cyan |
| `101` | Red + Blue | Magenta |
| `110` | Red + Green | Yellow |
| `111` | All on | White |
 
#### Scaling to 16+ Million Colors
 
Restricting each channel to "on/off" is limiting. By giving each channel **256 possible intensity levels** (i.e., 8 bits per channel) instead of just 2, the total expands dramatically:
 
```
256 × 256 × 256 = 16,777,216 colors
```
 
| Concept | Value |
|---|---|
| Bits per channel | 8 (1 byte) |
| Channels (R, G, B) | 3 |
| Total bits per color | 24 (3 bytes) |
| Total representable colors | 16,777,216 |
 
A full-color value might look like this in raw binary:
 
```
10100011 11101010 00101010
   R         G         B
```
 
— which is accurate, but impractical to read or type. This is precisely the problem **hexadecimal notation** solves.
 
---
 
### Hexadecimal Representation
 
Hexadecimal (base-16) groups **4 bits** into a single symbol, using digits `0`–`9` and letters `A`–`F` to represent values `10`–`15`.
 
| Hex Digit | Binary | Hex Digit | Binary |
|---|---|---|---|
| `0` | `0000` | `8` | `1000` |
| `1` | `0001` | `9` | `1001` |
| `2` | `0010` | `A` | `1010` |
| `3` | `0011` | `B` | `1011` |
| `4` | `0100` | `C` | `1100` |
| `5` | `0101` | `D` | `1101` |
| `6` | `0110` | `E` | `1110` |
| `7` | `0111` | `F` | `1111` |
 
Since each byte (8 bits) maps to exactly **two hex digits**, the earlier 24-bit color:
 
```
10100011 11101010 00101010   (binary)
       ↓
     A3EA2A                  (hexadecimal)
```
 
This is the standard format used in CSS, design tools, and graphics software (e.g., `#A3EA2A`).
 
#### Summary: Color Encoding
 
| Fact | Detail |
|---|---|
| Bits per color | 24 (3 bytes) |
| Values per byte | 256 (0–255) |
| Byte assignment | One byte each for Red, Green, Blue |
| Bits per hex digit | 4 |
| Hex digits per byte | 2 |
| Hex digits per color | 6 (e.g., `A3EA2A`) |
 
---
 
### Positional Number Systems
 
Any positional number system represents a value as a sum of digits multiplied by powers of its **base**.
 
#### Decimal (Base-10) — for reference
 
```
213 = 2×10² + 1×10¹ + 3×10⁰
    = 200 + 10 + 3
    = 213
```
 
#### Binary (Base-2)
 
Uses only digits `0` and `1`; each position represents a power of 2.
 
```
1001 = 1×2³ + 0×2² + 0×2¹ + 1×2⁰
     = 8 + 0 + 0 + 1
     = 9
```
 
**Binary-to-decimal reference table:**
 
| Binary | Calculation | Decimal |
|---|---|---|
| `0000` | 0+0+0+0 | 0 |
| `0001` | 0+0+0+1 | 1 |
| `0010` | 0+0+2+0 | 2 |
| `0011` | 0+0+2+1 | 3 |
| `1100` | 8+4+0+0 | 12 |
| `1101` | 8+4+0+1 | 13 |
| `1110` | 8+4+2+0 | 14 |
| `1111` | 8+4+2+1 | 15 |
 
> **Pattern:** The rightmost digit is always multiplied by `base⁰`; each position moving left increases the exponent by 1.
 
#### Hexadecimal (Base-16)
 
Uses digits `0`–`9` and `A`–`F`; each position represents a power of 16.
 
```
9BDF = 9×16³ + 11×16² + 13×16¹ + 15×16⁰
     = 9×4096 + 11×256 + 13×16 + 15×1
     = 36,864 + 2,816 + 208 + 15
     = 39,903
```
 
#### Octal (Base-8)
 
Uses digits `0`–`7`; groups **3 bits** per digit (rather than hex's 4).
 
| Decimal | Octal | Binary |
|---|---|---|
| 0 | `0` | `000` |
| 1 | `1` | `001` |
| 2 | `2` | `010` |
| 3 | `3` | `011` |
| 4 | `4` | `100` |
| 5 | `5` | `101` |
| 6 | `6` | `110` |
| 7 | `7` | `111` |
 
```
357 = 3×8² + 5×8¹ + 7×8⁰
    = 192 + 40 + 7
    = 239
```
 
---
 
### Number System Comparison
 
| System | Base | Digits Used | Bits Grouped per Digit | Common Use |
|---|---|---|---|---|
| **Binary** | 2 | `0`–`1` | — | Native machine representation |
| **Octal** | 8 | `0`–`7` | 3 | Unix file permissions |
| **Decimal** | 10 | `0`–`9` | — | Human-readable numbers |
| **Hexadecimal** | 16 | `0`–`9`, `A`–`F` | 4 | Memory addresses, color codes, MAC addresses |
 
