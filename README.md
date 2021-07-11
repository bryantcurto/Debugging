# Preamble
This document contains the strategies and tips I have accumulated for debugging. This is not meant to be an exhaustive guide for debugging, which has been done by others who have far more experience than me. Instead, this briefly describes *my approach* for debugging software (especially C/C++ code) as well as provides some tips and examples.

There are a few reasons I decided to create this document (considering that there are already so many documents on this topic).
- Many existing documents provide tips instead of a holistic approach to debugging.
- Not enough documents sufficiently stressed the importance of not only *understanding a bug* but also *being able to explain how the bug produced the buggy behavior*.
- I wanted to create a *brief* document that will serve as a jumping-off point for those learning how to debug.

My hope is to periodically update this document based on my own experiences and based on my experience helping others debug their code. I hope to also add any interesting debugging challenges I've encountered.


# Introduction:
According to [Wikipedia](https://en.wikipedia.org/wiki/Software_bug), a bug is "an error, flaw or fault in a computer program or system that causes it to produce an incorrect or unexpected result, or to behave in unintended ways." When debugging, your objective is to understand why observed behavior does not match expected behavior. One of your goals as a programmer should be to learn how to debug *effectively* and *efficiently*.

Debugging consists of *isolating the bug* and *squishing it*.

The process of *isolating the bug* consists of two equally important parts:
- identifying the piece of code that is to blame for the unexpected behavior, and
- understanding how the code caused the unexpected behavior well enough to explain it to someone else.

The process of *squashing the bug* is just a cute way of saying resolving the bug once you know what's broken. For obvious reasons, this document doesn't cover how to do this. However, I do discuss tips for preemptively simplifying the debugging process as you write new code.

# My Approach:
## Reproduce
The first step of debugging is to figure out a method for consistently reproducing the bug. If you can't reproduce the bug, then you cannot debug it using the methods I describe. Some bugs appear very infrequently/inconsistently (e.g., multi-threading bugs), so it may take multiple invocations of your application to make the bug reappear.

## Simplify
Once you've determined a method for reproducing the bug, it is often helpful to simplify the number of variables that result in the bug appearing by constructing a [minimal working example (MWE)](https://en.wikipedia.org/wiki/Minimal_working_example). ([Here](https://stackoverflow.com/help/minimal-reproducible-example) is stackoverflow's description of an MWE.) For the purpose of debugging, answering the question of how minimal an MWE needs to be is largely dependent on how far you've isolated the bug.
- If your script crashes when you supply it a specific set of command line arguments, your MWE should be a minimal set and complexity of arguments that reproduces the crash.
- If your function incorrectly computes the volume of a box given its length, width, and height, your MWE should be a minimally complicated triplet of numbers (i.e., few decimals, easy to remember) that exhibits the incorrect behavior.
- If your algorithm for computing the convex hull of a set of points is buggy, your MWE should consist of the minimal number of points (each of which is a minimally complicated pair of numbers) that reproduces the bug.
- If some git commit in a list of commits introduced a bug, your MWE should indicate the commit that introduced the bug.

**Keep in mind that the MWE must still consistently reproduce the bug!**

## Effective Approach
Once you produced an MWE, you can track down your bug in the following manner:
#### 1. Identify how observed behavior differs from expected behavior.
Sometimes this step is rather obvious: e.g., I expected the code not to crash. However, it's good to be clear about exactly *what* you're trying to solve.

#### 2. Collect information about the behavior.
This is one of the most important steps because, for most errors, the solution to your bug is right in front of you.
	- Most compilers point directly to the syntax errors causing the failed compilation.
	- Most programs log a warning/error message before crashing.

#### 3. Understand the information and try to infer from it.
If you don't know something, look it up! If you roughly know something, look it up! Debugging requires you to understand what's going on with your program.
Learning to effectively and efficiently search for information on the internet is a skill in and of itself. In brief, when searching for something on the internet, I've found success distilling my query into key phrases or terms. That being said, when I get an error message that I've never seen before, the first thing I do is copy and paste it directly into Google. Someone else has likely gotten that error message before.

#### 4. Use your understanding to hypothesize what might be happening.
This is the most challenging step for beginners because it takes practice, experience, and knowledge of the code base you're debugging.

#### 5. Determine if your hypothesis was correct.
This step is almost as challenging for beginners.
*Your objective in this step is to come up with the simplest method that will definitively tell you whether or not your hypothesis is correct.* Determining if your hypothesis is correct may be achieved by:
    - reading documentation,
    - designing and running one or more experiments,
    - extracting a buggy piece of code to test directly, or even
    - tracking each step of an algorithm with pencil and paper.

#### 6. Finish, or return to step (2) or (3) if the experiment provided additional information about the behavior.

## Efficient Approach
It may be enticing to dive deeply into an issue and pull out GDB immediately. It may also be enticing to change a few parameters and rerun your code. However, this is almost always an inefficient approach. Have a bisecting, breadth-first approach to debugging!
- Use a bisection approach by repeatedly splitting the problem space until you've isolated the issue -- like a binary search algorithm.
    - If a data structure gets modified at an unknown location, rerun your application with additional print statements (or comment out chunks of code) to narrow down the chunk of code where the modification occurs. ([Here](https://www.nics.tennessee.edu/faq/darter-how-do-i-use-code-bisection-method-find-bug) is a step-by-step description of how you might do this.)
     - If some git commit in a list of commits introduced a bug, literally perform a binary search to determine which commit introduced the bug. (See [this Stack Overflow post](https://stackoverflow.com/a/844022/6573510) for an example.)
- Use a breadth-first approach by avoiding focusing on one thing or one piece of code until you are certain that it is responsible for the bug.
    - A breadth-first approach entails doing the least amount of work/making the fewest number of code changes in order to verify that you're on the right path to tracking down or squashing a bug.
    - GDB is a great tool for inadvertently studying code that does not in fact contain a bug.

# Tips:
## Debugging Tips
- **Read (and understand) everything--warnings, errors, comments, and code!**
Not all bugs tell you that they exist by crashing your program. Sometimes, that compiler warning you've been ignoring may be an indication that you have a bug. Sometimes, your bug isn't a bug at all! Instead, it is a corner case that has yet to be covered but has been thoroughly documented in the source code with a `TODO`.
Programmers spend a surprising amount of time simply reading code.
- **Change one thing at a time!**
If you change more than one thing and then observe different behavior, you are not able to say which change caused the change in behavior.
- **More likely than not, you caused the bug**.
The sooner you train yourself to always assume that it's your code that's buggy, the more time you can save. Code bases are often rigorously tested for correctness.
- **Avoid context switching!**
Don't try to work on multiple things at once. Focus on one problem at a time. It'll help keep your thoughts organized and your mind focused.
- **Document your actions.**
Keep a notebook to jot down what you've tried, why you tried it, and what happened as you debug. It not only documents what you've done, but it also helps you organize your thoughts and therefore organize your approach in debugging.
- **Take a step back.**
If the bug is tricky and/or you've been working on it for a while, remember to step back and look at the big picture. I usually do this by getting up to get more coffee or going for a short walk. This lets me look at the problem from a new angle.
- **Make liberal use of print statements.**
Print statements can allow you to quickly and easily verify that data passing through the system is correct. Print statements are a key part of having a breadth-first debugging approach.
- **Use GDB/LLDB sparingly except possibly to get the stack trace.**
Unless you're very comfortable with GDB/LLDB or the application your debugging takes a long time to compile and run, I usually recommend avoiding such tools until you've narrowed down the issue substantially. It's very easy to get lost (in the code and in the tool itself) when using these tools. Often, the same thing can be accomplished using print statements and commenting out code.
    - If using GDB, make sure that you're compiling with debug information enabled (-g for GCC).
    - If GDB reports that the variable you want to read is optimized out, try disabling optimization (-O0 for GCC). *(Make sure you can still reproduce the bug!)*
    - [Here](https://stackoverflow.com/questions/6545763/how-can-i-rerun-a-program-with-gdb-until-a-segmentation-fault-occurs/6546674) is how you can have GDB repeatedly rerun your program until it segfaults.
- **Collect statistics if you can't use print statements.**
When debugging multi-threading bugs, print statements may prevent the thread interleaving allowing you to reproduce the bug. In that case, collect statistics in (possibly global) data structures and print them at program exit.
    - For C/C++, I've found [`atexit()`](https://man7.org/linux/man-pages/man3/atexit.3.html) and [`sigaction()`](https://man7.org/linux/man-pages/man2/sigaction.2.html) to be useful.
- **Be vigilant for [off-by-one errors/bugs](https://en.wikipedia.org/wiki/Off-by-one_error).**
- **Resolve compilation errors from first to last.**
Earlier errors can cascade and cause later errors.


## Programming Tips
- **Make liberal use of assertion statements.**
Assertions allow you to more easily detect bugs at their source instead of trying to hunt down their source.
- **Use a version control system (e.g., git).**
It's much easier to fix buggy code when you can compare it to code that worked.
    - Keep commits small and self contained. Doing so *greatly* simplifies debugging by making the process consist of isolating the commit that introduced the bug.
    - Remember to test your code before each commit.
- **Reread your code after you've written it.**

# Examples:
## Example 1:
**code**:
```c++
#include <stdio.h>

int main(char* argv[]) {
    printf("Hello world\n");
}
```
**output**:
```
$ g++ -o foo foo.cc 
foo.cc:3:5: error: first parameter of 'main' (argument count) must be of type 'int'
int main(char* argv[]) {
    ^
1 error generated.
```
1. I expected my program to compile but it did not.
2. The compiler says that my program has 1 error. The error's description is `first parameter of 'main' (argument count) must be of type 'int'`. The compiler must expect `main`'s first parameter to be an int. Looking at my code, I have `main`'s first parameter as a `char* []`.
3. From the [docs](https://en.cppreference.com/w/cpp/language/main_function), `main` can have one of several signatures.
4. The signature of `main` that I used is not valid.
5. Changing function main to have signature `main(int, char* [])` should and does fix this compiler error. 
6. Done!


## Example 2:
**code**:
```c++
#include <stdio.h>
#include <vector>

int main() {
    std::vector<int> vec;
    for (size_t i = 0; i < 5; i++) {
        vec.push_back(i);
    }
    
    // ... code block 1 ...
    
    // ... code block 2 ...
    
    // ... code block 3 ...
    
    // ... code block 4 ...
    
    for (size_t i = 0; i < vec.capacity(); i++) {
        printf("%d\n", vec[i]);
    }
    return 0;
}
```
**output**:
```
0
1
2
3
4
32767
1666466198
32767
```

Let's assume that I mistakenly believe that `std::vector::capacity()` has the functionality of `std::vector::size()`.

1. I expected the program to output five values but it instead output eight.
2. The latter numbers retrieved from vector `vec` don't seem to correspond with the prior numbers.
3. There's nothing obvious that I need to understand about this information. *(Though, one might be able to infer that the latter values could be invalid values.)*
4. I hypothesize that unexpected values were appended to the list somewhere within the program.
5. I narrow down the location(s) of where `vec` gets modified. I do this by using a bisecting approach.
    - For example, I can print out the contents of `vec` in between code blocks 2 and 3. If the output exhibits the bug, I then move the printing code to between blocks 1 and 2. In this way, I'm performing a binary search to determine which code block(s) are to blame for the buggy behavior.
6. If I continue to erroneously use `std::vector::capacity()` to print out `vec` in my bisecting approach, I'll end up observing that the unexpected values are inserted immediately after the initial values are inserted (see above code). If I don't erroneously use `std::vector::capacity()` to print out `vec` in my bisecting approach, I'll end up observing that the unexpected values are inserted immediately before `vec` gets printed (see above code). I reach an impossibility: either `vec` always contains those values or those values were never added to the vector. Either way, I come to the same realization.

Returning to step (3)

3. My previous experiment tells me that there is something wrong with how `vec` is being printed since the unexpected values where always present in `vec`/never inserted in `vec`. Since the unexpected values appear at the end of the array and don't follow the pattern of those at the start, maybe the latter values are (somehow) invalid values.
4. I hypothesize that the upper bound of the for-loop is too large resulting in the loop accessing memory outside the bounds of the vector. This must mean that `std::vector::capacity()` doesn't do what I think it does.
5. Looking at the [docs](https://en.cppreference.com/w/cpp/container/vector/capacity), `std::vector::capacity()` returns the number of elements that the vector has currently allocated space for. I want to instead use `std::vector::size()`. After making this change, I see that the program produces the expected output.
6. Done!

# Additional Resources
In order from most useful to more useful:
- [MIT 6.031: Software Construction - Debugging Lecture](http://web.mit.edu/6.031/www/sp17/classes/11-debugging/)
- [Cornell CS312 - Debugging Lecture](https://www.cs.cornell.edu/courses/cs312/2006fa/lectures/lec26.html)
- [Happy Coding - Debugging Tutorial](https://happycoding.io/tutorials/processing/debugging)
- [Larrylisky - The Art of Debugging](https://larrylisky.com/2012/09/03/the-art-of-debugging/)
- [Computer Programming Principles - Debugging Chapter](https://en.m.wikibooks.org/wiki/Computer_Programming_Principles/Maintaining/Debugging)
