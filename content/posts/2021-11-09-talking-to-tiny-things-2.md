---
title: "Talking to Tiny Things - Part II"
date: 2021-11-09T09:22:28+02:00
draft: true
---

This post will cover how to create a state machine in *SystemVerilog* by the example of a UART receiver module.
The basic idea is similar for basically all FSMs you could build in this HDL.

## Building a State Machine

Recall the state chart for the UART receiver we discussed in the last post:

![UART Receiver States](/imgs/uart_rx_states.svg "UART Receiver State Chart")

Our goal is to translate this state machine to a receiver module written in *SystemVerilog*.
I mentioned in the last post that UART is asynchronous, so there's no clock line connecting the transmitter and receiver.
However, each of the modules does need a clock input to count off the intervals between bits.
This leads us to two inputs: clock and data.
As outputs, we obviously want an 8-bit data bus to write our received byte to.
In addition, we'd like connected modules to be notified that our receiver module just read a new byte.
To do that, we'll add another single-bit output that we drive high for one clock cycle after reading a stop bit.

Putting this all together, we end up with the following:

```systemverilog
module uart_rx 
    #(parameter CYCLES_PER_BIT = 104)
    (
        input        clk,
        input        rx,
        output [7:0] data_out,
        output       read_done
    );

    const int IDLE  = 2'b00;
    const int START = 2'b01;
    const int DATA  = 2'b10;
    const int STOP  = 2'b11;

    logic [1:0] state;
    logic [2:0] bit_count;
    logic [7:0] bits;
    logic [7:0] cycles;
    logic       done;

    always @ (posedge clk) begin
        /* implement state machine here */
    end

    assign data_out  = bits;
    assign read_done = done;

endmodule
```

The `parameter` in the module header is a number we can set when instantiating the module.
The value `104` should seem familiar from last time: it's the number of microseconds per bit at a baudrate of 9,600 baud/s: 104 µs.
At a clock frequency of 1 MHz (one million cycles per second), the period of a clock cycle is exactly 1 µs.
So, assuming we have a 1 MHz clock, we have to wait for 104 clock cycles to read our next bit.
If we have another combination of clock frequency and baudrate, we can set the parameter accordingly.
This makes our module more reusable.

After our module's header, we define identifiers for our module's states.
Since we only have to keep track of four states, we'll need two bits, which we track in the `state` register.
We have to count up to 8 bits when reading in the `DATA` state.
This means we'll save the number of bits we've read in the `bit_count` register.
We save the bits we read into the `bits` register.
Finally, as mentioned previously, we have to count the number of clock cycles up to `CYCLES_PER_BIT` before reading our next bit in the transmission.
We'll count the number of clock cycles in the `cycles` register.
Seven bits would have been sufficient for counting to 104, but we'll add an eighth for fun.

After updating our state machine in the `always` block, we assign whatever's in our `bits` and `done` registers to the corresponding outputs.
The next step is implementing our state machine according to our design.

## Moving Through States

Now that we have the skeleton of our module set up, let's go into the meat of it.
I already set up the `always` block that's triggered on every rising edge of the clock signal.
Here's the body:

```systemverilog
case (state)
    IDLE: begin
        cycles <= 8'h00;
        bit_count <= 3'b000;
        done <= 0;
        bits <= 8'h00;
        if (rx == 0)
            state <= START;
        else
            state <= IDLE;
    end
    START: begin
        if (cycles == (CYCLES_PER_BIT - 1) / 2) begin
            if (rx == 0) begin
                cycles <= 8'h00;
                state <= DATA;
            end
            else
                state <= IDLE;
        end
        else begin
            cycles <= cycles + 1;
            state <= START;
        end
    end
    DATA: begin
        if (cycles > CYCLES_PER_BIT - 1) begin
            cycles <= 8'h00;
            bits[bit_count] <= rx;
            
            if (bit_count < 7) begin
                bit_count <= bit_count + 1;
                state <= DATA;
            end
            else begin
                bit_count <= 3'b000;
                state <= STOP;
            end
        end
        else begin
            cycles <= cycles + 1;
            state <= DATA;
        end
    end
    STOP: begin
        if (cycles > CYCLES_PER_BIT - 1) begin
            cycles <= 8'h00;
            state <= IDLE;
            done <= 1;
        end
        else begin
            cycles <= cycles + 1;
            state <= STOP;
        end
    end
    default: begin
        state <= IDLE;
    end
endcase
```

Notice the behavior of the `done` register: It's driven high for one clock cycle in the `STOP` state,
then switched back to low once the next rising edge causes the state machine to go into the `IDLE` state.
The resulting output is a one-clock-cycle impulse on the `read_done` output.
This allows other modules to set a trigger on that output and react to it in some way.
The programming style used to interconnect these modules in *SystemVerilog* is known as [reactive programming](https://en.wikipedia.org/wiki/Reactive_programming).
Every module *reacts* in some way to inputs from another module or external stimulus.

## But wait, there's more!

I still have one more post on this topic in the pipeline.
We'll be using the brand-new UART receiver to show some 8-bit values on the FPGA board's LED array.
Along with that, I'll be demonstrating a way to build a bitstream and program an FPGA from the command line.

Stay tuned!

~ nyando
