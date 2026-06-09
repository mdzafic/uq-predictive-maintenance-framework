# Kvantifikacija nesigurnosti u modelima umjetne inteligencije: okvir za prediktivno održavanje i analizu rizika

## Motivacija

Deterministički modeli za prediktivno održavanje daju samo tačkaste predikcije:

> *"Motor će otkazati za 50 ciklusa."*

Za sigurnosno-kritične industrijske primjene to nije dovoljno. Inženjer koji donosi odluku o zaustavljanju postrojenja treba znati ne samo *šta* model predviđa, nego i *koliko je siguran* u tu predikciju.

Ovaj rad razvija okvir koji uz svaku predikciju daje i mjeru pouzdanosti:

> *"Preostali životni vijek: 50 ± 8 ciklusa (95% interval pouzdanosti)."*

---

## Problem i hipoteza

**Problem:** Deterministički modeli daju samo predikciju RUL-a (Remaining Useful Life) bez mjere pouzdanosti, što nije dovoljno za sigurnosne industrijske primjene.

**Hipoteza:** UQ metode (MC Dropout, Deep Ensemble, Hibridni BNN) mogu pružiti pouzdane intervale nesigurnosti za predikciju RUL-a, uz zadržavanje prediktivne tačnosti usporedive s determinističkim Baselineom, pri čemu Conformal Prediction kao post-hoc tehnika može kalibrirati sve modele do statističke pouzdanosti od 95%.

---

## Dataset

**NASA CMAPSS FD001** — simulirani podaci rada mlaznih motora do otkaza.

- Senzorska mjerenja turbofan motora (21 senzorski kanal)
- Run-to-failure sekvence sa RUL oznakama
- Nakon EDA analize: uklonjeno **7 konstantnih senzora**, feature set reduciran sa **24 → 17 varijabli**
- RUL clipping na **125 ciklusa** (motor u ranim fazama ne pokazuje mjerljive znakove degradacije)

---

## Implementirane metode

Sve četiri metode dijele **identičnu LSTM osnovu** — arhitektura je kontrolirana varijabla, jedina razlika je UQ metoda.

| Metoda | Ulaz | Arhitektura | Izlaz |
|---|---|---|---|
| **Baseline LSTM** | 30 × 17 | LSTM(64) → Drop(0.3) → LSTM(32) → Drop(0.3) → Dense(32) | RUL |
| **MC Dropout** | 30 × 17 | Ista baza, Dropout **aktivan u inferenciji** (100×) | mean ± std |
| **Deep Ensemble** | 30 × 17 | 5 nezavisno treniranih instanci, ista baza | mean ± std |
| **Hibridni BNN** | 30 × 17 | LSTM slojevi deterministički, **DenseVariational** na Dense sloju (100×) | mean ± std |

### Conformal Prediction

Implementiran kao **post-hoc kalibracijska tehnika** (Sekcija 12) — kalibrira sve modele do ciljane statističke pouzdanosti od 95% bez ponovnog treniranja.

---

## Rezultati

### Tačnost i kalibracija (prije Conformal Prediction)

| Model | RMSE | MAE | Coverage (95%) | Interval Width | NLL | ECE |
|---|---|---|---|---|---|---|
| Baseline LSTM | 14.78 | 10.60 | — | — | — | — |
| MC Dropout | 14.86 | 11.08 | 0.81 | 32.37 | 4.76 | 0.155 |
| Deep Ensemble | **14.65** | **10.93** | 0.35 | **10.84** | 19.55 | 0.350 |
| Hibridni BNN | 14.80 | 11.13 | 0.75 | 27.51 | **4.86** | 0.205 |

### Efekt Conformal Prediction kalibracije

| Model | Coverage (prije) | Coverage (poslije) | Width (prije) | Width (poslije) |
|---|---|---|---|---|
| MC Dropout | 0.81 | **0.963** | 32.37 | 63.94 |
| Deep Ensemble | 0.35 | **0.950** | 10.84 | 61.69 |
| Hibridni BNN | 0.75 | **0.963** | 27.51 | 62.68 |

### Dekompozicija nesigurnosti

| Model | Aleatorička (std, ciklusi) | Epistemička (std, ciklusi) | Udio epistemičke |
|---|---|---|---|
| MC Dropout | 6.66 | 1.68 | 20.33% |
| Hibridni BNN | 5.65 | 1.43 | 20.41% |

Aleatorička komponenta (inherentni šum podataka) konzistentno dominira kroz cijeli raspon motora kod oba modela — što je očekivano za fizikalne sisteme s inherentnom stohastičnošću.

---

## Ključni nalaz

**Raskorak između tačnosti i kalibracije** je centralni nalaz rada:

- **Deep Ensemble** postiže najbolji RMSE, ali ima katastrofalnu kalibraciju (ECE = 0.350, Coverage = 0.35) — intervali su previše uski i ne pokrivaju stvarne vrijednosti.
- **MC Dropout** ima nešto slabiji RMSE, ali nudi najpouzdanije intervale (ECE = 0.155) i najbliži je idealnoj kalibraciji.

> Za industrijsku primjenu, sama tačnost predikcije nije dovoljan kriterij odabira modela. Model koji griješi, ali zna da griješi, superioran je modelu koji griješi s lažnom sigurnošću.

---

## Metrike evaluacije

Pored standardnih RMSE i MAE, korištene su metrike specifične za UQ:

- **Coverage Probability** — udio stvarnih vrijednosti koje padaju unutar prediktivnog intervala
- **Mean Interval Width** — prosječna širina prediktivnog intervala (uži = precizniji)
- **Negative Log-Likelihood (NLL)** — mjeri kvalitet probabilističke kalibracije
- **Expected Calibration Error (ECE)** — mjeri odstupanje nominalne i opažene pokrivenosti

---

## Struktura repozitorija

```
/code        Jupyter notebook s kompletnom implementacijom (EDA, trening, evaluacija)
/data        Linkovi ka CMAPSS datasetu i pre-procesirani podaci
/docs        Literatura, PDF verzija završnog rada i relevantni dokumenti
README.md    Opis projekta
```

---
