# Basic C Programming (Chapter 1 - Chapter 10)
## Chapter 1
### 3.1 History of C
- C is a bi-product of the UNIX operating system.
- UNIX was written in assembly language. Thompson designed B language for the sake of debugging simplicity comparing to assembly language. Later the language is extended and upgraded by Ritchie and renamed it to C.
- American National Standards Institute (ANSI) standardized the C language in 1989, and later ISO approved it in 1990. This version of C is referred to as C89 or C90.

### 3.2 Characteristics of C
- C provides access to machine-level concepts (bytes and addresses).
- C provides operations that correspond closely to a computer's built-in instructions (chdir).
- C assumes you know what you're doing so it doesn't do error checking automatically at the most of time.

### 3.3 How to write a good C program
- Lrean common pitfalls (best practicies)
- Lrean using debuggers like GDB/Valgrind
- Set the compiler to have more warning levels
- Don't reinvent, use libraries
- Adopt coding convension
- Don't force the code to be concise. Shorter version is often the hardest to comprehend.
- Stick to the standard

## Chapter 2. C Fundamentals
- **Directives** are the statements meant to be processed by the *preprocessor*.
- Both `printf` and `scanf` deal with *formatted* data. `printf` prints the value following the format and `scanf` gets an input that's in the expected format. For example, `scanf("%d", &i)` reads an integer from the input stream and saves it in the buffer `i`.
- Common *cc* options are `-Wall` or `-W[code]` to produce warning message. It's recommended to be used with `-O` option. `-pendantic` rejects programs that use non-standard features. `-std=version` sets the C standard version to use. `-g` enables debugging. 
- `exit(0)` is effectively as same as `return 0`, they both return 0 to the operating system.
- Old C compilers removed all the comments when compiling an C source code. However, modern compilers are required by the C standard (C99) to replace the comment line with a single space character.
- A decimal number must end with `f` in order to be a `float` type otherwise compiler will treat it as a `double` value.

## Chapter 3. Formatted Input/Output
In this chapter we deal with two built-in functions `printf` and `scanf`. They are reffered to as **formatted writing** and **formatted reading**.

### 3.1 The `printf` function
The `printf` function is designed to write the contents of a **string**, known as the **format string**, with values possibly inserted at specified positions in the string.
```
printf(format_string, expr1, expr2, ...);
```
```
What is a stream?
The concept of a stream is closely releated to a FILE. A FILE is nothing more than a continues sequence of bytes. A stream is an absraction of data flow; it has a source (input stream) or a sink (output stream). When we read or write to a FILE, we first obtain the handle to the FILE then we write/read the contents in/out in the form of a stream. 
```
The format string may contain both ordinary characters and **conversion specifications**, which begin with the **%** character. A conversion specification is a place-holder representing a value to be filled in during writing.

The letter followes the % character specifies how the value is *converted* from its internal form (bytes) to printed form (characters)--that's where the term "conversion specification" comes from.

Note that, C compilers aren't required to check that the number of conversion specifications in a format string. The following call of `printf` is valid. The format string checker can be turned on by `-Wall` or `-Wformat=1` gcc flags.
```c
printf("%d %d", i); // the string written by the 2nd place holder becomes uncertain in this case
```
