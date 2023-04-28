---
title: 泰勒
mathjax: true
categories: 高等数学
tags: 公式
---
一些常用的公式
<!--more-->
$e^x = \sum_{n=0}^{\infty} \frac{x^n}{n!}$

$\sin(x) = \sum_{n=0}^{\infty} \frac{(-1)^n x^{2n+1}}{(2n+1)!}$

$\cos(x) = \sum_{n=0}^{\infty} \frac{(-1)^n x^{2n}}{(2n)!}$

$\ln(1+x) = \sum_{n=1}^{\infty} \frac{(-1)^{n-1} x^n}{n}$

$\frac{1}{1-x} = \sum_{n=0}^{\infty} x^n$

$\arctan(x) = \sum_{n=0}^{\infty} \frac{(-1)^n x^{2n+1}}{2n+1}$

泰勒展开

在x-> 0处

$\sin(x) = x - \frac{x^3}{6} + \frac{x^5}{120} +o(x^5)$

$\cos(x) = 1 - \frac{x^2}{2} + \frac{x^4}{24} +o(x^4)$

$\ln(1+x) = x - \frac{x^2}{2} +\frac{x^3}{3} +o(x^3)$

$e^x = 1 + x + \frac{x^2}{2} + \frac{x^3}{6} + o(x^3)$

$(1+x)^{\frac{1}{x}} = e^\frac{\ln(1+x)}{x} = e^\frac{x - \frac{x^2}{2} +\frac{x^3}{3} +o(x^3)}{x}= e^{1-\frac{x}{2}+\frac{x^2}{3}+o(x^2)} = e*e^{-\frac{x}{2}+\frac{x^2}{3}+o(x^2)} = e*[1 + (-\frac{x}{2}+\frac{x^2}{3}+o(x^2))+ \frac{(-\frac{x}{2}+\frac{x^2}{3}+o(x^2))^2}{2}+o(-\frac{x}{2}+\frac{x^2}{3}+o(x^2))]$ 