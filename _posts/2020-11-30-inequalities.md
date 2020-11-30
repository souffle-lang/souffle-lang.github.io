---
sidebar: news_sidebar
title: 'Automatic Index Selection for Inequalities'
date: 2020-11-30
author: Samuel Isaac Arch
categories: [community]
permalink: inequalities.html
layout: post
---
Sam Arch has extended the existing automatic index selection technique in Soufflé to support indexed inequalities. An Honours thesis can be found [here](/pdf/samthesis.pdf), 
and the corresponding slides can be found [here](/pdf/sampres.pdf).

The work investigates two competing automatic index selection techniques designed to accelerate rules with inequality constraints. We present a new auto-index selection strategy that constructs a minimum cluster of B-Tree indexes that cover all searches with at most one inequality - the *B-Tree SPS* technique. The new technique ensures that rules with inequalities are evaluated efficiently, speeding up the evaluation time of real-world applications by up to 2.32x. The technique is also lightweight, incurring less than a 1% overhead in maximum memory usage and only a 6% overhead in compilation time. Furthermore, the technique is robust, with no practical degradation in performance for real-world applications when compared to the existing auto-index selection scheme.

{% include links.html %}

