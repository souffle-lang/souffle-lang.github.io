---
title: Arithmetic
permalink: /arithmetic
sidebar: docs_sidebar
folder: docs
toc: false
---
* Souffle supports standard arithmetic operations **+**, **-**, **&#42;**, **&#47;**, **&#94;** and **&#37;**. Examples of this are given below.
```
.decl e(x:number, t:symbol, y:number)
e(10 * 2,"10*2", 20).
e(10 + 2,"10+2", 12).
e(10 / 2,"10/2", 5).
e(10 ^ 2 , "10^2", 100).
e(10 % 3, "10%3", 1).
e(2^4%13 , "2^4%13",3).
```
* Souffle supports bitwise logical operations: **band** (bitwise and), **bor** (bitwise or), **bxor** (bitwise exclusive-or), **bshl** (bitwise shift left), **bshr** (bitwise shift right), and **bshru** (bitwise shift right/unsigned). 

Examples of this are given below.
```
e(0xFFF1 band 0xF, "0xFFF1 band 0xF", 0x1).
e(0xFF00 bor 0x000F, "0xFF00 bor 0x000F", 0xFF0F).
e(0xFFFF bxor 0x000F, "0xFFFF bxor 0x000F", 0xFFF0).
```

* Souffle supports logical operations that consider every non-zero number as true and always return 1 or 0: **land** (logical and), **lor** (logical or), **lxor** (logical exclusive-or), and **lnot** (logical not).

Examples of this are given below.
```
e(1 land 2, "1 land 2", 1).
e(1 land 0, "1 land 0", 0).
e(1 lor 0, "1 lor 0", 1).
```

* Souffle supports **max** and **min** operations over numbers.
```
e(max(3, 4), "max(3, 4)", 4).
e(min(3, 4), "min(3, 4)", 3).
```

* Souffle supports standard unary operation **-**.
```
e(-2*10,"-20", -20).
e(-2,"-2", -2).
e(--2,"--2", 2).
```

* Souffle supports standard binary operations **&#62;**, **&#60;**, **&#61;**, **&#33;&#61;**, **&#62;&#61;** and **&#60;&#61;**. Examples of this are given below.
```
A(a,c) :- a > c.
B(a,c) :- a < c.
C(a,c) :- a = c.
D(a,c) :- a != c.
E(a,c) :- a <= c.
F(a,c) :- a >= c.
```

* **&#36;** is used to generate unique random values to populate a table. It should be used with care as it may result in stepping outside the standard Datalog semantics.
```
.decl A (n:number)
.decl B (a:number, b:number)
.decl C (a:number, b:number)
.output C
A(0).
A(i+1) :- A(i), i<1000.		
B($,i) :- A(i).		
C(i,j) :- B(c,i), B(c,j), i!=j.
```
The above example does not output anything.

{% include links.html %}
