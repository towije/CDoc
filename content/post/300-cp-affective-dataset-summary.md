---
title: "Podsumowanie badania CP Affective"
subtitle: "Przetwarzanie surowego zbioru danych"
author: Tomasz Jerzyński
date: 2021-02-23
output: html_document
slug: [c-affective-dataset-summary]
categories: [Badanie]
tags: [analiza emocjonalnych, analiza n-gramów, R, sondaż]
---

Przetwarzanie i sprawdzenie surowego zbioru danych. Plik tekstowy UTF-16,
234.2Mb.

## Ustawienia i wczytywanie funkcji pomocniczych

Na początek zmiana UTF-16 na UTF-8. Następnie wczytanie bibliotek.

```r
library(data.table)
library(readxl)
library(stringr)
source("tonames.R")
source("fr.R")
source("ntil.R")
source("number_formatter.R")
```

## Czytanie zbioru

```r
d1 <- fread("data/SWR_Crisis_baza_23.02.2021-utf8.txt",
            encoding = "UTF-8")
d1[1:10, 1:10]
```

```
##     Id     V2 Wiek Liczba_mieszkańców klasa Płeć P1_1 P1_2 P1_3 P1_4
##  1:  1 284403   75              63328     2    F   NA   NA   NA   NA
##  2:  2 397923   45              66314     2    M   NA   NA   NA   NA
##  3:  3 402342   34              88015     2    F   NA   NA   NA   NA
##  4:  4 372407   39              22801     2    M   NA   NA   NA   NA
##  5:  5 273150   27              66314     2    M   NA   NA   NA   NA
##  6:  6 380685   25              65876     2    M   NA   NA   NA   NA
##  7:  7  59646   43              48922     2    F   NA   NA   NA   NA
##  8:  8 403993   41              68902     2    F   NA   NA   NA   NA
##  9:  9 405974   29              13853     2    M   NA   NA   NA   NA
## 10: 10 288572   70              93363     2    F   NA   NA   NA   NA
```

Zbiór został dostarczony w standardowej formie, gdzie w wierszach mamy
respondentów a w kolumnach poszczególne odpowiedzi. Jest to nieoptymalna i
nieodpowiednia forma dla tego typu zbioru danych.

## Usuwamy niepotrzebne zmienne demograficzno-społeczne

Na tym etapie nie ine interesują nas parametry respondentów.

```r
d1 <- d1[, -(2:6)]
```

## Struktura braku informacji

Każdy z respondentów oceniał tylko około 100 słów. Dlatego surowy zbiór składa
się głównie z pustych komórek.

```r
d1[1:10, 1:10]
```

```
##     Id P1_1 P1_2 P1_3 P1_4 P1_5 P2_1 P2_2 P2_3 P2_4
##  1:  1   NA   NA   NA   NA   NA   NA   NA   NA   NA
##  2:  2   NA   NA   NA   NA   NA   NA   NA   NA   NA
##  3:  3   NA   NA   NA   NA   NA   NA   NA   NA   NA
##  4:  4   NA   NA   NA   NA   NA   NA   NA   NA   NA
##  5:  5   NA   NA   NA   NA   NA   NA   NA   NA   NA
##  6:  6   NA   NA   NA   NA   NA   NA   NA   NA   NA
##  7:  7   NA   NA   NA   NA   NA   NA   NA   NA   NA
##  8:  8   NA   NA   NA   NA   NA   NA   NA   NA   NA
##  9:  9   NA   NA   NA   NA   NA   NA   NA   NA   NA
## 10: 10   NA   NA   NA   NA   NA   NA   NA   NA   NA
```

## Tworzymy długi zbiór

Pierwszym etapem porządkowania będzie utworzenie zbioru, gdzie każdy wiersz
będzie odpowiadał tylko jednej odpowiedzi respondenta.

```r
d2 <- melt(d1, id.vars = "Id")
d2
```

```
##             Id variable value
##        1:    1     P1_1    NA
##        2:    2     P1_1    NA
##        3:    3     P1_1    NA
##        4:    4     P1_1    NA
##        5:    5     P1_1    NA
##       ---                    
## 61269996: 6123  P2000_5    NA
## 61269997: 6124  P2000_5    NA
## 61269998: 6125  P2000_5    NA
## 61269999: 6126  P2000_5    NA
## 61270000: 6127  P2000_5    NA
```

```r
cells1 <- nrow(d2)
fen0(cells1)
```

```
## [1] "61,270,000"
```

W surowym zbiorze mamy ``61,270,000`` komórek.

## Usuwamy puste komórki

Ze zbioru w długiej formie usuwamy puste wiersze nie zawierające odpowiedzi.

```r
d2 <- d2[!is.na(value)]
d2
```

```
##            Id variable value
##       1:   45     P1_1     4
##       2:  135     P1_1     7
##       3:  141     P1_1     6
##       4:  172     P1_1     4
##       5:  196     P1_1     1
##      ---                    
## 1221451: 5804  P2000_5     1
## 1221452: 5957  P2000_5     1
## 1221453: 5958  P2000_5     3
## 1221454: 5959  P2000_5     1
## 1221455: 5960  P2000_5     1
```

```r
cells2 <- nrow(d2)
fen0(cells2)
```

```
## [1] "1,221,455"
```

W zbiorze pozostaje ``1,221,455`` wierszy zawierających odpowiedzi respondentów.

Dane stanowiły jedynie niecałe ``2%``
zbioru wyjściowego

## Identyfikatory słowa i emocji

Zbiór zawiera połączony identyfikator dla słowa i emocji. Dzielimy
identyfikator "słowo_emocja" na osobne zmienne "słowo" i "emocja".

```r
d2 <- d2[, c("word", "aff") := tstrsplit(variable, "_", fixed = TRUE)]
d2[, variable := NULL]
d2
```

```
##            Id value  word aff
##       1:   45     4    P1   1
##       2:  135     7    P1   1
##       3:  141     6    P1   1
##       4:  172     4    P1   1
##       5:  196     1    P1   1
##      ---                     
## 1221451: 5804     1 P2000   5
## 1221452: 5957     1 P2000   5
## 1221453: 5958     3 P2000   5
## 1221454: 5959     1 P2000   5
## 1221455: 5960     1 P2000   5
```

## Tworzymy docelowy zbiór

Docelowy zbiór w wierszach powinien odpowiadać poszczególnym słowom i
identyfikatorom respondentów. W kolumnach powinny znajdować się oceny kolejnych
emocji.

```r
d3 <- dcast(d2, word + Id ~ aff)
setnames(d3, tonames(names(d3)))
d3
```

Przy okazji wystandaryzowaliśmy nazwy zmiennych.

```
##         word   id x1 x2 x3 x4 x5
##      1:   P1   45  4  3  5  5  5
##      2:   P1  135  7  5  5  5  3
##      3:   P1  141  6  5  5  5  5
##      4:   P1  172  4  3  4  4  4
##      5:   P1  196  1  1  2  1  1
##     ---                         
## 244287: P999 5945  2  4  3  4  1
## 244288: P999 5960  1  2  1  2  2
## 244289: P999 6041  2  2  5  5  2
## 244290: P999 6110  1  1  1  4  1
## 244291: P999 6126  4  7  6  4  6
```

```r
summary(d3)
```

```
##      word                 id             x1              x2       
##  Length:244291      Min.   :   1   Min.   :1.000   Min.   :1.000  
##  Class :character   1st Qu.:1533   1st Qu.:1.000   1st Qu.:1.000  
##  Mode  :character   Median :3064   Median :3.000   Median :2.000  
##                     Mean   :3065   Mean   :3.351   Mean   :2.687  
##                     3rd Qu.:4597   3rd Qu.:5.000   3rd Qu.:4.000  
##                     Max.   :6127   Max.   :7.000   Max.   :7.000  
##        x3              x4              x5       
##  Min.   :1.000   Min.   :1.000   Min.   :1.000  
##  1st Qu.:1.000   1st Qu.:1.000   1st Qu.:1.000  
##  Median :2.000   Median :2.000   Median :2.000  
##  Mean   :2.809   Mean   :2.706   Mean   :2.553  
##  3rd Qu.:4.000   3rd Qu.:4.000   3rd Qu.:4.000  
##  Max.   :7.000   Max.   :7.000   Max.   :7.000
```

Wszystko się zgadza. Każda emocja ma oceny na skali 1-7.

## Podsumowanie

Mamy łącznie ``244,291`` ocen.

```r
t1 <- d3[, .N, word]
t1[, fr(ntil(N))]
```

```
## Liczba ocen  N      %
## [93,113]   403  20.15
## (113,119]  415  20.75
## (119,125]  445  22.25
## (125,131]  351  17.55
## (131,165]  386  19.30
## Sum       2000 100.00
## NA's         0   0.00
```

Średnio po ``122`` oceny na słowo.

## Opis zbioru

### Etykiety emocji

Sprawdzamy, czy identyfikatory zawsze oznaczają te same emocje.

Wczytujemy etykiety z arkusza dostarczonego przez *Ankieteo*.

```r
labs <- data.table(
  read_xlsx(
    "data/SWR_Crisis_etykiety_23.02.2021.xlsx",
    range = "a9:C10008", col_names = c("v1", "v2", "v3"))
)
```

Usuwamy niepotrzebną kolumnę.

```r
labs[, v2 := NULL]
```

Usuwamy z tekstu etykiety wszystko poza nazwą emocji.

```r
labs[, v2 := gsub("^.* - ", "", v3)]
labs
```

```
##             v1
##     1:    P1_1
##     2:    P1_2
##     3:    P1_3
##     4:    P1_4
##     5:    P1_5
##    ---        
##  9996: P2000_1
##  9997: P2000_2
##  9998: P2000_3
##  9999: P2000_4
## 10000: P2000_5
##                                                                                                                                                                   v3
##     1: Prosimy określić, w jakim stopniu fraza "POJAWIĆ SIĘ" wzbudza (wywołuje) w Tobie dany rodzaj emocji, wskazując na każdej ze skal ocenę od 1 do 7? - Szczęście
##     2:     Prosimy określić, w jakim stopniu fraza "POJAWIĆ SIĘ" wzbudza (wywołuje) w Tobie dany rodzaj emocji, wskazując na każdej ze skal ocenę od 1 do 7? - Gniew
##     3:    Prosimy określić, w jakim stopniu fraza "POJAWIĆ SIĘ" wzbudza (wywołuje) w Tobie dany rodzaj emocji, wskazując na każdej ze skal ocenę od 1 do 7? - Strach
##     4:    Prosimy określić, w jakim stopniu fraza "POJAWIĆ SIĘ" wzbudza (wywołuje) w Tobie dany rodzaj emocji, wskazując na każdej ze skal ocenę od 1 do 7? - Smutek
##     5:    Prosimy określić, w jakim stopniu fraza "POJAWIĆ SIĘ" wzbudza (wywołuje) w Tobie dany rodzaj emocji, wskazując na każdej ze skal ocenę od 1 do 7? - Wstręt
##    ---                                                                                                                                                              
##  9996:      Prosimy określić, w jakim stopniu słowo "POTĘGA" wzbudza (wywołuje) w Tobie dany rodzaj emocji, wskazując na każdej ze skal ocenę od 1 do 7? - Szczęście
##  9997:          Prosimy określić, w jakim stopniu słowo "POTĘGA" wzbudza (wywołuje) w Tobie dany rodzaj emocji, wskazując na każdej ze skal ocenę od 1 do 7? - Gniew
##  9998:         Prosimy określić, w jakim stopniu słowo "POTĘGA" wzbudza (wywołuje) w Tobie dany rodzaj emocji, wskazując na każdej ze skal ocenę od 1 do 7? - Strach
##  9999:         Prosimy określić, w jakim stopniu słowo "POTĘGA" wzbudza (wywołuje) w Tobie dany rodzaj emocji, wskazując na każdej ze skal ocenę od 1 do 7? - Smutek
## 10000:         Prosimy określić, w jakim stopniu słowo "POTĘGA" wzbudza (wywołuje) w Tobie dany rodzaj emocji, wskazując na każdej ze skal ocenę od 1 do 7? - Wstręt
##               v2
##     1: Szczęście
##     2:     Gniew
##     3:    Strach
##     4:    Smutek
##     5:    Wstręt
##    ---          
##  9996: Szczęście
##  9997:     Gniew
##  9998:    Strach
##  9999:    Smutek
## 10000:    Wstręt
```

Rozdzielamy identyfikator słowa od identyfikatora emocji.

```r
labs <- labs[, c("word", "aff") := tstrsplit(v1, "_", fixed = TRUE)]
labs[, v1 := NULL]
```

Zmieniamy typ identyfikatora emocji na liczbę całkowitą i liczymy jego średnią w
grupach wydzielonych przez nazwy emocji.

```r
labs[, aff := as.integer(aff)]
labs[, mean(aff), v2]
```

```
##           v2 V1
## 1: Szczęście  1
## 2:     Gniew  2
## 3:    Strach  3
## 4:    Smutek  4
## 5:    Wstręt  5
```

Wszystkie średnie pozostały liczbami całkowitymi, więc każdy identyfikator
jednoznacznie określa jedną emocję.

Nadajemy odpowiednie nazwy zmiennym.

```r
setnames(d3, c("word", "id", "h", "a", "f", "s", "d"))
d3
```

```
##         word   id h a f s d
##      1:   P1   45 4 3 5 5 5
##      2:   P1  135 7 5 5 5 3
##      3:   P1  141 6 5 5 5 5
##      4:   P1  172 4 3 4 4 4
##      5:   P1  196 1 1 2 1 1
##     ---                    
## 244287: P999 5945 2 4 3 4 1
## 244288: P999 5960 1 2 1 2 2
## 244289: P999 6041 2 2 5 5 2
## 244290: P999 6110 1 1 1 4 1
## 244291: P999 6126 4 7 6 4 6
```

### Etykiety słów

Oddzielamy token z treści etykiety. Zmieniamy litery na małe.

```r
labs[, v4 := str_extract(v3, "\".*\"")]
labs[, v4 := tolower(gsub("\"", "", v4))]
labs
```

```
##                                                                                                                                                                   v3
##     1: Prosimy określić, w jakim stopniu fraza "POJAWIĆ SIĘ" wzbudza (wywołuje) w Tobie dany rodzaj emocji, wskazując na każdej ze skal ocenę od 1 do 7? - Szczęście
##     2:     Prosimy określić, w jakim stopniu fraza "POJAWIĆ SIĘ" wzbudza (wywołuje) w Tobie dany rodzaj emocji, wskazując na każdej ze skal ocenę od 1 do 7? - Gniew
##     3:    Prosimy określić, w jakim stopniu fraza "POJAWIĆ SIĘ" wzbudza (wywołuje) w Tobie dany rodzaj emocji, wskazując na każdej ze skal ocenę od 1 do 7? - Strach
##     4:    Prosimy określić, w jakim stopniu fraza "POJAWIĆ SIĘ" wzbudza (wywołuje) w Tobie dany rodzaj emocji, wskazując na każdej ze skal ocenę od 1 do 7? - Smutek
##     5:    Prosimy określić, w jakim stopniu fraza "POJAWIĆ SIĘ" wzbudza (wywołuje) w Tobie dany rodzaj emocji, wskazując na każdej ze skal ocenę od 1 do 7? - Wstręt
##    ---                                                                                                                                                              
##  9996:      Prosimy określić, w jakim stopniu słowo "POTĘGA" wzbudza (wywołuje) w Tobie dany rodzaj emocji, wskazując na każdej ze skal ocenę od 1 do 7? - Szczęście
##  9997:          Prosimy określić, w jakim stopniu słowo "POTĘGA" wzbudza (wywołuje) w Tobie dany rodzaj emocji, wskazując na każdej ze skal ocenę od 1 do 7? - Gniew
##  9998:         Prosimy określić, w jakim stopniu słowo "POTĘGA" wzbudza (wywołuje) w Tobie dany rodzaj emocji, wskazując na każdej ze skal ocenę od 1 do 7? - Strach
##  9999:         Prosimy określić, w jakim stopniu słowo "POTĘGA" wzbudza (wywołuje) w Tobie dany rodzaj emocji, wskazując na każdej ze skal ocenę od 1 do 7? - Smutek
## 10000:         Prosimy określić, w jakim stopniu słowo "POTĘGA" wzbudza (wywołuje) w Tobie dany rodzaj emocji, wskazując na każdej ze skal ocenę od 1 do 7? - Wstręt
##               v2  word aff          v4
##     1: Szczęście    P1   1 pojawić się
##     2:     Gniew    P1   2 pojawić się
##     3:    Strach    P1   3 pojawić się
##     4:    Smutek    P1   4 pojawić się
##     5:    Wstręt    P1   5 pojawić się
##    ---                                
##  9996: Szczęście P2000   1      potęga
##  9997:     Gniew P2000   2      potęga
##  9998:    Strach P2000   3      potęga
##  9999:    Smutek P2000   4      potęga
## 10000:    Wstręt P2000   5      potęga
```

Usuwamy ze zbioru zreplikowane wiersze i zostawiamy tylko potrzebne kolumny.

```r
lword <- unique(labs[, .(word, v4)])
lword
```

```
##        word           v4
##    1:    P1  pojawić się
##    2:    P2   cały świat
##    3:    P3    dostęp do
##    4:    P4  cieszyć się
##    5:    P5     na ulicy
##   ---                   
## 1996: P1996       litera
## 1997: P1997         nota
## 1998: P1998 obowiązujący
## 1999: P1999        panel
## 2000: P2000       potęga
```

Mamy 2k tokenów.

Dodajemy brzmienia tokenów.

```r
setkey(lword, "word")
cp_affective <- merge(d3, lword)
setnames(cp_affective,
         c("token_id", "respondent_id",
           "h", "a", "f", "s", "d", "token")
)
cp_affective
```

```
##         token_id respondent_id h a f s d       token
##      1:       P1            45 4 3 5 5 5 pojawić się
##      2:       P1           135 7 5 5 5 3 pojawić się
##      3:       P1           141 6 5 5 5 5 pojawić się
##      4:       P1           172 4 3 4 4 4 pojawić się
##      5:       P1           196 1 1 2 1 1 pojawić się
##     ---                                             
## 244287:     P999          5945 2 4 3 4 1 konieczność
## 244288:     P999          5960 1 2 1 2 2 konieczność
## 244289:     P999          6041 2 2 5 5 2 konieczność
## 244290:     P999          6110 1 1 1 4 1 konieczność
## 244291:     P999          6126 4 7 6 4 6 konieczność
```

## Zapisujemy zbiór

```r
save(cp_affective, file = "cp_affective.RData")
```

Zbiór wynikowy w natywnym formacie binarnym ```R``` ma wielkość 1.1Mb.


