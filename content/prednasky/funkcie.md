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
{{< figure src="/images/prednasky/funkcie/functionExample.png" link="/images/prednasky/funkcie/functionExample.png">}}
Ak by sme chceli Pythonu povedať čo očakávame(a pre budúcich programátorov ktorí tento kód budú po nás čítať), dajú sa pridať tzv. type annotations.
Vráťme sa k nášmu príkladu a pridajme type annotations:

```python
def plus_dva_krat_dva(cislo:int)->int:
    return cislo*2+2

print(plus_dva_krat_dva(2)) # Outputs 6
print(plus_dva_krat_dva(1000)) # Outputs 20002
...
```
 --- 
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
Takýto `Traceback` sa dá čítať takto. Je tam napísané (most recent call last). To znamena že posledná zavolaná funkcia je posledná v tomto stack trace.
Jeden výpis tracebacku je v forme File <file v ktorej je daná funkcia zavolaná> line <riadok kde bola zavolaná> in <otcovská funkcia> a na
ďalšom riadku je ako bola zavolaná. Niečo takéto je veľmi užitočné keď sa snažíme zistiť kde bola chyba - vidíme ako bola naša funkcia zavolaná
a kde nastala chyba.

Tento traceback však nevypisuje všetky informácie v call stacku, konkrétne nie aktuálne definované premenné a ešte niektoré špeciálne hodnoty.
V textovej forme by to bolo totižto silne nečitateľné. Preto existuje debugger[^1] ktorý je typicky integrovaný do editorov a ukazuje viac
informácií. Ukážme si to na tomto príklade:
```python {hl_lines=[10]}
def prva_funkcia(pozdrav):
    b = 4
    print("Vnutri prvej funkcie " + pozdrav)
    return True

def druha_funkcia():
    print("Vnutri drhuej funkcie")
    a = 3
    return prva_funkcia("Python")
print(druha_funkcia())
```
Poďme po riadkoch.
[^1]: Používam [PyCharm](https://www.jetbrains.com/pycharm/) debugger. Ostatné editory(VSCode, Cursor) by to mali mať veľmi podobné.

{{< figure src="/images/prednasky/funkcie/StackTrace1.png" title="PyCharm debugger na riadku 10" link="/images/prednasky/funkcie/StackTrace1.png">}}
Keďže Python pri spustení preskakuje všetky definície funkcií, program začína na riadku 10. Na obrázku pre začiatok:
1. Naľavo vidieť call stack. Aktuálne obsahuje iba \<module\>. To predstavuje aktuálne spúštaný Python modul(typicky teda aktuálnu file).[^2]
2. V strede je aktuálny kontext. Keďže sme nedefinovali žiadne premenné okrem tých ktoré nám dá Python vždy(napríklad \_\_name\_\_)

 [^2]: V iných jazykoch(C, Java…) je typicky funkcia main() ktorá sa zavolá pri spustení kódu, a tým pádom každý kód spustený je vnútri nejakej funkcie. V pythone to tak nie je,
 preto existuje akoby neviditeľná funkcia \<module\>

Na riadku 10 Python chce zavolať funkciu 

Keď chceme debugovať funkciu hlbšie v call stack, treba spraviť „step in“. To znamená že vkročíme do funkcie ktorá bola zavolaná na tomto riadku(musíme to špecificky definovať
lebo často je to nežiadúce, napríklad nechceme vkročiť do funkcie ako `print` lebo veríme že tá funguje správne).

Keď sa dostaneme na koniec `druha_funkcia` vyzera stack trace takto:
{{< figure src="/images/prednasky/funkcie/StackTrace2.png" title="PyCharm debugger na konci druhej funkcie" link="/images/prednasky/funkcie/StackTrace2.png">}}

Zaujiímave veci čo sa zmenili od posledného stavu:
1. Na stack trace sa pridala funkcia `druha_funkcia` 
2. Do aktuálneho kontextu sa pridala premenná `a=3`.

Prečo to tak dopodrobna rozoberám je pre poriadne pochopenie kontextu(po anglicky scope) pravidiel v pythone. Pre viac čítania sa dá pozrieť na:  [tejto stránke](https://realpython.com/python-scope-legb-rule/)

Následne sa musí vyhodnotiť výraz za `return`, takže python spustí `prva_funkcia`.
Keď potom znovu "step in" do funkcie `prva_funkcia` vyzera stack trace takto:
{{< figure src="/images/prednasky/funkcie/StackTrace3.png" title="PyCharm debugger na začiatku prvej funkcie" link="/images/prednasky/funkcie/StackTrace3.png">}}

Dôležité je že v tomto prípade sa stratila lokálna premenná a. Je to preto, že premenné definované v kontexte jednej funkcie zostávajú iba v tomto kontexte.

Napríklad keby to tak nebolo by to mohlo viesť k zlému kódu kde by jedna funkcia očakávala že jej iná funkcia 
nastaví premenné a potom by nefungovala ak by bola zavolaná v inom poradí atď…

Taktiež parameter pozdrav bol definovaný ako lokálna premenná v tejto funkcii. 
{{< figure src="/images/prednasky/funkcie/StackTrace4.png" title="PyCharm debugger na konci prvej funkcie" link="/images/prednasky/funkcie/StackTrace4.png">}}
Na konci `prva_funkcia` vrátime True, čo znamená že na riadku 8 vyhodnotilo `return prva_funkcia("Python")` na `return True`. 

Tým pádom sa vyhodnotilo `print(druha_funkcia())` na `print(True)`. Takže výsledná konzola vyzerá takto: 
```text
Vnutri drhuej funkcie
Vnutri prvej funkcie Python
True
```

 ---
## Šialenstvo

Realisticky, toto je všetko čo by si potreboval vedieť o funkciách v Pythone. Ak ťa zaujíma viac, tu to je:

### Abstract Syntax Tree
Ako rozhoduje Python ktorý výraz vyhodnotí ako prvý? Vytvorí si tzv. Abstract Syntax Tree(AST). Je to strom kde každý list predstavuje niečo na čom závisí
rodič. To znamená že napríklad náš ukážkový program sa skompiluje do AST takto:
{{< figure src="/images/prednasky/funkcie/bigAST.png" link="/images/prednasky/funkcie/bigAST.png">}}

To je na začiatok trochu komplikované, preto sa pozrime na AST jednoduchšieho programu:
{{< figure src="/images/prednasky/funkcie/smallASTExample.png" link="/images/prednasky/funkcie/smallASTExample.png">}}
Pôvodný program vyzerá takto:
```python
print("Ahoj" + input())
```
Koreňom stromu je vrchol Module. To je začiatok každého programu, to isté ako \<module\> v stack trace vyššie.
Vrchol Expr označuje výraz, konkrétne náš riadok programu. Na to aby sme tento výraz mohli vyhodnotiť však musíme vyhodnotiť jeho deti,
pričom jeho najvrchnejšie dieťa je funkcia print. 

Ľavá vetva od print nás veľmi nezaujíma, v podstate to vraví Pythonu že pred tým ako chce spustiť print musí túto funkciu načítať do pamäte.

Pravá strana je však zaujímavá. BinOp znamená Binary Operation, takže operácia s dvoma vstupmi a jedným výstupom.
Na ľavej strane tejto operácie je konštanta("Ahoj") s ktorou už nič viac netreba spraviť, preto je to listový vrchol.
Operácia sa volá add, to nás tiež až tak nezaujíma, a pravá strana je znovu výraz ktorý treba vyhodnotiť.

Keď python spúšťa kód ide v presne opačnom poradí - od listov k vrcholu. To znamená že najprv by načítal funkcie input() a print(),
potom spustil input(), potom spustil operáciu add atď...

To ako sa AST tvorí a optimalizuje je výrazne nad rámec prednášky, ale jeho jediné využitia nie su v spúšťaní kódu. Niektoré cool využitia:
1. Funkcia "Rename symbol" v editoroch funguje na tomto princípe - rozbije si kód na takýto strom a postupne v ňom hľadá tento symbol
2. Optimalizácia kódu - v takomto strome vieme aplikovať pravidlá ako nejaké časti kódu optimalizovať. V praxi by to bolo omnoho zložitejšie ale napríklad 5*20 môžme
v takomto grafe nájsť a rovno nahradiť konštantou 100, prípadne odhalenie časti kódu ktorý logicky nemôže nikdy zbehnúť(`if False:`)
3. Formátovanie/linting. Ak vám niekedy editor vyhodil niečo na štýl: `a < b and b < c` can be replaced by `a<b<c` tak za toto môže práve linter.

### este by to chcelo dajaky zaver ale teraz sa mi nechce 