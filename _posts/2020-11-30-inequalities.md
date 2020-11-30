---
sidebar: news_sidebar
title: 'Automatic Index Selection for Inequalities'
date: 2020-11-30
author: Samuel Isaac Arch
categories: [community]
permalink: inequalities.html
layout: post
---
Sam Arch has extended the existing automatic index selection technique in Soufflé to support indexed inequalities.  
An Honours thesis can be found [here](/pdf/samthesis.pdf), 
and the corresponding slides can be found [here](/pdf/samslides.pdf).

The work investigates two competing techniques to select indexes that accelerate rules with inequalities. We present a new auto-index selection strategy to construct a minimum cluster of B-Tree indexes to cover all searches with at most one inequality - the *B-Tree SPS* technique. The new technique ensures that rules with inequalities are evaluated efficiently, speeding up the evaluation time of real-world applications by up to 2.32x while incurring less than a 1% overhead in memory usage and 6% compilation time overhead. Furthermore, the technique is robust, with no practical degradation in performance for real-world applications compared to the existing auto-index selection scheme.

{% include links.html %}

