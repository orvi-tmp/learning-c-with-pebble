Chapter 4: Making Decisions and Conditional Execution
=======
There is a theorem in programming language theory that states that there are only three types of statements needed for programming.  In a 1966 paper, Corrado Böhm and Giuseppe Jacopini show that we only need sequential execution, conditional execution of statements, and looping repeated execution of statements to write a program. 

We have already described sequential execution in a C program in Chapter 3.  In this chapter, we will discuss conditional execution and its various forms.  We will discuss loops and iteration in Chapter 5.

> **Böhm and Jacopini**
> 
The Böhm/Jacopini theorem can be found in "Flow diagram, Turing machines, and language with only two formation rules" in *Communications of the ACM*, Vol. 9, May 1966.  It's discussed as a "folk theorem" (read that as a theorem on the level of folklore) in [this paper by David Harel](http://www.wisdom.weizmann.ac.il/~dharel/SCANNED.PAPERS/OnFolkTheorems.pdf). In addition, it's been argued that the theorem is not correct; see [this paper by Kozen and Tseng](http://www.cs.cornell.edu/~kozen/papers/bohmjacopini.pdf).

### Conditions and Comparisons ###

Conditional execution is based on boolean decision making: should code block #1 or code block #2 be executed?  This decision making is binary, based on true or false values.  Therefore, in order to discuss conditional execution, we have to first define what conditions are in C.

As we discuss conditions, it is important to remember what we discussed about booleans in Chapter 3.  There is no boolean data type in C; C considers an integer value of 0 to be "false" and every other integer value to be "true".  This means that 

    if (x != 0) {...}

is the same as using

    if (x) {...}

We will discuss pitfalls and the messy ways we can express boolean values at the end of this chapter.  

#### Comparisons ####

There are six ways to compare values in C, listed in the table below.

<table>

<tr>
<td><b>Name</b></td>
<td><b>Symbol</b></td>
<td><b>Description</b></td>
</tr>

<tr>
<td>Greater than</td>
<td><code> > </code></td>
<td>x > y is true when x has a greater value than y and is false otherwise</td>
</tr>

<tr>
<td>Greater than or equal to</td>
<td><code> >= </code></td>
<td>x >= y is true when x has a greater value than y or is equal to y and is false otherwise</td>
</tr>

<tr>
<td>Less than</td>
<td><code> &lt; </code></td>
<td>x < y is true when x has a lesser value than y and is false otherwise</td>
</tr>

<tr>
<td>Lesser than or equal to</td>
<td><code> &lt;= </code></td>
<td>x &lt;= y is true when x has a lesser value than y or is equal to y and is false otherwise</td>
</tr>

<tr>
<td>Equals</td>
<td><code> == </code></td>
<td>x == y is true when the value of x is equal to the value of y and is false otherwise</td>
</tr>

<tr>
<td>Not equal</td>
<td><code> != </code></td>
<td>x != y is true when the value of x is not equal to the value of y and is false otherwise</td>
</tr>

</table>

Comparisons act as you would think they would, except C implements boolean values with integers.  So, trying to print a boolean value gets an integer result:

    int a = 1;
    int b = 2;
    printf("a<b is %d\n", a<b);

This code will print `a<b is 1`.  (Note the `printf` function will be used to print values; we will define it detail in Chapter 6.  For now, think of the string as defining a template that the rest of the value fit into.)  Changing the comparison will have the expected effect:

    int a = 1;
    int b = 2;
    printf("a==b is %d\n", a==b);

This code will print `a==b is 0`. 

Note that equality is expressed as `==` rather than `=`.  The `=` is already reserved for the assignment operator, so `==` is used.  However, it is probably one of the most insidious of errors to accidentally use the assignment operator for the equality comparison.  This is because of the use of integers for boolean values.  Refer to the section on "Big Messes" below for what happens when the symbols are interchanged.

#### Combining Comparisons ####

Comparison operations can be combined using *logical operators*.  We covered boolean expressions in Chapter 3 and here is a place where they become very applicable.  

Let's consider an example.

    int red = 0xFF0000;
    int green = 0x00FF00;
    int blue = 0x0000FF;
    int yellow = 0xFFFF00;
    int violet = 0xFF00FF;

    int true_colors = red+blue == violet && (red+green == yellow);
    int fake_colors = !(red+green == yellow) || !(red+blue == violet);

Here, we have combined several comparisons on colors (yes...it's a bit contrived).  The `&&` operator will be true only if both comparisons are true (which they are).  The `||` operator will produce a true value if at least one of the two "notted" comparisons are true (both sides produce a false value). 

Note the precedence in the above example.  Especially because they produce integer values, logical operators in C must be included in precedence rules.  In the computation of `true_colors` above, the `+` operator has precedence over the logical operators.  In the computation of `fake_colors`, the `!` operator is immediately associated with the expression to its right.  

Let's consider again an example from Chapter 3:

    int retired = false;
    int still_working = true;
    int gets_a_pension = (retired && ! still_working) || (age > 65 && hours < 20);

In Chapter 3, we had a diagram of the order of evaluation of the boolean expression:

<figure>
   <hr/>
   <img src='http://www.cs.hope.edu/~jipping/pebblebook/Figure3-1.png'>
   <figcaption>
      <b>Figure 4.1/3.1:</b> Computation Order for the Boolean Expression Example<br/>
   </figcaption>
   <hr/>
</figure>

Note that the parentheses served as grouping symbols, and the left-to-right association for the `!` operator made it the first operator to be evaluated.  The second and third steps were evaluating `age > 65` and `hours < 20` respectively.  This shows that comparison operators have a higher precedence than logical operators.

#### Short Circuiting ####

Consider the following example:

    float fuel_cost = 2.09;
    float fuel_level = 0.4;

    int do_we_buy_fuel = fuel_level < 0.25 || fuel_level < 0.5 && fuel_cost < 2.50;

Here, the decision to by fuel is based on two considerations: if the tank is below 25% full or the tank is below half and fuel costs less than 2.5.  Note that if the fuel level is below 25% we are going to refuel and we can skip the right hand side of the expression.  

This is an example of where we can *short circuit* the evaluation of the boolean expression. Consider that `false &&` *anything* has the value `false` and `true ||` *anything* has the value `true`.  Short circuiting the evaluation of a boolean expression means that we can stop evaluation when we are certain of the value of the outcome.  

C uses short circuiting in the evaluation of boolean expressions.  This is a very acceptable way to streamline evaluation and not waste execution time. 

Here's another example:

    if (a < b || c == x-21 && d+2 == e) 
        printf("We are here...\n");

In this example, we can find several ways to short circuit the expression evaluation.  

* If `a` has the value 10 and `b` has the value 20, `a < b` will evaluate to true and the rest of the expression will be ignored.  
* If `a` has the value 10, `b` has the value 5, `c` has the value 30, and `x` has the value 40,  `a < b` will evaluate to false as will `c == x-21`.  Evaluation will stop, because the left side of the `&&` operator guarantees the expression will evaluate to false.  

> **Short Circuiting Pitfalls**
> 
You have to be careful with short circuiting.  When a boolean expression is short circuited, the unevaluated portion of the expression is not even examined.  Bad values, bugs, and incorrect code can exist, but may not be revealed until the entire expression is evaluated.  Consider this example:
>
    int zero = 0;
    int five = 5;
    int ten = 10;
    int condition = ten / five < 3 || 10 / zero == zero;
>
When the right side of the `condition` assignment is evaluated, the left part of the expression will always be true, and will force the evaluation to never evaluate the right side of the `||` operator, which will cause a "division by zero" error.  Should the value of `ten` become `15` or greater, the right side *will* evaluate and the program will likely crash.  These kinds of conditional errors are extremely hard to track down and fix.

### If Statements ###

C has several statements that use comparison and boolean expressions to choose statements to execute.  The if statement is used the most.  It has the following two forms:

<pre>
if (<i>expression</i>) <i>statement</i>
</pre>

and 

<pre>
if (<i>expression</i>)
   <i>statement1</i>
else
   <i>statement2</i>
</pre>

#### If and Only If ####

The first form of the if statement can be called the "if and only if" statement.  The *statement* is executed if and only if the *expression* evaluates to a true value.  For example,

    if (x + y > z) a++;

If the sum of `x` and `y` is greater than the value of `z`, then `a` is incremented.  If the sum is less than or equal to `z`, `a` is not incremented.  

The boolean expression that the conditional execution hinges on is exactly the type of boolean expression -- the combination of comparisons and logical operations -- that we have seen in this chapter's first section.  In fact, remembering that boolean is the same as integer, you can really put any expression -- boolean or otherwise -- as the expression on which the execution of *statement* depends.  

Each of these are valid if statements

    if (a + b == c - d && f > g) x++;
    if (a && b) y--;
    if ( (miles_traveled < miles_requested) || 
         (miles_traveled - miles_requested == 0) ) 
            miles_reimbursed = miles_traveled;

As expressions get more complicated, it becomes very important to (a) use explanatory names and (b) format your code for readability.  Also, remember that short circuiting applies to boolean expression evaluation; in this example, if `miles_traveled < miles_requested`, the remainder of the expression will not be evaluated.

Because the if statement is itself a statement, you can *nest* if statements inside each other.  For example,

    if (x_coord1 < x_coord2) 
        if (y_coord1 < y_coord2)
            quadrant = 2;

In this example, the inner if statement forms the conditionally executed statement for the outer if statement.  The inner if statement is considered only if the expression for the outer if statement evaluates to a true value.  For this form of the if statement, using outer and inner conditions is the same as using a logical AND operator to combine the boolean expressions.  The example below behaves the same as the above example:

    if ( (x_coord1 < x_coord2) && (y_coord1 < y_coord2) ) quadrant = 2;

This is good place to remember that C uses the idea of *compound statements*.  Statements surrounded by `{}` brackets can be considered to be a single statement and, therefore, fit into the template of a if statement.  Consider this example.

    if ((position_x - ball_radius < 0)
        || (position_x + ball_radius > window_frame_width)) {
        x_velocity = x_velocity * -1;
        ball_radius ++;
    }

Here, we have two statements forming one statement when surrounded by `{}` brackets. The statements are indented inside the brackets to improve readability.

#### If-Else ####

The second form of the if statement uses an "else" part.  In this form, there are two statements: one that gets executed if the *expression* evaluates to a true value (*statement1*) and one that executes if the *expression* evaluates to a false value (*statement2*).  All of the semantics from the "if and only if" version apply, except that, in the false case, we add the execution of a second statement.  

Let's consider a simple example.

    if (checking_account < 100) {
        savings_account -= 100;
        checking_account += 100;
    } else {
        savings_account += 100;
        checking_account -= 100;
    }

Here, if the checking account balance is less than $100, we move money from the savings account to the checking account.  Otherwise, we shift $100 in the opposite direction.  Note that we do *something*, that is, we either execute the "true part" or the "false part".  

While this form of the if statement may seem straightforward, haphazard uses of "else" can make this confusing.  Consider this example:

    if (n < 10) 
    if (z > 4)
    x = 5;
    else
    y = 6;

To which "if" does the "else" give an alternative?  It's not clear from indentation or compound statement braces.  The rule in C is that an "else" is associated with the closest "else-less if".  So, using indentation to show association, the code will behave like this:

    if (n < 10) 
        if (z > 4)
            x = 5;
        else
            y = 6;

If the programmer really wanted the else associated with the other "if", then braces must be used, like this:

    if (n < 10) { 
        if (z > 4)
            x = 5;
    } else
        y = 6;

#### Else-If ####

As we have seen previously, the if statement is indeed a statement, so it may be used after an if statement -- or as the else part of an if statement.  This last construct makes if possible to string together if statements that explore multiple possibilities.  Consider this example.

    if (checking_account < 100) {
        savings_account -= 100;
        checking_account += 100;
    } else if (checking >= 100 && checking_account < 300) {
        savings_account += 50;
        checking_account -= 50;
    } else if (checking >= 300 && checking_account < 500) {
        savings_account += 75;
        checking_account -= 75;
    } else {
        savings_account += 100;
        checking_account -= 100;
    }
     
Here, there are 4 possible scenarios for moving money between the checking and the savings accounts.  We can string them together using this "else-if" construct.  This neatly constrasts all the choices.  

> **Remember It's a Trick!**
>
Remember, this way of using if statements and else parts is *really* a trick of text formatting.  The "else if" is really just an if statement inside an else statement.  We could really just write it like this -- with braces -- to show this:
>
    if (checking_account < 100) {
        savings_account -= 100;
        checking_account += 100;
    } else {
        if (checking >= 100 && checking_account < 300) {
            savings_account += 50;
            checking_account -= 50;
        } else {
            if (checking >= 300 && checking_account < 500) {
                savings_account += 75;
                checking_account -= 75;
            } else {
                savings_account += 100;
                checking_account -= 100;
            }
        }
    }
>
Using "else-if" constructions makes for a cleaner, more contrasted set of statements.    

### Switch Statements ###

Consider an "else-if" construct where one expression is being examined in a case-by-case basis:

    if (x_coord == 10)
        x_color = GColorBlack;
    else if (x_coord == 20) 
        x_color = GColorBlue;
    else if (x_coord == 30) 
        x_color = GColorRed;
    else if (x_coord == 40) 
        x_color = GColorGreen;
    else
        x_color = GColorWhite; 

Note that the color values (`GColorBlack`, `GColorBlue`, etc) are Pebble color values.  In this example, we repetitively examine `x_coord` in a way that might get lost in all the syntax.  

A switch statement is designed to be used for situations like the example above: where single comparisons are made in if statements.  The general form of a switch statement looks like this.

<pre>
switch (<i>expression</i>) {
    case <i>constant1</i>:
        <i>statements1</i>
    case <i>constant2</i>:
        <i>statements2</i>
    ...
    default:
        <i>statementsn</i>
}
</pre>

Using a switch statement, we might rewrite the "else-if" example as this:

    switch (x_coord) {
        case 10:
            x_color = GColorBlack;
            break;
        case 20:
            x_color = GColorBlue;
            break;
        case 30:
            x_color = GColorRed;
            break;
        case 40:
            x_color = GColorGreen;
            break;
        default:
            x_color = GColorWhite;
            break;
    }

There are several interesting elements in this switch statement.  First, note that only one value is used in each case.  Each case is *not* a list; it is a single value.  Second, note that the "default" part serves the function of the "else" part of the if statement -- a kind of "catch-all" statement.  Finally, note that each set of statements ends with a break statement.  The break statement stops execution of the current statement (it applies to several other statements as well).  Without it, one case's statement execution would flow into the next one.   

To that last point, let's say that we left out a couple of break statements.  Let's rewrite our example like this:

    switch (x_coord) {
        case 10:
            x_color = GColorBlack;
        case 20:
            x_color = GColorBlue;
            break;
        case 30:
            x_color = GColorRed;
        case 40:
            x_color = GColorGreen;
            break;
        default:
            x_color = GColorWhite;
            break;
    }

For this code, the first case (`x_coord == 10`) merges with the second case (`x_coord == 20`).  There is no reevaluation of the condition.  The above switch statement would be equivalent to this if statement set:

    if (x_coord == 10) {
        x_color = GColorBlack;
        x_color = GColorBlue;
    } else if (x_coord == 20) 
        x_color = GColorBlue;
    else if (x_coord == 30) {
        x_color = GColorRed;
        x_color = GColorGreen;
    } else if (x_coord == 40) 
        x_color = GColorGreen;
    else
        x_color = GColorWhite; 

Along the same lines, also note that the last "break" is technically not needed.  The execution will fall through, but there is no other statement, so the case statement ends. Developing a habit of adding a break is good, however, so adding a break here works as well.

Finally, there are valid reasons to leave out "break" statements.  In the example above, when `x_coord` has the value 10, there might be two valid operations to handle (other than changing colors twice).  Leaving out the "break" statement then, is a great way to add functionality withot needlessly duplicating code.

> **Syntactic Sugar**
> 
The switch statement -- and some other statements in C and other programming languages -- have been called "syntactic sugar".  It is a more convenient -- and perhaps aesthetic -- way of expressing the same semantics as a series of if statements.  For some, "convenient" and/or "aesthetic" are not good enough reasons to choose one code structure over another.
>
In response to that argument, others note the aesthetic code looks better and, as a result, reads better.  It's a better fit for the concept that is being coded and, therefore, is better documentation for the algorithm being implemented. 

### Inline Conditionals ###
 
Conditional execution is often used to decide what value to assign to a variable.   Let's revisit our checking account example:

    if (checking_account < 100) {
        savings_account -= 100;
        checking_account += 100;
    } else {
        savings_account += 100;
        checking_account -= 100;
    }

The amount that is assigned to variables is determined by the condition `checking_account < 100`.  

Inline conditional assignment flips the perspective.  Instead of using an if statement to determine assignment, we can embed a conditional into the assignment statement itself to choose between two expressions.  The form of this looks like

<pre>
<i>variable</i> = <i>condition</i> ? <i>expression1</i> : <i>expression2</i>;
</pre>

The choice between *expression1* and *expression2* works like you would expect: if the *condition* evaluates to a true value, the first expression is used otherwise the second expression is used.  

Using inline conditionals requires a change in perspective.  The focus is on the assignment, not the if statement.  The checking account example would have only two lines if we used inline conditionals:

    savings_account = checking_account<100 ? savings_account-100 : savings_account+100;
    checking_account = checking_account<100 ? checking_account+100 : checking_account-100;

There is still a fair amount of complexity here, but the focus is now on the assignment and the conditional has been worked into an expression. 

Inline conditionals can be very useful if they focus attention on the right programming element.  If the focus is on assignment and expressions, then inline conditionals can document and express an algorithm nicely.  If, however, the focus in an algorithm is on a larger segment of code chosen by a conditional, then an if statement is probably more appropriate.

### Big Messes and How to Avoid Them ###

As we have seen, C allows some interesting shortcuts that, when combined with its boolean-as-integer approach and inline conditionals, can get pretty messy.  Let's take some examples, then give some guidelines to follow to avoid messy code.

One big pitfall happens when assignment is used as part of the conditional.  For example,

    if ( (x = 2) == 5 ) {...}

or worse without parentheses:
 
    if ( y=x+1==z-1 ) {...}

Here, precedence rules matter a great deal.  The assignment happens first, producing a value that is then used in the evaluation of the `==` comparison.

In fact, precedence rules can fool you.  Consider this statement:

    if (2 + 5 > 6 ? 1 : 0 > 0) return true; else return false;

It's hard to figure out which part gets evaluated first.  Breaking this out into if statements helps and, as pointed out before, using parentheses helps.  Here, the `2 + 5` part is evaluated before the comparisons.

Using integers as booleans can be messy.  For example,

    if ( x-4 || y+2 ) {...}

This statement relies on the fact that `x-4==0` means false.  It's hard to remember integers are boolean values as well.  

Other problems arise when inline conditionals are nested.  For example,

    x = y>2 ? x>4 ? a ? b : c : d : e;

Figuring out the grouping is hard when nesting is itself nested.

Here are some guidelines to follow to avoid messy conditional code.

1. Avoid using integers as booleans, even if it makes sense in the code you are writing.  For example,

        if (x != 0) {...}

    makes much more sense than

        if (x) {...}

1. Use assignment operators as statements; avoid using them in conditionals.  

1. Parentheses are your friends.  They are difficult to overuse; parentheses should be used liberally to clarify expressions and conditions.  As an example, this rewritten assignment is much clearer than the inline conditional above:

        x = y>2 ? (x>4 ? (a ? b : c) : d) : e;

1. Braces are also your friends.  Even if there is a single statement in an if statement, using braces around that statement clarifies groupings and minimizes confusion.

### Project Exercises ###

The exercises for this chapter have you working with projects that implement clock-like features. They all deal with watchface applications.  So this means there will be some code that you are expected to understand yet.  However, the ability to deal with complexity while not completely understanding is a very good skill to develop.  It will help working with code you did not write.

#### Project 4.1 #### 

[Click here to get started with Project 4.1.](https://cloudpebble.net/ide/import/github/programming-pebble-in-c/project-4-1)
This application implements a second hand that sweeps in a circle and displays the numbers on a clock at 5 second intervals.  Run the application, then work through the following questions and challenges.

1. Add comments that explain the use of `second % 5 == 0` in the if statement in the `mark_the_second` function. 

1. The switch statement in the `mark_the_second` function works but is rather unwieldy.  Each of the cases 1 - 9 do the same thing.  Rewrite the switch statement into a compound if/else-if statement that does what the switch statement does.

1. The declaration `char text[3]` is a way to declare a string of (3) characters.  Strings will be discussed in detail later; right now, it's enough to know that a string is a sequence of characters, ending in a character whose integer value is zero.  Explain what the line `text[0] = 48 + number;` means by adding some comments to the code.

1. Notice that the text is drawn with a font called `FONT_KEY_GOTHIC_24` (lines 55-57).  Add code to make the number "12" drawn with the font `FONT_KEY_GOTHIC_28_BOLD`.

1. This question is a bit challenging.  You will notice, as the hand sweeps around the clock, that the numbers are displayed in awkward positions, not the positions you would expect for clock numbers.  Numbers are drawn in a 30 x 30 box, with the upper left corner the origin point (0,0).  
  * The text is drawn by the `graphics_draw_text` function at the end of the `mark_the_second` function.  Find the `GRect(position_x,position_y,30,30)` function.  Change `position_x` and `position_y` to new variables `text_x` and `text_y`.  Declare these variables at the top of `mark_the_second`.  Just before the `graphics_draw_text` function, make `text_x` and `text_y` equal to `position_x` and `position_y`.  Your code should now run the same way it did before these changes.
  * Now add a switch statement that will change `text_x` and `text_y` for each number around the clock.  For example, for "3", the correct coordinates are halfway down the left side of the 30 x 30 box surrounding the number.
  For this, `text_x` should still equal `position_x`, but `text_y` should equal `position_y +15`.  Add cases for all clock number 1 through 12.

1. Finally, claim the code by putting your name and the date in the header comments.  

You can find the answers and implementations for the assignments [above at this link.](https://cloudpebble.net/ide/import/github/programming-pebble-in-c/project-4-1-answer)

#### Project 4.2 ####

Now let's examine Project 4.2; [click here to import the project from Github into CloudPebble](https://cloudpebble.net/ide/import/github/programming-pebble-in-c/project-4-2).

This application puts time on the Pebble screen as text.  It's a simple program that updates once per second.  Most of the work is done in the `handle_tick` function, shown below:

    static void handle_tick(struct tm *tick_time, TimeUnits units_changed) {
        int hour = tick_time->tm_hour;
        int minute = tick_time->tm_min;
        int second = tick_time->tm_sec;
        static char time_string[] = "00:00:00";

        strftime(time_string, sizeof(time_string), "%T", tick_time);
        text_layer_set_text(text_layer, time_string);
    }

Let's walk through this code, since we have not seen much of it yet.  
* In the first declaration, we retrieve the hour and store it in the variable `hour`.
* In the second declaration, we get minutes after the hour and store the number in `minute`.
* Next we declare the `second` variable and store the seconds in there.
* Next, we declare a string of characters, stored in an *array*.  An array is a sequence of memory words or variables, stored sequentially in memory, so we can refer to them by number.  
* The next statement stores the current time, formatted neatly in the string. 
* The last statement puts that string, which is now formatted with the time, on the Pebble screen.

We will make changes in this code by focusing on the `handle_tick` function.  Work through the following assignments.

1. Make the watch app vibrate on the quarter hour.  To do this, we need to use one of two functions.  On the top of the hour, make the watch give a long vibrate by calling the `vibes_long_pulse()` function.  On each of the other quarter hour marks, call the `vibes_short_pulse()` function to give a short vibration.
 
1. Make the watch app slowly change the color of the time string as the hour progresses.  At the top of the hour, display the time in red; as the hour progresses, fade from red into white.  To do this, you have to declare a variable to hold the text color; this variable needs to be of the `GColor8` type.  Then, use the `text_layer_set_text_color` function to set the text color.  

    At the top of the hour, call `text_layer_set_text_color(text_layer, GColorRed)`.  For every other minute, you need to compute the color.  Use a fraction of time passed times the distance between red and white.  Use something like this:

    `text_color.argb = GColorRedARGB8 + (second / 60.0) * (GColorRedARGB8 - GColorWhiteARGB8));`

    Technically, that won't work.  You have to figure out how to cast some of the computations to the right data type.  But this will compute the fade.  Work it into the code with the right if statement.

1. Finally, claim the code by putting your name and the date in the header comments.  

You can find the implementations for the assignments [above at this link.](https://cloudpebble.net/ide/import/github/programming-pebble-in-c/project-4-2-answer)