Chapter 8: Pointers and Memory Allocation
=======
We have discussed many abstractions that are built into the C programming language.  Most of these abstractions intentionally obscure something central to storage: the address in memory where something is stored.  Pointers are a way to get closer to memory and to manipulate the addresses of memory.  

In this chapter, we will discuss pointers and how pointers are used to work with memory.  We will also discuss how memory can be dynamically allocated and manipulated using pointers.  

### Pointer Basics ###

In C, we know variables as abstractions of memory, holding a value.  That value is *typed*, defined by a data type definition in the variable declaration.  

A pointer is no different.  A pointer is a variable whose value is an address, typed by its declaration.  Pointers "point to" a variable with a typed value by referencing that variable, not by name, but by address.  

#### Declarations ####

Pointers are declared to point to a typed value.  This is the syntax of a declaration:

<pre>
<i>datatype</i> *<i>variable_name</i>
</pre>

Here are some examples:

    int *ptr1;
    float *ptr2;
    char *ptr3;

These declare `ptr1` to hold the address of an integer, `ptr2` to hold the address of a floating point number, and `ptr3` to hold the address of a character.  

Like all variables, *pointers do not have a value simply by being declared*.  That fact might seem intuitive for other data types, but it's hard to remember for pointers.  In the above examples, the variables are meant to contain the address of other variables, but they have not been initialized yet.

> **Declaration Notation**
> 
At first glance, the notation used to declare pointers might seem a little wrong.  Since the variable points to a variable of the declared data type, you might expect the declaration to look like this:
>
    int* ptr1;
>
Instead, the `*` symbol is associated with the variable name in the declaration.  This is intentional.  If we associate the `*` symbol with the variable name, we can declare a list of variable names, some of which are not pointers.  Here is an example:
>
    int *ptr1, width, height, *mem;
>
Note that not everything in the list is a pointer.  This is possible only if we associate the `*` with the variable name. 

#### Variables and Addresses ####

If pointers contain addresses, there should be a way to give them a value.  All variables have an address, a designation of where they are stored in memory.  We can derive the address of a variable by placing a "&" symbol in front of the variable name.  Here is an example:

    int miles = 10;
    int *ptri = &miles;
    printf("%u\n", ptri);

The variable `ptri` is assigned the address of the variable `miles` as its value.  The value of `miles` is not changed.

Now if we were to print the value of `ptri`, we would get a large number that really makes no sense *to us*, but makes sense to the runtime system. The `printf` function call above might print this as its value:

    3216074448

That value makes sense to the runtime system, but it is of no use to programmers.  Knowing the address does not help us work with the pointer or what it points to.  

#### Dereferencing a Pointer ####

Once a pointer has an address of a variable name, we can use it to work with the variable it references.  To do this, we have to *dereference* the pointer, that is, get to the variable or memory it points to.  

Dereferencing a pointer uses the same asterisk notation that we used to declare a pointer.  Consider the following example.

    int payment = 10;
    int *p = &payment;

    *p = 15;

This code starts by assigning the value `10` to the variable `payment`.  Then the pointer `p` takes the address of `payment` as its value.  The third statement changes `payment` to `15` by assigning the variable that `p` points to to the value `15`.  The `*p` in the statement is a derefence.  

Using the same syntax to declare pointers and to dereference pointers can be a bit confusing, especially if the declaration is used with an initial value, like in the above example.  Unfortunately, you just have to remember the difference between declaration and dereferencing.  

Let's look at one more example of dereferencing.

    int distance = 250, fuel = 10;
    float economy1, economy2;
    int *pd, *pf;

    pd = &distance;
    *pd = *pd + 10;
    pf = &fuel;
    *pf = *pf +5;
    economy1 = distance / fuel;
    *(&economy2) = economy1;

In this example, the variable `distance` is set to `250` and incremented by `10` by dereferencing the pointer `pd`.  Likewise, the variable `fuel` is set to `10`, then incremented by `5` by dereferencing the pointer `pf`.  The last statement is there to show that you can even dereference an address operator.  By dereferencing what the address of `economy2` points to we just reference the variable `economy2`.    

### Allocating Memory ###

While you can work with declared variables using the "&" operator, you can also create memory space while a program is executing and allow a pointer to reference it.  This memory space does not even need a name associated with it.

You create space in memory using the `malloc` function.  To create space, you need to know the *type* of space you want to create and the *size* in bytes of that space.  Fortunately, you don't have to know the size of everything in C; you can use a function to compute that.

The `sizeof` function will return the number of bytes its type parameter uses.  For example, 

    sizeof(int)

should return the value `4`: a variable declared to be of type `int` will take up 4 bytes in memory.  In fact, variables of the float and char data types will also take up 4 bytes.  

Once we know the number of bytes we want to allocate, calling `malloc` with the right size will create a space in memory of that size and will return the address of that space.  So, consider this example:

    int *p = malloc(sizeof(int));

Here, we have allocated enough memory to store a single integer and returned the address of that space to be assigned to the pointer `p`.  The `malloc` function returns an integer pointer by default.  Now, the *only* way to work with that space is through the pointer.  But if we use dereferencing correctly, this space can be used as if we have a variable, *because we do!* We do indeed have a variable; it just does not have a name associated with it.  

Here's another example:

    int *pa = malloc(5*sizeof(int));

In this allocation, we have created a space that is big enough to store 5 integers.  

Since this space is contiguous, that is, created from sequential memory locations, we have essentially created an array of 5 integers.  We will examine this further, but we need to first figure out how to access each integer in this space by doing arithmetic on pointers.

Pointers are *typed* in C.  When a pointer is declared, the data type it points to is recorded. As with other variables, if we try to assign values from incompatible types, errors will result.     

### Pointer Arithmetic ###

We have said that a pointer contains an address into memory.  If addresses are just numbers, then we can do computations with them.  Indeed, we can do pointer arithmetic in an intuitive fashion.  

As you might think, pointer arithmetic uses operators `+` and `-` to increment or decrement addresses.  However, to increment a pointer means to add enough to its value to move to the next element of the data type it points to.  For example, if a pointer contains the address of an integer, then adding one to that pointer means skipping 4 bytes to point to the next integer.  If the data type is larger, the increment will increase the pointer the correct amount of bytes.  Decrement works in an analogous way.  

This is especially useful when a pointer points to the beginning of an allocated area in memory.  Let's say that we have code that just allocated space in memory for 20 integers:

    int *bigspace = malloc(20 * sizeof(int));
    *bigspace = 10;

 By dereferencing the pointer, we gain access to the first integer in the space.  The rest of the integers are accessible through pointer arithmetic.  Consider the following code:

    *(bigspace + 1) = 20;
    *(bigspace + 2) = 30; 

This code assigns the values `20` and `30` to the the second and third integers in the space, respectively.  This is how we would access all integers in the allocated space.

There's a few notes we need to make about pointer arithmetic.

* The result of pointer arithmetic is a pointer.  As seen in the example above, we do the arithmetic inside the parentheses *and then* treat the result like it was a declared pointer.  In the example above, we dereferenced the result of the addition and it worked.
* As with all expressions, the above example simply computes where the next integer is located, but does not change the pointer itself.
* There is no way to derive *where* a pointer points in the allocated space.  We could get the pointer's value, which is an address, but that would not give us much information about which allocation element a pointer points to.

Let's walk through one more example.

    long *bigints = (long *) malloc(100 * sizeof(long));
    for (int i=0; i<100; i++, bigints++) *bigints = 0;
    bigints -= 100; 

In this example, we allocate 100 long integers and initializing each long integer in the space to the value `0`.  Then we "rewind" the pointer by subtracting `100*sizeof(long)` from it.  It is our only way to access all the long integers in the allocated space and we must be careful to work with the pointer so it accurately points to the elements we need.  

### Pointers and Arrays ###

We have defined arrays as a collection of elements of the same type, organized in sequence so that we can reference them with an integer index.  We have also described memory allocation as a way to create a collection of elements of the same type, placed sequentially in memory.  These two ideas are so very close that C treats them the same.

We will demonstrate that *pointers are arrays and arrays are pointers.*  

#### Pointers and Array References ####

C allows the same syntax to be used for both arrays and pointers.  Let's consider a previous example:

    int *bigspace = malloc(20 * sizeof(int));

We accessed and assigned values to memory in this way:

    *bigspace = 10;
    *(bigspace + 1) = 20;
    *(bigspace + 2) = 30; 

We could just as easily have used array reference syntax:

    bigspace[0] = 10;
    bigspace[1] = 20;
    bigspace[2] = 30;

Declaring `bigspace` as an array also works:

    int bigspace[20];

And both methods of accessing the memory space are still equally valid.

Pointers and arrays may be exchanged in assignment statements as well.  For example, consider the following:

    int space1[20];
    int *space2 = space1;

    *space1 = 10;
    space2[1] = 20;

The first two elements of the array `space1` have been initialized to `10` and `20`, respectively.  The reverse is true as well:

    int *space2 = malloc(20 * sizeof(int));
    int space1 = space2;

    *space1 = 10;
    space2[1] = 20;    

This illustrates our point: pointers are arrays and arrays are pointers. 

#### Pointer Arithmetic and Array Indicies ####

As we saw in the previous section, pointer arithmetic and using indicies and array notation are interchangeable.  We have also seen how to pointer arithmetic to move pointers through allocated memory space.  

As declared and initialized to a memory space, pointers point to the base, the first element, of that space.  This is item number 0 in array notation.  Whenever we add 1 to a pointer, the system computes the size, in bytes, of the data type that the pointer points to and increments that pointer the number of bytes that make up that data type.  Thus, an increment for the pointer is the same as a increment in array notation.  

For example, consider this code:

    short sa[10], *ps;
    long sl[10], *pl;

    ps = sa;
    pl = sl;

Now, `ps` points to `sa[0]` and `pl` points to `sl[0]`.  Now if we increment both pointers by 1, like this:

    ps++;
    pl++;

`ps` points to `sa[1]` and `pl` points to `sl[1]`.  Even though both pointers were incremented by 1, the addresses were incremented by a different number of bytes.  

If we wanted a pointer to start in the middle of an array, instead of the beginning, we would need to use the address of a selected item from that array.  For example, if we wanted `ps` from the example above to point to `sa[4]`, we would need to derive the address of `sa[4]` like this:

    ps = &sa[4];

Now, `ps` points into the array and not at the beginning.

#### Pointers and Multidimensional Arrays ####

Now let's consider how pointers interact with multidimensional arrays.  The syntax starts to get a bit clumsy, but if we remember how pointers work with memory, it gets a bit better.

Let's start with two dimensions: rows and columns.  Remember that a two-dimensional array is really just a big memory space, *organized* as rows and columns.  If that's true, then a pointer into that space could actually just work as we have described it so far.  Here's an example:

    long table[10][20];
    long *ptr = table;

This code describes a space comprised of 200 long integers.  We could work with `ptr` as if it was pointing into that 200 long integer space.  We could reference `ptr[30]`  or `(ptr+30)` and it would work, referencing a long integer, 30 items into that space.  

However, if we are declaring an array to have two dimensions, then it makes sense to try to use pointers in two dimensions.  When we have an array of two dimensions, we can think of it as an "array of arrays".  Using one dimension references an entire array from that collection.  So referencing `table[3]` references an entire array at the 4th row of the table.  

To use pointers with two dimensions, we need to think like this.  If one pointer reference points to an array, then we really need a *double* reference: one to the the array/row and one more to get the item at the column in the array/row.  Consider this type of declaration:

    long table[10][20];
    long **ptr = table;

Note the double asterisk: one for rows and one for columns.  Now, it would be an error to reference `ptr[30]` because there are not 30 rows in the table, only 10.  In addition, `(ptr+5)` skips over the first 5 arrays, or 100 long integers, giving us access to the array at `table[5]`.  

The same works to other dimensional arrays.  Three dimensions could work like this:

    long bigtable[5][10][20];
    long ***ptr;

Other dimensions work analogously.   

### Typed Pointers and Untyped Pointers ###

We have seen that, in C, pointers can be typed.  The data type that a pointer points to informs the compiler on how many bytes to increment a pointer's address when using pointer arithmetic and how to work with pointers in situations where data types matter (like computations or some types of parameters).  For example, in this code:

    float x, y, *pf;
    int a, b, *pi;

we cannot mix pointers to floats and integers in the same situations we can't mix actual floats and integers.  For example:

    *pf = 12.5;
    a = 10;
    pf = &a;
    b = *pf + a;

The last line is an error, because it mixes a floating point number and an integer, producing a floating point number, and tries to assign it to an integer.

There are situations where untyped pointers are appropriate.  Untyped pointers are declared to point to "void" type and may point to values of *any* type.  However, since they have no data type themselves, in order to dereference such a pointer, we must tell the compiler what it points to.  Consider this example.

    void main() {
        int numbers[6] = {10, 20, 1, 5, 19, 50};
        float average;

        void *p;
        p = &numbers[3];
        p = &average;

        average = computeAverage(numbers, 6);

        printf("Average of these values is %f\n", *(float*)p);
    }

In this code, assume that `computeAverage` computes the average of the integers in the array and returns that value as a float (we saw this example in Chapter 7).  First, note that the pointer `p` takes an address of an integer variable, then takes the address of a float variable, and that a compiler would think this is correct.  Second, in the call to printf, we had to inform the compiler that `p` was currently pointing to a float variable, and then we could dereference it.  

Untyped pointers are also useful as formal parameters to functions.  Using a void type for a pointer in a function specification allows flexibility in the actual parameter.  We will discuss this further in the next section.

### Pointers as Function Parameters ###

Pointers are the only way parameters can be changed in functions in a way that the caller of the function can access.  

We have said in Chapter 6 that functions use pass-by-value for function parameters.  In other words, values are copied from actual parameters to formal parameters when the call is made, but not copied back when the function returns.  This implies that it is impossible to send changed values back to a function caller.

However, using pointers as parameters to function makes this type of change possible.  If we send the pointer to memory to a function, any changes to the pointer itself will be ignored, but the function can dereference the pointer and make changes to memory that the pointer references.  Those changes are not part of the parameter list of the function and they will be reflected back to the caller. 

Let's consider the following example.

    void swap_integers(int first, int second) {
        int temp;
        temp = first;
        first = second;
        second = temp;
    }

Now, because C uses pass-by-value, calling this code like this will not swap anything:

    int x = 10, y = 20;
  
    swap_integers(x, y);

The variables `x` and `y` will retain the same values after the function call.

Now, let's change the parameters of `swap_integers` into pointers.  The code would look like this:

	void swap_integers(int *first, int *second) {
	   int temp;
	   temp = *first;
	   *first = *second;
	   *second = temp;
	}

Because the parameters are now pointers, we have to dereference them to get at the actual values to be swapped.  But now, since we are not changing the parameters, but rather the memory to which they point, memory is changed.  

We would call this function this way:

    int x = 10, y = 20;
  
    swap_integers(&x, &y);

And the values are actually swapped. 

> **Can We Make "swap" Generic with void Pointers?**
> 
If we can use typed pointers in a `swap_integers` function, could we use untyped pointers in a generic `swap` function?  That is, by using "void" pointers, could we swap values of *any* type?
>
The answer, unfortunately, is "NO".  While we can certainly specify parameters that are `void *` parameters, we cannot dereference a void pointer without knowing the type to which it points.  In addition, because we assign a value to `temp` in the above code, we must know what data types `first` and `second` point to, so the compiler knows how to make the assignment.   

### NULL Pointers ###

We have stated that pointers contain memory addresses as their values.  In addition to addresses, pointers can have a "no address" value, called "null".  This null value is actually a special value (many compilers make this value 0, although it could be another special value).  Null values are unique; null pointers of *any* type are guaranteed to be equal.  

Null pointers should not be confused with uninitialized pointers.  Uninitialized pointers, like uninitialized variables, have no defined value; they occupy space in memory and take on whatever value was left there by the previous variable that occupied that space.  Null pointers have a specific null value; uninitialized pointers have an undefined value.

Null pointers typically signify the end of data or an error condition.  Dereferencing a null pointer will typically cause a program-crashing error.

### Freeing up Memory ###

Dynamically creating memory with `malloc` is a great way to only take up the memory that your program needs.  Memory on the Pebble watch is limited, and using pointers in this way is frugal.

To continue this frugality, memory space that is allocated by your program must also be deallocated by your program.  To deallocate memory that was allocated with `malloc`, use the `free` function call.  Here's an example of allocating and immediately freeing up memory:

    long *racing = (long *)malloc(40 * sizeof(long));

    free(racing);

It's as easy as that.  It should be done in your program as soon as memory space is not needed.

When freeing memory space, you need to be aware of certain rules:

* You cannot free memory that was not allocated by `malloc`.  For example, if you assign a pointer the address of a declared variable, freeing that pointer's memory will cause an error.  Declared variables are allocated differently than dynamically allocation memory space.
* You cannot free a null pointer.  Null pointers are pointers with "no address" values and freeing them will cause an error.
* You cannot free uninitialized pointers.  These pointers do not point to anything and trying to free them will cause a program error.  

Sometimes, memory is allocated by function calls within functions.  That kind of allocation is usually freed up by a companion function to the function that allocated the space.  (See below in "Pointers and Pebble Programming" for examples.) 

### How to Avoid Messy Coding with Pointers ###

Pointers are notorious for creating messy, confusing code.  Here are some examples: 

    intar[i];
    *(intar+i);
    *(i+intar);

These are all equivalent references; they can be used with either `int intar[10]` or `int intar = malloc(10*sizeof(int));` declarations.  Here's another example:

    char *c = "Hello World";
    while (*c) printf("%c", *c++);

As we will see in the next chapter, strings in C are arrays of characters, ending with a character with a 0 value (a "null" character).  Knowing this, the above code will print each character of a string, incrementing to the next character for the next iteration of the loop.  Here's one more example:

	int main()
	{
	    int i, sum;
	    int *ptr = malloc(5 * sizeof(int));
	
	    for (i=0; i<5; i++)
	        *(ptr + i) = i;
	
	    sum = *ptr++;
	    sum += (*ptr)++;
	    sum += *ptr;
	    sum += *++ptr;
	    sum += ++*ptr;
	
	    printf("Sum = %d\n", sum);
	}

Running this example will print the value `8`, when the intention is to print the value `10`.  The confusion here is the various ways that pointer arithmetic has been done.

Here a few tips to remember to avoid confusion with pointers.

1. Never forget to initialize pointers.  This is a simple rule, but it is very confusing when a pointer uses old values.
2. Using array syntax with pointers can be a lot clearer than pointer syntax.  This is especially true of multidimensional arrays and array spaces. 
3. Be painfully clear when using pointer arithmetic.  Using shortcut arithmetic with dereferencing can be very confusing, as we see in the examples above.
4. When using memory allocation, always use the most flexible and meaningful expressions. Calling `malloc(8)` is not very expressive, but using `malloc( 4 * sizeof(int) )` is much more informative.  

> **Pointer Jokes**
>
Pointers can be messy and useful.  They can also be funny.  [This link is an xkcd pointer joke.](https://xkcd.com/138/)  Enjoy!

### Pointers and Pebble Programming ###

Pointers feature prominently in software written for Pebble watches.  You have seen this in the many project exercises that have been given in past chapters.  For example, every Pebble program needs a window on the screen so that it can interact with the user.  This window is declared like this:

    static Window *window;

and is created like this:

    window = window_create();   

The call to `window_create` returns a pointer to a `Window`.  This type of allocation is deallocated by a companion to `window_create`:

    window_destroy(window);

Pebble programming uses pointers for most system calls that work with the operating system.  Doing so allows these system objects to be allocated in memory and thus hidden from programmers.  The fact that they are hidden enhances the abstractness of using them: usually, programmers just care that they work as they are documented and they really don't want to examine every byte of the data used.  Using pointers for system calls also allows Pebble to update the system data structures without having to change app source code.  

The pattern of allocation, use and deallocation is very common among all system interfaces.  The creation/deallocation functions all have different, but similar, names.  You should get used to this pattern as you write more Pebble code.

### Project Exercises ###

#### Project 8.1 ####

We have said that pointers are arrays and arrays are pointers.  In this project exercise, you are asked to prove it!  Start with  the Bubble Sort in Project 7.1, [available at this link](https://cloudpebble.net/ide/import/github/programming-pebble-in-c/project-7-1-answer), and (1) leave the array *declarations* alone, but (2) change all the array references to pointer references.  You may add any variables you need. 

Don't forget the parameters to the sorting functions and the assignment of the `number_sorted` array from the `number` array in the `handle_init` function. 

In addition to the above, can you make the sort work with *any* data type? (*The answer is NO: but can you figure out why?*)

[You can find a solution here.](https://cloudpebble.net/ide/import/github/programming-pebble-in-c/project-8-1-answer)

>**You Cannot Use "void *" Here**
>
At first glance, you might think you could make this work with any type by using "void *" to declare the parameters to the sorting functions, like this:

    void bubble_sort(void *array)

>
This is fine, but it makes the comparisons in the function code invalid.  If the `array` can be of any type, then how do you know that the `<` operator works with the specific type that is used at runtime?  You could fix the code to use "int *" for comparison like this:

	void bubble_sort(void *array)
	{	
	 	for (int i=0; i < NUMBERS_MAX - 1; i++) {
	    	for (int j=0; j < NUMBERS_MAX - 1; j++) {
	        	if (*(int *)(array+j) > *(int *)(array+(j+1))) {
	            	int temp = *(int *)(array+j);
	            	*(int *)(array+j) = *(int *)(array+(j+1));
	            	*(int *)(array+(j+1)) = temp;
	        	}
	    	}
		}
	}

>
But this code defeats the purpose of using "void *".  It says that you can send any type to the sort code, but the code will compare them as integers.  And this means we need to be specific about the data type used as function parameters.

#### Project 8.2 ####

This exercise revisits Project 6.2 again (like we did for Project 7.2).  That project, whose answer can be found here, takes a sequence of characters in a string and considers them in groups of three, drawing 5 rows of three squares.  POINTER VERSION OF 7.2....MORE DETAILS AFTER I SEE 7.2 ANSWER

#### Project 8.3 ####

Remember Project 5.2?  This project asked you to change the colors of pixels by examining each one in a loop and changing the ones that matched a certain color. The main code for this project was a function called `replace_colors`, whose code is below:

	void replace_colors(int pixel_width, int pixel_height, GColor old_color, GColor new_color){
		int max_y = pixel_height; 
		int max_x = pixel_width;
		
		for(int y = 0; y < max_y; y++){
			for(int x = 0; x < max_x; x++){
				GColor pixel_color = get_pixel_color(x,y);
				if(gcolor_equal(pixel_color, old_color)){
				  set_pixel_color(x, y, new_color);
				}
			}
		}
	}

In this code, there was a bitmap that was allocated using a pointer, but referenced using an array.  The function `set_pixel_color` is a good example:

	void set_pixel_color(int x, int y, GColor color){
		bitmap_data[y*bytes_per_row + x] = color.argb;
	}

You are to rewrite this code to use pointers to access the bitmap data.  To do this you must (1) remove the function `set_pixel_color` and (2) you must rewrite the nested loops in `replace_color` to use a single loop and to reference the pixel colors with a pointer.   

#### Project 8.4 ####

For Project 8.4, you can get a started with a basic project here.

If you run the initial code, you will see that it's a simple image that bounces around the screen, reminiscent of the bouncing ball from Chapter 3.  There are two functions, `compute_x` and `compute_y`, that compute the next compute the next position for the image's X and Y coordinates, respectively.  These functions take reference parameters, that are the previous X or Y and the next X or Y.  Based on the previous X or Y, the new value is computed.

Currently, the code for these two functions is minimal.  We want a program that makes the image move randomly when the up button is pressed and in a bouncing manner when the bottom button is pressed.  You will need to add code in `up_click_handler` and `down_click_handler` to change between the two modes and you will need to add code to the `compute_x` and `compute_y` functions to handle new coordinates.  Remember to keep the reference parameters.

A completed project can be found here.
