---
title: "Wstępne przetwarzanie dla jednostki tekstu"
subtitle: "Funkcja"
author: Tomasz Jerzyński
date: 2020-11-17
output:
  html_document:
    highlight: "zenburn"
    css: style.css
    keep_md: true
    toc: true
    toc_float: true
slug: [preprocessing-single]
categories: [Wejście]
tags: [preprocessing, R]
---

Na początku ładujemy ```locale``` i ```stopwords_pl```.
Raz przy starcie sesji.

{{< code language="r" title="Nastawy środowiska" >}}
Sys.setlocale("LC_ALL", "English")
stopwords_pl <- readLines("stopwords_pl.txt")
{{< /code >}}

Także przy starcie sesji ładujemy definicję funkcji.


{{< code language="r" title="Funkcja preprocesingu dla pojedynczego tekstu" >}}
make_corpus <- function(txt) {
    txt <- stringi::stri_trans_general(tolower(txt), "latin-ascii")
    txt <- gsub("[^[:alpha:]]", " ", txt)
    txt <- strsplit(txt, " ", fixed = T)[[1]]
    txt <- txt[nchar(txt) > 2]
    txt <- txt[!is.element(txt, stopwords_pl)]
    txt <- paste(txt, collapse = " ")
    return(txt)
}
{{< /code >}}

Na wejściu podajemy, a na wyjściu dostajemy jeden łańcuch tekstowy. Nie wektor. Wektor nie wywali funkcji ale na wyjściu dostaniemy jeden łańcuch - czyli poszczególne teksty połączą się.

{{< code language="r" title="Przykład" >}}
make_corpus("Nie jestes zadnym doktorem, ale draniem i glupcem. Dobrze wiesz ilu twoich kolegów i kolezanek po fachu jest banowanych i szykanowanych za demaskowanie klamstw dotyczacych tej fake-epidemii a ty szmato bierzesz udzial w kreowaniu tej psychozy.\nTfu!!!")
{{< /code >}}

{{< code language="r" title="Wynik">}}
## [1] "jestes zadnym doktorem draniem glupcem wiesz ilu twoich kolegow kolezanek fachu banowanych szykanowanych demaskowanie klamstw dotyczacych fake epidemii szmato bierzesz udzial kreowaniu psychozy tfu"
{{< /code >}}

Do rozważenia pozostaje sprawa jak pluć jednostkami tekstu w sesję R i jak je na bieżąco odbierać.
