+++
title = "Funkcie"
date = "2025-07-11"
[ author ]
  name = "Lukas Lipka"
+++
Funkcia je veľmi jednoducho kus kódu ktorý by sme radi niekedy zopakovali len s drobnými zmenami. Jednoduchý príkladad v Pythone je napríklad:

```python
def plus_dva_krat_dva(cislo):
    vysledok = cislo*2+2
    return vysledok

print(plus_dva_krat_dva(2)) # Outputs 6
print(plus_dva_krat_dva(1000)) # Outputs 20002
...
```
Je tu toho na začiatok viacej. def je kľúčové slovo v Pythone ktorým definujeme funkciu.
 V zátvorkách sa nachádzajú jej parametre.
 Parameter je tá drobná zmena v funkcií - napríklad chceme rôzne čísla vynásobiť 2 a pripočítať k nim 2 - preto si definujeme takúto funkciu.

 Keďže funkcia predstavuje nejaký blok kódu tak rovnako ako napríklad cykly začína jej kód dvojbodkou a je odsadený.
 Potom nasleduje samotný kód funkcie, kde môžeme vidieť kľúčové slovo return. Return je značenie pre "Tu funkcia skončila, vypľuj to čo nasleduje za return". V našom prípade teda vypľujeme cislo*2+2.Funkcia môže obsahovať niekoľko returnov, ak neobsahuje žiaden vráti sa automaticky typ None(nič)

Inak sa dajú predsaviť funkcie aj ako skrinka do ktorej dáme niečo(parametre) a vypľuje niečo iné. V našom prípade by to mohlo vyzerať takto:
## Ilustrácia trust me bro

Ak by sme chceli Pythonu povedať čo očakávame(a pre budúcich programátorov ktorí tento kód budú po nás čítať), dajú sa pridať tzv. type annotations.
Vráťme sa k nášmu príkladu a pridajme type annotations:

```python
def plus_dva_krat_dva(cislo:int)->int:
    return cislo*2+2

print(plus_dva_krat_dva(2)) # Outputs 6
print(plus_dva_krat_dva(1000)) # Outputs 20002
...
```
Teraz nám Python dovolí dovnútra dať iba čísla a zároveň vieme očákavať že výstup bude tiež vždy číslo. Je to ako zmluva medzi nami a touto funkciou.

## Deeper look
Okej, základy by boli. Ako fungujú funkcie hlbšie?

Python je <a href="https://www.geeksforgeeks.org/compiler-design/difference-between-compiled-and-interpreted-language/"> interpretovaný jazyk</a> preto pri niektorých konceptoch nám bude jednoduchšie preskočiť do kompilovaného jazyku ako napríklad C. Zatiaľ nám bude stačiť Python.

V podstate každý jazyk keď spúšťa kód má tzv. call stack.
Call stack je dátova štruktúra <a href="https://www.geeksforgeeks.org/dsa/stack-data-structure/">stack</a>,takže vieme na ňu pridať aj odoberať z konca. Vždy keď zavoláme nejakú funkciu stane sa nasledovné:

 - Na call stack sa pridá nový prvok - aktuálny riadok kódu a všetky parametre
 - Aktuálny program counter (vysvetliť) sa nastaví na prvý riadok funkcie.

Obrázok by sa zišiel

Následne sa funkcia vykonáva ako každý iný kód až kým nenarazí na kľúčové slovo `return`. Keď narazí na return, stane sa toto:

 - Call stack odoberie svoj posledný prvok(nazvime ho n)
 - Program counter sa nastaví na riadok z ktorého sme volali túto funkciu(uložený v n)
 - Táto funkcia sa vyhodnotí na výsledok tejto funkcie(to čo `return`-la)

 V praxi to funguje takto:

```python
1 def plus_dva_krat_dva(cislo):
2    vysledok = cislo*2+2
3    return vysledok
4
5 print(plus_dva_krat_dva(2)) # Outputs 6
6 print(plus_dva_krat_dva(1000)) # Outputs 20002
```
