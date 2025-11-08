+++
title = "Procesorové optimalizácie"
date = "2025-07-11"
[ author ]
  name = "Lukas Lipka"
+++

Rád by som na začiatok začal s takým motivačným príkladom. Mám tu dve funkcie ktoré sčítajú obsah poľa v c++. Ktoré si myslíte že bude rýchlejšie a o koľko?
```cpp
int sum_1(int** arr) {
    int sum = 0;
    for (int i = 0; i < arr_size; i++) {
        for (int j = 0; j < arr_size; j++) {
            sum += arr[i][j];
        }
    }
    return sum;
}
```
```cpp
int sum_2(int** arr) {
    int sum = 0;
    for (int i = 0; i < arr_size; i++) {
        for (int j = 0; j < arr_size; j++) {
            sum += arr[j][i];
        }
    }
```

Pri kompilácií s `g++ main.cpp -o main` prvá funkcia zbehne za 0.13332 sekúnd, druhá za 0.307794 sekúnd(spriemerovaných 10 runs).

Pri kompilácií s ` g++ -O3 -mavx2 main.cpp -o main` prvá funkcia zbehne za 0.0166178 sekúnd, druhá za 0.32011 sekúnd.
(používajúc `arr_size`=10000)

Jediný rozdiel je riadok `sum += arr[i][j];` v prvej funkcii a `sum += arr[j][i];` v druhej funkcii a má to prekvapivo veľké dôsledky.

Za tieto urýchlenia môžu prevažne dve veci: CPU caching a C++ kompilátor. Poďme sa najprv pozrieť na caching.

## Pamäť a caching

Pamäť v počítači je jeden obrovský súvislý slíž. Keď chce CPU spraviť nejakú operáciu na tomto slíži, potrebuje si tento kus pamäte načítať do *registrov*. 

Register je jediná forma pamäte na ktorej vie robiť procesor operácie a ukladať medzivýsledky. Všetky ostatné typy pamäte musia byť načítane do registrov.
Keďže programy často potrebujú tú istú pamäť viackrát, používa sa *caching*. Cache je medzikrok medzi registrami a RAM, rýchlejšia ako RAM ale aj menšia. Je to 
z logistického dôvodu - spraviť tak rýchlu pamäť by bolo neskutočne drahé, možno aj nemožné. Na to aby si nemusel procesor viackrát pýtať ten istý kus pamäte,
uloží si ju do cache. Rýchlost pamäte sa udáva v _latency_, čo znamená čas ktorý trvá informácií prejsť odtiaľ k CPU.Typicky poznáme 3 levely CPU caches:

1. L1 cache(~1MB), latency okolo 1ns(!)[^1].
2. L2 cache(~20MB), latency okolo 4ns[^1].
3. L3 cache(~30MB, serverové procesory mávajú aj 300MB),  latency okolo 10ns[^1].
4. RAM(16GB, latency okolo 10-20ns[^1]).

[^1]:(1 nanosekunda = 1/1000000000 alebo jedna **miliardtina!** sekundy)

Procesor sa vždy pozrie či sa táto pamäť nenachádza najprv v jeho caches(najprv pozrie L1, potom L2 a tak ďalej...) aby nemíňal čas.
Ďalšia optimalizácia ktorú procesor robí je že často programy vyžadujú pamäť blízko pri sebe, preto načítava do cache
vždy extra kus okolo toho bodu ktorý si program vypýta. [^2]
[^2]: Tu je dôležitá poznámka že to čo sa nachádza v cache a čo v RAM si spravuje procesor úplne sám, inak by to bolo príliš pomalé a viedlo k rôznym kyberútokom-

Môžme sa pozrieť na rozloženie pamäte `int** arr`. Naše pole je pointer na pointery, preto vyzerá v pamäti nejak takto:
{{< figure src="/images/prednasky/procesorove_optimalizacie/img.png" title="Rozloženie pamäte" link="/images/prednasky/procesorove_optimalizacie/img.png">}}
Každý pointer ukazuje na nejaký úplne iný súvislý kus pamäte. Taktiež treba poznamenať že jednotlivé pointre na stĺpce sa nijako nevolajú len som ich pomenoval nech je zrejmé o čo sa jedná.

Keď sa teda pozrieme na náš pôvodný „pomalý" kód, vyzerá takto:
```cpp
int sum_2(int** arr) {
    int sum = 0;
    for (int i = 0; i < arr_size; i++) {
        for (int j = 0; j < arr_size; j++) {
            sum += arr[j][i];
        }
    }
```
Všimnime si že postupne páry i,j vyzerajú takto:

| hodnota i | hodnota j |
|-----------|-----------|
| 0         | 0         |
| 0         | 1         |
| 0         | 2         |
| ...       | ...       |
| 1         | 0         |

A tak ďalej. To znamená že j sa mení s každou iteráciou, pričom i iba raz za `arr_size` iterácií. 
Poďme po iteráciach:
1. `i=0,j=0` Program si vypýta hodnotu `arr[0][0]`. CPU načíta oblasť pamäte okolo `arr[0][0]` do cache.
2. `i=0,j=1` Program si vypýta hodnotu `arr[1][0]`. Keďže `row1` sa nachádza v pamäti niekde úplne inde, CPU musí vyprázdniť cache a načítať oblasť okolo `arr[1][0]` do cache
3. `i=0,j=2`. Program si vypýta hodnotu `arr[2][0]`. Keďže `row2` sa nachádza v pamäti niekde úplne inde, CPU musí vyprázdniť cache a načítať oblasť okolo `arr[2][0]` do cache
To znamená že každá iterácia vyžaduje čitanie z RAM(pre procesor je RAM extrémne pomalý zdroj dát). 

Pozrime sa naopak na náš rýchly kód:
```cpp
int sum_1(int** arr) {
    int sum = 0;
    for (int i = 0; i < arr_size; i++) {
        for (int j = 0; j < arr_size; j++) {
            sum += arr[i][j];
        }
    }
    return sum;
```
1. `i=0,j=0` Program si vypýta hodnotu `arr[0][0]`. CPU načíta oblasť pamäte okolo `arr[0][0]` do cache.
2. `i=0,j=1` Program si vypýta hodnotu `arr[0][1]`. CPU už má túto oblasť v cache, preto ju rovno využije.
3. ...
4. `i=1,j=0` Program si vypýta hodnotu `arr[1][0]`. CPU túto oblasť ešte načítanú nemá, preto musí spraviť ďalšie čítanie z pamäte.
Všimnime si že tu sa stane čítanie z RAM iba približne raz za `arr_size`, čím je tento kód výrazne rýchlejší.[^3] 

[^3]:Je dobré poznamenať že keď si to zrátame, RAM latency okolo 20ns v našom pomalom príklade vychádza na ~20\*10000\*10000 = 2000000000 ns = 2s. Preto je pomerne prekvapivé že to náš program zvládol za 0.3s. Odpoveďou na to je, že CPU má ešte veľa optimalizačných trikov ktoré používa(napríklad, často keď vyrobíme naraz veľké pole tak sa alokuje blízko pri sebe v pamäti, tým pádom nie každý riadok vyžaduje nové čítanie z RAM) a preto je toto číslo drasticky nižšie, no stále výrazne väčšie.


---

Okej, týmto sme vysvetlili túto časť:
> Pri kompilácií s `g++ main.cpp -o main` prvá funkcia zbehne za 0.13332 sekúnd, druhá za 0.307794 sekúnd(spriemerovaných 10 runs).

Ale stále je myslím veľmi zaujímavé sa pozrieť na toto:
>Pri kompilácií s ` g++ -O3 -mavx2 main.cpp -o main` prvá funkcia zbehne za 0.0166178 sekúnd, druhá za 0.32011 sekúnd.

Tu sa už nebudeme rozprávať až tak o tom ako fundamentálne funguje CPU a RAM, ale pozrieme sa na kompilátor.

## Kompilátor

Každý procesorový cyklus sa skladá z:
1. Fetch - Načíta inštrukciu ktorú ide vykonať
2. Decode - Dekóduje ktoré obvody musí spustiť
3. Execute - Vykoná inštrukciu

CPU inštrukcie ale nie sú z jazyka C ani Python, ale je to Assembler. Napríklad naša funkcia sum_1 vyzerá v assembleri takto:
```assembly
sum_1(int**):
        xor     edx, edx
        xor     esi, esi
        lea     rcx, [rdi+80000]
.L15:
        mov     rax, rdi
        vpxor   xmm1, xmm1, xmm1
.L16:
        vmovdqu ymm3, YMMWORD PTR [rax]
        vpcmpeqd        xmm5, xmm5, xmm5
        vpcmpeqd        xmm6, xmm6, xmm6
        add     rax, 64
        vmovdqu ymm4, YMMWORD PTR [rax-32]
        vpgatherqd      xmm0, DWORD PTR [rdx+ymm3*1], xmm5
        vpgatherqd      xmm3, DWORD PTR [rdx+ymm4*1], xmm6
        vinserti128     ymm0, ymm0, xmm3, 1
        vpaddd  ymm1, ymm1, ymm0
        cmp     rcx, rax
        jne     .L16
        vextracti128    xmm0, ymm1, 0x1
        add     rdx, 4
        vpaddd  xmm0, xmm0, xmm1
        vpsrldq xmm1, xmm0, 8
        vpaddd  xmm0, xmm0, xmm1
        vpsrldq xmm1, xmm0, 4
        vpaddd  xmm0, xmm0, xmm1
        vmovd   eax, xmm0
        add     esi, eax
        cmp     rdx, 40000
        jne     .L15
        mov     eax, esi
        vzeroupper
        ret
```
Ako sa asi dá všimnúť, fakt to nie je ľudsky čitateľné. Presne preto existujú programovacie jazyky. 

A ako sa preloží programovací jazyk do assembleru?
Preloží ho kompilátor(alebo interpreter, to teraz nebudeme riešiť).

Kompilátor má na starosti nejak požuť kód C a preložiť ho do assembleru. Klasicky sa to robí príkazom `g++ main.cpp -o main`.
Moderné kompilátory sú ale tak múdre že dokážu si všimnúť nejaké časti kódu ktoré sa dajú napísať efektívnejšie, a keď im to povolíme, automaticky
to vylepšia. Najjednoduchší príklad asi je tzv. constant unwinding, čo znamená že napríklad z `double c = 10*3/5.2` spraví `double c = 5.769230769`.
Takáto zmena síce nie je veľmi veľká ale ak sa tento kód spustí miliardukrát, ušetrí niekoľko desatín sekundy.

Kompilátoru povolíme tieto optimalizácie pomocou tzv. compiler flags. To sú parametre ktoré dáme kompilátoru a on pomocou nich si dovolí robiť rôzne optimalizácie.
Najčastejšie sú flags `-O3`, `-O2`, `-O1`, `-O0`, čo sú "balíčky" viacerých flags. -O0 znamená že neurob žiadne optimalizácie a -O3 znamená že skoro všetky[^4]
[^4]: Prečo by sme si nechceli vždy zapnút -O3? Typicky preto, že ak musí kompilátor hľadať tieto rôzne optimalizácie, na veľkých programoch to bude trvať dlho. Preto keď normálne programujeme, často používame -O0 nech vieme kód rýchlo otestovať a keď tento kód ideme vydať, použijeme -O3 pre maximálny výkon. V veľmi ojedinelých prípadoch sa môže kompilátor pomýliť a spôsobiť chybu pri optimalizácií, preto sa niekedy oplatí skúsiť spustiť kód s nižšou optimalizáciou.

-O3 je už samo o sebe dosť dobré. Napríklad bez -O3 má funkcia sum_2 v assembleri 33 riadkov a bez nej 17, čo nie vždy znamená rýchlejší kód ale často áno.

Následne tu je ešte flag `-mavx2`. Všetky flags ktoré začínajú na m(skratka pre machine) povedia kompilátoru niečo o počítači na ktorom bude výsledný kód bežať. Tieto nemôžu byť obsiahnuté v -O3, pretože 
-O3 funguje na (takmer) každom možnom počítači. V prípade že spustíme kód skompilovaný s `-mavx2` na procesore bez podpory pre [AVX2](https://en.wikipedia.org/wiki/Advanced_Vector_Extensions) tak ten program jednoducho nezbehne.

A čo to je to AVX2? Je to tzv. SIMD inštrukcia, alebo **S**ingle **I**nstruction **M**ultiple **D**ata inštrukcia, to znamená že dokáže spracovať niekoľko inštrukcií naraz.
Keď sa pozrieme na výstupný assembler po skompilovaní s avx2, funkcia sum_1, obsahuje inštrukciu `vpaddd  xmm0, xmm1, xmm0`. Bez detailov o tom ako funguje assembler,
inštrukcia vpadd je tzv. 256 bit vector inštrukcia, takže dokáže naraz spraviť 8x32bit operácie alebo 4x64bit operácie. Keďže klasický int je 32bit, tento kód by mal byť približne 8x rýchlejší. 
A naozaj, 0.13332÷0.0166178 = 8.022. 

Prečo teda sum_2 nedokázalo benefitovať z toho istého? 

Je to opäť kvôli pamäti. Zjednodušene, procesor musí načítavať z 8 rôznych miest z pamäte, a tie straty z toho majú také negatívne efekty že sa často kompilátor rozhodne že sa mu ani neoplatí robiť SIMD vektorizáciu. 
V podstate by sme sa zbavili 7 inštrukcí na sčítanie, lenže tie sú nič v porovnaní s 7 inštrukciami na čítanie z pamäte, preto by to aj tak kód veľmi neurýchlilo.