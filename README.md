# 1. Úloha IOS (2022)
# Popis úlohy
Cílem úlohy je vytvořit shellový skript pro analýzu záznamů osob s prokázanou nákazou koronavirem způsobujícím onemocnění COVID-19 na území České republiky. Skript bude filtrovat záznamy a poskytovat základní statistiky podle zadání uživatele.
# Specifikace rozhraní skriptu
## JMÉNO

- `corona` — analyzátor záznamů osob s prokázanou nákazou koronavirem způsobujícím onemocnění COVID-19
## POUŽITÍ

`corona [-h] [FILTERS] [COMMAND] [LOG [LOG2 [...]]`
## VOLBY

- `COMMAND` může být jeden z:
  - `infected` — spočítá počet nakažených.
  - `merge` — sloučí několik souborů se záznamy do jednoho, zachovávající původní pořadí (hlavička bude ve výstupu jen jednou).
  - `gender` — vypíše počet nakažených pro jednotlivá pohlaví.
  - `age` — vypíše statistiku počtu nakažených osob dle věku (bližší popis je níže).
  - dai`ly — vypíše statistiku nakažených osob pro jednotlivé dny.
  - `monthly` — vypíše statistiku nakažených osob pro jednotlivé měsíce.
  - `yearly` — vypíše statistiku nakažených osob pro jednotlivé roky.
countries — vypíše statistiku nakažených osob pro jednotlivé země nákazy (bez ČR, tj. kódu CZ).
  - `districts` — vypíše statistiku nakažených osob pro jednotlivé okresy.
  - `regions` — vypíše statistiku nakažených osob pro jednotlivé kraje.
- `FILTERS` může být kombinace následujících (každý maximálně jednou):
  - `-a DATETIME` — after: jsou uvažovány pouze záznamy PO tomto datu (včetně tohoto data). `DATETIME` je formátu YYYY-MM-DD.
  - `-b DATETIME` — before: jsou uvažovány pouze záznamy PŘED tímto datem (včetně tohoto data).
  - `-g GENDER` — jsou uvažovány pouze záznamy nakažených osob daného pohlaví. `GENDER` může být `M` (muži) nebo `Z` (ženy).
  - `-s [WIDTH]` u příkazů `gender, age, daily, monthly, yearly, countries, districts` a `regions` vypisuje data ne číselně, ale graficky v podobě histogramů. Nepovinný parametr `WIDTH` nastavuje šířku histogramů, tedy délku nejdelšího řádku, na `WIDTH`. Tedy, `WIDTH` musí být kladné celé číslo. Pokud není parametr `WIDTH` uveden, řídí se šířky řádků požadavky uvedenými níže.
- `-h` — vypíše nápovědu s krátkým popisem každého příkazu a přepínače.
# Popis
1. Skript filtruje záznamy osob s prokázanou nákazou koronavirem způsobujícím onemocnění COVID-19. Pokud je skriptu zadán také příkaz, nad filtrovanými záznamy daný příkaz provede. Pokud příkaz zadán není, implicitně se použije příkaz `merge`.
2. Pokud skript nedostane ani filtr ani příkaz, opisuje záznamy na standardní výstup.
3. Skript umí zpracovat i záznamy komprimované pomocí nástrojů `gzip` a `bzip2` (v případě, že název souboru končí `.gz` resp. `.bz2`).
4. V případě, že skript na příkazové řádce nedostane soubory se záznamy (`LOG`, `LOG2`, …), očekává záznamy na standardním vstupu.
5. Grafy jsou vykresleny pomocí ASCII a jsou otočené doprava. Hodnota řádku je vyobrazena posloupností znaku mřížky `#`.
# Podrobné požadavky
1. Skript analyzuje záznamy ze zadaných souborů v daném pořadí.

2. Formát souboru se záznamy je CSV, kde oddělovačem je znak čárky ,. Celý soubor je v kódování ASCII. Formát je řádkový, každý neprázdný řádek (prázdné řádky jsou takové, které obsahují jen bílé znaky) odpovídá záznamu o jedné nákaze osoby ve tvaru (následující je hlavička CSV souboru)

```
id,datum,vek,pohlavi,kraj_nuts_kod,okres_lau_kod,nakaza_v_zahranici,nakaza_zeme_csu_kod,reportovano_khs
```

kde

   - `id` je identifikátor záznamu (řetězec neobsahující bílé znaky a znak čárky `,`),
   - `datum` je ve formátu `YYYY-MM-DD`,
   - `vek` je kladné celé číslo,
   - `pohlavi` je `M` (muž) nebo `Z` (žena),
   - `kraj_nuts_kod` je kód kraje, kde byla nákaza zjištěna,
   - `okres_lau_kod` je kód okresu, kde byla nákaza zjištěna,
   - `nakaza_v_zahranici` značí, zda zdroj nákazy byl v zahraničí (1 značí, že zdroj - - - nákazy byl v zahraničí, prázdné pole značí, že nebyl),
   - `nakaza_zeme_csu_kod` je kód země vzniku nákazy (pro nákazu vzniklou v zahraničí),
   - `reportovano_khs` značí, zda byla nákaza reportována krajské hygienické stanici.

Příklad souboru se třemi záznamy:

```
id,datum,vek,pohlavi,kraj_nuts_kod,okres_lau_kod,nakaza_v_zahranici,nakaza_zeme_csu_kod,reportovano_khs
6f4125cb-fb41-4fb0-a478-07b69ba106a4,2020-03-01,21,Z,CZ010,CZ0100,1,IT,1
f6b08ff5-203d-4a3e-aab0-a5d39ac9ab9e,2020-03-11,32,M,CZ080,CZ0804,,,1
b03dcf40-04cd-4f7b-a13d-767fc43c3013,2020-03-14,38,M,,,,,
```
   - První záznam z 1. března 2020 značí nákazu 21leté ženy v kraji “Hlavní město Praha” (kód CZ010), v okrese “Hlavní město Praha” (kód CZ0100). Žena byla nakažena v Itálii (kód IT) a nákaza byla reportována krajské hygienické stanici.
   - Druhý záznam značí vnitrostátní nákazu 32letého muže v Moravskoslezském kraji (kód CZ080), v okrese Nový Jičín (kód CZ0804), zjištěnou 11. března 2020.
   - Třetí záznam značí nákazu 38letého muže zjištěnou 14. března 2020, u níž nejsou k dispozici další informace.
3. Skript žádný soubor nemodifikuje. Skript nepoužívá dočasné soubory.

4. Záznamy ve vstupních souborech nemusí být uvedeny chronologicky a je-li na vstupu více souborů, jejich pořadí také nemusí být chronologické.

5. Pokud není při použití přepínače `-s` uvedena šířka `WIDTH`, pak každý výskyt symbolu `#` v grafu odpovídá počtu nákaz (zaokrouhleno dolů) dle příkazu následujícím způsobem:

   - `gender` — 100 000
   - `age` — 10 000
   - `daily` — 500
   - `monthly` — 10 000
   - `yearly` — 100 000
   - `countries` — 100
   - `districts` — 1 000
   - `regions` — 10 000
6. Při použití přepínače `-s` s uvedenou šířkou `WIDTH` je počet nákaz na mřížku upraven podle největšího počtu výskytů v řádku grafu. Řádek s největší hodnotou bude mít počet mřížek `WIDTH` a ostatní řádky budou mít proporcionální počet mřížek vzhledem k největší hodnotě. Při dělení zaokrouhlujte dolů. Tedy např. při `-s` 6 a řádku s největší hodnotou 1234 bude řádek s touto hodnotou vypadat takto: `######`.

7. Pořadí argumentů stačí uvažovat takové, že nejdřív budou všechny přepínače, pak (volitelně) příkaz a nakonec seznam vstupních souborů (lze tedy použít `getopts`).

8. Předpokládejte, že vstupní soubory nemůžou mít jména odpovídající některému příkazu nebo přepínači.

9. V případě uvedení přepínače `-h` se vždy pouze vypíše nápověda a skript skončí (tedy, pokud by za přepínačem následoval nějaký příkaz nebo soubor, neprovede se).

10. V případě neexistující hodnoty atributu u příkazů` gender, age, daily, monthly, yearly, districts, regions` agregujte záznamy s neexistující hodnotou pod hodnotu `None`, kterou ve výpisech uvádějte jako poslední.

11. Přepodkládejte, že je-li v záznamu uvedena hodnota pro nějaký atribut záznamu, pak je korektní (tj. není potřeba validovat vstup) s následujícími výjimkami:

    - ve `sloupci` datum je očekáváno korektní datum ve formátu `YYYY-MM-DD`, které odpovídá skutečnému dni (tedy, např., `yesterday` nebo `2020-02-30` jsou nevalidní hodnoty).
    - ve sloupci `vek` je očekáváno nezáporné celé číslo (tedy, např., `-42`, `18.5` nebo `1e10` jsou nevalidní hodnoty).
V případě detekování záznamu porušujícího některou z podmínek uvedených výše vypište chybu na chybový výstup a pokračujte ve zpracovávání dále (chybějící hodnota data/věku není porušením). Formát pro chybu je následující:

```
Invalid date: 6f4125cb-fb41-4fb0-a478-07b69ba106a4,2020-04-31,21,Z,CZ010,CZ0100,1,IT,1
Invalid age: 033fb060-2a10-42ce-80c1-72c2e39b1981,2020-03-05,dvacet,Z,CZ042,CZ0421,,,1
```
Kontrolu validity záznamů provádějte před případným filtrováním.

12. U příkazu age pracujte s následujícími intervaly a zarovnáním:

```
0-5   : 
6-15  :
16-25 :
26-35 :
36-45 :
46-55 :
56-65 :
66-75 :
76-85 :
86-95 :
96-105:
>105  :   
```

13. Implementace přepínačů `-d` a `-r` je nepovinná; korektní implementace může vynahradit jiné bodové ztráty.

14. U příkazů `gender, daily, monthly, yearly, countries, districts, regions` (bez přepínačů `-d `a `-r`) stačí vypisovat výstup ve formátu `hodnota: pocet` (bez mezery před dvojtečkou a s právě jednou mezerou za dvojtečkou), případně (pro přepínač `-s`) `hodnota: ###...#`. U příkazu `age` je zarovnání specifikováno výše.

Pro nepovinné přepínače `-r` a `-d` je dvojtečka na pozici o jedna větší, než je počet symbolů nejdelší hodnoty, tj. např.

```
hodnota      : 42
delsi_hodnota: 1337
```
15. Příkaz `countries` nevypisuje počet nákaz v České republice (kód `CZ` nebo chybějící hodnota).

16. Hodnoty ve sloupcích `nakaza_v_zahranici` a `reportovano_khs` ignorujte (tj. například u příkazu `countries` není třeba brát `nakaza_v_zahranici` do úvahy).

17. Záznamy nemusí nutně mít patřičný počet sloupců. V případě chybějícího sloupce postupujte stejně, jako kdyby v něm chyběla hodnota (pokud záznamu chybí N polí, znamená, to, že chybí hodnota `N` nejpravějších sloupců, tedy, např., pokud záznam obsahuje jen 7 polí, pak chybí hodnoty sloupců `nakaza_zeme_csu_kod` a `reportovano_khs`).

18. Nekontrolujte, zda obsahy sloupců `kraj_nuts_kod`, `okres_lau_kod` a nakaza_zeme_csu_kod odpovídají daným číselníkům. V případě implementace rozšíření `-d` a `-r` při použití hodnoty nedefinované v souboru s definicemi okresů/krajů vypisujte dané záznamy na chybový výstup v následujícím formátu:
```
Invalid value: 07958a56-6867-4245-b042-29c291c20359,2020-08-16,5,M,CZ099,CZ0999,,,1
```
# Implementační detaily
1. Skript by měl mít v celém běhu nastaveno `POSIXLY_CORRECT=yes`.

2. Skript by měl běžet na všech běžných shellech (`dash, ksh, bash`). Pokud použijete vlastnost specifickou pro nějaký shell, uveďte to pomocí direktivy interpretu na prvním řádku souboru, např. `#!/bin/bash` nebo `#!/usr/bin/env` `bash` pro `bash`. Můžete použít GNU rozšíření pro `sed` či `awk`. Jazyky Perl, Python, Ruby, atd. povoleny nejsou.

    UPOZORNĚNÍ: některé servery, např. `merlin.fit.vutbr.cz`, mají symlink `/bin/sh -> bash`. Ověřte si proto, že skript skutečně testujete daným shellem. Doporučuji ověřit správnou funkčnost pomocí virtuálního stroje níže.

3. Skript musí běžet na běžně dostupných OS GNU/Linux, BSD a MacOS. Studentům je k dispozici virtuální stroj s obrazem ke stažení zde: http://www.fit.vutbr.cz/~lengal/public/trusty.ova (pro VirtualBox, login: `trusty` / heslo: `trusty`), na kterém lze ověřit správnou funkčnost projektu.

4. Skript nesmí používat dočasné soubory. Povoleny jsou však dočasné soubory nepřímo tvořené jinými příkazy (např. příkazem `sed -i`).