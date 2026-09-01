# Ekspertni sistem za kupovinu računara (Fuzzy + ANN + Ekspertni)

Kratak vodič kroz cijeli zadatak — šta je urađeno, kojim redom, i koji kod služi za testiranje svakog dijela.

Sistem na osnovu **6 karakteristika računara** daje **odluku o kupovini**, i to na tri načina koja se porede:
1. **Fuzzy** — ekspertna pravila
2. **ANN** — neuronska mreža naučena iz podataka (Excel)
3. **Ekspertni** — kombinacija ANN + fuzzy

---

## Fajlovi u radnom folderu

Svi ovi fajlovi moraju biti u **istom folderu** (i taj folder = Current Folder u MATLAB-u):

| Fajl | Šta je |
|---|---|
| `fuzzySistem.fis` | glavni fuzzy sistem (6 ulaza, 1 izlaz, 47 pravila) |
| `fuzzy2.fis` | pomoćni fuzzy (1 ulaz, 1 izlaz) za ekspertni dio |
| `modelPrvi.slx` | Simulink model: Constant → ANN → izlaz |
| `modelDrugi.slx` | Simulink model: Constant → ANN → fuzzy2 → izlaz (daje ANN i ekspertni) |
| `VIES_Tabela.xlsx` | podaci (1000 redova) za treniranje/testiranje mreže |
| `app1.mlapp` | GUI (App Designer) |

---

## KORAK 1 — Fuzzy sistem

Otvoren komandom `fuzzy` (Fuzzy Logic Designer), tip **Mamdani**.

### Ulazi i rasponi (izvučeni iz min/max Excel tabele)

| Varijabla | Raspon | Fuzzy skupovi |
|---|---|---|
| cijena | [0 10] | niska / srednja / visoka |
| procesor | [0 100] | los / srednji / dobar |
| ekran | [0 12] | manji / srednji / veliki |
| memorija | [20 100] | mala / srednja / velika |
| tip_racunara | [0 20] | laptop / desktop / gaming |
| stanje | [0 20] | polovno / novo |
| **odluka** (izlaz) | [0 6] | lose / srednje / dobro |

### Funkcije pripadnosti (trapmf = trapez, trimf = trougao)

> Pravilo: krajnji skupovi = trapez sa "ramenom", srednji = trougao. Susjedni skupovi se **preklapaju** (meki prelaz).
> Napomena: procesor (35/65) i memorija (40/70) su granice date u tekstu zadatka.

| Varijabla | Skup | Tip | Parametri |
|---|---|---|---|
| cijena | niska / srednja / visoka | trap/tri/trap | [0 0 2 4] / [3 5 7] / [6 8 10 10] |
| procesor | los / srednji / dobar | trap/tri/trap | [0 0 20 40] / [35 50 65] / [60 80 100 100] |
| ekran | manji / srednji / veliki | trap/tri/trap | [0 0 3 5] / [4 6 8] / [7 9 12 12] |
| memorija | mala / srednja / velika | trap/tri/trap | [20 20 35 45] / [40 55 70] / [65 80 100 100] |
| tip_racunara | laptop / desktop / gaming | trap/tri/trap | [0 0 4 8] / [6 10 14] / [12 16 20 20] |
| stanje | polovno / novo | trap/trap | [0 0 7 12] / [8 13 20 20] |
| odluka | lose / srednje / dobro | trap/tri/trap | [0 0 1.5 2.5] / [2 3 4] / [3.5 4.5 6 6] |

*(Provjeri da se ovi brojevi poklapaju s onim što si stvarno upisao u Designeru — ako si negdje stavio malo drugačije, upiši te vrijednosti.)*

### Pravila (47)

Logika prati podatke: **bolji parametri → veća odluka**. Tri vrste pravila:
- 3 "čista" (svi ulazi na istom nivou)
- "presudni faktor" (samo 2 ulaza, ostalo se ignoriše) — npr. *procesor dobar & memorija velika → dobro*
- mješovita (3–4 ulaza)

Pravila su ubačena skriptom `upisivanjePravila.m` (koristi `addRule`), umjesto ručnog klikanja.

---

## KORAK 2 — ANN mreža

1. Učitani podaci: `ulazi` (1000×6) i `izlaz` (1000×1).
2. `nftool` → Import (Predictors = ulazi, Responses = izlaz, samples u redovima).
3. Split 70/15/15, 10 neurona, **Train** (Levenberg-Marquardt).
4. **Regression ≈ 1** (podaci su idealni → mreža nauči skoro savršeno).
5. **Export to Simulink** → dobijen blok "Function Fitting Neural Network".
6. Oko bloka dodato: Constant (`x1`) za ulaz + To Workspace (`izlazSim`, Array) za izlaz → snimljeno kao **modelPrvi**.

Constant blok vrijednost: `[cijena;procesor;ekran;memorija;tip_racunara;stanje]`, čekirano "Interpret vector parameters as 1-D".

---

## KORAK 3 — Ekspertni sistem

1. `fuzzy2` — mali fuzzy: 1 ulaz `[0 6]` + 1 izlaz `[0 6]`, po 3 iste funkcije (lose/srednje/dobro), 3 pravila jedan-na-jedan. Snimljen kao `fuzzy2.fis`.
2. **modelDrugi** = kopija modelPrvi + Fuzzy Logic Controller (FIS name: `readfis('fuzzy2')`) ubačen iza mreže.
3. Dva izlaza:
   - `izlazSim1` = odmah iza mreže (**čist ANN**)
   - `izlazSim` = iza fuzzy2 (**ekspertni**)

Lanac: `Constant → ANN →┬→ izlazSim1 (ANN)` i `└→ fuzzy2 → izlazSim (ekspertni)`

---

## KORAK 4 — GUI (App Designer)

Napravljen komandom `appdesigner` → Blank App.

**Nazivi komponenti:**
- Ulazi: `cijenaField`, `procesorField`, `ekranField`, `memorijaField`, `tipField`, `stanjeField` (Numeric)
- Izlazi riječ: `fuzzyField`, `annField`, `ekspertniField` (Text)
- Izlazi broj: `vrijednost1Field` (fuzzy), `vrijednost2Field` (ann), `vrijednost3Field` (ekspertni) (Numeric)
- Dugmad: `pokreni`, `stop`, `reset`

Callback dugmeta POKRENI: čita 6 polja → pokreće fuzzy (`evalfis`) i modelDrugi (`sim`) → upisuje brojeve i riječi.
Prevod broja u riječ: `<2.5 → Lose`, `2.5–4 → Srednje`, `>4 → Dobro`.

---

# TESTNI KODOVI (sortirano po dijelovima)

Sve zalijepiti u **Command Window**. Kod modela: prva linija (vrijednosti) MORA ići prije `sim`.

## T0 — Učitavanje podataka i raspona

```matlab
podaci = readmatrix('VIES_Tabela.xlsx');
ulazi  = podaci(:, 1:6);
izlaz  = podaci(:, 7);
[min(ulazi); max(ulazi)]   % rasponi ulaza
[min(izlaz)  max(izlaz)]   % raspon izlaza (Odluka)
```

## T1 — Test FUZZY sistema

```matlab
fis = readfis('fuzzySistem.fis');
% dobar racunar -> ocekuj ~5-6
evalfis(fis, [9  90  11  95  18  18])
% los racunar -> ocekuj ~1-2
evalfis(fis, [1  10  2   25  2   2 ])
```

## T2 — Test ANN mreže (modelPrvi)

```matlab
cijena=9; procesor=90; ekran=11; memorija=95; tip_racunara=18; stanje=18;
out = sim('modelPrvi');
out.izlazSim(1)     % ocekuj ~5.5
```

## T3 — Test EKSPERTNOG sistema (modelDrugi)

```matlab
cijena=9; procesor=90; ekran=11; memorija=95; tip_racunara=18; stanje=18;
out = sim('modelDrugi');
disp(out.izlazSim1(1))   % ANN        -> ~5.5
disp(out.izlazSim(1))    % ekspertni  -> ~5

% los racunar:
cijena=1; procesor=10; ekran=2; memorija=25; tip_racunara=2; stanje=2;
out = sim('modelDrugi');
disp(out.izlazSim1(1))   % -> ~1
disp(out.izlazSim(1))    % -> ~1
```

## T4 — Test mreže na uzorcima iz Excela (poređenje sa stvarnim)

Koristi skriptu `testirajANN.m` (uzima redove iz tabele, poredi predikciju sa stvarnom Odlukom). Razlika treba biti sićušna (R≈1).

---

# Česte greške i rješenja

| Greška | Uzrok | Rješenje |
|---|---|---|
| `Invalid input variable name in rule description` | ime u pravilu ≠ ime u .fis (mala/velika slova, `tip_racunara` vs `tipRacunara`) | uskladi imena; MATLAB razlikuje velika/mala slova |
| `names must be valid MATLAB identifiers` | razmak ili naše slovo (č,š,ž) u imenu | koristi samo slova/cifre/`_`, bez razmaka i kvačica |
| model daje ~0.65 umjesto pravog broja | `sim` pokrenut bez postavljanja 6 varijabli | postavi vrijednosti u ISTOM potezu prije `sim` |
| `file 'modelDrugi' not found` | pogrešan Current Folder | postavi folder gdje su .slx/.fis fajlovi |
| `izlazSim1 unrecognized` | To Workspace blok pogrešno nazvan | provjeri imena blokova u Simulinku |
| `guide` ne radi | uklonjen u novim verzijama | koristi App Designer (`appdesigner`) |
