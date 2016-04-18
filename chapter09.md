Chapter 9: Strings
=======
Of all the structures in the Caribbean programming language, strings are perhaps the paradoxical.  They are extremely useful, yet their use can lead to of most convoluted code in C.  They are necessary for writing programs, but can be extremely annoying.  They are conceptually easy and practically difficult.

This chapter will work through the easiness and the difficulty that strings represent.  We will start off with the easy part: the idea and usefulness of strings.  And we will end with the difficulty: the messy code that using strings can generate.

### The Basics of Strings ###

Strings are simply sequences of characters.  The word "sequence" should remind you that arrays are sequences of identically typed elements.  Therefore, we can think of strings as arrays of characters.  

String literals are sequences of characters, surrounded by double quotes.  For example:

    "Hello, World!"
    "Nice to see you."
    "Tea.  Earl Grey.  Hot!"

These are all string literals.  In C, unlike some languages, the quotes are not interchangeable.  Single quotes surround single character literals, not strings.  Strings require double quotes.

### Strings are Arrays and Pointers ###
 
As we stated above, we can think of strings as arrays of characters.  In fact, there is no "string" data type in C; we declare strings exactly as we would declare an array of characters.  For example,

    char title[40];
   
This declares a string to contain 40 characters, while also declaring an array to hold 40 characters.

Since strings are arrays, we can initialize strings the same way we initialize strings. For example,

    char title[40] = 
        { 'E', 'n', 'c', 'o', 'u', 'n', 't', 'e', 'r', ' ', 
          'a', 't', ' ', 'F', 'a', 'r', 'p', 'o', 'i', 'n', 't' };  

This will initialize the character array `title` to contain the string "Encounter at Farpoint".  

While this method of initializing works, it's pretty tedious.  C also allows a more convenient initialization of strings:

    char title[40] = "Encounter at Farpoint";

From Chapter 8, we know that pointers are arrays and arrays are pointers. So strings can also be manipulated by pointers as well as array notation.  We can do the following:

    char quote[40] = "Make it ";
    char *select = quote;

    select += 8;
    *select = 's';
    *(select + 1) = 'o';
    *(select + 2) = '!';

It should be clear that we can manipulate strings using array and pointer notation in ways we are, by now, accustomed to.

One warning needs to be made about string assignment.  We can initialize strings, be we cannot assign strings through the assignment operator.  Consider this code:

    char quote[40] = "Make it so";
    quote = "Engage!";

The first line works because it is an initialization within a declaration.  The second line is an error, because you cannot assign arrays to each other.  To make assignment of strings work, you must *copy* one string to another character-by-character.  There are functions that we can use for this; see a following section below for description of string copy functions.

### Strings are Null Terminated ###

If we are going to be able to work with a string, we are going to have to know the length of a string.  In C, we can't actually encode the length of a string with the string itself, so we place a marker at the *end* of a string.  By knowing what the marker looks like, we can count the characters in a string and compute its length.

In C, strings are terminated by a character whose integer value is 0, called the "null" character.  

This is a judicious choice for a termination character.  Consider how array initializations are made.  When we initialize an array, like with `title` in the example above, if we give the number of initialization values that is less than the full array size, the remainder of the array is initialized to the value 0.  This is convenient, and makes all the string examples we have given automatically terminated with a null character.  Even the pointer manipulation example produces a correct string because the part of the string array not initialized or changed is a collection of 0 values, terminating whatever string is created.

This choice for a termination character also makes certain computations about strings very easy.  Consider how we could compute the size of an array:

    int size = 0;
    while (!title[size]) size += size + 1;

Eventually, `title[size]` will have the value 0 when the computation is at the end of the string.  With pointers, we can abbreviate this code to 

    int size = 0;
    for (select = title; *select; size++,select++);

Using a null character for termination also means that we have to be careful when manipulating the array of characters that makes up a string.  For example, we can truncate a string easily this way:

    char title[40] = "Encounter at Farpoint";
    title[9] = 0;

Now the string contained in the array `title` has the value "Encounter" because it is terminated by a value of 0 in the character position after the "r".  There are actually 2 strings now in the `title` array, each terminated with a 0 value.  

We also have to be careful about filling the array up to capacity; we have to remember that the last character must have the value 0.  

>** Strings are Not Objects **
>
It's important to emphasize here that strings in C are just character arrays.  In other languages, they are specialized datatypes or data objects.  In Java, for instance, a "String" is a built-in class and, once assigned, you can work with objects from that class in many built-in ways.  Length is determined by calling a function built into the class; various functions are also built-in: concatenation, reverse, up- and lower-casing.  
>
In C, all we have is a character array, with the string terminated with a null character.  We have to compute the size and do operations like concatenation and reverse by manipulating the characters in the actual array.

### String Functions ###

C provides a number of functions that work with strings.  They are provided in a library of standard C functions that can be used from any program. 

One of the most common functions is a string copy.  Let's say you have two strings, declared as below:

    char quote1[80] = "I will take your word for it. This is very amusing.";
    char quote2[60];

As we have noted before, simply assigning one array to the other will not copy one array to another.  The assignment will make both arrays work with the same data.  To make a copy work, we must move the data, character by character, from one array to the other.  Again, the choice of a 0 value was a judicious choice to terminate a string, because we can have code like this:

    int i = 0;
    while (quote1[i] != '\0') {
        quote2[i] = quote1[i];
        i++;
    }

This is rather clunky, but it copies each character from `quote1` to `quote2` until the code reaches the terminator.  However, the terminator is *not* copied, so this is not correct.  We can condense this code, and make it correct, in the following way: 

    char *ptr = quote2;
    while(*quote2++=*quote1++);
    quote2 = ptr;

Here, we use pointer dereferencing  and pointer arithmetic to copy the strings. It's more cryptic than you might normally use, but it works.  

We could have just used the `strcpy` function.  The header for this function looks like:

    void strcpy(char *dest, char *src);

And we could have used it with our quote strings like:

    strcpy(quote2, quote1);

That looks a little simpler to use.   Note that this works with string literals as well.  Outside of initialization in declaration, we cannot use assignment to set up strings.  We need something like this:

    strcpy(quote2, "Tell him he is a pretty cat.");

There are several other common functions that are very useful.
* `size_t strlen(char *string);`
This function counts the number of characters in a string, without the terminator, thus returning its length.  `strlen(quote1)` returns 51.  Note that the "size_t" data type is equivalent to an unsigned integer data type (which, in this case, is more accurate, because lengths cannot be negative).
* `char *strcat(char *string1, char *string2);`
This function concatenated `string2` to `string1` and returns the result.  Consider this example:
    char day1[40] = "I would be chasing an untamed";
    char day2[20] = "ornithoid without cause.";
    char *days = strcat(day1, day2);
Thus making `days` reference a string describing a wild goose chase.  
* `int strcmp(char *string1, char *string2)`
This function compares `string1` to `string2` lexicographically and returns an integer: -1 if `string1` is less than `string2`; 0 if `string1` is equal to `string2`; and 1 if `string1` is greater than `string2`.  

There are "n" versions of these functions: `strncpy`, `strncat`, and `strncmp`.  These "n" versions take an extra integer as the last parameter and only work for the number of characters in the value of this parameter.   For example,

    char reg1[10] = "1701 C";
    char reg2[20] = "1701 E";
    int cmp = strncmp(reg1, reg2, 4);

In this code, `cmp` would have value 0, because the first 4 characters of each string are the same.    

There are several other functions in the C string library that work with strings.  [Here is a good listing link to check them out.](http://www.cplusplus.com/reference/cstring/)  

### Common Pitfalls with Strings ###

Because strings are closely related to arrays and pointers, we have to be careful when handling them.  There are many ways to fall down when using strings.  Here are a few bits to guide your string handling.

* Beware accidentally handling the null character.  Placing a 0 value in the middle of a string will truncate it.  
* Because strings are character arrays, you have to be careful with boundaries.  When you work with string values rather than individual characters and indexes, it's easy to make out of bounds references. For example: 

    char data[40] = "The need for more research is clearly indicated.";

This reference will overflow the boundaries of the `data` array, because the string is longer than 40 characters.  However, because of the way C works with out-of-bounds references, it is not defined how this overflow will affect program code and/or other variables.
* Avoid using `strncpy` to copy fixed numbers of characters.  If the destination string is not as long as the source string, this function will fill the destination, but will not terminate the string with the null character terminator.  
* Be aware that calling string functions typically analyzes each string array for every call.  This means that code like this:
    int i;
    for (i=0; i<strlen(data); i++) do_something_with(data[i]);
actually processes the entire `data` string once for every character reference.  HOwever, this code:
    int i, len = strlen(data);
    for (i=0; i<len; i++) do_something_with(data[i]);
analyzes the `data` string once, then processes each character of the string.  For long strings, the second version has significant performance improvements.  

In addition to these tips, many programmers advise not using functions when the "n" variant is available.  There are several reasons for this, most of which note the "safety" of the "n" variants. For example, the `strcmp` function depends on the fact that both strings contain the null character terminator.  Using the `strncmp` variant on this function is safer because this version will compare two strings whether they are different lengths or whether one or both are missing their terminator.

### Avoid Messy Code with Strings ###

Strings are arrays of characters, which are also pointers.  So all the warnings about arrays and pointers apply to strings.  However, the null terminator introduces a new way to get messy.  The string copy example above demonstrates this.  Consider this line:

    while(*quote2++=*quote1++);

This does indeed copy the string in `quote1` to `quote2`.   But it is also uses several C features that are ill-advised: using assignment to return a value, using pointer arithmetic inside an assignment operation, and stealthily depending on the null terminator to shut down the loop, to name just three.  The dependence on the 0 value to make the condition for the while loop false is not here, especially when it depends on the assignment operator to give up that 0 value.  

While C puzzles are interesting and even fun, it is better programming practice to be painfully and obviously clear about your algorithm.  The first string copy code example uses `quote[i] != '\0'` in the while loop condition when simply stating `quote[i]` would have accomplished the same functionality.  Obvious is clearer and better.  

### Project Exercises ###

#### Project 9.1 #####

The starter code for Project 9.1 is here.  Copy the project and run the code.  The starter code displays a cursor that is positioned underneath letters in a word.  The "select" button will move between letters: a press will move right and a long press will move left.  In the starter project, the letters are all "A".

You are to fill in code for `up_handler` and `down_handler`, code that handles presses of the "up" and "down" buttons, respectively.  Each up press should advance the letter the cursor is on to the next letter of the alphabet; each down press should move the letter to the previous letter in the alphabet. In either case, the string must be rebuilt and displayed again on the screen.  

Note that you *could* simply redisplay each letter as it is changed.  But that is not good enough for this project.  Each string needs to be rebuilt using string functions and redisplayed on the Pebble screen.

An answer to this project can be found here.

#### Project 9.2 ####

This project creates a "word calculator".  The starter code, which can be found here, uses the buttons on a Pebble smartwatch to cycle numbers and operators.  The "select" button moves to the next position.  When the operator "=" is selected, and the "select" button is selected, the app will display the value resulting from the computation displayed.

You are to write a function `num2words` that will take an integer parameter and produce a string that expresses the value in words.  For example, if you call the function like this

    char *words = num2words(52);

then `words` will point to the value "fifty two".  Set the maximum value sent to the function to 100.  

You are also to use this function to replace the numbers that a user puts on the Pebble screen with words.  When the select button is pushed, erase the number that was just added and replace it with the words from the function.  Redisplay the current computation "sentence".  

You will have a few issues here to work out.  How will you wrap your "sentence" around the Pebble screen?  Which operators will you allow?  And what happens when the "sentence" is too long for the screen?

You can find an answer here.

**Extra Challenge:** Extend your words substitution to operators.  Replace operators, like "+", with words on the screen (e.g., "plus").

**Extra Extra Challenge:** Don't even use numbers.  Cycle through words that represent numbers with a user uses the "up" and "down" buttons.

#### Project 9.3 ####

There are many weather sites on the Internet.  This project begins by fetching data from one of those sites.  It's your job to display relevant data on the Pebble screen.

You can begin with this starter code.  Read through the code and notice that there are several functions that work together to get weather data.  The main function for this is `get_weather`, which returns Web page contents that have the current conditions.

You are to use the string processing functions you have available to trim down the Web page data and to extract the strings relevant to the current weather conditions.  You will have to wade through a lot of HTML to do this.  Display these on the watch screen.  See previous code for ways to display strings on the Pebble screen.

You can find an answer for this project here.

#### Project 9.4 ####

Get the starter code for this project here.   Read through the code, paying attention to the functions defined.

Among the functions in the starter code that run the watch app, there are three functions that tap into the sensors on the watch.  `get_compass` gets information from the compass sensor. `get_accelerometer` gets data from the accelerometer.   `get_light` gets light level data.   Each function has code to get the data from its respective sensor and each function returns a string that contains that information in a form that can be drawn on the watch screen.

You are to fill in each of the three functions to convert the data derived from the sensor to a string that can be returned.

Remember that strings can be depicted as arrays or pointers.  In each function, you need to dynamically allocate a string using `malloc` and return that as the `char *` return type from the function.

You can find a solution for this project here.
