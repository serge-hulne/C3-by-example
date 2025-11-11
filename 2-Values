## 🔢 C3 by Example: Values

C3 supports a familiar set of basic types including integers, floats, booleans, and characters. You can define constants, use type suffixes, and initialize variables much like in C — but with some improvements and safety rules.

`📄 Source Code: values.c3`


```
import std::io;

fn void main() {
    // Integers
    int a = 5;
    long b = 10000000000;
    uint c = 0xff_u32; // Hexadecimal with suffix

    // Floats
    float f = 3.14;
    double d = 2.718281828459;

    // Booleans
    bool isReady = true;
    bool isDone = false;

    // Characters
    char ch = 'C';
    ushort ch2 = 'C3';
    uint ch4 = 'TEST';

    // Printing values
    io::printfn("a = %d", a);
    io::printfn("b = %ld", b);
    io::printfn("c = %u", c);
    io::printfn("f = %f", f);
    io::printfn("d = %lf", d);
    io::printfn("isReady = %s", isReady ? "true" : "false");
    io::printfn("ch = '%c'", ch);
    io::printfn("ch2 = '%u'", ch2);
    io::printfn("ch4 = '%u'", ch4);
}
```

### 🧠 Explanation

- `Integer Types`: C3 offers signed/unsigned types from 8 to 128 bits (int, long, uint, int128, etc.).
- `Float Types`: Includes float, double, float128. Float to int conversions require explicit casting.
- `Booleans`: Stored as a byte. Use true and false.
- `Character Literals`: 'A' is a char; 'AB' is a ushort; 'TEST' is a uint.

### 🔍 Suffixes for Constants

You can use type suffixes like u32, i64, etc. For example:

- `uint id = 1234u;`
- `long big = 9999999999i64;`

✨ Nice Touch: `Readable Literals`

- Underscores can be used for readability:
int price = 1_000_000;
▶
### ️ Run the Example
`c3c compile-run values.c3`

Expected Output:

```
a = 5
b = 10000000000
c = 255
f = 3.140000
d = 2.718282
isReady = true
ch = 'C'
ch2 = '17219'
ch4 = '1415934836'
```

### Note: 
- `ch2` and `ch4` are printed as unsigned integers — they contain multiple characters packed in memory (like FourCC codes).

### 🔖 Notes
Type safety is stricter than C — implicit narrowing conversions are mostly not allowed.
Integer constants are assumed to be signed int unless they don’t fit, or you use a suffix.
Floating-point literals default to float unless assigned to a wider type.