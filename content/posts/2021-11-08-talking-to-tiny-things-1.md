---
title: "Talking to Tiny Things - Part I"
date: 2021-11-08T09:22:28+02:00
draft: false
---

Last week's post on services failed to materialize, mostly to some design changes I'm currently mulling over for `factorix`, so this week I have something different planned.
I wanted to take a break from my regular posts on my factory simulator project to talk about another project I'm currently working on.
The post will involve a little more low-level programming, but in a "high-level" language.

## Coding a CPU

The project I'm talking about is a [Java processor](https://en.wikipedia.org/wiki/Java_processor).
Most of you will know that the Java compiler, rather than compiling to native machine code like most C compilers,
takes Java code and compiles it down to a Java-specific machine code, also known as the **Java bytecode**.
This bytecode is then executed in Java's runtime environment, affectionately known as the Java Virtual Machine (JVM).

There are a few implementations of actual hardware able to execute Java bytecode.
The most widely known is probably the [Java Optimized Processor (JOP)](https://www.jopdesign.com/) project.
Still, I decided to try my hand at creating my own implementation of a (very simplified) Java processor.
The processor will be implemented on an FPGA and designed in the SystemVerilog HDL.
I named this project `bali`, those among you who know their geography will probably realize why.
Giving my projects corny names makes it easier for me to want to invest time in them.
Have I mentioned that `factorix` is a portmanteau of **facto**ry and Mat**rix**? No?
Well, now you know.

## FPGA? HDL?

FPGA stands for **F**ield **P**rogrammable **G**ate **A**rray.
As the second half of the name says, it's basically an integrated circuit that consists of a giant array of logic gates.
We can basically impress any desired integrated circuit logic we want onto this array by telling it to activate a certain configuration of these gates.
This is done by writing a binary code known as a **bitstream** onto the FPGA that tells it which gates to activate and connect to each other.

HDL is short for **H**ardware **D**escription **L**anguage.
You may have heard of the fact that there is a correspondence between logical circuits and algorithms.
Any algorithm is expressable as a logical circuit, and vice-versa.
That's why it's possible to emulate a CPU. That old Game Boy game you're playing on your phone?
This is the principle at work behind it.
HDLs essentially allows us to express integrated logic in code.

## Building Hardware Modules

The basic building block of an HDL project is a *module*.
You can think of it as a sort of equivalent to a class in object-oriented languages.
A module can have one or more inputs and outputs and any internal wiring you could possibly want.
If you're familiar with digital electronics, a module is a component in a block wiring diagram: the inputs are the wires going into the block, and the outputs are the outgoing wires.
Depending on the inputs and internal state of the module, the outputs change.
We can then instantiate these modules and wire them together to create new modules.
To put it in a more object-oriented way: a module defines an *interface* for a set of input and output values.
An instance of a module is a stateful object that determines the outputs from a set of inputs and its state at a given time.

This was very abstract, so let's look at an example of a 4-bit adder in an HDL:

```systemverilog
module adder(input  [3:0] a,
             input  [3:0] b,
             output [3:0] res,
             output       c_out);
    
    logic [3:0] result;
    logic carry_out;

    always @ (a or b)
    begin
        {carry_out, result} = a + b;
    end

    assign res   = result;
    assign c_out = carry_out;

endmodule
```

For this project, I'm using *SystemVerilog*, an extension of *Verilog*, one of the two major players in the HDL game, along with *VHDL*.
So what is this code fragment? Well, we can represent it as a block, much like you would see in a digital block wiring diagram:

![Adder block](/imgs/adder.svg "Block diagram of a simple 4-bit adder")

This logical circuit adds two four-bit numbers.
Let's decompose the above code a bit: We start off by defining the module `adder` with two 4-bit input buses (that's what that `[3:0]` notation is for).
These buses input the two numbers we want to add.
A four-bit number is a hexadecimal digit, so we have possible input values ranging from `0x0` to `0xF` for both `a` and `b`.
The output `res` is the bus that carries the 4-bit result of our addition operation.
Now, say that the sum of our addition is larger than `0xF`.
Worst case: both of our inputs have a value of `0xF` and our actual result `0x1E` gets truncated to `0xE`.
We can avoid this by adding a second single-bit output, `c_out`, or the carry bit, which we set when we get an overflow.

In the body of the module, we create two registers, one for storing our result and one for storing the carry bit.
Then we specify that we want the module to do add `a` and `b` together each time one of the inputs changes.
We do this by putting the addition operation into an `always` block and setting the trigger for this block to `a or b`.
This means that the code within the block is executed each time either `a` or `b` changes value.
Then we simply store the result of our addition in our internal register `result` (extended by `carry_out`).
The `assign` statements at the end, finally, specify that the values in the registers `result` and `carry_out` should be assigned to our two outputs.

## Talk the Talk

Now that the basics are out of the way, we can discuss making the FPGA talk to my computer.
My FPGA board offers two pins subsumed in its USB interface that can be used for communication via *UART*.

**UART** stands for **U**niversal **A**synchronous **R**eceiver-**T**ransmitter.
*Asynchronous* means that it's independent of a clock signal, so there's only a single data line on which data is sent.
In lieu of the clock, both ends of the UART connection specify a *baudrate*, which defines the number of bits (or other symbols, but mostly bits) sent per second.

### The Transmission Process

When not sending data, a UART line continually transmits a value of 1.
A single UART transmission consists of a start bit (0), seven or eight data bits, an optional parity bit to check data correctness, and a stop bit (1).
We'll run through an example transmission here.

For this demo, we'll assume a baudrate of 9,600 baud/s, eight data bits, and no parity bit.
So now we want to transfer a total of ten bits per packet.
Let's say we're transmitting the byte `0x80`, which is a one followed by seven zeroes.
Since we're transmitting 9600 bits every second, we'll hold the value of each bit for `1/9600`th of a second, which is about 104 µs.

![UART Transmission](/imgs/uart_tx.svg "UART Transmission of the value 0x80")

We start by driving the output low to transmit a zero.
Then we transmit each data bit by setting the output to the corresponding bit value, holding each for 104 µs.
When we're done, we drive the output high again and hold for another 104 µs before ending the transmission.

## The Receiving End

On the receiving end, we obviously have to know the parameters of the transmission: 9,600 baud/s, eight data bits, no parity bit.
Once that's cleared up, all we have to do is wait for our input to go low.
After detecting that falling edge, we wait for 52 µs (half of our transmission time per bit).
This will ensure that we land in the "middle" for every bit we receive.
We wait 104 µs, then look at the value of our input and note it down.
Once we've written all eight values and detected a stop bit, we can notify whoever's listening that we're done receiving a transmission.

![UART Receiver State Diagram](/imgs/uart_rx_states.svg "Finite State Machine of a UART Receiver")

As pictured in the above state diagram, this results in four states.
Or eleven, depending on how you count the eight individual states in the `DATA` state.
We can see that UART is relatively simple to implement in hardware.

## Coming Up

In the next post, I'll put what we've learned about UART together with a little FPGA development knowledge and construct a small module for receiving UART transmissions.
I'll also write up some things on the development environment I'm using and how to automate it with tools like `make` and the Tcl scripting language.
If I have the time, I'll also give some open source FPGA toolchains a try and share my experience.

See you then!

~ nyando
