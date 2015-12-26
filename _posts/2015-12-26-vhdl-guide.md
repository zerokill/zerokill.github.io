---
layout: post
title: "VHDL guide"
description: ""
category: 
tags: []
---
I've been toying around with the idea of writing a short something about VHDL. This will consist of multiple posts where we are going to look at writing your first VHDL program and simulating it in Modelsim. Topics that I want to cover include:

* Combinatorial logic
* Sequential logic
* Memory
* Verification techniques

Finally, I would like to implement a simple C algoritm in VHDL. I am not planning on actually implementing the design in an FPGA.

Lets start with the first part!

# Combinatorial logic

To quote wikipedia about combinatorial logic:

> In digital circuit theory, combinational logic (sometimes also referred to as time-independent logic) is a type of digital logic which is implemented by Boolean circuits, where the output is a pure function of the present input only. 

We are going to create a digital block with 4 inputs (A, B, C, and D) and 1 output (F). The block will implement the following combinatorial function:

    F = AC' + AB' + BCD' + AD'

We start our design by telling the compiler what libraries to use. Because we may want to implement this design in an FPGA we are going to use std_logic. The std_logic_1164 library gives us access to std_logic signals and vectors that we can drive to 0, 1, Z (high impedance), - (dontcare), and others. This is also the place where we can include libraries and packages that we have developerd ourselves!

{% highlight vhdl %}
library ieee;
use ieee.std_logic_1164.all
{% endhighlight %}

Next, we give an entity description. The entity is basically the "box" that will surround our logic. In the entity we define the input and output signals. We can also pass parameters to the entity, called generics. We will discuss generics later. For our program we are going to define an entity called "comb_logic" with four inputs called (A, B, C, and D) and 1 output (F).

{% highlight vhdl %}
entity comb_logic is
    port (  A   : IN  std_logic;
            B   : IN  std_logic;
            C   : IN  std_logic;
            D   : IN  std_logic;
            F   : OUT std_logic
            );           
end comb_logic;
{% endhighlight %}

At the momemt all we have is a black box with four inputs and one output. We can already connect this blackbox to something but the simulator will have no idea what it should do inside the blackbox. All signals are basically unconnected.

Lets implement our logical function. For this we will need to define an architecture containing RTL. Here we define our boolean function, or more specifically, how the output F changes based on changes on the input. Signal assignments are done using the "<=" operator.

{% highlight vhdl %}
architecture RTL of comb_logic is;
begin
    F <= (A and not C) or (A and not B) or (B and C and not D) or (A and not D);
end RTL;
{% endhighlight %}

As you can see we its very straightforward to implement a boolean function! All input signals are connected to the output signal and the simulator can model how the output changes based on the input signals. In the next part we are going to verify that our boolean function works.

## Verification

When writing VHDL code we need to take care that our design actually implements the logic that we want. To verify this we are going to create a testbench. A testbench is basically a module that can generate test signals and verify that the expected output appears. The difference between a testbench and RTL code is that a testbench will never be implemented in hardware. For example, in a testbench it is perfectly legal to make a statement like "change signal A from 0 to 1 after 20ns". A testbench is executed using a simulator and the simulator will change the signal A after 20ns.

However, implementing a statement like that in hardware is not possible. There is no way for the hardware to now what 20ns is. Unless you implement an actual clock to measure time, hardware has no idea of physical things such as "time". Testbenches are usefull because it allows us to simulate the usage of hardware by invoking functions that are only possible in simulation.

Lets start with our testbench! We are going to use std_logic vectors to connect to our RTL so we will need the std_logic_1164 libraries.

{% highlight vhdl %}
library ieee;
use ieee.std_logic_1164.all;
{% endhighlight %}

Our testbench is the toplevel. We are going to connect the RTL inside the testbench. Because of this our testbench does not need any input or output ports. Because of this we can define our entity as follows:

{% highlight vhdl %}
entity comb_logic_tb is
end comb_logic_tb;
{% endhighlight %}

Now for the actual testbench code:

{% highlight vhdl %}
architecture DUT of comb_logic_tb is
{% endhighlight %}

First we need signals that we are going to use to connect to our RTL. For this testbench you should think of signals as wires (I now this is a gross simplification but for this part its fine). Signals are different from ports because they do not have a direction. We can read from a signal and we can assign to a signal.

{% highlight vhdl %}
signal A, B, C, D, F    : std_logic;

begin
{% endhighlight %}

First we are going to connect the signals to RTL code. We do this by instantiating the RTL block inside our testbench. Note that there are many ways to instantiate components inside other components. We will discuss some other ways later.

{% highlight vhdl %}
    rtl_i : entity work.comb_logic(RTL)
    port map(   A => A,
                B => B,
                C => C,
                D => D,
                F => F
                );
{% endhighlight %}

The testbench is going to run inside a VHDL process. A VHDL process allows us to perform functionality sequentially. We will discuss processes later.

{% highlight vhdl %}
    process
    begin
{% endhighlight %}

Now we are ready to start generating some testsignals to verify the operation of our RTL. We are going to generate testsignals for all possible input combinations.

{% highlight vhdl %}
        A <= '0'; B <= '0'; C <= '0'; D <= '0'; wait for 10 ns;
        A <= '0'; B <= '0'; C <= '0'; D <= '1'; wait for 10 ns;
        A <= '0'; B <= '0'; C <= '1'; D <= '0'; wait for 10 ns;
        A <= '0'; B <= '0'; C <= '1'; D <= '1'; wait for 10 ns;
        ...
        
        wait;
    end process;
end DUT;
{% endhighlight %}

Thats it, your first testbench. We can use a simulator such as modelsim to compile the RTL of testbench and to simulate the operation of the testbench. We can verify that the RTL of working correct by for example viewing the output wave form.

## Conclusion

Currently, this is all rather basic. The RTL is not doing anything really interesting and we still manually need to verify that the RTL of producing the correct output for each input signal. In the next parts we are going to expand the RTL code to do more interesting stuff and we are going to expand the testbench to actually test if the RTL of behaving correct.

You can find all the code for the post on my github!
