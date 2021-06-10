---
title: Wyjście miar dystansowych dla emocji
author: Tomasz Jerzyński
date: 2021-06-10
output:
    html_document:
        keep_md: true
        toc: true
        toc_float: false
categories: [Wyjście]
tags: [analiza emocjonalnych, postprocessing, R]
---

## Opis

Skrypt tworzy szereg liczb. Kolejne liczby odpowiadają kolejnym emocjom.
Wyjście dotyczy danego okresu (ramki czasu) określonej przez liczbę wypowiedzi w pliku wejściowym.

## Dla jednej wypowiedzi

Dla jednej wypowiedzi dane wymagają jedynie obliczenia proporcji, odwrócenia.

```r
t_dst_wmp_n = prop.table(
  mean_weighted_distances_a,
  mean_weighted_distances_f,
  mean_weighted_distances_s,
  mean_weighted_distances_h,
  mean_weighted_distances_d
) * 100

prop.table(1 - t_dst_wmp_n) * 100
```

## Dla ramki czasu

### Funkcja tworzące wagę dla obliczeń

```r
makew <- function(in_vec, ...) {
  # reskalujemy do odwrotności 0-1
  out_vec <- rescale(in_vec, to = c(1, 0), ...)
  # naciągamy średnia do 1
  out_vec <- out_vec + (1 - mean(out_vec, ...))
}
```

### Wagi dla dystansów

```r
tw_h = makew(stdev_distances_h)
tw_a = makew(stdev_distances_a)
tw_f = makew(stdev_distances_f)
tw_s = makew(stdev_distances_s)
tw_d = makew(stdev_distances_d)
tw_n = makew(stdev_distances_n)
```


### Średnie ważone ze średnich ważonych dystansów

```r
t_dst_wmp_n = prop.table(
    weighted.mean(mean_weighted_distances_a, tw_h),
    weighted.mean(mean_weighted_distances_f, tw_a),
    weighted.mean(mean_weighted_distances_s, tw_f),
    weighted.mean(mean_weighted_distances_h, tw_s),
    weighted.mean(mean_weighted_distances_d, tw_d)
) * 100
```


### Wyjście

Miary dystansu należy odwrócić, aby większy poziom wskaźnika wskazywał lepsze – a nie gorsze – powiązanie z danym sentymentem.

```r
prop.table(1 - t_dst_wmp_n) * 100
```


## Skrypt ```R```

### Dla 1 wypowiedzi

```r
t_dst_wmp_n <- d1[, prop.table(c(
  mean_weighted_distances_a,
  mean_weighted_distances_f,
  mean_weighted_distances_s,
  mean_weighted_distances_h,
  mean_weighted_distances_d)
) * 100]

outt <- prop.table(1 - t_dst_wmp_n) * 100
```


### Dla ramki czasu


```r
library(data.table)
library(scales)
d1 <- fread("../data/nawl_cpa/nawl_cpa2.csv",
            encoding = "UTF-8")

makew <- function(in_vec, ...) {
  # reskalujemy do odwrotności 0-1
  out_vec <- rescale(in_vec, to = c(1, 0), ...)
  # naciągamy średnia do 1
  out_vec <- out_vec + (1 - mean(out_vec, ...))
}

tw_h <- d1[, makew(stdev_distances_h)]
tw_a <- d1[, makew(stdev_distances_a)]
tw_f <- d1[, makew(stdev_distances_f)]
tw_s <- d1[, makew(stdev_distances_s)]
tw_d <- d1[, makew(stdev_distances_d)]
tw_n <- d1[, makew(stdev_distances_n)]

t_dst_wmp_n <- d1[, prop.table(c(
  weighted.mean(mean_weighted_distances_a, tw_h),
  weighted.mean(mean_weighted_distances_f, tw_a),
  weighted.mean(mean_weighted_distances_s, tw_f),
  weighted.mean(mean_weighted_distances_h, tw_s),
  weighted.mean(mean_weighted_distances_d, tw_d))
) * 100]

outt <- prop.table(1 - t_dst_wmp_n) * 100
names(outt) <- c("H", "A", "F", "S", "D")
outt
```

```
##        H        A        F        S        D 
## 20.50925 20.39895 21.05587 16.26237 21.77355
```

```r
outb <- data.table(Emo = factor(names(outt),ordered=T), Pwr = outt)
library(ggplot2)
ggplot(outb, aes(x=Emo,y=Pwr, group=1))+
        geom_point()+
        scale_y_continuous(limits = c(0,100))+
        coord_polar()
```

![Przykład](/img/280-out-twm/out-twm.png)<!-- -->
