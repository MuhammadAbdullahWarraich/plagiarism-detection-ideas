# plagiarism-detection-ideas

# How to change one code into another

1. Branch-ful to Branch-less Programming
How will we do that? Well, first we need to understand branchless programming. Branchless programming is programming with no(well not completely) if statements. The core idea is demonstrated as follows:
```C
int x = initialize_x();
int y = initialize_y();
if (condition) {
  x = 10;
  y += 15;
  mutate_x(&x); // void mutate_x(int *x) { *x += 100; }
}
```
the above code will become:
```C
int x = initialize_x();
int y = initialize_y();

bool flag = condition;
x = tmp * !condition + 10 * condition;
y += 15 * flag;
mutate_x_optionally(&x, condition); // void mutate_x_optionally(int *x, bool condition) { *x += 100 * condition; }
```

it is very easy to see how we can programmatically convert branch-ful to branch-less programming. You just have to look for conditionals, and then convert them into branch-free programming. We can do and modify functions that are being called as above, or we can allow ourselves to have if statements for thos function calls. Therefore we can have various versions of the same program depending upon how much branches we eliminated in different contexts in the whole program.

One important thing to note is that we can go back as far as we want with eliminating branches, but we can't obviously modify code that we don't own like libraries unless we vendor our dependencies. But in the context of plagiarism detection, modifying dependencies would look like a deliberate attempt to change code, so we won't do that here.

It is also not possible to eliminate code in case of system calls because that code is owned by kernel space and we don't have access to that(unless we are using MS DOS or we also add our own OS with the assignment, haha).

For lots of else if's, we can repeat the above process but this flag will be changed like this:
```C
if (condition1) {
block1;
} else if (condition2) {
block2;
} else {
}
```
this will become:
```C
bool flag = condition1;
modified_block1;// modified like above
flag ^= condition2;
modified_block2;
flag = !flag;
modified_else_block;
```
and nested conditionals can be resolved as well:
```C
if (c1) {
  code_1;
  if (c2) {
    code_2;
  }
  code_3;
}
```
this will become:
```C
bool flag = c1;
modified_code_1;// modify like example 1
bool flag2 = flag && c2;
modified_code_2; // modify for flag2
modified_code_3; // modify for flag
```
2. Selective branch removal and Hints to CPU branch predictor
If you don't want to make a large number of copies of your code, you can be smart about your branch removal to make the use of branchless programming seem more natural. But fret not, we still can do it automatically! We can run the program over and over again and analyze which condition was hit more for each set of conditionals. We can do that naively by just putting a print statement in each condition and then analyzing logs to identify frequencies of different conditions being hit over multiple program runs with different inputs. Then we can remove branches which get hit most. We can optionally also put specifiers (like [[likely]] in case of C) to signal to the CPU branch predictor that this branch is likely, which would show to the evaluator of the assignment that you are a cracked programmer who knows a lot about optimization, but in reality it is my technique that helped the student.
3. Transforming boolean expressions
Continuing our arc on transforming if-statements, we can convert one condition into a different looking but logically equivalent boolean expression. For this purpose, we need to have a dictionary of boolean identities, and we need to search the syntax tree of the condition for patterns equal to one side of those identities. And when a match hits, we can replace the matching side of the identity with the other side of the identity. This way we can fake different ways of forming conditions for different students' versions of the assignment. You can mimic smart students by only transforming when the replacement is shorter than the original, or oversmart students(some people admire complexity) by only transforming when the replacement is longer.
4. Converting nested conditionals into flat conditionals
This is a simpler one to implement, but really changes the look of your code. You can understand how to do it from the following example:
```C
if (c1) {
  some_code_1;
  if (c2) {
    some_code_2;
  }
  some_code_3;
}
```
This will become:
```C
if (c1) {
  some_code_1;
}
if (c1 && c2) {
  some_code_2;
}
if (c1) {
  some_code_3;
}
```
For converting flat to nested, just convert like the following:
```C
if (c1 && c2) {
  do_some_things;
}
```
Convert to:
```C
if (c1) {
  if (c2) {
    do_some_things;
  }
}
```
