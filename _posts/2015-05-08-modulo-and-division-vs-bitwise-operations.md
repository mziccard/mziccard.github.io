---
layout: post
title: Modulo and Division vs Bitwise Operations
description: Performance comparison of hardware division (modulo) instructions and bitwise operations
keywords: modulo, division, hardware, assembly, x86, bitwise, and, or, xor
---

The modulo and integer division operators are widely used by 
programmers, regardless of the language used, 
the type of application or its goal. 
Modern off-the-shelf processors provide 
machine instructions to perform such operations. 
For instance x86 architecture has the `DIV` 
instruction that performs a division and places the reminder in the `AX` register. 
Division and modulo operations are frequently executed as 
recurrent parts of more complex functionalities. 
Their impact on performance is, however, often underestimated. 
Divisions in modern architectures, in fact, 
may take several clock cycles, depending on the operands' values. 
Other integer or bitwise operations, on the other hand, usually 
take only one cycle.
Due to their high frequency of execution and possibly longer latency, 
modulo and division should be avoided in favor of bitwise 
operations whenever possible.

**How to avoid division and modulo?**

Modulo can be easily translated into a bitwise `AND` if 
the divisor is a power of two. Consider, for instance, 
the following C code:

{% highlight C %}
int remainder = value % 1024;
{% endhighlight %}

It can be translated into:

{% highlight C %}
int remainder = value & 0x3FF;
{% endhighlight %}

In general, if `divisor` is a power 
`n` of two, the modulo operation can be translated to 
a bitwise AND with `divisor-1`. Similarly, 
the integer division `value / divisor` corresponds to 
a right shift of `n` positions: `value >> n`.

**What is the real impact of modulo?**

Saying that division/modulo should be avoided when possible 
requires some evidence to be provided. 
The question is: does the use of division/modulo 
operation have a real impact on the 
performance of the application when compared to 
using the bitwise and?  
To answer to this question I run the following C program on an x86 machine:

{% highlight C %}
int main(int argc, char** argv) {
  int remainder;
  int value = 1301081;
  int divisor = 1024;
  int i = 0;
  while (i < 10000000) {
    remainder = value % divisor;
    i++;
  }
}
{% endhighlight %}

And compared its execution times with the same program 
implemented using binary AND (`&`) operator.
Results are shown below (1000 runs of the program 
for both `%` and `&` versions):
![Comparison of modulo and bitwise AND](/public/images/MODvsANDresults.png "Comparison of modulo and bitwise AND")
As you can see, in the average case the program with 
the modulo operation is 62% slower than the one 
using `&` operator.

**Example**

It's not always possibile to get rid of modulo and division in favor of 
bitiwise operations. However, there are cases in which this 
is possible. If you are in need for a circular buffer, for 
instance, of size _s_ you might consider setting the 
size to the power of two closer to _s_ to speed up performance 
on accessing the buffer. 
If _s_ is close to the nearest power of two 
then the space wasted will be totally worth it.
