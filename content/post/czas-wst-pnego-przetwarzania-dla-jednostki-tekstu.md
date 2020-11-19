---
title: Czas wstępnego przetwarzania dla jednostki tekstu
author: Tomasz Jerzyński
date: '2020-11-13'
slug: [preprocessing-time]
categories: [Wejście]
tags: [preprocessing, R]
---


## Czas przetwarzania

### Podsumowanie

- Czasy przetwarzania pojedynczego wpisu o średniej długości są tak małe, że interpreter nie jest w stanie ich zmierzyć.
- Przy 10K słów przetwarzanie trwa około 0.2 sekundy.
- Całe przetwarzanie wstępne, nawet wzbogacone o kilka innych elementów, może być realizowane w czasie rzeczywistym, a do repozytorium może trafiać już "namoczony" materiał.



[ Obliczenia na jednym rdzeniu. ]

### 30 słów
Średnio w komentarzu pojawia się 25 znaczących słów. 30 wszystkich.
Wybierzmy sobie zupełnie dowolny przypadek spełniający to założenie.

```r
library(data.table)
load("data/DUMP SO/fruktus137.RData")
load("stopwords_pl.RData")
txt <- fruktus[137, Tresc.wypowiedzi]
print(txt)
```

```
## [1] "Nie jestes zadnym doktorem, ale draniem i glupcem. Dobrze wiesz ilu twoich kolegów i kolezanek po fachu jest banowanych i szykanowanych za demaskowanie klamstw dotyczacych tej fake-epidemii a ty szmato bierzesz udzial w kreowaniu tej psychozy.\nTfu!!!"
```

Urocze i na temat.

#### Na małe litery.

```r
system.time(txt <- tolower(txt))
```

```
##    user  system elapsed 
##       0       0       0
```

#### Tylko litery

```r
system.time(txt <- gsub("[^abcdefghijklmnopqrstuvwxyzęćółąćśżźń]", " ", txt))
```

```
##    user  system elapsed 
##       0       0       0
```

#### Bez 1 i 2 literowych

```r
system.time(txt<-gsub("\\b\\w{1,2}\\b", " ",txt))
```

```
##    user  system elapsed 
##       0       0       0
```
#### Bez stopwords

```r
system.time({
  txt <- paste0(" ", txt, " ")
  stopwords_pl <- paste0(" ", stopwords_pl, " ")
  for (idx in stopwords_pl) {
    txt <- gsub(idx, " ", txt, fixed = T)
  }
})
```

```
##    user  system elapsed 
##    0.01    0.00    0.01
```
#### Bez nadmiarowych spacji

```r
system.time(txt<-gsub("\\s+", " ",txt))
```

```
##    user  system elapsed 
##       0       0       0
```

#### Wszystko razem

```r
txt <- fruktus[137, Tresc.wypowiedzi]
system.time({
  txt <- tolower(txt)
  txt <- gsub("[^abcdefghijklmnopqrstuvwxyzęćółąćśżźń]", " ", txt)
  txt<-gsub("\\b\\w{1,2}\\b", " ",txt)
  txt <- paste0(" ", txt, " ")
  stopwords_pl <- paste0(" ", stopwords_pl, " ")
  for (idx in stopwords_pl) {
    txt <- gsub(idx, " ", txt, fixed = T)
  }
  txt<-gsub("\\s+", " ",txt)
})
```

```
##    user  system elapsed 
##       0       0       0
```

Czasy przetwarzania sa tak małe, że interpreter nie jest w stanie ich zmierzyć. Jedyne drgnięcie widać przy stopwords, bo tam interpreter musiał przelecieć, przez listę kilkudziesięciu słów. Wchodzi czas dostępu, a nie samo przetwarzanie.

### 10K słów (czyli 333 razy więcej)

```r
txt <- rep(fruktus[137, Tresc.wypowiedzi],333)
system.time({
  txt <- tolower(txt)
  txt <- gsub("[^abcdefghijklmnopqrstuvwxyzęćółąćśżźń]", " ", txt)
  txt<-gsub("\\b\\w{1,2}\\b", " ",txt)
  txt <- paste0(" ", txt, " ")
  stopwords_pl <- paste0(" ", stopwords_pl, " ")
  for (idx in stopwords_pl) {
    txt <- gsub(idx, " ", txt, fixed = T)
  }
  txt<-gsub("\\s+", " ",txt)
})
```

```
##    user  system elapsed 
##    0.22    0.00    0.22
```

Przy 10K słów przetwarzanie trwa jakieś 0.2 sekundy.

Całe przetwarzanie wstępne, nawet wzbogacone o kilka innych elementów, może być spokojnie realizowane w czasie rzeczywistym, a do repozytorium może trafiać już "namoczony" materiał.
