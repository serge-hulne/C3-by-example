# C3 Programming Manual 
(non-official quick introduction).

> A practical introduction to the C3 programming language, with emphasis on memory management, arrays, slices, maps, error handling, and file operations.

## Table of Contents

1. [Introduction to C3](#1-introduction-to-c3)
2. [First Program](#2-first-program)
3. [Variables, Types, and Naming Conventions](#3-variables-types-and-naming-conventions)
4. [Important Differences from C](#4-important-differences-from-c)
5. [Fixed-Size Arrays](#5-fixed-size-arrays)
6. [Slices](#6-slices)
7. [Slicing Syntax](#7-slicing-syntax)
8. [Passing Arrays and Slices to Functions](#8-passing-arrays-and-slices-to-functions)
9. [Conversions Between Arrays, Slices, and Pointers](#9-conversions-between-arrays-slices-and-pointers)
10. [Dynamic Allocation](#10-dynamic-allocation)
11. [Memory Management](#11-memory-management)
12. [Strings](#12-strings)
13. [Dynamic Lists](#13-dynamic-lists)
14. [Copying Between Slices](#14-copying-between-slices)
15. [Maps](#15-maps)
16. [Summary of Collections and Conversions](#16-summary-of-collections-and-conversions)
17. [Practical Collection Style](#17-practical-collection-style)
18. [Error Handling](#18-error-handling)
19. [Files: Opening, Reading, Writing, and Closing](#19-files-opening-reading-writing-and-closing)
20. [C Interoperability](#20-c-interoperability)
21. [Comparison with C, Go, Zig, and Rust](#21-comparison-with-c-go-zig-and-rust)
22. [Practical Mental Model](#22-practical-mental-model)

---

## 1. Introduction to C3

### 1.1. General Overview

C3 is a modern systems programming language designed as an evolution of C rather than as a complete break from it. Its goal is to preserve a syntax and mental model close to C while adding safer and more convenient tools.

C3 provides, among other things:

- a module system;
- slices;
- optional result types for error handling;
- contracts;
- semantic macros;
- better variable initialization;
- direct integration with the C ABI;
- more structured memory and resource management.

The general spirit of C3 can be summarized as follows: modernize C without completely replacing its programming model.

C3 targets a precise niche:

- simpler than C++;
- less radical than Rust;
- more modern and safer than C;
- suitable for low-level programs, system tools, libraries, C bindings, small engines, native WebView applications, and lightweight desktop tools.

For C3/WebView projects, C3 is an interesting option: it keeps a C-like logic while offering more structure.

---

## 2. First Program

### 2.1. Minimal Program Structure

A C3 program looks like modernized C.

```c
module hello;

import std::io;

fn void main(String[] args)
{
    io::printn("Hello from C3!");
}
```

The program starts with a module declaration:

```c
module hello;
```

Libraries are imported with `import`:

```c
import std::io;
```

A function is declared with the `fn` keyword:

```c
fn void main(String[] args)
{
    ...
}
```

The `main` function is the program entry point. In this example, it prints a message with:

```c
io::printn("Hello from C3!");
```

Standard library functions are usually accessed through namespaces. For example, input/output functions are accessed through `io::`.

---

## 3. Variables, Types, and Naming Conventions

### 3.1. Basic Types

C3 keeps familiar types for C programmers.

```c
int age = 40;
double price = 12.5;
bool active = true;
char c = 'A';
```

Common types include:

- `int` for integers;
- `double` for double-precision floating-point numbers;
- `bool` for booleans;
- `char` for characters;
- `String` for higher-level strings.

### 3.2. Naming Conventions

C3 enforces more syntactic conventions than C.

User-defined types generally start with an uppercase letter. Functions and variables generally start with a lowercase letter. This rule helps the compiler avoid some historical ambiguities from C.

Example structure:

```c
struct Person
{
    String name;
    int age;
}
```

Unlike C, one does not write:

```c
struct Person { ... };
```

The final semicolon after a `struct` declaration is not used.

---

## 4. Important Differences from C

### 4.1. Zero Initialization

In C, an uninitialized local variable contains an indeterminate value. In C3, local variables are initialized to zero by default, unless explicitly marked as uninitialized with `@noinit`.

```c
int a;           // equals 0
int b @noinit;   // uninitialized, C-style
```

This rule prevents many bugs caused by forgotten or incorrectly initialized local variables.

### 4.2. Arrays as Values

In C3, fixed-size arrays are real value types. They do not automatically decay into pointers as they do in C. The size is part of the type: `int[4]` and `int[8]` are different types.

```c
int[3] a = { 1, 2, 3 };
int[3] b = a;

b[0] = 99;

// a remains { 1, 2, 3 }
// b is now { 99, 2, 3 }
```

Copying a fixed-size array copies its elements. This behavior is more explicit than in C.

---

## 5. Fixed-Size Arrays

### 5.1. Declaring a Fixed-Size Array

A fixed-size array is written by specifying the element type and the size.

```c
int[4] values = { 10, 20, 30, 40 };
```

The actual type is:

```c
int[4]
```

The size is part of the type. Therefore, `int[4]` and `int[5]` are different types.

### 5.2. Size Inference

To let C3 infer the size of a fixed-size array, use `[*]`.

```c
int[*] values = { 10, 20, 30 };
```

In this example, C3 infers:

```c
int[3]
```

### 5.3. Reading the Length

The length of an array is available through `.len`.

```c
int[4] values = { 1, 2, 3, 4 };

io::printfn("%d", values.len); // 4
```

### 5.4. Copying a Fixed-Size Array

A fixed-size array is copied by value.

```c
fn void test()
{
    int[3] a = { 1, 2, 3 };
    int[3] b = a;

    b[0] = 99;

    // a = { 1, 2, 3 }
    // b = { 99, 2, 3 }
}
```

This behavior differs strongly from C, where arrays are very often manipulated as pointers.

---

## 6. Slices

### 6.1. Purpose of Slices

In C, one often passes a pointer and a length separately.

```c
void process(int* data, int len);
```

In C3, a slice is preferred.

```c
fn void process(int[] data)
{
    // data contains a pointer + a length
}
```

An `int[]` slice is a view over an array or over a contiguous memory region. It essentially contains two pieces of information:

- a pointer to the data;
- a length.

It can be mentally represented as:

```c
struct SliceInt
{
    int* ptr;
    sz len;
}
```

### 6.2. Creating a Slice from an Array

To create a slice from a fixed-size array, use a slicing operation.

```c
int[5] values = { 10, 20, 30, 40, 50 };

int[] all = values[..];
```

The slice `all` does not own the data. It points to the array `values`.

### 6.3. Modifying Data Through a Slice

Modifying a slice modifies the underlying memory.

```c
fn void test()
{
    int[3] a = { 1, 2, 3 };
    int[] s = a[..];

    s[0] = 99;

    // a is now { 99, 2, 3 }
}
```

The slice does not copy the elements. It only provides a view.

### 6.4. Lifetime of a Slice

A slice does not necessarily own the memory it points to. Its lifetime therefore depends on the underlying memory.

Do not return a slice pointing to a local stack array.

```c
fn int[] wrong()
{
    int[3] values = { 1, 2, 3 };
    return values[..]; // wrong: the memory disappears at the end of the function
}
```

To return a durable collection, use heap allocation or explicitly transfer ownership of the memory.

---

## 7. Slicing Syntax

### 7.1. Slicing with Start and End Bounds

The following form extracts a range of elements:

```c
int[5] a = { 10, 20, 30, 40, 50 };

int[] s = a[1..3];
```

In C3, with `..`, the final bound is inclusive. The slice therefore contains the elements at indexes `1`, `2`, and `3`.

The result is:

```c
{ 20, 30, 40 }
```

This rule matters because several languages, such as Go, Rust, Python, and JavaScript, use an exclusive final bound.

### 7.2. Slicing with Start and Length

The following form uses a start index and a length:

```c
int[] s = a[1 : 3];
```

This expression means: start at index `1` and take `3` elements.

The result is also:

```c
{ 20, 30, 40 }
```

This form is often clearer in code that manipulates buffers because it avoids recalculating the final index.

### 7.3. Full Slicing

Several forms can be used to obtain the whole array as a slice.

```c
int[5] a = { 10, 20, 30, 40, 50 };

int[] s1 = a[0..4];
int[] s2 = a[..4];
int[] s3 = a[0..];
int[] s4 = a[..];

int[] s5 = a[0 : 5];
int[] s6 = a[:5];
```

### 7.4. Indexing from the End

C3 also supports indexing from the end with `^`.

```c
int[5] a = { 10, 20, 30, 40, 50 };

int[] last_two = a[^2..]; // { 40, 50 }
```

`^1` refers to the last element, `^2` to the second-to-last element, and so on.

---

## 8. Passing Arrays and Slices to Functions

### 8.1. Passing a Fixed-Size Array by Value

To require exactly an array of a given size, write:

```c
fn void print_values(int[4] values)
{
    // values is copied
}
```

This function accepts exactly an `int[4]`. The array is passed by value.

### 8.2. Passing a Slice

In practice, it is often better to receive a slice.

```c
fn void print_values(int[] values)
{
    foreach (v : values)
    {
        io::printfn("%d", v);
    }
}
```

This version accepts arrays of different sizes, dynamic slices, or subranges.

```c
int[4] data = { 1, 2, 3, 4 };

print_values(data[..]);
```

The slice does not copy the elements. It points to them.

---

## 9. Conversions Between Arrays, Slices, and Pointers

### 9.1. Slice to Pointer

A slice can be converted to a pointer to its first element.

```c
int[4] a = { 1, 2, 3, 4 };
int[] s = a[..];

int* p = s;
```

This conversion loses the length.

A raw pointer is therefore less safe than a slice:

```c
fn void dangerous(int* p)
{
    // no information about the available size
}

fn void safer(int[] s)
{
    // s.len is available
}
```

When passing an internal buffer to a C3 function, `T[]` is generally preferable to `T*`.

### 9.2. Pointer to Array

C3 distinguishes between a pointer to an element and a pointer to a complete array.

```c
int* p;
int[4]* arr_ptr;
```

`int*` is a pointer to an integer.

`int[4]*` is a pointer to a complete array of four integers.

```c
int[3] a = { 1, 2, 3 };
int[3]* p = &a;
```

To access the second element:

```c
int x = (*p)[1];
```

C3 also provides a shortcut:

```c
int x = p.[1];
```

### 9.3. Reconstructing a Slice from a Pointer

To reconstruct a slice from a pointer, a length must be provided.

```c
fn void test()
{
    int[4] a = { 1, 2, 3, 4 };

    int[] s = a[..];
    int* p = s;

    int[] s2 = p[0..3];
}
```

This operation is dangerous if the provided length does not match the actual memory.

```c
int x = 123;
int* p = &x;

int[] wrong = p[0..10]; // dangerous: invalid memory
```

### 9.4. Conversion Summary

| Source | Destination | Copy? | Comment |
|---|---:|---:|---|
| `int[4]` | `int[4]` | yes | copies elements |
| `int[4]` | `int[]` | no | view through `a[..]` |
| `int[]` | `int[]` | no | copies only pointer + length |
| `int[]` | `int*` | no | loses length |
| `int[4]*` | `int*` | no | pointer to first element |
| `int*` | `int[]` | no | reconstruction with explicit length |
| `int[]` | `int[4]*` | no | possible cast, but dangerous |

---

## 10. Dynamic Allocation

### 10.1. Slice Owning Allocated Memory

A slice can represent dynamically allocated memory.

```c
int[] values = mem::new_array(int, 10);
defer mem::free(values);
```

Here, `values` is a slice, but it points to heap-allocated memory.

### 10.2. Initialized and Uninitialized Allocation

To allocate a dynamic array initialized to zero:

```c
int[] values = mem::new_array(int, n);
```

To allocate an uninitialized dynamic array:

```c
int[] raw = mem::alloc_array(int, n);
```

`new` usually means zero-initialized allocation.  
`alloc` usually means uninitialized allocation.

### 10.3. Function Returning a Dynamic Slice

To produce a dynamic slice, allocate the memory inside the function and transfer responsibility for freeing it to the caller.

```c
fn int[] make_values(int n)
{
    int[] values = mem::new_array(int, n);

    for (int i = 0; i < n; i++)
    {
        values[i] = i * 10;
    }

    return values;
}

fn void main()
{
    int[] values = make_values(5);
    defer mem::free(values);

    // values = {0, 10, 20, 30, 40}
}
```

The slice is only a handle to the memory. The caller must free that memory.

---

## 11. Memory Management

### 11.1. General Model

C3 does not have a garbage collector. As in C, memory is generally managed manually.

An object can be:

1. placed on the stack;
2. dynamically allocated on the heap;
3. temporarily allocated through the temporary allocator.

The general rule is simple: durable allocated memory must be freed.

### 11.2. Stack Memory

Local memory is placed on the stack.

```c
fn void test()
{
    int x = 12;
    int[3] values = { 1, 2, 3 };
}
```

`x` and `values` live until the end of the function. Do not return a pointer or a slice pointing to this memory.

```c
fn int[] wrong()
{
    int[3] values = { 1, 2, 3 };
    return values[..]; // wrong
}
```

### 11.3. Heap Allocation

C3 provides functions close to C, such as `malloc`, `calloc`, and `free`. However, it is usually better to use typed standard library functions when possible.

Conceptual example:

```c
fn int[] create_array(int n)
{
    int* data = malloc(n * int::size);

    for (int i = 0; i < n; i++)
    {
        data[i] = i;
    }

    return data[:n];
}
```

Usage:

```c
fn void main(String[] args)
{
    int[] values = create_array(10);

    // use values...

    free(values);
}
```

More idiomatic version:

```c
int[] values = mem::new_array(int, n);
defer mem::free(values);
```

### 11.4. Using `defer`

To clean up a resource properly, use `defer`.

```c
fn void process()
{
    int[] values = create_array(10);
    defer free(values);

    // use values
    // free(values) will be called automatically when leaving the scope
}
```

`defer` is not a C++ destructor and not full RAII. It is an explicit instruction requesting that an operation be executed when leaving the scope.

It usefully replaces traditional manual cleanup patterns using `goto cleanup`.

### 11.5. Temporary Allocator

C3 provides a temporary allocator for short-lived data. Allocate with `tmalloc`, `tcalloc`, or similar functions, and all temporary memory is freed at once at the end of an `@pool` block.

```c
fn void work()
{
    @pool()
    {
        void* buffer = tmalloc(1024);

        // use buffer here

    }; // all temporary memory from the pool is freed here
}
```

The temporary allocator is useful for:

- parsing text;
- temporarily building a string;
- loading a small JSON document;
- preparing a response;
- manipulating intermediate buffers;
- avoiding a long sequence of `malloc` / `free`.

### 11.6. Safety Rule for `@pool`

Never keep a pointer to a temporary allocation after leaving the `@pool` block.

Bad example:

```c
String global_name;

fn void bad()
{
    @pool()
    {
        String tmp = string::tformat("Name: %s", "Ash");
        global_name = tmp; // dangerous
    };
}
```

Good usage:

```c
fn void good()
{
    @pool()
    {
        String tmp = string::tformat("Name: %s", "Ash");
        io::printn(tmp);
    }; // tmp is no longer used afterwards
}
```

For projects involving JS â C3 communication, JSON parsing, rendering buffers, or string conversions, the temporary allocator is very useful. It does, however, require strict discipline about lifetimes.

---

## 12. Strings

### 12.1. `String`

C3 has a `String` type, defined as a view over a `char[]`. It is therefore essentially a pointer plus a length.

```c
String s = "Hello";
```

As with slices, the underlying memory must remain valid.

### 12.2. `ZString`

For interoperability with C, C3 also provides `ZString`, which corresponds to a zero-terminated C string.

### 12.3. Dynamic Strings

To build strings dynamically, C3 provides types such as `DString`.

The important questions remain:

- who owns the memory?
- who must free it?
- how long does the string remain valid?

---

## 13. Dynamic Lists

### 13.1. Purpose of `List`

To build an array that grows progressively, use `List`.

Example in C3 0.7.x style:

```c
import std::collections;

alias IntList = List {int};

fn void test()
{
    IntList list;
    list.init(mem);
    defer list.free();

    list.push(10);
    list.push(20);
    list.push(30);

    foreach (value : list)
    {
        io::printfn("%d", value);
    }

    int second = list[1]; // 20
}
```

`List` represents a dynamic contiguous collection. It usually owns:

- a pointer to a buffer;
- a length;
- a capacity.

### 13.2. Converting a `List` to a Slice

A `List` can conceptually provide a slice view over its internal buffer. The exact function name depends on the standard library version.

The principle is:

```text
List<T>  -> dynamic owner of contiguous data
T[]      -> ptr + len view
```

A `List` â `T[]` conversion can therefore happen without a copy if it returns a view.

This view becomes fragile if the list is later modified.

```c
IntList list;
list.init(mem);
defer list.free();

list.push(1);
list.push(2);

// imagine a view:
int[] view = list_as_slice(list);

list.push(3); // may trigger reallocation

// view may become invalid
```

The practical rule is: a view over a `List` remains valid as long as the `List` is not reallocated or freed.

To obtain independent durable data, copy it.

---

## 14. Copying Between Slices

### 14.1. Assigning to a Range

C3 allows assigning a value to a range.

```c
int[5] a = { 1, 2, 3, 4, 5 };

a[1..3] = 0;

// a = { 1, 0, 0, 0, 5 }
```

### 14.2. Copying One Range to Another

A range can also be copied into another range.

```c
int[3] a = { 1, 20, 50 };
int[3] b = { 2, 4, 5 };

a[1..2] = b[0..1];

// a = { 1, 2, 4 }
```

Avoid copying between overlapping ranges unless the behavior is explicitly guaranteed.

To move data within the same memory region, use a suitable function such as `memmove`, or write the logic explicitly in the correct direction.

---

## 15. Maps

### 15.1. Purpose of Maps

In C3, a map is not a magical built-in type like in Go. It is a standard library collection.

Conceptually, it is similar to:

```c
HashMap<Key, Value>
```

In C3 0.7.x syntax, one often sees a form like:

```c
HashMap {String, int}
```

Conceptual example:

```c
import std::collections;

alias Scores = HashMap {String, int};

fn void test()
{
    Scores scores;
    scores.init(mem);
    defer scores.free();

    scores.set("Ash", 15);
    scores.set("Padma", 40);

    int ash_age = scores["Ash"];
}
```

Exact details can vary depending on the compiler and standard library version.

### 15.2. Difference Between Maps and Slices

A map is not a contiguous collection like an array or a slice.

An array or slice maps an index to a value:

```text
0 -> "Ash"
1 -> "Padma"
2 -> "Daniel"
```

A map associates a key with a value:

```text
"Ash"    -> 15
"Padma"  -> 40
"Daniel" -> 40
```

There is therefore no natural direct conversion between a map and a slice.

To convert a map into a linear collection, decide what to extract:

- the keys;
- the values;
- the key-value pairs.

### 15.3. Converting a Map to a List of Pairs

To extract a map as a linear collection, define a structure.

```c
struct Entry
{
    String key;
    int value;
}
```

Then build a list:

```c
alias EntryList = List {Entry};

EntryList entries;
entries.init(mem);
defer entries.free();

foreach (key, value : map)
{
    entries.push({ .key = key, .value = value });
}
```

Then, to obtain a slice view, use the API available in the version of `List` being used.

### 15.4. Converting a Map to a Fixed-Size Array

A direct conversion to a fixed-size array is rarely appropriate because the size of a map is known at runtime, not at compile time.

A declaration like this is therefore not suitable:

```c
Entry[map.len] entries;
```

A C3 fixed-size array requires a size known at compile time.

To convert a map into a linear collection, use instead:

- a `List`;
- a dynamically allocated slice.

Conceptual example:

```c
Entry[] entries = mem::new_array(Entry, map.len);
defer mem::free(entries);

int i = 0;

foreach (key, value : map)
{
    entries[i] = { .key = key, .value = value };
    i++;
}
```

### 15.5. Converting a Slice to a Map

To convert a slice to a map, iterate over the elements and insert values one by one.

Example:

```c
struct Person
{
    String name;
    int age;
}
```

To build a `name -> age` map:

```c
fn Scores people_to_scores(Person[] people)
{
    Scores scores;
    scores.init(mem);

    foreach (p : people)
    {
        scores.set(p.name, p.age);
    }

    return scores;
}
```

Pay attention to key memory. If `p.name` points to temporary memory, the map may contain invalid keys.

A map containing `String` values does not necessarily copy the text as a garbage-collected language would. Check the exact behavior of the API being used.

### 15.6. Converting an Array to a Map

To convert a fixed-size array to a map, first pass it as a slice.

```c
Person[3] people = {
    { .name = "Ash", .age = 15 },
    { .name = "Padma", .age = 40 },
    { .name = "Daniel", .age = 40 },
};

Scores scores = people_to_scores(people[..]);
defer scores.free();
```

This approach is preferable:

```c
fn Scores people_to_scores(Person[] people)
```

to this one:

```c
fn Scores people_to_scores(Person[3] people)
```

The first version accepts a `Person[3]`, a `Person[10]`, a dynamic slice, or a subrange.

---

## 16. Summary of Collections and Conversions

### 16.1. Fixed-Size Array to Slice

```c
int[4] a = { 1, 2, 3, 4 };

int[] s = a[..];
```

This conversion does not copy the elements.

### 16.2. Slice to Pointer

```c
int* p = s;
```

This conversion does not copy the data, but it loses the length.

### 16.3. Pointer to Slice

```c
int* p = ...;
int[] s = p[0..9];
```

This conversion does not copy the data, but it is dangerous if the length is wrong.

### 16.4. Slice to Fixed-Size Array

There is no safe automatic conversion. A cast is possible, but it should be avoided unless the memory correspondence is certain.

```c
int[4]* p = (int[4]*)s;
```

### 16.5. Slice to Dynamic Copy

```c
int[] copy = mem::new_array(int, s.len);
defer mem::free(copy);

copy[..] = s[..];
```

This operation performs a real copy.

### 16.6. Fixed-Size Array to Fixed-Size Array

```c
int[4] b = a;
```

This operation copies the elements.

### 16.7. List to Slice

A `List` can provide a view over its internal buffer if the API of the version being used supports it. This view can become invalid if the list is reallocated.

### 16.8. Map to Slice

A map does not convert directly to a slice. Extract the keys, values, or key-value pairs.

### 16.9. Slice to Map

A slice does not automatically convert to a map. Iterate and insert each element.

---

## 17. Practical Collection Style

### 17.1. Functions Receiving Collections

To receive a collection, usually use a slice.

```c
fn void process_items(Item[] items)
{
    ...
}
```

This form is preferable to:

```c
fn void process_items(Item[10] items)
```

unless the function really requires exactly ten elements.

### 17.2. Functions Producing Dynamic Collections

To produce a variable-size collection, return a dynamically allocated slice.

```c
fn Item[] make_items(...)
{
    Item[] items = mem::new_array(Item, n);
    return items;
}
```

The caller must then free the memory.

```c
Item[] items = make_items(...);
defer mem::free(items);
```

### 17.3. Progressive Construction

To progressively build a collection, use a `List`.

```c
List {Item} items;
items.init(mem);
defer items.free();

items.push(...);
items.push(...);
```

### 17.4. Lookup by Key

To retrieve values by key, use a `HashMap`.

```c
HashMap {String, Item} items_by_name;
items_by_name.init(mem);
defer items_by_name.free();

items_by_name.set("Ash", item);
```

### 17.5. Final Mental Model

The essential distinction is:

```text
int[4]  = the data itself
int[]   = a view over data
int*    = an address without length
List    = a dynamic owner of contiguous data
HashMap = a key-value collection
```

---

## 18. Error Handling

### 18.1. General Principle

C3 does not use classic exceptions like Java, C#, Python, or C++. It uses optional results.

A function can return:

- either a real value;
- or an error called a `fault`.

An optional type is written with `?`.

Examples:

```c
int?
char[]?
void?
```

### 18.2. Function That Can Fail

To write a function that can fail, use an optional return type.

```c
fn int? divide(int a, int b)
{
    if (b == 0) return DIVISION_BY_ZERO~;
    return a / b;
}
```

Here, `int?` means: either an `int` or a fault.

### 18.3. Declaring Faults

Errors are represented by values of type `fault`. Custom faults are declared with `faultdef`.

```c
faultdef DIVISION_BY_ZERO, FILE_IS_INVALID;
```

To return a fault, use the `~` suffix.

```c
return DIVISION_BY_ZERO~;
```

### 18.4. Automatic Propagation with `!`

To automatically propagate an error, use the `!` operator.

```c
fn int? compute()
{
    int value = maybe_function()!;
    return value * 2;
}
```

This means: if `maybe_function()` succeeds, retrieve the value; otherwise, immediately return the fault from the current function.

The previous code is conceptually similar to:

```c
fn int? compute()
{
    int? temp = maybe_function();

    if (catch excuse = temp)
    {
        return excuse~;
    }

    int value = temp;
    return value * 2;
}
```

C3 therefore gives a feeling of automatic propagation similar to exceptions, but without hiding the possibility of failure: the return type explicitly shows that the function can fail.

### 18.5. Local Handling with `if (catch ...)`

To catch an error locally, use `if (catch ...)`.

```c
fn void main()
{
    int? result = divide(10, 0);

    if (catch excuse = result)
    {
        io::printfn("Error: %s", excuse);
        return;
    }

    io::printfn("Result: %d", result);
}
```

After an `if (catch ...)` block that exits the current flow with `return`, `break`, `continue`, or `!`, the compiler can treat the remaining value as valid.

### 18.6. `void?` Functions

A function may return no useful value and still fail.

```c
fn void? save_file()
{
    if (problem) return io::FILE_NOT_FOUND~;
    return;
}
```

This form is close to:

```go
func saveFile() error
```

in Go, but it is integrated into C3's type system.

`void?` can be used as a function return type. Depending on the current documentation, it should not be used as an ordinary variable type.

### 18.7. Default Value with `??`

To replace an error with a default value, use `??`.

```c
int value = foo_may_error() ?? -1;
```

This expression means: if `foo_may_error()` succeeds, use its result; otherwise, use `-1`.

`??` can also be used to transform one fault into a more specific fault.

### 18.8. Forced Unwrapping with `!!`

To forcibly unwrap an optional value, use `!!`.

```c
value = function_that_should_never_fail()!!;
```

If the value contains a fault, the program panics and stops.

This operator is useful for prototypes, tests, or cases where failure is truly impossible. In robust code, prefer handling or propagating the error.

### 18.9. Positive Handling with `if (try ...)`

To execute a block only if an optional value contains a real value, use `if (try ...)`.

```c
int? result = maybe_function();

if (try result)
{
    io::printfn("Value: %d", result);
}
```

Inside the block, `result` can be used as a normal value.

### 18.10. Useful Macros

C3 provides macros to reduce repetitive code.

```c
fault excuse = @catch(optional_value);
bool ok = @ok(optional_value);
```

`@catch` retrieves the fault without performing the automatic unwrapping of `if (catch)`.

`@ok` simply indicates whether a value is present.

Some recent versions also add macros such as:

- `@try`;
- `@try_catch`.

These macros simplify cases where an expected error must be handled locally while other errors are propagated.

---

## 19. Files: Opening, Reading, Writing, and Closing

### 19.1. General Principle

File handling in C3 is close in spirit to C, but with three important improvements:

1. errors go through optionals;
2. errors can be propagated with `!`;
3. resources can be closed cleanly with `defer`.

The basic API is located in:

```c
std::io
std::io::file
```

The central function is:

```c
file::open(filename, mode)
```

It returns a `File?`.

Closing is done with:

```c
file.close()
```

### 19.2. Opening and Closing a File

To open a file, use `file::open`.

```c
import std::io;
import std::io::file;

fn void? test_open()
{
    File f = file::open("notes.txt", "r")!;
    defer (void)f.close();

    io::printn("File opened successfully.");
}
```

The opening mode is a C-style string:

- `"r"` for reading;
- `"w"` for writing;
- `"a"` for appending;
- `"rb"` for binary reading;
- `"wb"` for binary writing.

The following instruction guarantees that the file is closed when leaving the block:

```c
defer (void)f.close();
```

The `(void)` cast is used to ignore a possible error returned by `close`.

### 19.3. Reading into a Buffer

To read into a buffer, open the file, prepare a slice, and call `read`.

```c
import std::io;
import std::io::file;

fn char[]? read_some(String filename)
{
    char[] buffer = mem::new_array(char, 100);
    defer free(buffer);

    File f = file::open(filename, "r")!;
    defer (void)f.close();

    sz n = f.read(buffer)!;

    return buffer[:n];
}
```

The `File.read(buffer)` method reads into a `char[]` slice and returns the number of bytes read.

In this example, the `!` operator automatically propagates any possible error.

### 19.4. Handling a Read Error

To call the previous function, catch the error with `if (catch ...)`.

```c
fn int main()
{
    char[]? data = read_some("notes.txt");

    if (catch err = data)
    {
        io::printfn("Read error: %s", err);
        return 1;
    }

    io::printfn("Read content: %s", (String)data);
    return 0;
}
```

### 19.5. Reading a Whole File

To read an entire file, the standard library provides helpers such as:

- `file::load`;
- `file::load_temp`;
- `file::load_buffer`.

For a small tool or a prototype, `file::load_temp` is convenient.

```c
import std::io;
import std::io::file;

fn int main()
{
    char[]? data = file::load_temp("notes.txt");

    if (catch err = data)
    {
        io::printfn("Unable to read file: %s", err);
        return 1;
    }

    io::printfn("Content: %s", (String)data);
    return 0;
}
```

For more controlled code, use an explicit allocator.

```c
char[] data = file::load(mem, "notes.txt")!;
defer free(data);
```

### 19.6. Writing to a File

To write to a file, open the file in write mode, write the data, then automatically close it with `defer`.

```c
import std::io;
import std::io::file;

fn void? save_text(String filename, char[] data)
{
    File f = file::open(filename, "w")!;
    defer (void)f.close();

    f.write(data)!;
}
```

Usage:

```c
fn int main()
{
    if (catch err = save_text("output.txt", "Hello from C3\n"))
    {
        io::printfn("Write error: %s", err);
        return 1;
    }

    return 0;
}
```

### 19.7. Writing a Whole File with `file::save`

To write a whole file at once, use `file::save`.

```c
import std::io;
import std::io::file;

fn int main()
{
    if (catch err = file::save("output.txt", "Hello\nWorld\n"))
    {
        io::printfn("Unable to write: %s", err);
        return 1;
    }

    return 0;
}
```

### 19.8. Idiomatic File Pattern

The general pattern is:

```c
File f = file::open("file.txt", "r")!;
defer (void)f.close();

// read or write here
```

This pattern means:

1. open the file;
2. propagate the error if opening fails;
3. automatically schedule closing;
4. read or write;
5. leave the scope and close the file cleanly.

Summary of common operations:

```c
file::open(...)!     // open or propagate the error
defer f.close()      // close automatically
f.read(buffer)!      // read or propagate the error
f.write(data)!       // write or propagate the error
file::load(...)      // read a whole file
file::save(...)      // write a whole file
```

---

## 20. C Interoperability

### 20.1. Calling C Functions

C3 is designed to integrate well with C. A C function can be declared with `extern fn`.

```c
extern fn int puts(char* text);

fn void main(String[] args)
{
    puts("Hello from libc");
}
```

This compatibility is especially useful for:

- calling C libraries;
- writing native bindings;
- accessing libc;
- using system APIs;
- building lightweight wrappers;
- integrating C3 into WebView or desktop projects.

### 20.2. Usage Model

C3 is particularly suitable when one wants to write a performant native layer without moving to the complexity of C++ or the strict discipline of Rust.

---

## 21. Comparison with C, Go, Zig, and Rust

### 21.1. Compared with C

C3 is more pleasant than C thanks to:

- modules;
- slices;
- optionals;
- better initialization;
- `defer`;
- contracts;
- more structured macros.

However, it keeps a low-level model close to C.

### 21.2. Compared with Go

C3 is lower-level than Go.

It has no garbage collector and gives more direct control over memory. It is therefore better suited for a small, performant native layer, but less comfortable for large conventional web services.

### 21.3. Compared with Zig

C3 is often closer to C in style.

Zig enforces more discipline through explicit allocators almost everywhere. C3 offers a compromise: allocators are available, but the syntax remains more familiar to C programmers.

### 21.4. Compared with Rust

C3 does not try to guarantee memory safety through a borrow checker.

It remains a manual memory management language. It is therefore easier to approach than Rust, but it protects the programmer less.

---

## 22. Practical Mental Model

### 22.1. Core Rules

To write clean C3 code, use the following rules:

1. Use the stack when the lifetime is local.
2. Use slices to pass arrays together with their length.
3. Prefer `mem::new_array` and `mem::alloc_array` to raw `malloc` when possible.
4. Use `defer mem::free(...)` to avoid leaks.
5. Use `@pool` and temporary memory for short-lived data.
6. Never return a view pointing to local or temporary memory.
7. Always ask: who owns this memory?
8. For long-lived resources such as files, sockets, or durable buffers, provide a `free`, `destroy`, or `close` function.

### 22.2. Stack, Heap, and Temporary Memory

The most useful rule is:

```text
stack for local data
heap + free for durable data
tmem + @pool for temporary data
```

This distinction is probably the key to writing clean and reliable C3 programs.

---

## 23. Complete Example: Durable Allocation

```c
module demo;

import std::io;

fn int[] make_numbers(int n)
{
    int[] values = mem::new_array(int, n);

    for (int i = 0; i < n; i++)
    {
        values[i] = i * 10;
    }

    return values;
}

fn void main(String[] args)
{
    int[] numbers = make_numbers(5);
    defer free(numbers);

    foreach (n : numbers)
    {
        io::printfn("%d", n);
    }
}
```

In this example:

- `make_numbers` allocates memory on the heap;
- `main` becomes responsible for that memory;
- `defer free(numbers)` guarantees that the memory is freed.

---

## 24. Complete Example: Temporary Allocation

```c
module demo;

import std::io;

fn void main(String[] args)
{
    @pool()
    {
        String message = string::tformat("Hello %s", "Serge");
        io::printn(message);
    }; // message is automatically freed here
}
```

In this example, the string is temporary. It is valid only inside the `@pool` block.

---

## 25. Practical Conclusion

C3 is interesting when one wants to write low-level code without the complexity of C++ and without Rust's strict mental model.

For small WebView frameworks, JSON parsing, native bindings, lightweight desktop tools, and system utilities, C3 is coherent and practical.

However, C3 should not be treated as a âsafe languageâ in the Rust sense. It improves on C, but it remains a manual memory management language. Discipline is still necessary.

The real advantage is that C3 provides better tools for disciplined programming:

- slices;
- optionals;
- `defer`;
- temporary allocators;
- stricter conventions;
- cleaner interoperability with C.

The essential habit is:

```text
stack for local data
heap + free for durable data
temporary allocator + @pool for short-lived data
```
