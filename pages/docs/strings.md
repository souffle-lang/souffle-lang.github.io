---
title: Strings
permalink: /strings
sidebar: docs_sidebar
folder: docs
---

* **cat(*string*, *string*)** is used to concatenate two strings together. It can be nested to concatenate more than two strings.
```
.type A
.decl Y(a:A, b:A)
.decl Z(a:A, b:A, c:A)
.output Z
Y("a","b").
Y("c","d").
Z(a,b, cat(cat(a,b), a)) :- Y(a,b).
```
The output would be:
```
a	b	aba
c	d	cdc
```

* **contains(*string1*, *string2*)** is used to check if the latter string contains the former string.
```
.type String
.decl stringTable(t:String)
.decl substringTable(t:String)
.decl outputData(substr:String, str:String)
.output outputData
outputData(x,y) :- substringTable(x), stringTable(y), contains(x,y).
stringTable("aaaa").
stringTable("abba").
stringTable("bcab").
stringTable("bdab").
substringTable("a").
substringTable("ab").
substringTable("cab").
```
The output would be:
```
a	aaaa
a	abba
a	bcab
a	bdab
ab	abba
ab	bcab
ab	bdab
cab	bcab
```

* **match** is used to check if the latter string matches a wildcard pattern specified in the former string.
```
.type String
.decl inputData(t:String)
.decl outputData(t:String)
.output outputData
outputData(x) :- inputData(x), match("a.*",x).
inputData("aaaa").
inputData("abba").
inputData("bcab").
inputData("bdab").
```
The output would be:
```
aaaa
abba
```

* **ord(*string*)** is used to return the ordinal number associated with *string*. **This is not a lexicographic ordering.** The ordinal number is based on the order of appearance (see example below).
```
.type Name
.decl n(x:Name)
n("Homer").
n("Marge").
n("Bart").
n("Lisa").
n("Maggie").
.decl r(x:number)
.output r
r(1) :- n(x), n(y), ord(x) < ord(y), x="Homer", y="Bart".
r(2) :- n(x), n(y), ord(x) > ord(y), x="Maggie", y="Homer".
r(3) :- n(x), n(y), ord(x) > ord(y), x="Marge", y="Bart".
```
The output would be:
```
1
2
```
`r(3)` is not set, since ord("Marge") is less than ord("Bart") (the string "Marge" appears before the string "Bart", therefore it has a smaller ordinal number).

* Equality operations (**=** and **!=**) are also available for string types, by performing an ordinal comparison.

* **strlen(*string*)** returns the length of *string* as number.
```
.decl length(n:number)
.output length
length(n) :- n=strlen("Hello").
length(n) :- n=strlen("World!").
```
The output would be:
```
5
6
```

* **substr(*string*, *index*, *length*)** is used to return the substring starting at *index* with length *length* of *string*. The index is zero-based.
```
.type String
.decl substring(s:String)
.output substring
substring(s) :- s=substr("Hello_", 2, 3).
substring(s) :- string="World!", s=substr(string, 3, strlen(string)).
```
The output would be:
```
llo
ld!
```

* **to_number(*string*)** transforms a string representing a number to its associated number.
```
.decl tonumber(n:number)
.output tonumber
tonumber(n) :- n=to_number("123").
tonumber(n) :- n=to_number("1534").
```
The output would be:
```
123
1534
```
The reverse operation **to_string(*number*)** also exists, which turns a number to its string representation.
