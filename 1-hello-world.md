## 🧪 C3 by Example: Hello World

The classic first program in any language. This shows how to print to the terminal using C3's standard I/O library.

`📄 Source Code: hello_world.c3`

```
import std::io;

fn void main() {
    io::printn("Hello, World!");
}
```

### 🧠 Explanation

`import std::io;`
Imports the standard I/O module to use output functions.

`fn void main()`
Declares the main function as the program's entry point with no return value.

`io::printn("Hello, World!");`
Prints a string followed by a newline. This is equivalent to puts in C or println! in Rust.

### ▶️ Run the Example

Compile and run the program with the C3 compiler:

`c3c compile-run hello_world.c3`

### Expected Output:
> Program linked to executable 'hello_world'
> Launching hello_world...
Hello, World!

### 🔖 Note
- In C3, main must be explicitly declared with void return type.
- `io::printn` is newline-appending. Use io::print to omit the newline.
You can use module path shortening: io::printn instead of full std::io::printn.