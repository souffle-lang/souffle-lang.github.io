---
sidebar: news_sidebar
title: 'An Efficient Interpreter for Soufflé'
date: 2020-07-23
author: Xiaowen Hu
categories: [community]
permalink: interpreter.html
layout: post
---
Xiaowen Hu has implemented an efficient interpreter for Soufflé. 
An Honours thesis can be found [here](/pdf/xiaowenthesis.pdf), 
and the corresponding slides can be found [here](/pdf/xiaowenslides.pdf).

The work investigates various techniques to implement a high performance interpreter for Soufflé,
so that the performance gap between Soufflé and synthesised C++ Soufflé code can be improved.
We present a new strategy for implementing an efficient tree-walk interpreter - 
the *Switch-based Shadow Tree* technique.
The new technique ensures a high-performance tree-walk interpreter for 
a coarse-grained instruction set and has sound engineering principles for the ease of development.

{% include links.html %}

