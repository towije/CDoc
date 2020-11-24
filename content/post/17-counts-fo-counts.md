---
title: "Powtarzalność tagów i fraz"
author: Tomasz Jerzyński
date: 2020-11-16
slug: [counts-of-counts]
categories: [Wejście]
tags: [analiza emocjonalnych, preprocessing, R, rozkład]
---

Oszacowanie kształtu rozkładów występowania słów i fraz w danych.

## Tagi - jedno słowo

W przypadku tagów, warte zainteresowania/przejrzenia są pozycje powyżej 50 wystąpień - ``181`` sztuk, lub powyżej 15 wystąpień - ``354`` sztuk.

![oi](/img/unnamed-chunk-2-1.png)<!-- -->


{{< code language="r" title="Rozkład powtarzalności tagów" >}}
##                  N      %
## [0,15]        1893  84.25
## (15,25]         86   3.83
## (25,50]         87   3.87
## (50,100]        62   2.76
## (100,1e+03]     91   4.05
## (1e+03,1e+05]   28   1.25
## Sum           2247 100.00
## NA's             1   0.04
{{< /code >}}

## Frazy dwu- wyrazowe

![](/img/unnamed-chunk-4-1.png)<!-- -->

W przypadku fraz dwu- wyrazowych, warte zainteresowania/przejrzenia są pozycje powyżej 50 wystąpień - ``112`` sztuk, lub powyżej 15 wystąpień - ``187`` sztuk.


{{< code language="r" title="Rozkład powtarzalności fraz 2- wyrazowych" >}}
##                 N      %
## [0,15]        503  72.90
## (15,25]        39   5.65
## (25,50]        36   5.22
## (50,100]       24   3.48
## (100,1e+03]    54   7.83
## (1e+03,4e+06]  34   4.93
## Sum           690 100.00
## NA's            0   0.00
{{< /code >}}

## Frazy trzy- wyrazowe

![](/img/unnamed-chunk-6-1.png)<!-- -->

W przypadku fraz dwu- wyrazowych, warte zainteresowania/przejrzenia są pozycje powyżej 15 wystąpień - ``133`` sztuk.

{{< code language="r" title="Rozkład powtarzalności fraz 3- wyrazowych" >}}
##                 N      %
## [0,15]        288  68.41
## (15,25]        18   4.28
## (25,50]        22   5.23
## (50,100]       23   5.46
## (100,1e+03]    39   9.26
## (1e+03,6e+06]  31   7.36
## Sum           421 100.00
## NA's            0   0.00
{{< /code >}}
