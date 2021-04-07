---
title: Skrypt oceny dla jednej wypowiedzi
author: Tomasz Jerzyński
date: 2021-03-27
output: html_document
toc: true
categories: [Wyjście]
tags: [analiza emocjonalnych, postprocessing, R]
---

## Symbole

### Sprawdzenie, czy występują sklasyfikowane symbole

Jeśli nie, to pomijamy tę sekcję i zakładamy zerowy wektor wynikowy.


```r
if (one[, count_symbols_scored] == 0) stop("Brak sklasyfikowanych symboli.")
```

### Czynniki dla symboli

#### 1 czynnik z klasyfikacji symboli

jest rozkładem częstości poszczególnych ocen.


```r
outsk <- one[, prop.table(c(
  pozytywne = count_symbols_positive,
  neutralne = count_symbols_neutral,
  negatywne = count_symbols_negative))]
outsk
```

```
##  pozytywne  neutralne  negatywne 
## 0.36363636 0.54545455 0.09090909
```

Ten czynnik pomijamy.

### Trzy czynniki z punktacji dla symboli

są to rozkłady częstości dla sum, średnich i średnich ważonych poszczególnych ocen.

#### 2 czynnik -- sumy


```r
outss <- one[, prop.table(c(
  pozytywne = sum_scores_positive,
  neutralne = sum_scores_neutral,
  negatywne = sum_scores_negative))]
outss
```

```
##  pozytywne  neutralne  negatywne 
## 0.66625249 0.27012922 0.06361829
```

#### 3 czynnik -- średnie


```r
outsm <- one[, prop.table(c(
  pozytywne = mean_scores_positive,
  neutralne = mean_scores_neutral,
  negatywne = mean_scores_negative))]
outsm
```

```
## pozytywne neutralne negatywne 
## 0.6677460 0.2684932 0.0637609
```

#### 4 czynnik -- średnie ważone


```r
outsmw <- one[, prop.table(c(
  pozytywne = mean_weighted_scores_positive,
  neutralne = mean_weighted_scores_neutral,
  negatywne = mean_weighted_scores_negative))]
outsmw
```

```
## pozytywne neutralne negatywne 
## 0.6674662 0.2707620 0.0617718
```

### łączenie czynników dla symboli



Analiza czynnikowa wykazała, że w całym zbiorze miliona wypowiedzi średnie proporcje ładunków dla sum, średnich i średnich ważonych poszczególnych ocen wynosiły:


```r
ssw
```

```
##   sum   mea w.mea 
##  0.24  0.38  0.38
```

Trzy rodzaje wskaźników dla poszczególnych emocji zagregowane zostały do średnich ważonych powyższymi proporcjami.


```r
outs <- rbind(outss, outsm, outsmw)
outs <- c(
  pozytywne = weighted.mean(outs[, 1], ssw),
  neutralne = weighted.mean(outs[, 2], ssw),
  negatywne = weighted.mean(outs[, 3], ssw))
outs
```

```
##  pozytywne  neutralne  negatywne 
## 0.66728121 0.26974798 0.06297082
```

## Słowa

### Sprawdzenie czy występują sklasyfikowane słowa

Jeśli nie, to pomijamy tę sekcję i zakładamy zerowy wektor wynikowy.


```r
if (one[, count_tokens_scored] == 0) stop("Brak sklasyfikowanych słów.")
```

### Redukcja emocji

Wszystkie emocje poza szczęściem uśredniamy jako negatywne. Osobno dla sum, średnich i średnich ważonych.


```r
one[, c("sum_distances_neg",
        "mean_distances_neg",
        "mean_weighted_distances_neg") := .(
  mean(sum_distances_a,
       sum_distances_f,
       sum_distances_s,
       sum_distances_d),
  mean(mean_distances_a,
       mean_distances_f,
       mean_distances_s,
       mean_distances_d),
  mean(mean_weighted_distances_a,
       mean_weighted_distances_f,
       mean_weighted_distances_s,
       mean_weighted_distances_d)
)]
```


### Trzy czynniki z punktacji dla słów

są to rozkłady częstości dla sum, średnich i średnich ważonych poszczególnych dystansów.

#### 1 - czynnik z sum dystansów


```r
outts <- one[, prop.table(c(
  pozytywne = sum_distances_h,
  neutralne = sum_distances_n,
  negatywne = sum_distances_neg))]
outts
```

```
## pozytywne neutralne negatywne 
## 0.3076923 0.2692308 0.4230769
```

#### 2 - czynnik ze średnich dystansów


```r
outtm <- one[, prop.table(c(
  pozytywne = mean_distances_h,
  neutralne = mean_distances_n,
  negatywne = mean_distances_neg))]
outtm
```

```
## pozytywne neutralne negatywne 
## 0.3098941 0.2668173 0.4232886
```

#### 3 - czynnik ze średnich ważonych dystansów


```r
outtmw <- one[, prop.table(c(
  pozytywne = mean_weighted_distances_h,
  neutralne = mean_weighted_distances_n,
  negatywne = mean_weighted_distances_neg))]
outtmw
```

```
## pozytywne neutralne negatywne 
## 0.2939417 0.2817540 0.4243043
```

### łączenie czynników dla słów



Analiza czynnikowa wykazała, że w całym zbiorze miliona wypowiedzi średnie proporcje ładunków dla sum, średnich i średnich ważonych poszczególnych ocen wynosiły:


```r
tdw
```

```
##   sum   mea w.mea 
##  0.07  0.46  0.47
```

Trzy rodzaje wskaźników dla poszczególnych emocji zagregowane zostały do średnich ważonych powyższymi proporcjami.


```r
outt <- rbind(outts, outtm, outtmw)
outt <- c(
  pozytywne = weighted.mean(outt[, 1], tdw),
  neutralne = weighted.mean(outt[, 2], tdw),
  negatywne = weighted.mean(outt[, 3], tdw))
outt
```

```
## pozytywne neutralne negatywne 
## 0.3022424 0.2740065 0.4237511
```
### Odwrotność dystansu

Miary dystansu należy odwrócić, aby większy poziom wskaźnika wskazywał lepsze -- a nie gorsze -- powiązanie z danym sentymentem.


```r
outt <- prop.table(1 - outt)
```

## Wynik


```
##    sentyment    symbole     slowa
## 1: pozytywne 0.66728121 0.3488788
## 2: neutralne 0.26974798 0.3629968
## 3: negatywne 0.06297082 0.2881244
```

![](S:/analizy/CP/250-data-table-analise/273-one-indicator_files/figure-html/unnamed-chunk-19-1.png)<!-- -->

W przypadku dodania kolejnych systemów ocen, każdy mogłby być prezentowany w postaci kolejnego trójkolorowego paska.

## Uproszczenie

Analiza czynnikowa wykazała, że najlepszymi wskaźnikami są średnie ważone.
Można zatem uprościć całą procedurę korzystając jedynie z nich.

### Sprawdzenie czy występują sklasyfikowane symbole

Jeśli nie, to pomijamy tę sekcję i zakładamy zerowy wektor wynikowy.


```r
if (one[, count_symbols_scored] == 0) stop("Brak sklasyfikowanych symboli.")
```

### Punktacja symboli


```r
outs <- one[, prop.table(c(
  pozytywne = mean_weighted_scores_positive,
  neutralne = mean_weighted_scores_neutral,
  negatywne = mean_weighted_scores_negative))]
```


## Dystanse dla słów

### Sprawdzenie czy występują sklasyfikowane słowa

Jeśli nie, to pomijamy tę sekcję i zakładamy zerowy wektor wynikowy.


```r
if (one[, count_tokens_scored] == 0) stop("Brak sklasyfikowanych słów.")
```

### redukcja emocji


```r
one[, mean_weighted_distances_neg :=
        mean(mean_weighted_distances_a,
             mean_weighted_distances_f,
             mean_weighted_distances_s,
             mean_weighted_distances_d)]
```

### Dystans słów


```r
outt <- one[, prop.table(c(
  pozytywne = mean_weighted_distances_h,
  neutralne = mean_weighted_distances_n,
  negatywne = mean_weighted_distances_neg
))]
outt
```

```
## pozytywne neutralne negatywne 
## 0.2939417 0.2817540 0.4243043
```

## Wyniki z uproszczenia


```
##    sentyment   symbole     slowa
## 1: pozytywne 0.6674662 0.2939417
## 2: neutralne 0.2707620 0.2817540
## 3: negatywne 0.0617718 0.4243043
```

![](S:/analizy/CP/250-data-table-analise/273-one-indicator_files/figure-html/unnamed-chunk-25-1.png)<!-- -->

Jak widać wyniki pełnej i uproszczonej analizy są podobne.

## Cały uproszczony skrypt

```r
# Title     : Uproszczony skrypt oceny dla jednej wypowiedzi - funkcja
# Created by: Tomasz Jerzyński
# Created on: 26.03.2021, 14:03
rm(list = ls())
library(data.table)
library(ggplot2)
library(RColorBrewer)
library(colorspace)

one_result <- function(one) {
  if (one[, count_symbols_scored] > 0) {
    # symbole
    outs <- one[, prop.table(c(
      pozytywne = mean_weighted_scores_positive,
      neutralne = mean_weighted_scores_neutral,
      negatywne = mean_weighted_scores_negative))]
  } else {
    outs <- c(pozytywne = 0,
              neutralne = 0,
              negatywne = 0)
  }
  # tokeny
  if (one[, count_tokens_scored] > 0) {
    # redukcja emocji
    one[, mean_weighted_distances_neg :=
            mean(mean_weighted_distances_a,
                 mean_weighted_distances_f,
                 mean_weighted_distances_s,
                 mean_weighted_distances_d)]
    # słowa
    outt <- one[, prop.table(c(
      pozytywne = mean_weighted_distances_h,
      neutralne = mean_weighted_distances_n,
      negatywne = mean_weighted_distances_neg
    ))]
    # odwrotność
    outt <- prop.table(1 - outt)
  } else {
    outt <- c(pozytywne = 0,
              neutralne = 0,
              negatywne = 0
    )
  }
  # output
  out1 <- data.table(
    sentyment = names(outs),
    symbole = outs,
    slowa = outt)
  print(out1)
  out1 <- melt(out1, id.vars = 1)
  # grafika
  ggplot(out1, aes(x = variable, y = value, fill = sentyment)) +
    geom_col() +
    scale_fill_brewer(palette = "Set1") +
    coord_flip() +
    xlab(NULL) +
    theme_minimal() +
    theme(axis.line.x = element_blank(),
          axis.text.x = element_blank(),
          axis.ticks.x = element_blank(),
          axis.title.x = element_blank(),
          panel.grid.major.x = element_blank(),
          panel.grid.minor.x = element_blank())
}
```