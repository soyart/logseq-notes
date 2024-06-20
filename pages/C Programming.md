- > Note: the notes are tested on macOS clang on M3 Pro
- # C types
	- > The standard (as per K&R) only specifies a few types
	- `int` (see also [[Integers]])
		- The standard does not guarantee size of `int`, but most implementations follow the machine's word size.
		- 2 size modifiers `short` and `long` applies to `int`
			- We can omit `int` when used with these 2 modifiers, i.e. `short int i = 7;` statement is identical to `short i = 7;`
			- On modern 64-bit architectures, `short` is 16-bit, `int` 32-bit, and `long` is 64-bit
		- 2 [sign](((66535f02-198f-4a35-9ae9-3a69721729ad))) modifiers `signed` and `unsigned` are also available for `int`, and plain `int` is always `signed int`.
		- The standard allows C compiler to choose **sizes** for `short`, `int`, and `long` for their hardware however they want, but with 3 rules:
			- `int` must be at least 16-bit
			- `long` must be at least 32-bit
			- `short` <= `int` <= `long`
		- If we want to set/mask bits in a portable way, we can take advantage of [C type conversion](((6659e12f-ed51-4b62-ac5a-f34008127a90)))
			- For example, if we want to set in `x` the leftmost 6 bits to zero, we can do:
			  ```c
			  x & ~077 // 077 is:  000111111
			           // ~077 is: 111000000
			           // So the code always sets the 6 leftmost bits to 0, regardless of x's length
			  ```
			- Using `~077` is great, because we don't have to assume length of `x`
			- Since `077` is a constant, C will (at compile time) promote it to whatever `x` is according to its conversion rule
			- It would then apply the `~` operator, which flips all bits in the promoted value
	- `char`
		- > The standard library provides `ctypes.h` for working with C characters. It defines functions such as `isdigit`, `isspace`, `isupper`, `tolower`, etc.
		- `char` is a type meant for holding a character, usually 1 byte in size
		- `char` can be operated with arithmetics with other ints
		- The standard defines 3 separate types for `char`, namely `unsigned char`, `signed char`, and `char`.
		  id:: 6658bc6b-ee7c-4d11-8c97-15b80f0c8220
			- On gcc, `char` is by default signed. This allows us to use non-ASCII values to encode something else, like (stdio EOF) `-1` which is i32 on Apple M3. This behavior is unlike `int` - an `int` declaration without anything equivalent to `signed int`
			- To find out whether a plain `char` is signed or unsigned in an implementation, use `limits.h`:
			  ```c
			  #include <limits.h>
			  #include <stdio.h>
			  
			  int main(void)
			  {
			      printf("CHAR_MIN %d\n", CHAR_MIN);
			      printf("CHAR_MAX %d\n", CHAR_MAX);
			      printf("CHAR_BIT %d\n", CHAR_BIT);
			  
			      printf("SCHAR_MIN %d\n", SCHAR_MIN);
			      printf("SCHAR_MAX %d\n", SCHAR_MAX);
			  }
			  ```
			  
			  LLVM and gcc output (Apple M3):
			  ```
			  CHAR_MIN -128
			  CHAR_MAX 127
			  CHAR_BIT 8
			  SCHAR_MIN -128
			  SCHAR_MAX 127
			  ```
		- Because `char` can be signed or unsigned, and implementations differ across machines, conversion from `char` to integers might produce negative integers (even if all printable characters are positive)
			- On some machines, if a `char`'s leftmost bit is set to `1`, then converting it to integers might produce negative values
			- On other machines, a `char` is first promoted to integers by padding zeroes to the left (sign extension), and thus only producing positive value
		- This is why we might see small variables assigned to `unsigned char` - it's equivalent to a byte, or `u8` in Rust and `uint8` in Go
	- `float`
		- Single-precision floating point data type, usually 32-bit
	- `double` (double-precision floating point)
		- Double-precision floating point type.
		- Modifier `long` can be applied to produce a `long double`, which offers greater precision than a `double`, but the standard only requires `long double` to be *at least* as precise as `double`
	- ## Size (width) of C types
	  id:: 6658bded-c248-4077-b601-29a3225ccb23
		- On Apple M3 (arm64) using clang, here's the sizes of each types:
		  ```c
		  #include <stdio.h>
		  #include <stdlib.h>
		  
		  long countw(void);
		  
		  int main(void)
		  {
		      printf("size of short int in bytes: %ld\n", sizeof(short int));
		      printf("size of int in bytes: %ld\n", sizeof(int));
		      printf("size of long int in bytes: %ld\n", sizeof(long int));
		      printf("size of char in bytes: %ld\n", sizeof(char));
		      printf("size of float in bytes: %ld\n", sizeof(float));
		      printf("size of double in bytes: %ld\n", sizeof(double));
		      printf("size of long double in bytes: %ld\n", sizeof(long double));
		  }
		  ```
		  LLVM output on Apple M3:
		  ```
		  size of short int in bytes: 2
		  size of int in bytes: 4
		  size of long int in bytes: 8
		  size of char in bytes: 1
		  size of float in bytes: 4
		  size of double in bytes: 8
		  size of long double in bytes: 8
		  value of EOF -1
		  size of EOF in bytes: 4
		  ```
	- ## Type conversion
	  id:: 6659e12f-ed51-4b62-ac5a-f34008127a90
		- Floating point <-> integer conversions are rounded
		- `float` <-> `double` is implementation-dependent
		- ### No `unsigned` involved
			- Promotion from *narrower* types to *wider* types
				- > Exception: `float` are not automatically promoted to `double` to minimize memory
				- `char` is a small integer, so it can be promoted to integer types and participate in arithmetics as integers
					- We know that [`char` can be signed or unsigned](((6658bc6b-ee7c-4d11-8c97-15b80f0c8220))), so the conversion from `char` to integer types are quite tricky and machine-specific
				- In `float` and `int` arithmetics like `f+i`, the integer is promoted to `float` and the arithmetics are done on `float`
			- Lossy conversion may be allowed, but some warnings will be made
			- Nonsense conversion, like using a floating point as array index, is outright illegal
		- ### With `unsigned`
			- Generally *unsigned integer types of identical width are considered to be wider* than their signed counterparts
			- Behaviors are machine-dependent, due to the fact that it depends on integer types on the hardware
			- > Assume `int` is 16-bit and `long` is 32-bit.
			- `-1L < 1U`
				- Signed long `-1L` ints are wider than unsigned int `1U`.
				- `1U` is promoted to *signed long* `1L`
				- So this expression actually tested `-1L < 1L`
			- `-1L > 1UL`
				- `-1L` will be promoted to `unsigned long`
				- So `-1L` will have a `1` as its leftmost bit (from its previously signed bit, which, after conversion, is treated as a magnitude bit)
		- ### In assignments
			- When conversion is needed in assignments, the righthand side of the assignment is converted to the lefthand side and may cause data loss and overflows
			- #### Integers
				- When converting wider integers to narrower ones by dropping higher-order bit
				- In this snippet, `c` value is unchanged (lossless) because `int` is wider than `char`:
				  ```c
				  char c = 'a';
				  int i;
				  
				  i = c;
				  c = i;
				  ```
				- Reversing the order of assignments might lose data:
				  ```c
				  #include <stdio.h>
				  
				  int main(void)
				  {
				      int i = 300;
				      char c = 0;
				  
				      printf("i init %3d %o\t%x\n", i, i, i);
				      printf("c init %3d %o\t%x\n", c, c, c);
				  
				      c = i;
				      i = c;
				  
				      printf("i done %3d %o\t%x\n", i, i, i);
				      printf("c done %3d %o\t%x\n", c, c, c);
				  }
				  
				  ```
				  ```
				  i init 300 454	12c
				  c init  0  0	 0
				  i done 44 54	2c
				  c done 44 54	2c
				  ```
				- Or overflows:
				  ```
				  i init 256 400	100
				  c init   0 0	0
				  i done   0 0	0
				  c done   0 0	0
				  ```
				  ```
				  i init 255 377	ff
				  c init   0           0         0
				  i done  -1 37777777777	ffffffff
				  c done  -1 37777777777	ffffffff
				  ```
		- Explicit conversions have the following form:
		  ```
		  (type_name) expr
		  ```
		  So, if we have `double sqrt(double);` definition, and a `float`, then  we can call `sqrt` with:
		  ```c
		  // Like 'sqrt(float64(f))' in Go
		  // The value of f is converted to a double,
		  // and sent to sqrt. f itself did not change.
		  sqrt((double) f)
		  ```
		  Note that we might not have to do this if we have function prototype for `sqrt`, since function prototypes will coerce C to convert argument values to proper types as defined in the function's prototype.
- # C functions
	- > All C functions are *external* (like external variables)
	- ## Caveat
		- If more than 1 functions are called in an expression, the order of these calls is undefined
		- So in this snippet:
		  ```c
		  int i = 8;
		  int result = foo(&i) + bar(&i)
		  ```
		  We have no way of knowing if `foo` or `bar` is going to be called first, and risk bugs when both functions modify the same value.
		- To ensure order of execution, use a temporary variable:
		  ```c
		  int i = 8;
		  int result = foo(&i);
		  result = bar(&i);
		  ```
	- ## Parameters
		- If we have this declaration:
		  ```c
		  int f();
		  ```
		  The declaration *does not* say that `f` takes no arguments. **Instead, when omitted, parameters are not type-checked at all when we call it somewhere else**.
		- To declare that `f` takes absolutely no argument, you'll have to explicitly declare void parameter:
		  ```c
		  int f(void);
		  ```
		- By explicitly typing out the parameter types, the compiler should be able to see if some expression calls `f` with variables of invalid types, i.e. for `void` the compiler guarantees that it'll produce an error if `f` is called with any argument
	- ## Return values
		- **Default return type is int**, unless explicitly stated.
			- If the function definition does not explicitly state that a function returns nothing (e.g. `void f();`), then its return value is inferred to be `int`
			- We can omit the `return` statement:
			  ```c
			  int f(void);
			  int f(void) {};
			  ```
			  Here, the return value from calling `f` is some garbage unless we explicitly returns a value:
			  ```c
			  int f(void);
			  int f(void) { return 9; };
			  ```
		- Values in the expression following `return` are implicitly converted to the function's return type
			- ```c
			  int f(unsigned char c);
			  int f(unsigned char c) {
			      return c;
			  }
			  ```
			  Here, the return value of calling `f` is `unsigned char c` converted to signed `int` (only the value, note that `c` itself is unchanged).
			  Or we can explicitly convert it:
			  ```c
			  int f(unsigned char c);
			  int f(unsigned char c) {
			      return (int) c;
			  }
			  ```
		- We can declare functions along with variables in 1 statement, for example the statement
		  ```c
		  double sum, atof(char s[]);
		  ```
		  declares 2 names: `double sum` and `double atof(char s[])`. This means that we can do something like:
		  ```c
		  sum += atof("21.8")
		  ```
	- ## Functions defined in other files
		- Let's say our `main.c` program uses functions defined in `1.c` and `2.c`, then, if we run:
		  ```sh
		  cc 'main.c' '1.c' '2.c'
		  ```
		  The C compiler will produce 3 *object code* files: `main.o`, `1.o`, and `2.o`, and then it would load them all into a single executable `a.out`
	- ## Visibility of external variables
		- Functions can see external variables if their declarations appear before the function definition.
		- In this example, `main` does not see `c`, but `f` does:
		  ```c
		  char f();
		  
		  int main(void)
		  {
		      int i = f();
		    	int c_int = c; // Compile error! no such variable c!
		  }
		  
		  char c = 'c';
		  char f()
		  {
		      return c;
		  }
		  
		  ```
		- Even if we put declaration of `char c;` in `main`, that `c` is not the same as external variable c:
		  ```c
		  #include <stdio.h>
		  
		  char f();
		  
		  int main(void)
		  {
		      char c;
		      int i = f();
		  
		      printf("main.c: %d\n", c);
		      printf("main.i: %d\n", i);
		      printf("i == c: %s\n", i == c ? "true" : "false");
		  }
		  
		  char c = 'c';
		  
		  char f()
		  {
		      return c;
		  }
		  ```
		  ```
		  main.c: 0
		  main.i: 99
		  i == c: false
		  ```