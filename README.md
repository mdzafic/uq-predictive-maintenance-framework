# Kvantifikacija nesigurnosti u modelima umjetne inteligencije: okvir za prediktivno održavanje i analizu rizika

Framework za predikciju preostalog životnog vijeka (RUL) industrijskih sistema uz kvantifikaciju nesigurnosti predikcija korištenjem modernih metoda mašinskog učenja.

## Problem

Klasični modeli za prediktivno održavanje uglavnom daju samo tačkaste predikcije, npr:

> "Motor će otkazati za 50 ciklusa."

Međutim, u realnim industrijskim sistemima ovakav pristup nije dovoljan, jer model može biti nesiguran ili nestabilan pri malim promjenama ulaznih podataka.

Cilj ovog projekta je razvoj framework-a koji, pored same predikcije, procjenjuje i pouzdanost modela, npr:

> "Preostali životni vijek: 50 ± 5 ciklusa."

Na taj način moguće je:
- bolje procijeniti rizik,
- identifikovati nesigurne predikcije,
- povećati robusnost AI sistema.

## Fokus projekta

- epistemic uncertainty (nesigurnost modela),
- aleatoric uncertainty (šum i inherentna nesigurnost podataka),
- robusnost modela,
- pouzdanost AI predikcija u uslovima promjene podataka.

## Implementirane metode

### 1. Monte Carlo Dropout
Aproksimacija Bayesovske nesigurnosti korištenjem dropout-a tokom inferencije.

### 2. Deep Ensembles
Kombinovanje više nezavisno treniranih modela radi procjene varijanse predikcija.

### 3. Bayesian Neural Networks
Modeliranje težina neuronske mreže kao distribucija vjerovatnoće.

## Dataset

Za eksperimente se koristi NASA CMAPSS dataset za prediktivno održavanje mlaznih motora.

Dataset sadrži:
- senzorska mjerenja,
- operativne parametre,
- run-to-failure sekvence,
- Remaining Useful Life (RUL) oznake.

Dataset:
[Kaggle - NASA CMAPSS Dataset](https://www.kaggle.com/datasets/behrad3d/nasa-cmaps)

## Ciljevi evaluacije

Metode se porede prema:
- tačnosti predikcije,
- kalibraciji nesigurnosti,
- stabilnosti modela,
- robusnosti pri promjeni uslova,
- kvalitetu procjene povjerenja modela.

## Struktura repozitorija

```text
/code - Jupyter notebook-ovi i Python skripte za obradu podataka i trening modela.
/data - Linkovi ka datasetu i pre-procesirani podaci.
/docs - Literatura, PDF verzija završnog rada i ostali relevantni dokumenti.
README.md - Opis projekta.
