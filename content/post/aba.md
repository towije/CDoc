---
title: Skrypt oceny wypowiedzi dla ramki czasu
author: Tomasz Jerzyński
date: 2021-04-21
output: html_document
toc: true
categories: [Wyjście]
tags: [analiza emocjonalnych, postprocessing, R]
---


## Opis

Skrypt tworzy wskaźniki dla symboli i tokenów zebranych w wypowiedziach z danego okresu.

Skrypt można uprościć, biorąc pod uwagę tylko średnie ważone (pomijając sumy i średnie). Analogicznie jak w przypadku skryptu dla jednej wypowiedzi.

## Czytanie zbioru

Zbiór zawiera całą ramkę czasu. Jest odpowiedni dla podejścia zakładającego sąsiadujące ramki czasu.

### Lista zmiennych

Tylko zmienne, których używamy.

#### Dla symboli

```
#  [9,] "sum_scores_negative"
# [10,] "mean_scores_negative"
# [11,] "mean_weighted_scores_negative"
# [12,] "stdev_scores_negative"
# [13,] "sum_scores_neutral"
# [14,] "mean_scores_neutral"
# [15,] "mean_weighted_scores_neutral"
# [16,] "stdev_scores_neutral"
# [17,] "sum_scores_positive"
# [18,] "mean_scores_positive"
# [19,] "mean_weighted_scores_positive"
# [20,] "stdev_scores_positive"
```

#### Dla słów

```
# [32,] "sum_distances_h"
# [33,] "mean_distances_h"
# [34,] "mean_weighted_distances_h"
# [35,] "stdev_distances_h"
# [36,] "sum_distances_a"
# [37,] "mean_distances_a"
# [38,] "mean_weighted_distances_a"
# [39,] "stdev_distances_a"
# [40,] "sum_distances_f"
# [41,] "mean_distances_f"
# [42,] "mean_weighted_distances_f"
# [43,] "stdev_distances_f"
# [44,] "sum_distances_s"
# [45,] "mean_distances_s"
```

## Symbole

Dla symboli zajmujemy się tylko punktacją.

### Rozkład sum, średnich i średnich ważonych punktów przypisanych do poszczególnych emocji dla symboli — oceny z punktacji

#### Sumy

Wektor z trzema liczbami.

Funkcja ```prop.table``` tworzy liczbowy szereg proporcji powstały na podstawie szeregu liczb. Proporcje zawsze sumują się do 1.

```r
st_scores_pcounts <- prop.table(
    sum(sum_scores_negative),
    sum(sum_scores_neutral),
    sum(sum_scores_positive)
) * 100
```

#### Funkcja tworzące wagę dla obliczeń

```r
makew <- function(in_vec, ...) {
  # reskalujemy do odwrotności 0-1
  out_vec <- rescale(in_vec, to = c(1, 0), ...)
  # naciągamy średnia do 1
  out_vec <- out_vec + (1 - mean(out_vec, ...))
}
```

#### Waga dla symboli

```r
sw_negative <- makew(stdev_scores_negative)
sw_neutral <- makew(stdev_scores_neutral)
sw_positive <- makew(stdev_scores_positive)
```

#### Średnie ważone ze średnich sum punktów

Wektor z trzema liczbami.

```r
st_scores_w_means <- prop.table(
    weighted.mean(mean_scores_negative, sw_negative),
    weighted.mean(mean_scores_neutral, sw_neutral),
    weighted.mean(mean_scores_positive, sw_positive)
) * 100
```

#### Średnie ważone ze średnich ważonych sum punktów

Wektor z trzema liczbami.

```r
st_scores_w_wmeans <- prop.table(
    weighted.mean(mean_weighted_scores_negative, sw_negative),
    weighted.mean(mean_weighted_scores_neutral, sw_neutral),
    weighted.mean(mean_weighted_scores_positive, sw_positive)
) * 100
```

### Wskaźnik zagrożenia dla symboli

Macierz 3 × 3. W kolumnach emocje w wierszach typy wskaźników.

Funkcja ```rbind``` łączy wektory, traktując je jak kolejne wiersze.

```r
rbind(
  st_scores_pcounts,
  st_scores_w_means,
  st_scores_w_wmeans
)
```

Uśredniamy typy wskaźników ze względu na emocje (średnie kolumnowe).
Wygrywa kolumna z największą liczbą punktów. Zapamiętujemy jej nazwę i liczbę punktów.

Dobroć wskaźnika to odchylenie standardowe w zwycięskiej kolumnie. Odwrócone i przeskalowane. Do zakresu ```1-0``` z zakresu ```0-max_sd```, gdzie ```max_sd``` jest maksymalnym odchyleniem standardowym dla szeregu trzech liczb z zakresu 0 do 100.

```r
max_sd <- sd(c(0, 100, 100)) # to jest stała, można ją gdzieś zachować
symbol_goodness <- sd(symbol_summary[, get(names(symbol_winner))])
symbol_goodness <- rescale(
  symbol_goodness,
  from = c(0, max_sd),
  to = c(1, 0))
```

### Wyjście 

Na wyjściu dostajemy listę 3 wartości.

1) Nazwę zwycięskiej emocji.
2) Liczbę uzyskanych przez nią punktów.
3) Dobroć oszacowania.

## Słowa

Dla słów robimy dokładnie to samo. Zajmujemy się tylko dystansami.

### Rozkład sum, średnich i średnich ważonych dystansów od poszczególnych emocji dla słów

#### Suma dystansów

```r
t_dst_sp_n <- prop.table(
    sum(sum_distances_h),
    sum(sum_distances_a),
    sum(sum_distances_f),
    sum(sum_distances_s),
    sum(sum_distances_d)
) * 100
```

#### Wagi dla dystansów

```r
tw_h <- makew(stdev_distances_h)
tw_a <- makew(stdev_distances_a)
tw_f <- makew(stdev_distances_f)
tw_s <- makew(stdev_distances_s)
tw_d <- makew(stdev_distances_d)
tw_n <- makew(stdev_distances_n)
```

#### Średnie z dystansów

```r
t_dst_mp_n <- prop.table(
    weighted.mean(mean_distances_a, tw_h),
    weighted.mean(mean_distances_f, tw_a),
    weighted.mean(mean_distances_s, tw_f),
    weighted.mean(mean_distances_h, tw_s),
    weighted.mean(mean_distances_d, tw_d)
) * 100
```

#### Średnie ważone z dystansów

```r
t_dst_wmp_n <- prop.table(
    weighted.mean(mean_weighted_distances_a, tw_h),
    weighted.mean(mean_weighted_distances_f, tw_a),
    weighted.mean(mean_weighted_distances_s, tw_f),
    weighted.mean(mean_weighted_distances_h, tw_s),
    weighted.mean(mean_weighted_distances_d, tw_d)
) * 100
```

### Wskaźnik zagrożenia dla słów

Macierz 3 × 5. W kolumnach emocje, w wierszach wskaźniki.

```r
outt <- rbind(
  t_dst_sp_n,
  t_dst_mp_n,
  t_dst_wmp_n
)
```

### Odwrotność dystansu

Miary dystansu należy odwrócić, aby większy poziom wskaźnika wskazywał lepsze – a nie gorsze – powiązanie z danym sentymentem.

```r
outt <- prop.table(1 - outt)
```

Uśredniamy typy wskaźników ze względu na emocje (średnie kolumnowe).
Wygrywa kolumna z największą odwrotnością dystansu (najmniejszym dystansem). Zapamiętujemy jej nazwę i punktację.

Miarą dobroci będzie tu uśredniony dystans od kategorii N z każdego typu wskaźnika. Przeskalowany do zakresu 0 do 1.

```r
token_goodness <- mean(
  sum(sum_distances_n),
  weighted.mean(mean_distances_n, tw_n),
  weighted.mean(mean_weighted_distances_n, tw_n)
) / 100
```

### Wyjście 

Na wyjściu dostajemy listę 3 wartości.

1) Nazwę zwycięskiej emocji.
2) Uzyskany przez nią dystans.
3) Dobroć oszacowania.
