Chapter 10: Structured Data Types
=======
Chapter 7 introduced the idea of data collections by focusing on arrays.  As demonstrated in Chapter 9, arrays are collections of identical things, organized sequentially.  If we can build a collection of identical things, it makes sense that we could build a collection of things that are not the same.  These collections are structured, but different than arrays.

This chapter explores structured collections of data that are different than arrays.  Structs and unions represent such collections. Enums represent a different type of collection: a collection of structured constants.  We will cover each of these three data types, give plenty of examples and uses, and discuss how they can be used for good code and not-so-good code.  

### The Basics of Structs ###

Structs are collections of data items.  These data items can be completely different from each other, or they can be the same.  WIthin the struct, each item is named and is declared to have its own data type.  

Structs have the following syntax for declaration:

<pre>
struct <i>struct_name</i>{
    <i>data_item_declarations</i>
} 
</pre>

The *struct_name* is optional.  If it is omitted, you can use this as an actual type declaration.  Consider the following examples.

    struct {
        char appname[25];
        float cost;
        long when_purchased;
    } watchapp1, watchapp2;

    struct app_props {
        char appname[25];
        float cost;
        long when_purchased;
        int days_used;
    } watchapp3, watchapp4;

    struct app_props watchapp5, watchapp6;

Here we have 6 variables declared by structs. The first is an *unnamed* struct, declaring `watchapp1` and `watchapp2`.  This struct  cannot be used again, because it cannot be referenced, but it declares the variables nicely.  The second struct is a *named* struct that adds a field `days_used`.  This type of struct can indeed be referenced later, by name, as is shown in the declaration of watchapp5` and `watchapp6`.  Naming a struct declaration is very useful, through out C code, as we will see.

Referencing items in a struct is done by name using something called "dot notation".    Let's say we  need to reference an app as `watchapp1`:

    strcpy(watchapp1.appname, "lazerbeam");
    watchapp1.cost = 2.60;
    watchapp1.when_purchased = 1470052800;

This example sets up `watchapp1` to describe the properties for the app "lazerbeam". The elements of the struct are referenced by naming the data items connected to the `watchapp1` variable through the dot.  

Dot notation works for both named and unnamed struct declarations.

#### Nesting Structs ####

Structs are collections of data items.  It would follow that structs could be included in the collection. 

Nesting structs within structs follows the syntax that we have already introduced.  For example:

    struct new_app_props {
        struct {
            char first[50];
            char surname[50];
        } owner;
        struct app_props props;
    } watchapp8;

Here, we can work with `watchapp8` in this way:

    strcpy(watchapp8.owner.first, "Miguel");
    strcpy(watchapp8.owner.surname, "Rodriguez");
    watchapp8.props.cost = 4.5;
    watchapp8.cost.when_purchased = 1470055800;

Notice that the more structs are nested, the longer the references get.  Dot notation is used with dot notation to get to the actual data items being modified.  Also notice that any kind of declaration can be done; in the above example, we used both named and unnamed struct declarations.

#### Referencing Struct Elements Through Pointers ####

Pointers can be used to dynamically allocate space for structs.  Here is another place (beside declaration) where named structs come in handy.  Referencing struct elements through pointers also use a different syntax than dot notation.

To allocate structs with pointers, we have to declare the pointer correctly.  If we are going to use the above app example, we might declare a pointer this way:

    struct app_props *newapp;

This combines what we know of pointer declaration with our new understanding of struct declaration.  To allocate memory for the new struct declaration, we use `malloc` as we have before:

    newapp = (struct app_props *)malloc(sizeof(struct app_props));

Now here is the usefulness of a named struct.  All this casting and sizing would not be possible without structs that we could reference by name.

Referencing struct data items through a pointer uses a new syntax.  To reference with a pointer, we replace the dot with a "->" sequence.  Consider this example, given the declaration above:

    strcpy(newapp->appname, "shockster");
    newapp->cost = 1.00;
    newapp->when_purchased = 1470139200;
    newapp->days_used = 15;

The pointer notation works as well as the dot notation.  Each allows access to the data items of the struct. 

> **Orthogonal Notation that Works**
> 
Given the penchant for C to allow crazy notations, you can combine the ideas of the "&" operator and pointer notation for structs.  You can to this, using the declarations of `watchapp4` above:

    (&watchapp4)->cost = 1.00;

>
Note the use of the parentheses to make sure the pointer notation is associated with the result of the "&" operator.
>
This is example of *orthogonal* design.  Orthogonal design is design of language features that are independent and only cross at useful points.  So the design of "&" operator, pointer variables and pointer notation of structs are orthogonal to each other.  They work independently of each other, but can be useful when they cross, as above.  
>
There are many language features that are *not* orthogonal. MORE HERE.  

#### Initializing Structs ####

Just like we have special syntax for initializing arrays as we declare them, we can also use syntax to initialize structs as they are being declared.  

The syntax for initialization is similiar to that used for arrays.  We can use bracket notation like this:

    struct app_props myapp = {"zapme", 2.5, 1470229200, 10};

Each item in the struct gets a value here.  Note that we can assign strings in this way like we have done before, without the need for `strcpy`.  

Nested structures also work with initializing syntax.  Consider the `new_app_props` declaration above.  We could initialize a declaration this way:

    struct new_app_props myapp2 = { {"Miguel", "Rodriguez"}, {"zapme", 2.5, 1470229200, 10} };

This syntax assumes that *every* element of a struct is to be initialized.  Partial initialization is possible.  To initialize only a segment of a struct, you should reference the data items directly with dot notation or you can initialize nested elements completely.  Consider this example:

    struct new_app_props myapp3 = { {"Ester", "Williams"} };

In this case, the remainder of the `myapp3` struct will be initialized to "intuitive" values, that is, zeroes and blank strings.  In the above example, `myapp3.props.appname` will be blank, `myapp3.props.cost` will have the value `0.0`, and  `myapp3.props.when_purchased` and `myapp3.props.days_used` will both have the value `0`.  

Consider this declaration:

    struct new_app_props myapp4 = { .owner = {"Ester", "Williams"}, .props.cost=4.3 };

In this example, we used dot notation to reference parts of the struct.  Since we are already in the context of a declaration of the struct `new_app_props`, we can use partial dot notation.  `.owner` references that part of the struct being declared.  In this example, `myapp4.props.appname` will be blank and  `myapp4.props.when_purchased` and `myapp4.props.days_used` will both have the value `0`.  `myapp4.props.cost` will have the value 4.3.

#### Passing Structs as Parameters to Functions ####

Using structs is a convenient way to package data together.  However, structs can be very inconvenient when they get to be large collections.  Using structs as parameters to function must be done with some care, especially when passing large structs.

When specifying structs as parameters to function, it's important to remember that the type name of a struct parameter includes the word "struct" and the struct name.  For example:

    void listprops (struct app_props props) { ... }

This declaration needs a struct as an actual parameter, which gets copied to the formal struct parameter `props`. 

Note that parameters in C are passed either by copy or by reference.  So when we note that the actual struct parameter is *copied* to the formal struct parameter, an actual copy takes place.  In the case of our example, the size of a `struct app_props` data type is *only* 40 bytes (as counted by `sizeof`), but even 40 bytes can take up more precious time that we want to devote to other parts of an app's execution.  

To avoid copying structs to function parameters, it is best to use pass-by-reference when passing struct parameters.  We can retool the definition of `listprops` above to enforce passing by reference:

    void listprops (struct app_props *props) { ... }

Now, references can be quickly passed from actual parameter to formal parameter.  In addition, changes can be made to the formal parameter that can be reflected back to the actual parameter of the caller.  

#### Comparing Structs and Classes ####

Structs are often compared to classes from object-oriented languages.  Many programmers have had experience in object-oriented languages and have used classes and objects.  Structs are a kind of precursor to classes.

Structs are simliar to classes in several ways.  Both are ways to encapsulate different kinds of data into one structure.  Both constructs allow dynamic allocation of instances.

However, there are some fundamental differences between structs and classes.  Structs do not contain methods as classes do, which means they cannot contain constructors or destructors.  In most languages that use classes, objects are declared separately and can only be used when instantiated.  In C, variables can be declared as structs and used without instantiation.  

In addition, class assignment in most object-oriented languages is done using references or pointers.  Passing classes as parameters is also pass-by-reference.  Structs, on the other hand, are assigned by copying the contents of one variable to another.  Passing structs as parameters will also copy from the source to the destination, unless the parameters are explicitly declared as pointers.

### The Basics of Unions ###

Unions are collections of data items, like structs, that may be of different data types.  Unlike structs, the data items in a union are defined to occupy the same memory space.

Compare an integer and a character.  The output of this code fragment shows the size of each.  

    int integer;
    char character;

    printf("Size of int = %d and size of char = %d\n", sizeof(int), sizeof(char));

As output from the `printf` function call, we get

    Size of int = 4 and size of char = 1

Integers are 4 bytes long and characters are 1 byte.  So, if we stored them in the same memory word, they would like this:

IMAGE

If we assigned `integer` to have the value 65, and we overlaid the two variables, it would look like this in binary:

IMAGE

From this example, we can see that we can assign 65 to `integer` and `character` would have the value 'A'.

This is what happens with unions.  Unions are declared just like structs.  Consider the example below:

    union example1 {
        int integer;
        char character;
    }  ex1, ex2;

In this union declaration, there are two variables, each of which are made up of an integer overlaid with a character in memory.  Each variable takes up one memory word, even though there are two fields declared within it.  This declaration mirrors our example above.

Using the above example, we can work with the union using dot notation as we have before.

    ex1.integer = 65;
    printf("Character = %c\n", ex1.character);

The output of this printf statement is the letter "A" *even though we made no assignment to that field of the union*.  Because the parts occupy the same memory space, assigning a value to `ex1.integer` automatically makes an assignment to `ex.character`.

Unions occupy space for the largest data item contained within them.  For example, if an array stored in a single variable is the largest single declaration, all other fields will be stored in the space allocated for that array.  Consider this code: 

    union example2 {
        int bigarray[50];
        struct inner {
            char label[20];
            float cost;
        } in;
        float computations[10];
    } ex3;

There are three fields here: an integer array, a struct, and a float array.  They will all occupy the same memory space.  This means that the following code will make intertesting assignments to the `bigarray` field:

    strcpy(ex3.in.label, "Union Fun!");
    ex3.in.cost = 17.1;
    int i;
    for (i=0; i<49; i++) printf("int at %d = %d\n");

This code will  print the following output (let's only consider the first several lines):

OUTPUT

Unions have a limited but important usefulness.  They are very useful to extract information from packed data.  For example, consider the format of a pixel in an image.  In 32 bits, a pixel packs 4 values, depicting red, green and blue colors, and a transparency (alpha) value:

   IMAGE

In chapter 12, we will discuss the many ways we can manipulate bits, but remembering shifting and boolean operations, we can extract the information in a pixel like this:

    int pixel = get_pixel(…);
    int blue = pixel & 0xFF;
    int green = pixel >> 8 & 0xFF;
    int red = pixel >> 16 & 0xFF;
    int alpha = pixel >> 24 & 0xFF;

We could also get the same values by using a union like this:

    union {
        int pxl;
        struct {
            char blue;
            char green;
            char red;
            char alpha;
        } parts;
    } pixel;

    pixel.pxl = get_pixel(…);

Once a value is assigned to `pixel.pxl`, we can reference `pixel.parts.blue` or `pixel.parts.red`.  We use `char` in the declarations, because it is 8 bits wide and four of them fit neatly inside an integer, splitting it into the necessary parts.  In this example, if `pixel.pxl` gets a value of `0x1020A0F`, then `pixel.parts.blue` will have the value `15`; `pixel.parts.green` will have the value `10`; `pixel.parts.red` will have the value `2`; and `pixel.parts.alpha` will have the value `1`.

> **History of Unions**
> 
How and why did unions come to be implemented in C?  Given how close some features of C come to assembly and machine language, it's probably not surprise that something like unions ended up in C.  They can be used to easily dissect data formats and example how data is represented.
>
Unions go as far back as the language COBOL.  COBOL was invented in 1959 and uses the `RENAMES` keyword to implement a union-type of declaration.  Algol 68 also influenced the creation of unions.  Despite being invented in 1971, C did not adopt unions until 1976, when it was introduced with *typedef* and some other interesting type definitions.  
>
More history of the C language can be found in a paper by Dennis Ritchie, [which can be found here](https://www.bell-labs.com/usr/dmr/www/chist.html).

### Using Enums ###

When you program in C, you use integers frequently.  Integers are very useful for many things, but they are not very descriptive.  For example, you could use integers to represent directions on a compass, with "1" meaning "NORTH" and "2" meaning "EAST", etc, but even if you documented this with comments, you would be likely to forget this from time to time.  Connecting the value "1" to "NORTH" is just not obvious.

There are ways to make this better.  One way is to use a *macro*.  Macros are textual elements that can be stand for other  textual elements and are expanded in C code by the C preprocessor (we will discuss the C preprocessor in Chapter XX).  To work with compass directions,  we could make the following definitions:

    #define NORTH 1
    #define EAST 2

and so forth.  This would work fairly well; we could work with these definitions like this:

    int direction = get_compass(…);
    if (direction == NORTH) {
        ….
    }

There are still some issues with this approach.  We still define a "direction" as an integer.  This applies to declarations, as well as to function definitions and program code.   It is still easy to forget that, when something is defined as an integer, you need to think of it as a compass direction.  In addition, because macros are replaced by the C preprocessor before the compiler gets the code, all debugging information about compass directions will be handled as integers.  Finally, because macros definitions are replaced before compilation, they are, in a sense, global to the entire program and cannot be subject to accessibility rules.

A better way to make this kind of definition would be to create an entirely new data type.  To do this, we need to define values and operations for that data type.  *Enum* types in C will take care of this quite well.   

Enums are declared with the following form:

<pre>
enum [<it>tag</it>] { <it>constant_list</it> } 
</pre>

As an example, we can define our compass directions as follows:

    enum Compass { NORTH, EAST, SOUTH, WEST };

We would use this definition like this:

    enum Compass direction = get_compass(...);
    if (direction == NORTH) { ... }

Now we have a more description, if not more verbose, declaration for `direction`.  As long as the function `get_compass` returns something of the Compass enum type, this code works.

In the interest of complete disclosure, enums in C are actually integers.  This should not be surprising, since many language elements are implemented with integers.  And the C language design is quite transparent about these integer implementation.

So, in terms of a new data type, we have the values in an enum defined, and we have the operations to complete the data type definition.  Because enums are integers, integer operations apply to them.  So, while it makes very little sense, we can add "NORTH" to "EAST" and get "EAST", because `0 + 1 = 1`.  We can even influence the assignment of values to our enum constants.  Here is an example:

    enum Months { January=10, February=20, March=100, April=200, 
                  May, June, July, August, September, October, November,
                  December=1000 } calendar;

With this declaration, the constant `January` will be represented with the integer `10`, `February` with `20`, `March` with `100`, `April` with 200, `May` with `201`, `June` with `202`, `July` with `203`, etc.  When there is not a specific assignment, the compiler will assign a value that is 1 greater than the previous value.  Without any specifications, the first value in the constant list has the value 0.  

There are other issues with enums.  TOO BIG (http://stackoverflow.com/questions/17125505/what-makes-a-better-constant-in-c-a-macro-or-an-enum)

> **Enums or Macros?**
>
If enums are useful but not completely a new data type and macros are useful but not completely a new data type, which is better?  In general, it's better to use language contructs than preprocessor substitution, but in this case, there is really no clear winner.
>
The most convincing reason to use enums over macros is readbility.  It makes more sense to declare a function to take a "Compass" data type for a parameter than an "int" for the same parameter.  "Compass" is simply more descriptive.  However, one can use a typedef (see next section) to make typename aliases.  So a macro with a typedef can be just as descriptive.
>
An enum is implemented as a signed int data type.  This means that there are several situations where enums cannot be used.  If you need an unsigned int, or a number larger than `sizeof(int)`, then enums will not work.  And, you cannot have enums stand for non-integer values.  Macros, on the other hand, when combined with a typedef, can be used in any situation.  
>
So…there really is no wrong answer to the "which is better" question.  If you still seek an answer to this, staying with language features instead of macro expansion is probably better, if only for "purist" reasons.  

### Typedefs Make Declarations Easier ###

This chapter has described several ways to structure data.  Each structured data method is described by a declaration, then variables are declared to have the described structure.  These declarations can be a bit verbose and typedefs are here to help declarations to be more succinct.

To see how declarative verbosity gets in the way, let's reconsider the example we discussed in the struct section.  We described a structure that could be used for application properties:

    struct app_props {
        char appname[25];
        float cost;
        long when_purchased;
        int days_used;
    } watchapp3, watchapp4;

We can use this to declare pointers to structures and to allocate memory for these structures this way:

    struct app_props *props = (struct app_props *)malloc(sizeof(struct app_props));

That's a lot of verbose repetition.  

A typedef defines names that can stand in the place of other declarations, perhaps serving to reduce wordiness or to clarify syntax.  A typedef looks like this

<pre>
typedef <it>type_description</it> <it>substitute</it> 
</pre>

Once a typedef name is defined, it can be used anywhere the *type_definition* would be used.  

For our example above, we could use this typedef:

    typedef struct app_props properties;

Using this definition, we can streamline the memory allocation like this:

    properties *props = (properties *) malloc(sizeof(properties));

To be honest, this is still wordy and repetitive, but it's better.

Another reason for using typedef definitions is to ensure compatibility as software changes.  If definitions change, typedefs with the appropriate "old" definition can make designs use the same declarations with newer, updated software.

### Avoiding Structured Messes ###

We have seen a lot of messy programming in C.  Now that we have explored structured types, we can get even structured messes.  Here are some tips that will help avoid messy programming with structured data types.

1. Avoid anonymous declarations.  Good solid naming conventions are essential to clear code.  Use tagged structs to assist in typing and in reading code.
2. Initialize all parts of structs.  We have said this before, but now the problem has multiplied.  We now have variables with lots of parts, collected into a struct package.  Make sure they all are initialized before you use them.  This *especially* applies to dynamically allocated structures.  It's easy to forget to initialize them, because they are not in a declaration statement.
3. While you can use partial references to structures, use complete references whenever possible. This especially applies to initializing larger structures.  It's easy to lose sight of the parts that are being initialized, so use full references to remind you.
3. Use unions *very* sparingly.  Unions have their place, but those use cases are few.  As always, be obvious and clear when manipulating data. 

### Project Exercises ###

#### Project 10.1 ####

Let's start by creating a watchface.  Recall Project 8.2: we generated random digits and drew them on the smartwatch screen.  Now let's replace the random digits with the time.

Start with [the answer to Project 8.2, available here.](https://cloudpebble.net/ide/import/github/learning-c-with-pebble/project-8-2-answer)  Work through the following changes to the code.

1. In the function `canvas_update_proc`, remove the four `choice = rand()%10;` statements.
2. Locate the `main_window_load` function and add the following call to subscribe to the "tick timer", making a call to `tick_handler` every minute:
       tick_timer_service_subscribe(MINUTE_UNIT, tick_handler);
3. Add the following code in `main_window_unload` to unsubscribe from the "tick timer":
       tick_timer_service_unsubscribe();
3. Add `tick_handler` before `main_window_unload`:
       static void tick_handler(struct tm *tick_time, TimeUnits units_changed){
          layer_mark_dirty(s_canvas_layer);
       }
4. Finally, in `canvas_update_proc`, add these statements before the drawing of the 4 digits to get the current time:
       time_t now = time(NULL);
       struct tm *t = localtime(&now);

Now, we have the time in a pointer to `struct tm`.  This is a well-defined struct; [look up the contents of this structure here](https://sourceware.org/newlib/libc.html#Timefns).  It contains many time elements; we are only concerned about the hour and the minute.

Now, make the following changes to the code:

1. Separate the tens digit and the ones digit for the hour and the minute.  You will need to reference the struct elements through the pointer `t`.  
2. Using the same `draw_digit` calls as the starter code, draw these digits in the right place.  

Now you have a working watchface.  [See the answer to this project here.](https://cloudpebble.net/ide/import/github/learning-c-with-pebble/project-10-1-answer)

**Extra Challenge:** Add a seconds indicator.  Change `MINUTE_UNIT` to `SECOND_UNIT` in the call to `tick_timer_service_subscribe`.  Then in `canvas_update_proc` draw a block with colors that alternatively draws and erases every other second.

#### Project 10.2 ####

Snake is a game where a snake moves around the screen, directed by user input. The user moves the snake to eat some fruit, which causes the snake to grow. If the snake crosses the edge boundary or crosses itself, the game ends. It's a basic game most devices with good graphics will implement.  It's a sort of "Hello World" for user interaction.  

[Find starter code for a snake game here.](https://cloudpebble.net/ide/import/github/learning-c-with-pebble/project-10-1).  It's based on [an original Snake game by Nick Reynolds](https://github.com/thenickreynolds/pebblesnake) for the Pebble Classic smartwatch.  Take a few moments to review the code.  Run the code to make sure you know it works.  Answer these questions as you review the code.

1. Notice all the lines that begin with #define.  These are preprocessor statements (see Chapter 13) that define textual substitutions. How are all these #define statements used?  
2. Find the structs in the code: one for `Position` and one for `Snake`.  They are declared with a mix of typedef with an unnamed struct. Why do you think this declaration method was used?
3. There are no enum declarations here.  Which #define statements would be suitable for an enum?
4. In many of the functions, parameters that are `Position` types are declared as pointers.  Why would this be helpful (or necessary)?

Change the code in the following way.

1. Establish a direction enum to depict the values `UP`, `RIGHT`, `DOWN` and `LEFT`. You also have to change `DIRECTION_COUNT`?  How does that have to be declared?  Does the code have to change any further?
2. Make an enum for `CLOCKWISE` and `COUNTER_CLOCKWISE` values.  Again, how much does the code need to change?
3. You are to add an "autopilot mode" to the game.  When the "select" button is pressed, the game toggle between autopilot mode and regular mode.  Autopilot mode means that when the snake comes up to the edge of the screen, it automatically switches positions without an error. It also grows when it hits the edge until the snake is a certain length (determine this yourself), then it resets.  The up and down buttons should be ignored in autopilot mode.

[An answer to this project can be found here.](https://cloudpebble.net/ide/import/github/learning-c-with-pebble/project-10-2-answer)

**Extra Challenge #1:** In autopilot mode, the snake should not turn in one direction only.  Make sure the snake turns randomly in either direction.  Decide the probability of turning clockwise or counter clockwise.

**Extra Challenge #2:** In autopilot mode, the snake will eventually follow the edge of screen.  Help the snake by making it turn at the fruit position as well.  Make it turn toward the fruit.

[An answer to these challenges can be found here.](https://cloudpebble.net/ide/import/github/learning-c-with-pebble/project-10-2-challenges-answer)

#### Project 10.3 ####

After Project 10.2, we have a snake that can go on autopilot.  Using your knowledge of the time struct and your new snake skills, let's make a snake watchface.  

Start with the answer code from Project 10.2.  Change your code to make the snake *always* work in autopilot mode.  Remove any random direction turning; only turn in one direction.  However, make the snake turn at the fruit position.

Draw the time on the snake's tail.  You will have to add code to `tick_game()` so that the time is drawn after the tail and follows the snake. Try to keep the numbers in the proper orientation!

[An answer to this project can be found here.](https://cloudpebble.net/ide/import/github/learning-c-with-pebble/project-10-3-answer)

**Extra Challenge:** Make the time digits part of the snake's body.  Use either the first squares or the last ones.

#### Project 10.4 ####

Recall Project 8.4: the bouncing rectangle.  It bounced and randomly jumped all over the Pebble screen.  [You can find the answer to this project here.](https://cloudpebble.net/ide/import/github/learning-c-with-pebble/project-8-4-answer)

Locate the function `update_display`.  In this function, the fill color for the rectangle is set to `GColorWhite`.  You are to change this to gradually change the color of the rectangle through reds, greens, and blues.  Do this in the following way:

1. Set up a union that can change colors using the Pebble definitions.  [Check this link for the way color definitions are declared. ](https://developer.pebble.com/docs/c/Graphics/Graphics_Types/Color_Definitions/) 
2. Set up the code to reset the color to `GColorWhite` when the "select" button is pressed.
3. Every time the `update_display` is called, change the color just a bit and use this new color in the `graphics_context_set_fill_color` function.

[An answer to this project can be found here.](https://cloudpebble.net/ide/import/github/learning-c-with-pebble/project-10-4-answer)

**Extra Challenge:** Instead of computing colors here, can you use an enum and simply step through colors?  Could that enum be based on the Pebble `GColor` type?
