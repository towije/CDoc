---
title: Oszacowanie trafności oryginalnego i oczyszczonego zbioru danych
author: Tomasz Jerzyński
date: 2021-04-14
output: html_document
toc: true
categories: [Wejście]
tags: [analiza emocjonalnych, preprocessing, R]
---

## Wartości odchyleń standardowych w zbiorze oryginalnym i oczyszczonym

Najprostszym sposobem oszacowania trafności analiz prowadzonych na aryginalnym i oczyszczonym zbiorze danych jest porównanie odchyleń standardowych dystansów do danych emocji. Możemy przyjąć, że mniejsze odchylenie standardowe świadczy od bardziej spójnym kategoryzowaniu wyników, a to z kolei wskazuje, że wstępne oczyszczenie zbioru z nazw własnych (imiona, nazwiska, miejscowości itp.) redukuje w analizach przypadkową wariancję.

Należy pamiętać, że porównanie zostało przeprowadzone na różnych zbiorach danych. Nie został więc utrzymany kanon jedynej różnicy. Poniższa ocena ma jedynie charakter szacunkowy.


## Różnice pomiędzy odchyleniami standardowym w zbiorze oryginalnym i oczyszczonym

W oczyszczonym zbiorze danych odchylenia standardowe są mniejsze niż w zbiorze oryginalnym dla wszystkich poza szczęściem emocji.

![f1](/img/320-comparison-with-clean-dataset/fig1-1.png)<!-- -->

## Wartości odchyleń standardowych w zbiorze oryginalnym i oczyszczonym

Warto zauważyć, że różnice pomiędzy odchyleniami standardowymi dla poszczególnych emocji dla zbiorów oryginalnego i oczyszczonego są minimalne względem ich absolutnych wartości.

![f2](/img/320-comparison-with-clean-dataset/fig2-1.png)<!-- -->

