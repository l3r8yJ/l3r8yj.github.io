---
title: On the cutting edge of exception handling
layout: post
date: '2023-10-15'
last_modified_at: '2023-10-19'
categories:
  - exceptions
  - java
  - coding
tags:
  - exceptions
  - exception
  - java
  - programming
  - coding
---
Unfortunately, in work,
we're using [Spring Framework](https://en.wikipedia.org/wiki/Spring_Framework).
Not too long ago I was able to discover an interesting detail in our approach to exception handling.
<img height="420" title="Catching guy" alt="Catching guy" src="/assets/images/catch.gif">
Usually it looks like some class (most often a service) throws an exception.
There is an exception handler in another part of the program that does the job.
```asm
// class that throws
public class DrinkService {
    public drinkAndDoThings() {
        throw new HangoverException("You have a headache!")
    }
}

// class that handles
@ControllerAdvice
public class ExceptionHandler {
    
    @ExceptionHandler(HangoverException.class)
    @ResponseStatus(HttpStatus.BAD_REQUEST)
    public ResponseEntity<?> onHangoverException(final HangoverException exc) {
        return ResponseEntity
            .status(HttpStatus.BAD_REQUEST)
            .body(exc.getMessage());
    }
}
```
And user of our API gets response like 
```asm
{
    "error": "You have a headache!"
}
```
So, first of all, this looks like a mess. You probably didn't want to read this code, or you had to struggle to realize what is written here.
But that's beside the point, just a quick note.
Let's look at it from the other side. When a programmer writes the `throw` keyword, he or she knows that somewhere in the program he or she has a piece of code that will be executed. Looks familiar, doesn't it?
To refresh your memory, take a look at this
```asm
_start:
    ; Entry point of the program

    ; Initialize a counter (e.g., ECX) with a value (e.g., 5)
    mov ecx, 5

loop_start:
    ; Your code here
    ; For example, print something to the screen, perform some calculations, etc.

    ; Decrease the counter
    dec ecx

    ; Check if the counter is zero
    jz loop_exit  ; If it is zero, exit the loop

    ; Jump back to the start of the loop
    jmp loop_start

loop_exit:
    ; Your code here after the loop
    ; Exit the program
    mov eax, 1       ; syscall number for sys_exit
    int 0x80         ; Call kernel
```
Take a closer look at the `jmp` keyword, it does the same thing as `throw`. In fact, **we are still using assembler instructions, in 2023**. It seems we still live in a procedural paradigm.
I don't think it can be avoided in Java, but for example, in Rust they handle errors as follows
```asm
use std::fs::File;

fn main() {
    let data_result = File::open("data.txt");

    // using match for Result type
    let data_file = match data_result {
        Ok(file) => file,
        Err(error) => panic!("Problem opening the data file: {:?}", error),
    };

    println!("Data file", data_file);
}
```
In my opinion, this looks much better. First, you can see where the error was handled. Second, it obliges you to handle the exception yourself.
Unfortunately, the errors in [EOLANG](https://www.eolang.org/) have the same `goto` or `jmp` way of thinking (example was taken from [here](https://news.eolang.org/2022-07-18-error-and-try-catch.html)).
```asm
[x] > check
  if. > @
    x.eq 0
    error "Can't divide by zero"
    42.div x
```
Line

> `error "Can't divide by zero"` 

looks same as

> `throw new ZeroDivisionException("Can't divide by zero")` 

to me.

This forces programmers to use [exceptions as control flow](https://softwareengineering.stackexchange.com/questions/189222/are-exceptions-as-control-flow-considered-a-serious-antipattern-if-so-why). I hope that EOLANG will move on from this to a different way of handling errors.
