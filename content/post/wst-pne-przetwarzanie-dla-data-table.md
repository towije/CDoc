---
title: Wstępne przetwarzanie dla data.table
author: Tomasz Jerzyński
date: '2020-11-13'
output:
    html_document:
        keep_md: true
        toc: true
        toc_float: true
editor_options: 
chunk_output_type: inline
slug: [preprocessing-data-table]
categories: [Wejście]
tags: [analiza emocjonalnych, preprocessing, R, data.table]
---

Wstępne przetwarzanie ma na celu unifikację oraz kompresję tekstu.
Wykonujemy kilka standardowych przekształceń. W zależności od potrzeb możemy dodać inne lub usunąć któreś z istniejących. Większość z nich polega na zastąpieniu fragmentu tekstu znakiem odstępu.

## Przekształcenia

Tutaj wczytamy cały zbiór i wykonamy przekształcenia dla całości. W przypadku przetwarzania strumieniowego będziemy to robili dla pojedynczego wpisu, ale o tym dalej.

Najpierw ustawiamy ```locale``` na angielskie. Funkcje tekstowe często oparte sa na nastawach narodowych. Następnie ładujemy bibliotekę do szybkich operacji na zbiorach danych.


{{< code language="r" title="Środowisko i zmienne" >}}
Sys.setlocale("LC_ALL","English")
library(data.table)
file_name<-"lowicz_from201908250000_to202008242359_exportedAt202008241255.csv"
data_set_name<-"lowicz"
dir_name<-"data/DUMP SO/"
encoding<-"UTF-8"
{{< /code >}}

### Nazwy zmiennych

Wczytujemy dane do obiektu. Następnie ustawiamy nazwy. W nazwach mogą pojawiać się narodowe znaki diakrytyczne oraz całe spektrum przeróżnego śmiecia włącznie z kodami sterującymi. Teoretycznie może to działać, ale praktyka mówi, że pozostawienie tego to kłopoty.
```make.names``` doprowadzi nazwy do jedynie słusznej formy, usuwają wszelkie herezje. Stąd - między innymi - potrzebne były angielskie ```locale```, gdyby pozostały polskie, pozostały by także polskie znaki.

{{< code language="r" title="Dane i nazwy" >}}
data_set<-fread(paste0(dir_name,file_name),encoding = encoding)
setnames(data_set,make.names(names(data_set)))
{{< /code >}}

### Data.table

Muszę skrótowo opisać ```data.table```. Obiekt tego typu ma 3 parametry.

1. zakres wierszy,
2. zakres kolumn,
3. zmienna grupująca (dla nas, na razie, tu nie przydatna).

To bardzo *brudny* opis. Nie pokazujcie tego nikomu bo to wstyd. Przykłady:

{{< code language="r" title="Indeksy w ```data.table```" >}}
DT[1,2] # komórka z pierwszego wiersza i drugiej kolumny tabeli
DT[ ,2] # wszystkie wiersze z drugiej kolumny - czyli cała druga kolumna
DT[1, ] # wszystkie kolumny z pierwszego wiersza - czyli cały pierwszy wiersz
DT[1:10] # wiersze od 1 do 10 w całości
DT[1:10, ] # j.w.
DT[ , 1:10] # kolumny od 1 do 10 w całości
DT[ , tagi] # cała kolumna o nazwie tagi
{{< /code >}}

Ten znak: ```:=``` to przypisanie.


{{< code language="r" title="Przykład" >}}
DT[c(12,23,666) , tagi:=0]
{{< /code >}}

W wierszach 12, 33 i 666 wartości kolumny tagi zostały zamienione na 0.

I dużo dużo więcej, ale tyle nam wystarczy.

### Zmiana wielkości liter na małe

Ujednolicenie wielkości liter na samym początku analizy treści jest dobrym zwyczajem ułatwiającym późniejsze porównania bez potrzeby dbania o to, czy wyrazy były pisane z małej, czy dużej litery, kapitalikami, czy też zupełnie przypadkowo. Pomijanie rozróżniania wielkości liter już w trakcie wyszukiwania wielokrotnie wydłuża czas operacji.
Przy okazji kopiujemy zmienną zawierającą wypowiedzi.


{{< code language="r" title="" >}}
data_set[,tagi:=tolower(`Tresc.wypowiedzi`)]
data_set[2,tagi] # przykładowy tekst
{{< /code >}}

```
## [1] "u niej w domu, w kuchni, w szafce @martika nie wpisalas gdzie te pomidory znalazlas edit - ok, widze, ze juz dodane, biedra... ok, jezeli jutro bede w biedrze, zerkne przy okazji czy i u mnie nie ma. \"\"zimnyzuber\"\" 15 g, 2 min\nedytowane przez: hmmm a gdzie? art_sheffield 28/11/2019 20:08"
```

### Polskie znaki

Do przemyślenia. Można pozbyć się polskich znaków. Pozwoli to zunifikować piszących z ich użyciem i bez. Kiedyś wiele osób pisałem bez polskich znaków, teraz nie wiem, ale można przypuszczać, że jakaś część robi tak w dalszym ciągu.


{{< code language="r" title="" >}}
data_set[,tagi:=stringi::stri_trans_general(tagi, "latin-ascii")]
data_set[2,tagi] # przykładowy tekst
{{< /code >}}

```
## [1] "u niej w domu, w kuchni, w szafce @martika nie wpisalas gdzie te pomidory znalazlas edit - ok, widze, ze juz dodane, biedra... ok, jezeli jutro bede w biedrze, zerkne przy okazji czy i u mnie nie ma. \"\"zimnyzuber\"\" 15 g, 2 min\nedytowane przez: hmmm a gdzie? art_sheffield 28/11/2019 20:08"
```

### Tylko litery

Usunięcie wszystkich znaków za wyjątkiem liter. Cyfry, znaki przestankowe nie niosą znaczenia zajmując miejsce.
Inaczej będzie podczas analizy fraz dwu i tzy wyrazowych. Wtedy znaki przestankowe będą pierwszym poziomem podziału.


{{< code language="r" title="" >}}
data_set[,tagi:=gsub("[^abcdefghijklmnopqrstuvwxyzęćółąćśżźń]", " ",tagi)]
{{< /code >}}

Nie używamy makra ```[^[:alpha:]]`` bo zderzymy się z polskimi znakami, które nie są uznawana ze litery. Jeśli usunęliśmy polskie znaki to wystarczy makro.


{{< code language="r" title="" >}}
data_set[,tagi:=gsub("[^[:alpha:]]", " ",tagi)]
data_set[2,tagi] # przykładowy tekst
{{< /code >}}

```
## [1] "u niej w domu  w kuchni  w szafce  martika nie wpisalas gdzie te pomidory znalazlas edit   ok  widze  ze juz dodane  biedra    ok  jezeli jutro bede w biedrze  zerkne przy okazji czy i u mnie nie ma    zimnyzuber      g    min edytowane przez  hmmm a gdzie  art sheffield                 "
```

### Jedno- i dwu-literowce

Usunięcie słów krótszych niż trzy znaki. I znowu, w przypadku analizy fraz należy to przemyśleć.

{{< code language="r" title="" >}}
data_set[,tagi:=gsub("\\b\\w{1,2}\\b", " ",tagi)]
data_set[2,tagi] # przykładowy tekst
{{< /code >}}

```
## [1] "  niej   domu    kuchni    szafce  martika nie wpisalas gdzie   pomidory znalazlas edit      widze    juz dodane  biedra       jezeli jutro bede   biedrze  zerkne przy okazji czy     mnie nie      zimnyzuber           min edytowane przez  hmmm   gdzie  art sheffield                 "
```

### Stopwords

Usunięcie słów nieznaczących. Lista tych słów powinna być modyfikowana w oparciu o doświadczenie.
W każdym języku występują słowa mające niewielkie znaczenie dla treści, pełniące funkcje służące składni. Pozbycie się ich jest znacząco skraca czas przyszłych analiz.
Listy słów *niepotrzebnych* mogą różnić się w zależności od zastosowania, twórcy lub właściciela. Własną — dostosowaną ściśle do potrzeb danego zadania można utworzyć wychodząc od jednego z gotowych zbiorów. Tu używamy listy polskich stopwords udostępnionej przez Wikipedię.

W celu usunięcia stopwords wykonano tyle wyszukiwań, ile słów znalazło się na liście.

{{< code language="r" title="" >}}
load("stopwords_pl.RData") # bez 1 i 2 literowych
# wyszukujemy całe wyrazy
stopwords_pl <- paste0(" ", stopwords_pl, " ")
# jeżeli wcześniej usunęliśmy polskie diakrytyki to ze stopwords też musimy
stopwords_pl<-stringi::stri_trans_general(stopwords_pl, "latin-ascii")
# otaczamy spacjami każdą wypowiedź
data_set[, tagi := paste0(" ", tagi, " ")]
# iterujemy wzdłuż całej listy
for (idx in stopwords_pl) {
  data_set[, tagi := gsub(idx, " ", tagi, fixed = T)]
}
data_set[2,tagi] # przykładowy tekst
{{< /code >}}

```
## [1] "     domu    kuchni    szafce  martika wpisalas   pomidory znalazlas edit      widze    dodane  biedra       jutro bede   biedrze  zerkne okazji          zimnyzuber           min edytowane  hmmm    art sheffield                  "
```

Użyta tu pętla ```for``` nie jest optymalna. Są szybsze sposoby, ale użyłem jej tu bo w każdym języku wygląd mniej więcej tak samo.

### Sprzątamy

Wszystkie transformacje spowodowały namnożenie się w tekście wielokrotnie po sobie następujących znaków odstępu, na które zamieniane były elementy usunięte, lub które służyły jako separatory w innych celach. Pozbywamy się ich nadmiaru.


{{< code language="r" title="" >}}
data_set[,tagi:=gsub("\\s+", " ",tagi)] # dup spcs
data_set[,tagi:=gsub("^\\s+|\\s+$", "", tagi)] # trim
data_set[2,tagi] # przykładowy tekst
{{< /code >}}

```
## [1] "domu kuchni szafce martika wpisalas pomidory znalazlas edit widze dodane biedra jutro bede biedrze zerkne okazji zimnyzuber min edytowane hmmm art sheffield"
```

Oczyszczony text zapisujemy wraz z całym zbiorem.


{{< code language="r" title="" >}}
# nadajemy nazwę
assign(data_set_name,data_set, envir = .GlobalEnv)
# i zapisujemy
save(list=data_set_name,
       file = paste0(dir_name,data_set_name,".RData"))
{{< /code >}}

