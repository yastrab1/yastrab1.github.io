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

## Deeper look
Okej, základy by boli. Ako fungujú funkcie hlbšie?

Python je <a href="https://www.geeksforgeeks.org/compiler-design/difference-between-compiled-and-interpreted-language/"> interpretovaný jazyk</a>
preto pri niektorých konceptoch nám bude jednoduchšie preskočiť do kompilovaného jazyku ako napríklad C. Zatiaľ nám bude stačiť Python.

V podstate každý jazyk keď spúšťa kód má tzv. call stack.
Call stack je dátova štruktúra <a href="https://www.geeksforgeeks.org/dsa/stack-data-structure/">stack</a>.
V skratke sa to dá predstaviť ako veža palaciniek vedľa seba - na vežu vieme iba pridať na vrch, a aj iba z vrchu odobrať. 
Call stack si pamätá aktuálne spustené funkcie a ich kontext(napríklad premenné definované vnútri tejto funkcie). Napríklad si predstavme:


V skutočnosti ste už call stack určite stretli. Keď v pythone vznikne chyba ktorá nie je napravená alebo chytená, python spanikári, ukončí program a vypíše traceback. Určite ste už videli niečo takéto:
```
Traceback (most recent call last):
  File "/home/luki/PycharmProjects/testing/funkcie.py", line 11, in <module>
    druha_funkcia()
  File "/home/luki/PycharmProjects/testing/funkcie.py", line 10, in druha_funkcia
    prva_funkcia()
  File "/home/luki/PycharmProjects/testing/funkcie.py", line 6, in prva_funkcia
    raise ValueError("Nejaka chyba")
ValueError: Nejaka chyba
```
Takýto `traceback` sa dá čítať takto. Je tam napísané (most recent call last). To znamena že posledná zavolaná funkcia je zároveň aj posledná v tomto stack trace.
Jeden výpis tracebacku je v forme File <file v ktorej je daná funkcia zavolaná> line <riadok kde bola zavolaná> in <otcovská funkcia> a na
ďalšom riadku je ako bola zavolaná. Niečo takéto je veľmi užitočné keď sa snažíme zistiť kde bola chyba - vidíme ako bola naša funkcia zavolaná
a kde nastala chyba.

Tento traceback však nevypisuje všetky informácie v call stacku, konkrétne nie aktuálne definované premenné a ešte niektoré špeciálne hodnoty.
V textovej forme by to bolo totižto silne nečitateľné. Preto existuje debugger[^1] ktorý je typicky integrovaný do editorov a ukazuje viac
informácií. Ukážme si to na tomto príklade:
```python
def prva_funkcia():
    b = 4
    print("Vnutri prvej funkcie")

def druha_funkcia():
    print("Vnutri drhuej funkcie")
    a = 3
    prva_funkcia()
druha_funkcia()
```
Poďme po riadkoch.
[^1]: Používam [PyCharm](https://www.jetbrains.com/pycharm/) debugger. Ostatné editory(VSCode, Cursor) by to mali mať veľmi podobné.

{{< figure src="/images/prednasky/funkcie/StackTrace1.png" title="PyCharm debugger na riadku 9" link="/images/prednasky/funkcie/StackTrace1.png">}}
Keďže Python pri spustení preskakuje všetky definície funkcií, program začína na riadku 9. Na obrázku pre začiatok:
1. Naľavo vidieť call stack. Aktuálne obsahuje iba \<module\>. To predstavuje aktuálne spúštaný Python modul(typicky teda aktuálnu file).[^2]
2. V strede je aktuálny kontext. Keďže sme nedefinovali žiadne premenné okrem tých ktoré nám dá Python vždy(napríklad __name__)

 [^2]: V iných jazykoch(C,Java...) je typicky funkcia main() ktorá sa zavolá pri spustení kódu, a tým pádom každý kód spustený je vnútri nejakej funkcie. V pythone to tak nie je,
 preto existuje akoby neviditeľná funkcia \<module\>

Následne sa funkcia vykonáva ako každý iný kód až kým nenarazí na kľúčové slovo `return`. Keď narazí na return, stane sa toto:

 - Call stack odoberie svoj posledný prvok(nazvime ho n)
 - Program counter sa nastaví na riadok z ktorého sme volali túto funkciu(uložený v n)
 - Táto funkcia sa vyhodnotí na výsledok tejto funkcie(to čo `return`-la)

 V praxi to funguje takto:

```python
def plus_dva_krat_dva(cislo):
   vysledok = cislo*2+2
   return vysledok

print(plus_dva_krat_dva(2)) # Outputs 6
print(plus_dva_krat_dva(1000)) # Outputs 20002
```

