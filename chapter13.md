Chapter 13: Storage Classes and Qualifiers
=======
We have seen that storage in C takes the form of variables declared in C programs.  Over the lifetime of a C program, the specific locations that variables are declared and where they are referenced in a program will determine where they are stored in memory and how long they exist.  This "lifetime" of variable storage is something that can be controlled by a programmer.

This chapter will discuss storage and ways to control how storage is implemented for programs.  This control will take the form of special types of declaration syntax that will describe how storage is to be used.  We will be describing two types of syntax: *storage classes* and *storage qualifiers*.  We will be defining the differences between these and how they can be used effectively.  We will conclude with a brief look at storage declarations in Pebble programs.

### The Lifetime and Accessibility of Storage ###

Variable storage, that is, memory usage, has a lifetime in a C program.  The idea of variable lifetime goes together with the idea of variable name accessibility.  We have considered this before: global variables are accessible throughout a C program; local variables are only accessible inside the block in which they are declared.  It would make sense, then, for variables to be allocated only when they are accessible.  Allocating memory for every variable when a program starts would be very inefficient.

In addition, program semantics demand that variables are allocated when the block in which they are declared is entered.  Consider recursive functions.  When a function calls itself, the newly called function instance needs a new set of variables allocated, separate from the caller's variables, even if they are the same function and have the same set of declarations.

This management of memory is achieved by the use of *activation records* or *stack frames*.  When a function is called, its declared memory requirements are allocated in a group, together with any other data items needed to run the function (for example, the place in the code to return to when the function completes).  This group becomes the activation record and is pushed onto a system stack.  The record at the top of the stack is always the one used for the function currently being executed.  When a function calls another function, a new activation record is formed and pushed.  When a function is completed its activation record is popped and the memory is reused.

Consider an integer variable declared in the outermost block of a C program.  This storage will be allocated when a program begins execution and deleted when a program is terminated.  This means that there is always at least one activation record pushed onto the stack, that of the outermost, global block.  

Consider local variables declared inside a function's block.  Part of the overhead of a function call is the creation of storage in the AR and the manipulation of the stack.

One exception to this rule is the class of dynamically allocated storage.  Memory that is dynamically allocated is referenced by pointer variables that exist in an activation record.  However, while these pointer variables exist in the AR, the actual memory is allocated in a different area of memory called a *heap*.  Activation records are chunks of memory that are fixed in size; their size can be computed by the compiler from the source code.  This means that ARs can be pushed onto the stack in fixed amounts.  Dynamic memory, however, is not fixed in size; the amount of dynamic memory cannot be determined by analyzing a program's source code. Using a separate area of memory is the best way to accomodate the changing needs of dynamic memory.

Using a heap also has accessibility implications.  Because the entire heap is accessibile to all program code, dynamic memory is availble to all parts of an application.  Note, however, that, because dynamic memory is only available through pointer varaibles, and pointer variables have accessibility restrictions, dynamic memory is also restricted.  

We can control the lifetime and accessiblity of variables and memory through storage classes and descriptors.

### The "auto" Storage Class ###

The *auto* storage class is the storage class for all local variables.  It is the default storage class for variables without an explicit storage class declared.  Variables exist in the auto storage class with or without explicit declarations.

The term "auto" refers to storage that is automatically allocated when the block surrounding the variables is entered.  This refers to the activation record method of variable storage.  

For example, consider this example:

    void collect() {
        int a, b, c;
        auto float x, y;
        float z;
        
        ...
    }
    
Each of these variables are local to the `collect` function and are in the auto storage class.  Some are explicitly declared as "auto"; all are in the auto class.  They will be automatically allocated in the function block's activation record and pushed onto the system stack while the function is executing.   

### The "static" Storage Class ###

Variables in the *static* storage class are allocated just before a program begins execution and are maintained throughout the lifetime of the program.  While these might seem like global variables, there are a few subtle differences between static and other types of variables:

* Static variables are allocated before program code is executed.  Variables global to the main program are created in an activation record and pushed onto the system stack.  Since static variables are allocated first, they are less restrained by memory restrictions.
*  Static variables are available to all program components, even those that exist externally.  
*  Static variables exist throughout the lifetime of a program.  It is possible to make local variables static; this allows them to keep their values between function calls.

The last point above needs some examination.  Consider the example below:

    void shout() {
       static int counter = 10;
       i++
       printf("counter is %d\n", counter);
    }
    ...
    for (int i=0; i<5; i++) shout();

Here, the local variable `counter` is initialized *once*, even though memory space for the function `shout` is created every time it is called.  The output looks like this:

    counter is 11
    counter is 12
    counter is 13
    counter is 14
    counter is 15
    
If the "static" declaration was left off, the counter would be initialized to 10 at every call and every line would read `counter is 11`.

### The "extern" Storage Class ###

C programs can exist in multiple files, each of which contains code and definitions (hopefully organized to belong together).  These are combined when the executable code is being generated.  Sometimes, definitions that are given in one file are needed in another.  

These types of declarations, those that are required but external, are declared as "extern".  Such external definitions are considered global to the code being defined, while contained in the file of the declaration.  Both variable and function declarations can be declared as "extern".

Consider this example:

    extern int distance;
    extern char *replace(char *string, char *original, char *replacement);
    
Here, the variable `distance` is actually declared in an external file.  This might seem like a redundant declaration, seeing as this declaration specifies that `distance` is an integer, but this variable is now shared between two files: both global variables, both actually the same location in memory.  This type of declaration is convenient for organizational purposes.

The function `replace` in the above example is given in prototype form and the `extern` keyword specifies its definition is in a separate file.   Without the `extern` keyword, this prototype declaration would require the function definition to be given later in the same file.  However, here the definition *is assumed* to be in another file.

Note that the compiler *assumes* an external definition exists in an external file, but does not check to see if it actually exists.  When the executable is being built from all the files that make up the application, the linker will check to see if all definitions have been given.  An error will occur at that stage if the external definitions are not given in some file.

### The "register" Storage Class ###

Sometimes it is useful to require that a variable be stored in a register instead of memory.  This is desirable because register access is faster than memory access.  Placing the keyword "register" before a declaration specifies that the variable should be stored in a register.

For example, consider this code:

    register int xaxis, yaxis;
    int zaxis;
    
Here, two variables are specified as stored in a register.  The third variable is simple stored in memory.

Register declarations can seem to be ver usefule.  However, they restrict how variables can be allocated and used.

* Variables allocated to a register can only be a large as a register, usually a single word.  For example, "double" size variables cannot be allocated to registers.
* Variables allocated to registers cannot be used with a unary "&" operator.  Recall that this operator gives the address in memory of storage.  Variables stored in registers have no memory address.

In addition, compilers are free to ignore the "register" directive.  This means that, while the above restrictions are enforced, a "register" variable *might* be stored in a register.  Or not.  And a compiler might store a variable in a register *without* the "register" request, depending on how it is used.

### The "const" Storage Qualifier ###

The "const" keyword in declarations does not declare a storage class, but rather a way use the declared variable.  If a variable is declared with the "const" keyword, it can be assigned a value *once*, in the declaration, and it cannot be changed again.  

Consider this example:

    const int distance = 532;
    const float pi = 3.14159;
    
    distance = 561;
    
Here we have two constants, initialized once.  Any further attempt to change the value of these variables will result in a compiler error.  The last line will be trapped by the compiler with the error below:

    ../src/test.c:51:5: error: assignment of read-only variable 'distance'

Note that program code can change variable values in subtle and unexpected ways.  Assignment statements are obvious ways that variable can change values and these statements would not be allowed.  Variables can change values through memory references.  Consider the following declarations:

    const int distance = 10;
    int *dist = &distance;
    *dist = 15;

The compiler will not allow the pointer to address constant memory.  It give an error on the above code at the pointer delcaration:

    ../src/test.c:47:17: error: initialization discards 'const' qualifier from pointer target type [-Werror]

The "const" keyword is also useful in function parameter declaration.  When used with parameter declaration, it sepcifies that the parameter may not be changed.  This may seem superfluous, since C uses call-by-value semantics to pass parameters.  However, it is especially useful for pointers in parameter lists; this use says that the memory referenced by the pointer cannot be changed.  

Consider this example:

    void load_memory(const char *location, char *mem) { ... }
    
Let's assume this function uses the value of `location` to find data that it loads into the memory pointed to by `mem`.  We could call this function like this:

    char *loc = "details.txt";
    load_memory(loc, memory);
    
But we could also call this function like this:

    load_memory("details.txt", memory);
    
Using "const" in the parameter declaration guarantees no changes to the memory pointed to by hte first parameter.  This means we can use a string literal as the first parameter.  If we removed the "const" declarator, we could only call this function using the first method, because we could not guarantee that changes to memory would not occur.

### The "volatile" Storage Qualifier ###

The "volatile" keyword is a qualifier that, like "const", is used when a variable is declared. It tells the compiler that the value of the variable may change at any time.  This may seem obvious, we are declaring a *variable* after all, but this keyword implies that changes may happen *at any time*, even outside code that finds this variable accessible.

This situation occurs when the program being compiled interfaces with the hardware of the computer.  Often, when these kinds of interfaces occur, the system connects memory/variable storage to a register that represents a device interface.  When that device has data that a program needs, it moves that data to the connected register, which shows up to the program as a variable that has the same value as that register.

The compiler uses information of the volatility of data to *not* optimize or check variables.  When a compiler optimizes code, it can change the order of statements or remove variable storage altogether if these changes do not affect the outcome of the code.  If a variable's value is changed by the device data rather than the code being compiled, such optimization should not be done.  Declaring such a variable as "volatile" will inform the compiler to treat the variable as special and not optimize code around it.

Let's take a simple example.  Consider this code:

    int flag = 0;
    while (flag == 0) { ... }
    
If we assume that the `flag` variable is not changed by the code of the while loop, the compiler will likely remove the equality check, replacing it with a simple `while (true) { ... }` substitute.  If, however, `flag` was tied to some hardware register that changes outside the control of the program, removing the equality check would not be recommended.  Instead, we should change the `flag` declaraion:

    volatile int flag = 0;
    while (flag == 0) { ... }
    
Now the compiler will leave the equality check alone and it will behave as we expect.

### The "__attribute__" Qualifier of GNU C ###

Pebble SDKs use the Gnu C compiler (GCC) to compile code for its smartwatches.  The GCC recognizes a keyword that serves to instruct the compiler about how to generate code about storage structures.  While this is unique to the GCC, it fits in this chapter because it is focused on storage.

The "__attribute__" keyword instructs the compiler to manipulate types in specific ways.  The keyword is followed by an attribute specification inside double parentheses.  There are many attributes, even special ones that apply only to specific CPU architectures.  

In the Pebble SDK, the attribute that is used most often is "__packed__".  The "__packed__" attribute specifies that each member (other than zero-width, padding, bit fields) of the structure or union being declared is placed to minimize the memory required. This means no padding is inserted by the compiler to fit better in memory (such as aligning with memory words).  This can be used with an with a declaration; it then indicates that the smallest usable type should be used for values.

For example, the next chapter will consider data from the accelerometer sensor.  That data is structured with the `AccelRawData` struct, shown below:

    typedef struct __attribute__((__packed__)) {
      int16_t x;
      int16_t y;
      int16_t z;
    } AccelRawData;

By specifying the structure is "__packed__", the designer means to have these data items placed directly next to each other in memory, even in the same memory word.  The compiler might decide to place each of these 16-bit items in their own 32-bit memory space, but the "__packed__" attribute stipulates that these items be packed together as tightly as possible.   In this case, it is likely they will end up in 2 memory words.

### Storage Designators in Pebble Smartwatch Programs ###

There are some storage classes and qualifiers that are common in Pebble Smartwatch programs.

Much of the storage in Pebble programs fall into the "auto" storage class.  Most storage does not need special consideration or handling.

In most Pebble programs, you will see is that many global definitions of Pebble system structures are declared as "static".  Because static storage is allocated before a program starts execution and static variable storage is shared between all code files that are used for an application, static storage is the best storage to use with system variables.  Without static storage, variables global to each code file would be allocated as program execution begins and would quite likely duplicate storage in other code files.

Looking at the declaration of system data structures, you will see that most are declared with a "__packed__" attribute.  As an example, consider the `AccelRawData` structure in the previous section.  Packing data structures makes sense since it is best to be as efficient with memory as possible, which means leaving as little space unused as possible.

Larger Pebble programs can be nicely organized into a set of files, necessitating the use of the "extern" storage class.  Organizing code into files focused on functionality is a good way to organize.  This means that declarations using external storage should be used to facilitate the use of code from other files.

