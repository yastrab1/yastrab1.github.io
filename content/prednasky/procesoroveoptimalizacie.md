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

Pri kompilácií s ` g++ -O3 -mavx2 -mfma main.cpp -o main` prvá funkcia zbehne za 0.0166178 sekúnd, druhá za 0.32011 sekúnd.

> Ako je toto možné? Je to skoro identický kód, jeden ale zbehne skoro 30x rýchlejšie.
>  - Ja, 2025

Za tieto urýchlenia môžu prevažne dve veci: CPU caching a C++ kompilátor. Poďme sa najprv pozrieť na caching.

## Pamäť a caching

Pamäť v počítači je jeden obrovský súvislý slíž. Keď chce CPU spraviť nejakú operáciu na tomto slíži, potrebuje si tento kus pamäte načítať do *registrov*. 

Register je jediná forma pamäte na ktorej vie robiť procesor operácie a ukladať medzivýsledky. Všetky ostatné typy pamäte musia byť načítane do registrov.
Keďže programy často potrebujú tú istú pamäť viackrát, používa sa *caching*. Cache je medzikrok medzi registrami a RAM, rýchlejšia ako RAM ale aj menšia. Je to 
z logistického dôvodu - spraviť tak rýchlu pamäť by bolo neskutočne drahé, možno aj nemožné. Na to aby si nemusel procesor viackrát pýtať ten istý kus pamäte,
uloží si ju do cache. Rýchlost pamäte sa udáva v _latency_, čo znamená čas ktorý trvá informácií prejsť odtiaľ k CPU.Typicky poznáme 3 levely CPU caches:

1. L1 cache(~1MB), latency okolo 1ns(!)[^1].
2. L2 cache(~20MB), latency okolo 4ns[^1].
3. L3 cache(~30MB, serverové procesory mávajú aj 300MB),  latency okolo 40ns[^1].
4. RAM(16GB, latency okolo 80ns[^1]).

[^1]:(1 nanosekunda = 1/1000000000 alebo jedna **miliardtina!** sekundy)

Procesor sa vždy pozrie či sa táto pamäť nenachádza najprv v jeho caches(najprv pozrie L1, potom L2 a tak ďalej...) aby nemíňal čas.
Ďalšia optimalizácia ktorú procesor robí je že často programy vyžadujú pamäť blízko pri sebe, preto načítava do cache
vždy extra kus okolo toho bodu ktorý si program vypýta. [^2]
[^2]: Tu je dôležitá poznámka že to čo sa nachádza v cache a čo v RAM si spravuje procesor úplne sám, inak by mohlo viesť k tzv. cache poisoning attacks