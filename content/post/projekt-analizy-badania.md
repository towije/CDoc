---
title: "Projekt analizy wyników planowanego Badania **CP affective**"
subtitle: "Aplikacja na wygenerowanych danych"
author: Tomasz Jerzyński
date: 2020-11-27
output:
  html_document:
    highlight: "zenburn"
    css: style.css
    keep_md: true
    toc: true
    toc_float: true
slug: [projekt-analizy-badania]
categories: [badanie]
tags: [badanie, ankieta, analiza emocjonalnych, R]

---

Odtworzenie analizy jakiej dokonali na swoich danych badacze z Nenckiego.

{{< code language="r" title="Środowisko" >}}
library(data.table)
library(ggplot2)
library(scales)
{{< /code >}}


Przypuśćmy, że otrzymaliśmy 30 odpowiedzi dotyczących jednego z tagów.
Respondenta mamy w wierszach, a w kolumnach numer respondenta i kolejno jego odpowiedzi na pytania dotyczące kolejnych emocji.

Emocje to **h, a, s, f, d, h**. Pytanie jest jedno. *Czy i jak bardzo kojarzy sie Panu/Pani to słowo/ta fraza/ten symbol z przedstawionymi niżej emocjami.* Prosimy wybrać odpowiednią wartość dla każdej z emocji na skali od (1) w ogóle mi się nie kojarzy, do (7) bardzo mi sie kojarzy. Ta siedmiopunktowa skala pochodzi z NAWL. Nic nie stoi na przeszkodzie żeby zrobić większa lub zupełnie ciągłą ustawianą suwakami. Jednak byłbym temu przeciwny, bo to niepotrzebne komplikowanie sprawy dla respondenta.


{{< code language="r" title="Generator odpowiedzi" >}}
set.seed(666)
dsc <- data.table(no = 1:30,
                  h = rnorm(30, 5, .7),
                  a = rnorm(30, 3, 1),
                  s = rnorm(30, 6, .3),
                  f = rnorm(30, 2, .4),
                  d = rnorm(30, 4, .6))
dscm <- melt(dsc, id.vars = 1)
dscm[, v2 := round(rescale(value, c(1, 7), range(value))), variable]
dscm[, .(min(v2), max(v2)), variable]
{{< /code >}}

{{< code language="r" title="Zbiór 30 odpowiedzi dotyczących jednego obiektu" >}}
##     no        h        a        s        f        d
##  1:  1 5.527318 3.754996 5.701896 1.805206 3.535786
##  2:  2 6.410048 2.358511 5.728173 1.640015 4.740117
##  3:  3 4.751406 4.431132 5.604531 2.128192 4.382766
##  4:  4 6.419717 2.375449 6.580348 2.125892 4.616456
##  5:  5 3.448188 3.228995 6.203419 1.583683 3.866918
...
## 26: 26 3.962408 2.179176 6.301601 2.428893 3.395388
## 27: 27 4.211346 2.763879 6.120934 1.324820 4.390885
## 28: 28 3.765310 2.230296 6.429752 1.820284 4.907308
## 29: 29 4.256167 3.719994 6.041202 1.912873 4.067490
## 30: 30 4.059901 4.279951 5.597548 1.179955 4.093062
##     no        h        a        s        f        d
{{< /code >}}


{{< code language="r" title="Statystyki opisowe zbioru" >}}
##        no              h               a                
##  Min.   : 1.00   Min.   :1.000   Min.   :1.00     
##  1st Qu.: 8.25   1st Qu.:2.000   1st Qu.:2.25     
##  Median :15.50   Median :4.000   Median :4.00     
##  Mean   :15.50   Mean   :3.733   Mean   :3.80     
##  3rd Qu.:22.75   3rd Qu.:5.000   3rd Qu.:5.00     
##  Max.   :30.00   Max.   :7.000   Max.   :7.00     
##        s               f               d      
##  Min.   :1.000   Min.   :1.000   Min.   :1.0  
##  1st Qu.:4.000   1st Qu.:3.000   1st Qu.:4.0  
##  Median :5.000   Median :3.500   Median :5.0  
##  Mean   :4.533   Mean   :3.733   Mean   :4.4  
##  3rd Qu.:5.000   3rd Qu.:5.000   3rd Qu.:5.0  
##  Max.   :7.000   Max.   :7.000   Max.   :7.0
{{< /code >}}

{{< code language="r" title="Obrazek" isCollapsed="true" >}}
ggplot(dscm, aes(x = value, color = variable)) +
  geom_density(size = 3) +
  scale_color_brewer("Emocje", palette = "Set1") +
  scale_x_continuous(breaks = 1:7, limits = c(1, 7)) +
  xlab("Wartości na skali odpowiedzi") +
  theme_bw(base_size = 16)
{{< /code >}}

![Gęstość rozkładów odpowiedzi](/img/density-pab.png)<!-- -->

Ocena danego słowa opiera się na bardzo prostym systemie szacowania dystansu euklidesowego między wektorami wzorcowymi, a wektorami otrzymanymi od respondentów. Przykładowo wektor wzorcowy szczęścia to ```(7,1,1,1,1)```. Respondent numer 1 odpowiedział, że dany obiekt kojarzy mu się bardzo z obrzydzeniem i tak sobie z cała resztą - jego wektor wygląda tak: ```(1,2,1,5,7)```. Zatem dystans danego obiektu według danego respondenta wynosi ```sqrt(sum(vector1 - vector2)^2) = 9.43```

{{< code language="r" title="Wektory wzorcowe dla poszczególnych emocji" >}}
pure_h <- c(7, 1, 1, 1, 1) # for happiness,
pure_a <- c(1, 7, 1, 1, 1) # for anger,
pure_s <- c(1, 1, 7, 1, 1) # for sadness,
pure_f <- c(1, 1, 1, 7, 1) # for fear,
pure_d <- c(1, 1, 1, 1, 7) # for disgust,
pure_n <- c(1, 1, 1, 1, 1) # neutral state.
{{< /code >}}

Następnie liczymy odległości wektorów każdego z respondentów do każdego z wektorów wzorcowych.

{{< code language="r" title="Dystans" >}}
eudi <- function(vector1, vector2) sqrt(sum((vector1 - vector2)^2))
dist_names <- c("dist_h", "dist_a", "dist_s", 
                "dist_f", "dist_d", "dist_n")
dsc[, (dist_names) := list(eudi(pure_h, c(h, a, s, f, d)),
                           eudi(pure_a, c(h, a, s, f, d)),
                           eudi(pure_s, c(h, a, s, f, d)),
                           eudi(pure_f, c(h, a, s, f, d)),
                           eudi(pure_d, c(h, a, s, f, d)),
                           eudi(pure_n, c(h, a, s, f, d))), no]
{{< /code >}}

{{< code language="r" title="Odległości dodane do zbioru" >}}
##     no h a s f d   dist_h    dist_a   dist_s   dist_f   dist_d    dist_n
##  1:  1 5 5 3 3 3 5.656854  5.656854 7.483315 7.483315 7.483315  6.633250
##  2:  2 7 2 4 3 6 6.244998  9.949874 8.660254 9.327379 7.141428  8.660254
##  3:  3 4 6 3 5 5 8.366600  6.782330 9.055385 7.615773 7.615773  8.366600
##  4:  4 7 2 7 5 5 8.306624 11.357817 8.306624 9.643651 9.643651 10.246951
##  5:  5 1 4 5 3 4 8.602325  6.164414 5.099020 7.071068 6.164414  6.164414
...
##  6:  6 5 4 4 5 5 7.348469  8.124038 8.124038 7.348469 7.348469  8.124038
##  7:  7 2 4 5 5 5 9.055385  7.615773 6.782330 6.782330 6.782330  7.615773
##  8:  8 3 1 5 3 4 6.708204  8.306624 4.582576 6.708204 5.744563  5.744563
##  9:  9 2 4 7 4 5 9.746794  8.426150 5.916080 8.426150 7.681146  8.426150
## 10: 10 4 7 5 4 3 8.602325  6.164414 7.874008 8.602325 9.273618  8.602325
##     no h a s f d   dist_h    dist_a   dist_s   dist_f   dist_d    dist_n
{{< /code >}}

I na koniec liczymy średnią dystansów ocen od wzorców dla wszystkich odpowiedzi dotyczących danego obiektu.
Dodajemy także odchylenie standardowe i liczbę obserwacji, żeby policzyć przedziały ufności i wyskalować wyniki.

{{< code language="r" title="Tabela wynikowa" >}}
data.table(
  dist_names,
  Suma = c(dsc[, lapply(.SD, sum), .SDcols = dist_names]),
  SD = c(dsc[, lapply(.SD, sd), .SDcols = dist_names]),
  N = c(dsc[, lapply(.SD, length), .SDcols = dist_names])
)

##    dist_names     Suma       SD  N
## 1:     dist_h 230.0837 1.304941 30
## 2:     dist_a 226.4177 1.672985 30
## 3:     dist_s 210.6684 1.268008 30
## 4:     dist_f 231.2384 1.044297 30
## 5:     dist_d 213.1395 1.435005 30
## 6:     dist_n 224.1478 1.221349 30
{{< /code >}}

Metoda jest banalna, ale jednocześnie bardzo efektywna.
