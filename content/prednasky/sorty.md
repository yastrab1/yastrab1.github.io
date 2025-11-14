+++
title = 'Sorty'
date = 2025-11-13T20:37:51+01:00
draft = false
+++

V programovaní sa nám často stáva že máme nejakú postupnosť prvkov(často čislel, ale nie vždy), ktorú treba zoradiť vzostupne/zostupne. 
Na to máme sorty. Je to rodina algoritmov ktorá každá funguje nejak inak a má rôzne výhody a nevýhody. V tejto prednáške si ich popíšeme.

## Min sort:
Min sort je algoritmus ktorý je veľmi jednoduchý. Prejde celé pole a nájde v ňom najmenší prvok, ten dá na začiatok ďalšieho poľa a odoberie ho z pôvodného.
Toto zopakuje `n`-krát až kým nie je výsledné pole celé utriedené.

Časová zložitosť tohto algoritmu je `O(n^2)` keďže v ňom n-krát spravíme n porovnaní. 
## Merge sort
Merge sort používa klasickú metódu 'rozdeľuj a panuj'. Základný prístup vyzerá asi takto:

1. Rozdelíme si pole na dve polovice
2. Utriedime tie
3. Nájdeme spôsob ako ich spojiť efektívne

Krok 1. je jednoduchý. Ako utriedime dve polky poľa? Jednoducho! Predsa na nich zavoláme merge sort, tým pádom ich znovu rozdelíme. 
Takto delíme až kým nie je každá polka dĺžky jedna, čo znamená že už ďalej sa deliť nedá. Utriediť takéto pole ide jednoduchým porovnaním.

Krok 3. je zaujímavý. Keď mám dve utriedené polia, na ich spojenie mi stačí `n` operácií. V každej operácií vyberiem buď prvok z 
prvého poľa alebo z druhého poľa a presuniem ho na koniec výsledného poľa. Prečo to funguje? (yappik)

Časová zložitosť tohto algoritmu je `O(n log n)`. Prečo? Musím spraviť toto spájanie(s komplexitou `O(n)`) toľkokrát, koľkokrát sa dá
rozdeliť toto pole na polovice. To nám hovorí funkcia `log` (log s základom 2 má rovnakú komplexitu ako log s základom 10, preto sa zanedbáva).

