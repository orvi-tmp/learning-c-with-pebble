Chapter 5: Loops and Iteration
=======
As you invent solutions for various programming challenges, there will be many times when you must execute a statement many times.  This kind of iterative code is very common and C provides several choices on how to execute statements in a loop. 

In this chapter, we examine how C supports statement iteration via looping structures.  Loops provide a way to repeat blocks of code and to control that repetition.  We will also briefly consider the goto statement, an infamous programming feature that was around before C was invented.  

### Looping Concepts ###

C provides different types of loops for different situations.  First, there are conditional and counted loops. A *conditional* loop is used when the end of iteration is dependent on a condition -- and can't be known ahead of time. A *counted* loop is used when you know -- or can compute -- the number of times a loop will execute.  

C also provides loops that differ on where the decision to terminate the loop is tested.  *Pretest* loops test for termination before the body of code in the loop is executed.  *Post test* loops test for termination at the end of loop code execution.  Pretest loops may never execute their code, because the first termination test might succeed.  Likewise, post test loops will *always* execute their code at least once, because their first termination test is after the first pass through the loop code.

C gives us while and do/while loops as conditional loops and for loops as counted loops.  It also has while and for loops as pretest loops and the do/while loop as a post test loop.  The goto can be any kind of the loop you want...which we will see can be a problem.

### The While Loop ###

The while loop has the following standard form:

<pre>
while (<i>condition</i>)
     <i>statement</i>
</pre>

The *condition* is the same boolean expression-based condition we have seen before with the if statement.  The loop evaluates the *condition* and, if the evaluation produces a true value, executes the statement.  Then it evaluates again.  It continues the evaluation/execution pattern until *condition* evaluates to a false value.  

Let's consider this example.  

    int x = 10;
    int y = 20;
    while (x > 0) {
        y ++;
        x /= 2;
    }

In this simple example, the condition `x > 0` is evaluated repeatedly and, as long as it's true, the code block will be executed. Eventually, `x/2` will give a zero value, because of integer arithmetic, which will cause the condition to fail and the loop to stop.

Let's take another example.  Let's say we want to compute a factorial.  Assume we have an integer value stored in the variable `number` and we want to compute a value (the factorial of `number`) which will be stored in the variable `factorial`.  This while loop should do the job: 

    factorial = 1;
    while (number > 0) {
        factorial = factorial * number;
        number--;
    }

In this code, assuming `number` is non-negative to begin with, decrementing the variable each iteration will eventually make it equal to zero and the loop will terminate.  

As a pretest loop, the while loop might never execute it's loop statement code.  In the above example, if `number` were negative or zero before the loop statement, the loop would terminate without execution.

### The For Loop ###

The for loop is a pretest loop that you would use when you know, or can compute, the number of iterations in a loop.  The for loop, in fact, is a while loop in disguise.  You can always interchange the two.  

The for loop has this form:

<pre>
for (<i>initialization</i>; <i>continuation condition</i>; <i>increment</i>) 
      <i>statement</i>
</pre>

We can rewrite a for loop as a while loop, which may make it easier to understand.  We can rewrite it as a while loop this way:

<pre>
<i>initialization</i>
while (<i>continuation condition</i>) {
    <i>statement</i>
    <i>increment</i>
}
</pre>
  

Consider this simple example:

    for (int i=0; i<10; i++) {
        sum += i;
    }

This loop initializes `i` to zero, then tests the condition `i<10`.  While the condition evaluates to a true value, the loop statement executes, followed by the increment `i++`.  Then we test the condition again.  Eventually, `i` will take on the value 10 and the loop will terminate.

For another example, we can rewrite the factorial code from the previous section as follows:

    for (; number > 0; number--)
        factorial = factorial * number;

Notice we left the *initialization* part empty.  Our example assumed we already had a number value, therefore we did not need to initialize anything.  The loop will work without any kind of initialization.

In fact, we can leave all parts empty:

    for (;;) { ... }

This would be an infinite loop we would have to break out of from in the loop body.

We mentioned that for loops are just while loops.  We can rewrite our first simple example to be represented as a while loop like this:

    int i=0;
    while (i<10) {
        sum += 1;
        i++;
    }

The semantics between the two forms is the same.  

If it takes multiple statements to complete any part of the for loop specification, you can combine statements with commas.  For example:

    for (x=2,y=3; x+y < 20; x++,y++) {...}

This code will execute the initialization assignments *in order*, which might make a difference if the statements between the parentheses depend on each other.  So the above code is the same as executing this code sequence:

    x = 2;
    y = 3;
    for (; x+y < 20; ) {
        ...
        x++;
        y++;
    }

Finally, let's note that a for loop is a code block, which is especially important to know for name accessibility.  Consider the loops below:

    for (int row=0; row < total_rows; row++) {
        for (int column=0; row < total_columns, column++) {
            if (color_distance(get_pixel(row, column), GColorBlue) < 5) {
                setPixelColor(row, column, GColorBlack);
            }
        }
        printf("We ended up on column %d\n", column);
    }
    printf("We finished on row %d\n", row);

Here we have two loops, one nested inside the other, working on the pixels of an image.  Both of the `printf` function calls have errors.  For the inner `printf`, the `column` variable is not accessible outside the for loop.  Likewise for the outer `printf`, the variable `row` is not accessible outside the outer for loop.  The reason for this is that it is declared in the for loop specification.   If we had pulled the declaration of the variables outside the loops, everything would be correct, as below:

    int row, column;
    for (row=0; row < total_rows; row++) {
        for (column=0; row < total_columns, column++) {
            if (color_distance(get_pixel(row, column), GColorBlue) < 5) {
                setPixelColor(row, column, GColorBlack);
            }
        }
        printf("We ended up on column %d\n", column);
    }
    printf("We finished on row %d\n", row);

It is best to keep accessibility of variables and other names as tight as possible.  However, with for loops, sometimes that rule conflicts with convenience, so it's up to the programmer which method to choose.

> **Variable Name Accessibility And Other Habits**
> 
C is an extremely flexible language that allows many good and bad programming elements.  A programmer needs to build good habits to navigate programming in C.  Variable name accessibility is a good example.  Keeping a tight boundary around the area where names are used is a good habit to get into.  Using that rule means that if you accidentally use the variable where you *thought* it was accessible, but it really wasn't, the compiler error will alert you to your mistake.  Compiler errors are much easier to fix than execution errors later.
>
Other habits could be declaring variable names together at the beginning of code sections and using curly braces to implement code blocks even when one statement is in the block.  We will point out others as we develop our understanding of C.

### The Do/While Statement ###

We have mentioned that while loops and for loops are *pretest* loops.  The condition for termination is tested before the loop code and it's possible that the loop code will never execute.  Sometimes, an algorithm makes more sense if the condition is tested after the loop code executes.  We have called this a *post test* loop.  

C implements this with a do/while loop.  The form of a do/while loop is below.

<pre>
do
    <i>statement</i>
while (<i>condition</i>);
</pre>

This loop will execute the *statement* as long the *condition* returns a true value.  

Let's take another look at the factorial example.  We can write for do/while loops as follows:

    factorial = 1;
    do {
        factorial = factorial * number;
        number--;
    } while (number > 0);

This computes a factorial just like the original example, except for the case when `number` is equal to 0.  Then this will give a result that is in error (`factorial` will be equal to 0; we know that 0! = 1).  So, the factorial example is a good example of an algorithm that needs the condition checked before loop execution, when it's possible that no loop execution will happen.

Let's look at another example.  Let's write some code that will compute the first *n* numbers in a Fibonnaci sequence, where each term is equal to the sum of the two previous terms.  Note that this assumes that *n* is greater than 0, which means that at least 1 term will be generated, which means we will execute loop code at least once, which means we can use a do/while.

     int first=0, second=1, next;
     do {
         next = first + second;
         first = second;
         second = next;
         printf("%d\n",next);
         number --;
     } while (number > 1);

Here, `number` is assumed to have a value greater than 0 and is the number of Fibonnaci terms to produce.  

### The Break and Continue Statements ###

Occasionally, it is necessary in an algorithm to stop a loop prematurely or to skip an execution of a loop body, moving to the next iteration.  The break and continue statements are used for these situations.

The break statement will terminate the execution of a loop.  Whenever the break statement is encountered, everything is terminated and execution continues after the loop statement.  As an example, we could repair our do/while loop implementation of factorials this way:

    factorial = 1;
    do {
        if (number == 0) break;
        factorial = factorial * number;
        number--;
    } while (number > 0);

Here, we check `number` for quality to zero.  A true evaluation will shut down the loop and start execution at the next statement after the while. 

The continue statement simply terminates the current execution of loop code and starts the next iteration.  A simple example might be code that counts by twos:

    int sum=0;
    for (i=0; i<20; i++) {
        if (i%2 != 0) continue;
        sum += i;
    }

This code skips the summation for every time `i` is not even.  Of course, by using `i += 2` instead of `i++` in the for loop set up specification would mean we don't need the continue line.  But it's a good example.

### Infinite Loops ###

An infinite loop is a loop whose termination condition is *never* true.  Most often, infinite loops are an error by a programmer who actually meant something different.

Consider this example: 

    for (int x=100; x<50; x++) {...}

this code would be an infinite loop because `x` will never be less than 50. 

Most of the time, infinite loops are not so obvious.  Try this one:

    for (int y=10; y=100; y++) {...}

This is also an infinite loop.  The use of the assignment statement instead of the an equality for the continuation condition will always result in a true value (the assignment statement returns the value `100`, which is true). 

Here's another common error:

    int n=100;
    while (n > 50);
    {
        sum += n;
        n--;
    }  

In this example, the semicolon after the while specification will cause the while loop to run an empty statement infinitely.  

Most infinite loops happen because of errors in your code. Perhaps, like above, an assignment is used instead of an equality.  Or, perhaps, the condition used in a while loop is always true because of code somewhere else.  Infinite loops can be hard track down.  The key is to work with the variables in the loop conditional parts and make sure you know their values.  

Occasionally, however, an infinite loop can be useful, especially when combined with a break statement. There are situations where the decision to terminate a loop must occur in the middle of the loop body, not at the beginning or the end.  In these cases, a break statement is needed -- with an if statement attached -- to terminate the loop.  

## The Infamous Goto Statement ###

The C language supports a goto statement.  Goto statements include a label that marks some other statement.  When a goto statement is encountered, execution is paused, then resumed at the statement marked with a label.  The general form is:

<pre>
goto <i>label</i>;

...

<i>label</i>: <i>statement</i>
</pre>

The *label* name format must adhere to C naming rules (like variable name rules) and must be accessible.  

Goto statements can jump forward or backward.  Consider this example:

    int x=10;
    LOOP: while (x < 20) {
        x++;
        if (x == 15) {
            goto LOOP;
        } else {
            printf("The value of x is %d\n", x);
        }
    }

This code will print values of `x` from 10 to 19, but it will skip 15.  When `x` equals 15, the goto statement will force execution to start back at the top of the loop.  The behavior here is like a continue statement.

In fact, we can use a goto statement to rewrite many code structures.  We can implement the while loop from the above example this way:

    int x=10;
    LOOP: if (x >= 20) goto LOOPEND;
    x++;
    if (x == 15) goto LOOP;
    printf("The value of x is %d\n", x);
    goto LOOP;
    LOOPEND: ...

which is semantically equivalent.  However, this code also points out problems with the goto statement.  The biggest problem is that code can look chaotic with goto statements.  Compare the two examples above.  The while loop is much more expressive about the algorithm being described than the goto implementation.  In fact, if we eliminated the goto entirely, it would be much more elegant and expressive:
 
    int x=10;
    while (x < 20) {
        x++;
        if (x != 15) {
            printf("The value of x is %d\n", x);
        }
    }

The problem with the goto statement is that it encourages very bad code.  As threads of execution start to wind around blocks of code, a program can be very hard to follow.  *Structured programming constructs*, like while loops and for loops, were invented to eliminate the goto statement with statements that directly addressed the logic that the goto was implementing.  

In addition to code that is hard to understand, the goto statement poses some very difficult problems when it interacts to more structure program components.  For example, consider this code:

    int x=10;
    LOOP: while (x < 20) {
        x++;
        if (x == 15) {
            goto LOOPEND;
        } else {
            printf("The value of x is %d\n", x);
        }
    }
    LOOPEND: x = y-2;
    y++;
    goto LOOP;

This code forces a jump to code *outside* the loop, then continues the loop.  This is very bad form and is quite hard to understand.  

For these reasons -- and some others -- the goto statement is considered to be a statement that should never be used.  There are some languages that don't even include a goto statement.  Suffice it to say you should never use it and that every situation where you think a goto could be used can be addressed by another language construct.  

> **More on the GOTO Statement Evil**
>
The goto statement has long been considered "harmful" by computer scientists and language designers.  Back in 1968, computer pioneer Edsger Dijkstra [wrote a letter to the editor](http://homepages.cwi.nl/~storm/teaching/reader/Dijkstra68.pdf) of *Communications of the ACM* that became a famous treatise against the goto statement. The campaign against the goto statement is, in part, what led to structure programming languages.  These languages built structures -- like while loops and switch statements -- that make goto statements unnecessary.  [Dijkstra also has come things to say about that too.](http://www.cs.utexas.edu/~EWD/ewd13xx/EWD1308.PDF)
    
### Avoiding Messy Loop Code ###

C provides many opportunities for messy loop code.  Because integers can stand for conditions and for loop specification can have many forms, code can get very difficult to read.

Consider this example:

    for (; y<z++, printf("%d\n",z); ) ;
        
Here we put all the loop code in the continuation/conditional part of the for loop.  The comma allows us to put `y<z++` and the `printf` function call together.  In addition, we have used `z++` in a conditional statement.  We could rewrite this as

    while (y < z) {
        z++;
        printf("%d\n",z);
    }

This code is much clearer.

Consider this code:

    do{ not* look *at;
        me; for(;!'a';);}while(1);

Given the right variable declaration, this is valid C code.  This code has the same effect as the following:

    do { } while(1);

which is an infinite loop.  All the internal loop code are either expressions or a for loop that will stop immediately

Here are some guidelines about looping code.

1. Only use computations in the conditionals and specifications of loops.  **Do not use assignments.** It will be very tempting to use assignments when we get to C code that reads file input, but stand firm against the temptation.

1. For loops have a specific use -- as counted loops.  When a for loop has an empty specification, consider using a while loop instead.

1. Code formatting is important to readability.  Indent wherever you can.

1. If at all possible, avoid break and continue statements.  In a sense, they are substitutes for the goto statement and can also result in convoluted logic that is hard to follow.  

### Project Exercises ###

For a chapter on loops, we need exercises that demonstrate how loop might be used on a Pebble watch.

#### Project 5.1 ####

Let's start with Project 5.1.  [Click here to get the project into your CloudPebble editor.](https://cloudpebble.net/ide/import/github/programming-pebble-in-c/project-5-1) Run the project and see what it displays on the Pebble.  It generates a random decimal number and displays that number.  There's a second number that displays, but it's just a set of zeroes.

Instead of zeroes, display the random decimal number in binary.  To do this, we will need a loop that will iterate over the following steps:

1. AND the random number with 1.
2. Display the result.
3. Shift the number right.

We will have to do this until the decimal number equals 0.  Note that you are to display each binary digit as you figure it out, not save up the whole number and somehow display the whole thing at once.  This will mean you have to compute the (x,y) position of each digit as you display them. 

In the `rand2bin.c` file, locate the `update_display` function.  There is a declaration of `number` in that function; add your code under that declaration.  You can use the `display_bit` function to display each bit as it is computed.  Keep track of an X coordinate for each of the bits.  (Hint: start at 124, and work right-to-left by decrementing your X coordinate by 8 for each bit.) 

Make sure you add comments to your code and add your name.

You can find the answers and implementations for the code [above at this link.](https://cloudpebble.net/ide/import/github/programming-pebble-in-c/project-5-1-answer)

#### Project 5.2 ####

Now let's look at Project 5.2.  Click here to get the code for the project.  This project gets an image from the Internet and displays that image on the Pebble screen.  Look at the code in the function `get_and_display_image` to see how that's done.

For this project, We are going to change the color of the person's hair.  To do this, we will use a function called `color_distance` that will compute the distance between two colors.  When that distance is small, say 10 or less, we will say the two colors are the same.  

Look for the function `change_hair_color`.  It is empty; there is no code between the curly braces.  You are to add code that will use the `color_distance` function to check each pixel and change the hair color to `GColorRed`.  You will need two nested loops, one for a row of pixels and one for each pixel in a row.  

Now, did other pixels change in addition to the hair?  This is probably because some parts of the image have the same color as the hair does.  We need to restrict the view to a box that examines a smaller region, just the area where the hair is.  Change the loops so that the area examined is closer to the box where the hair is.

You can find an implementation for this project at this link.

#### Project 5.3 ####

Let's do one more project.  Click here to get the starting code for Project 5.3.  This code displays a message at the top left of the Pebble screen.  There is a function called `fill_screen` that only displays a single message.

You are to rewrite the `fill_screen` function so that it completely fills the Pebble screen with a message repeated over and over.  See the image below for an example.

IMAGE

Notice that the messages are placed neatly next to each other both horizontally and vertically.  

You should be able to use two loops, one for the row and one for the column.  However, these loops will not likely be for loops, because the (x,y) coordinate of the beginning of the message will change based on message length and font size.  

Now that you have used two loops, can you rewrite the code using only one loop?

You can find an implementation of the two loop solution here, and an implementation of the single loop solution here.
