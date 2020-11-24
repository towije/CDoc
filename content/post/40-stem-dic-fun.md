---
title: Stemming w ```R```
author: Tomasz Jerzyński
date: '2020-11-24'
output:
    html_document:
        keep_md: true
        toc: true
        toc_float: true
editor_options: 
chunk_output_type: inline
slug: [stemming-R]
categories: [Badanie]
tags: [analiza emocjonalnych, analiza n-gramów, stemming, morfologik, R, benchmark]
---

```Morfologik``` jest chyba jedynym i przede wszystkim bardzo udanym słownikiem stemmingowym dla języka polskiego.
Jednak jego użycie w zastosowaniach masowych czy strumieniowych wymaga dużo większej sprawności, niż może zaoferować interpreter ```R```.

## Struktura słownika

Adaptacja słownika ```morfologik```.

{{< code language="r" title="Przykładowe rekordy ze słownika ```morfologik```" >}}
sdp[sample(1:nrow(sdp),20)]
           core             flex                           typ1        typ2
 1:  przemarzły     przemarzłymi                    przymiotnik przymiotnik
 2:   pustawość     pustawościom                     rzeczownik  rzeczownik
 3:   odtańczyć     odtańczonymi imieslów przymiotnikowy bierny    imieslów
 4:  powykradać  powykradaliście                      czasownik   czasownik
 5: tamaryndowy    tamaryndowego                    przymiotnik przymiotnik
 6:    zaświnić  niezaświnionych imieslów przymiotnikowy bierny    imieslów
 7:   zaciemnić      zaciemniłeś                      czasownik   czasownik
 8:   skowyczeć         skowyczę                      czasownik   czasownik
 9:   obmotywać    obmotywałobym                      czasownik   czasownik
10:      magnon         magnonów                     rzeczownik  rzeczownik
11:   odpytywać      odpytywanie      rzeczownik odczasownikowy  rzeczownik
12:  unifikować    unifikowanych imieslów przymiotnikowy bierny    imieslów
13: zaszczepiać   zaszczepiającą imieslów przymiotnikowy czynny    imieslów
14:     osprzęt       osprzętowi                     rzeczownik  rzeczownik
15:   furgonowy       furgonowej                    przymiotnik przymiotnik
16:      sacrum           sacrów                     rzeczownik  rzeczownik
17:   zmianować    niezmianowaną imieslów przymiotnikowy bierny    imieslów
18:   niematowy        niematowe                    przymiotnik przymiotnik
19:   nachrapać nachrapalibyście                      czasownik   czasownik
20:      ukajać           ukajam                      czasownik   czasownik
{{< /code >}}

Zmienne:
 
- ```core``` to forma **podstawowa** słowa;
- ```flex``` to forma **odmieniona**;
- ```typ1``` szczegółowa nazwa części mowy;
- ```typ2``` ogólna nazwa części mowy. 

## Funkcja

Funkcja zwracająca formę podstawową słowa.
Wyszukiwanie słowa w słowniku ```morfologik```.

Znajdź ```słowo```
- Przeszukaj ```core```,
    - jeśli znaleziono to zwróć ```słowo```,
    - jeśli nie znaleziono to przeszukaj ```flex```,
        - jeśli znaleziono to zwróć ```core```,
        - jeśli nie znaleziono to zwróć ```słowo```.

{{< code language="r" title="Funkcja semmująca" >}}
stem<-function(slo) {
  if (nrow(sdp[core == slo])) {
    return(slo)
  } else if (nrow(sdp[flex == slo])) {
    return(sdp[flex == slo][1, core])
  } else {
    return(slo)
  }
}
{{< /code >}}

{{< code language="r" title="Przykład działania ```stemmera```" >}}
stem("domięśniowych")
[1] "domięśniowy"
{{< /code >}}

## Czas na jedno słowo

{{< code language="r" title="Czas stemmingu jednego słowa" >}}
microbenchmark(stem("oliwilaby"))
Unit: milliseconds
              expr     min       lq     mean   median       uq      max neval
 stem("oliwilaby") 31.1895 35.21415 269.1546 51.43565 54.48905 1274.868   100
{{< /code >}}

## Czas stemmingu dla 30 słów

W wypowiedzi jest średnio 30 słów.

{{< code language="r" title="Czas stemmingu dla 30 słów" >}}
microbenchmark(
  tag1s <- sapply(tag1[s30], stem)
)
Unit: seconds
                            expr      min       lq     mean   median      uq     max neval
tag1s <- sapply(tag1[s30], stem) 3.225535 3.416776 4.449229 4.431668 4.95623 9.94925   100
{{< /code >}}


Średnio 5 sekund na jedną wypowiedź. Za dużo na robienie tego w ```R```.

## Masa

Dla naszego badania chcemy przeanalizować dużą próbę słów - prawie 12 milionów (11,895,001).
Stemming tego zbioru trwałby ponad 20 dni. (Jeden rdzeń, procesor Intel(R) Core(TM) i7-4720HQ CPU @ 2.60GHz, 2594 MHz).
Na Ryzenie 7 pewnie dałoby radę zejść do 15 dni.
to działanie jednorazowe tylko na potrzeby naszego badania.
Tak czy inaczej, jeśli będziemy tego potrzebować trzeba będzie zrobić to poza ```R```.

{{< code language="r" title="Ile dni?" >}}
((11895001 / 30) * 4.449229) / 60 / 60 / 24
[1] 20.41805
{{< /code >}}
