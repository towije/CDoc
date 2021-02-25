---
title: "Wynikowy zbiór danych"
subtitle: "Podsumowanie przetwarzania"
author: Tomasz Jerzyński
date: 2021-02-21
output:
    html_document:
        keep_md: true
        toc: true
        toc_float: false
---

## Uwagi, ostrzeżenia, pułapki



**Dotyczy zbioru:** ```analyzed_events_sentiment_data_notext.csv```

W kilku komórkach pojawił się tekst. Na końcu zbioru pojawiły się 3
puste zmienne. Może to oznaczać, że przy eksporcie ```csv``` kolumny się
"rozjechały" --- wygląda t tak, jakby pojawiły się znaki sterujące.
Dotyczyło to tylko 96 przypadków z 36K więc mogły to byc na
przykład jakieś zaszłości z wcześniejszych faz przetwarzania.

Niepokojące jest także, że suma wszystkich tokenów nie jest liczbą całkowitą.

## Symbole

### Liczebności

W zbiorze danych było ponad 16k symboli.
Wśród wszystkich symboli mamy po 71% unikalnych i sklasyfikowanych.

{{< rawhtml >}}
<table class="table table-striped table-hover table-condensed" style="margin-left: auto; margin-right: auto;">
 <thead>
  <tr>
   <th style="text-align:left;"> .id </th>
   <th style="text-align:right;"> Wszystkich </th>
   <th style="text-align:right;"> Unikalnych </th>
   <th style="text-align:right;"> Sklasyfikowanych </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> Liczba symboli </td>
   <td style="text-align:right;"> 16800 </td>
   <td style="text-align:right;"> 11969.00 </td>
   <td style="text-align:right;"> 11903.00 </td>
  </tr>
  <tr>
   <td style="text-align:left;font-weight: bold;"> Procent symboli % </td>
   <td style="text-align:right;font-weight: bold;"> 100 </td>
   <td style="text-align:right;font-weight: bold;"> 71.24 </td>
   <td style="text-align:right;font-weight: bold;"> 70.85 </td>
  </tr>
</tbody>
</table>
{{< /rawhtml >}}

### Klasyfikacja

Wyniki pochodzące z klasyfikacji nie są zbieżne z wynikami pochodzącymi z
liczby punktów jakie dany symbol otrzymał w każdej kategorii sentymentu.
Zarówno rozkład sumy punktów jaki i rozkład średniej i średniej ważonej są
zbliżone do siebie. Mamy tu spore możliwości konstrukcji wskaźnika.

{{< rawhtml >}}
<table class="table table-striped table-hover table-condensed" style="margin-left: auto; margin-right: auto;">
 <thead>
  <tr>
   <th style="text-align:left;"> .id </th>
   <th style="text-align:right;"> negatywne </th>
   <th style="text-align:right;"> neutralne </th>
   <th style="text-align:right;"> pozytywne </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> Symbole zaklasyfikowane jako (%) </td>
   <td style="text-align:right;"> 19.82 </td>
   <td style="text-align:right;"> 44.88 </td>
   <td style="text-align:right;"> 35.29 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Suma punktów dla symbolu (%) </td>
   <td style="text-align:right;"> 18.27 </td>
   <td style="text-align:right;"> 24.74 </td>
   <td style="text-align:right;"> 56.99 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Średnia punktacja dla symbolu (%) </td>
   <td style="text-align:right;"> 17.04 </td>
   <td style="text-align:right;"> 25.02 </td>
   <td style="text-align:right;"> 57.94 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Średnia ważona punktacja dla symbolu (%) </td>
   <td style="text-align:right;"> 16.82 </td>
   <td style="text-align:right;"> 25.02 </td>
   <td style="text-align:right;"> 58.16 </td>
  </tr>
</tbody>
</table>
{{< /rawhtml >}}

### Wskaźnik zagrożenia

Najprostszym i najbardziej efektywnym jest średnia z punktacji, której
maksimum da nam wynik: pozytywne -- neutralne -- negatywne.


```r
symbol_winner <- symbol_summary[2:4, 2:4][, sapply(.SD, mean)]
symbol_winner <- names(symbol_winner[symbol_winner == max(symbol_winner)])
symbol_winner
```

```
## [1] "pozytywne"
```

Z kolei odchylenie standardowe ze wszystkich parametrów --- klasyfikacji i
punktacji --- dla wygranego sentymentu posłuży do konstrukcji miary dobroci
uzyskanego wyniku. Miara dobroci będzie przyjmowała wartości od 0 do 100.


```r
symbol_goodness <- symbol_summary[, sd(get(symbol_winner))]
max_sd <- sd(c(0, 0, 100, 100))
symbol_goodness <- rescale(
  symbol_goodness,
  from = c(0, max_sd),
  to = c(100, 0))
symbol_goodness
```

```
## [1] 80.57534
```

## Tokeny

### Liczebności

```94%``` tokenów pojawia się w poszczególnych wypowiedziach tylko raz.
Jedynie ```3.4%``` tokenów znajduje się w klasyfikacji. Wulgaryzmy stanowią
pomijalny margines.

{{< rawhtml >}}
<table class="table table-striped table-hover table-condensed" style="margin-left: auto; margin-right: auto;">
 <thead>
  <tr>
   <th style="text-align:left;"> .id </th>
   <th style="text-align:right;"> Wszystkich </th>
   <th style="text-align:right;"> Unikalnych </th>
   <th style="text-align:right;"> Sklasyfikowanych </th>
   <th style="text-align:right;"> profan </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> Wszystkicj tokenów </td>
   <td style="text-align:right;"> 516013.4 </td>
   <td style="text-align:right;"> 483630.00 </td>
   <td style="text-align:right;"> 17341.00 </td>
   <td style="text-align:right;"> 15 </td>
  </tr>
  <tr>
   <td style="text-align:left;font-weight: bold;"> Rozkład tokenów (%) </td>
   <td style="text-align:right;font-weight: bold;"> 100.0 </td>
   <td style="text-align:right;font-weight: bold;"> 93.72 </td>
   <td style="text-align:right;font-weight: bold;"> 3.36 </td>
   <td style="text-align:right;font-weight: bold;"> 0 </td>
  </tr>
</tbody>
</table>
{{< /rawhtml >}}

### Klasyfikacja

81% głosów zebranych w ocenach dla wszystkich tokenów przypadło kategorii U
(niezdefiniowany). Pominąwszy tę kategorię uzyskujemy rozkład w którym na
prowadzenie wysuwa się kategoria H (szczęście) a potem N (neutralność).
Pominąwszy także tę kategorię otrzymujemy aż 50% oddanych na H i o połowę
mniej na F (strach).

Rozkład zsumowanych dystansów pokazuje, że tagi w analizowanej grupie
najbliższe są N. Nie bierzemy pod uwagę U. Pominąwszy N najmniejszy dystans
prowadzi do H. Podobnie dla średniego i ważonego średniego dystansu, który po
pominięciu N  najbliższy jest H.

{{< rawhtml >}}
<table class="table table-striped table-hover table-condensed" style="margin-left: auto; margin-right: auto;">
 <thead>
  <tr>
   <th style="text-align:left;"> .id </th>
   <th style="text-align:right;"> H </th>
   <th style="text-align:right;"> A </th>
   <th style="text-align:right;"> F </th>
   <th style="text-align:right;"> S </th>
   <th style="text-align:right;"> D </th>
   <th style="text-align:right;"> N </th>
   <th style="text-align:right;"> U </th>
  </tr>
 </thead>
<tbody>
  <tr grouplength="4"><td colspan="8" style="border-bottom: 1px solid;"><strong>Klasyfikacja</strong></td></tr>
<tr>
   <td style="text-align:left; padding-left:  2em;" indentlevel="1"> Suma głosów dla tokenów </td>
   <td style="text-align:right;"> 1255.00 </td>
   <td style="text-align:right;"> 172.00 </td>
   <td style="text-align:right;"> 644.00 </td>
   <td style="text-align:right;"> 427.00 </td>
   <td style="text-align:right;"> 12.00 </td>
   <td style="text-align:right;"> 700.00 </td>
   <td style="text-align:right;"> 13497.00 </td>
  </tr>
  <tr>
   <td style="text-align:left; padding-left:  2em;" indentlevel="1"> Rozkład głosów dla tokenów (%) </td>
   <td style="text-align:right;"> 7.51 </td>
   <td style="text-align:right;"> 1.03 </td>
   <td style="text-align:right;"> 3.85 </td>
   <td style="text-align:right;"> 2.56 </td>
   <td style="text-align:right;"> 0.07 </td>
   <td style="text-align:right;"> 4.19 </td>
   <td style="text-align:right;"> 80.79 </td>
  </tr>
  <tr>
   <td style="text-align:left; padding-left:  2em;" indentlevel="1"> Rozkład głosów dla tokenów(-u) (%) </td>
   <td style="text-align:right;"> 39.10 </td>
   <td style="text-align:right;"> 5.36 </td>
   <td style="text-align:right;"> 20.06 </td>
   <td style="text-align:right;"> 13.30 </td>
   <td style="text-align:right;"> 0.37 </td>
   <td style="text-align:right;"> 21.81 </td>
   <td style="text-align:right;"> NA </td>
  </tr>
  <tr>
   <td style="text-align:left; padding-left:  2em;" indentlevel="1"> Rozkład głosów dla tokenów(-nu) (%) </td>
   <td style="text-align:right;"> 50.00 </td>
   <td style="text-align:right;"> 6.85 </td>
   <td style="text-align:right;"> 25.66 </td>
   <td style="text-align:right;"> 17.01 </td>
   <td style="text-align:right;"> 0.48 </td>
   <td style="text-align:right;"> NA </td>
   <td style="text-align:right;"> NA </td>
  </tr>
  <tr grouplength="2"><td colspan="8" style="border-bottom: 1px solid;"><strong>Suma dystansów</strong></td></tr>
<tr>
   <td style="text-align:left; padding-left:  2em;" indentlevel="1"> Sumy dystansów (%) </td>
   <td style="text-align:right;"> 13.96 </td>
   <td style="text-align:right;"> 18.44 </td>
   <td style="text-align:right;"> 17.78 </td>
   <td style="text-align:right;"> 18.64 </td>
   <td style="text-align:right;"> 19.39 </td>
   <td style="text-align:right;"> 11.79 </td>
   <td style="text-align:right;"> NA </td>
  </tr>
  <tr>
   <td style="text-align:left; padding-left:  2em;" indentlevel="1"> Sumy dystansów(-n) (%) </td>
   <td style="text-align:right;"> 15.82 </td>
   <td style="text-align:right;"> 20.91 </td>
   <td style="text-align:right;"> 20.16 </td>
   <td style="text-align:right;"> 21.14 </td>
   <td style="text-align:right;"> 21.98 </td>
   <td style="text-align:right;"> NA </td>
   <td style="text-align:right;"> NA </td>
  </tr>
  <tr grouplength="3"><td colspan="8" style="border-bottom: 1px solid;"><strong>Średnie dystansów</strong></td></tr>
<tr>
   <td style="text-align:left; padding-left:  2em;" indentlevel="1"> Średnie dystansów </td>
   <td style="text-align:right;"> 1.67 </td>
   <td style="text-align:right;"> 2.13 </td>
   <td style="text-align:right;"> 2.10 </td>
   <td style="text-align:right;"> 2.19 </td>
   <td style="text-align:right;"> 2.28 </td>
   <td style="text-align:right;"> 1.42 </td>
   <td style="text-align:right;"> NA </td>
  </tr>
  <tr>
   <td style="text-align:left; padding-left:  2em;" indentlevel="1"> Rozkład średnich dystansów (%) </td>
   <td style="text-align:right;"> 14.19 </td>
   <td style="text-align:right;"> 18.05 </td>
   <td style="text-align:right;"> 17.82 </td>
   <td style="text-align:right;"> 18.58 </td>
   <td style="text-align:right;"> 19.35 </td>
   <td style="text-align:right;"> 12.01 </td>
   <td style="text-align:right;"> NA </td>
  </tr>
  <tr>
   <td style="text-align:left; padding-left:  2em;" indentlevel="1"> Rozkład średnich dystansów(-n) (%) </td>
   <td style="text-align:right;"> 16.12 </td>
   <td style="text-align:right;"> 20.51 </td>
   <td style="text-align:right;"> 20.25 </td>
   <td style="text-align:right;"> 21.12 </td>
   <td style="text-align:right;"> 21.99 </td>
   <td style="text-align:right;"> NA </td>
   <td style="text-align:right;"> NA </td>
  </tr>
  <tr grouplength="3"><td colspan="8" style="border-bottom: 1px solid;"><strong>Średnie ważone dystansów</strong></td></tr>
<tr>
   <td style="text-align:left; padding-left:  2em;" indentlevel="1"> Średnie ważone dystansów </td>
   <td style="text-align:right;"> 1.68 </td>
   <td style="text-align:right;"> 2.13 </td>
   <td style="text-align:right;"> 2.11 </td>
   <td style="text-align:right;"> 2.20 </td>
   <td style="text-align:right;"> 2.29 </td>
   <td style="text-align:right;"> 1.43 </td>
   <td style="text-align:right;"> NA </td>
  </tr>
  <tr>
   <td style="text-align:left; padding-left:  2em;" indentlevel="1"> Rozkład średnich ważonych dystansów (%) </td>
   <td style="text-align:right;"> 14.20 </td>
   <td style="text-align:right;"> 18.01 </td>
   <td style="text-align:right;"> 17.80 </td>
   <td style="text-align:right;"> 18.56 </td>
   <td style="text-align:right;"> 19.33 </td>
   <td style="text-align:right;"> 12.09 </td>
   <td style="text-align:right;"> NA </td>
  </tr>
  <tr>
   <td style="text-align:left; padding-left:  2em;" indentlevel="1"> Rozkład średnich ważonych dystansów (-n) (%) </td>
   <td style="text-align:right;"> 16.16 </td>
   <td style="text-align:right;"> 20.49 </td>
   <td style="text-align:right;"> 20.25 </td>
   <td style="text-align:right;"> 21.11 </td>
   <td style="text-align:right;"> 21.99 </td>
   <td style="text-align:right;"> NA </td>
   <td style="text-align:right;"> NA </td>
  </tr>
</tbody>
</table>
{{< /rawhtml >}}

### Wskaźnik zagrożenia

Także i tu użyjemy średniej z punktacji z pominięciem kategorii N, której
maksimum da nam najbardziej prawdopodobną emocję.


```r
token_means <- tokens_table[c(6, 9, 12), 2:6][, sapply(.SD, mean)]
token_winner <- names(token_means[token_means == min(token_means)])
token_winner
```

```
## [1] "H"
```

Miarą dobroci będzie tu dystans od kategorii N. Przeskalowany do zakresu 0
do 100.


```r
token_goodness <- tokens_table[c(5, 8, 11), mean(N)]
token_goodness <- token_goodness
token_goodness
```

```
## [1] 11.96333
```

## Wyniki

Nie odnaleźliśmy oznak kryzysu w dostarczonej partii informacji.
Zarówno analiza symboli jak i tokenów wykazała, że ogólny przekaz zebranych
wypowiedzi był raczej pozytywny a dominującym uczuciem było szczęście.

{{< rawhtml >}}
<table class="table table-striped table-hover table-condensed" style="margin-left: auto; margin-right: auto;">
 <thead>
  <tr>
   <th style="text-align:left;"> Type </th>
   <th style="text-align:left;"> Winner </th>
   <th style="text-align:right;"> Goodness </th>
  </tr>
 </thead>
<tbody>
  <tr>
   <td style="text-align:left;"> Symbol </td>
   <td style="text-align:left;"> pozytywne </td>
   <td style="text-align:right;"> 80.58 </td>
  </tr>
  <tr>
   <td style="text-align:left;"> Token </td>
   <td style="text-align:left;"> H </td>
   <td style="text-align:right;"> 11.96 </td>
  </tr>
</tbody>
</table>
{{< /rawhtml >}}

## Dynamika

W ten sposób przetworzone zbiory z kolejnych ramek czasu, pozwolą na
uzyskanie szeregu, na podstawie którego będzie można uzyskać prognozy.
