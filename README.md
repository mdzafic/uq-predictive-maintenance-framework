# Kvantifikacija nesigurnosti u modelima umjetne inteligencije: okvir za prediktivno održavanje i analizu rizika

> Ovaj repozitorij implementira UQ (Uncertainty Quantification) okvir za prediktivno održavanje industrijskih sistema, razvijen u sklopu BSc završnog rada na Elektrotehničkom fakultetu Univerziteta u Sarajevu.

> **Student:** Muhamed Džafić &nbsp;·&nbsp; Odsjek za računarstvo i informatiku

---

## Motivacija

Deterministički modeli za prediktivno održavanje daju samo tačkastu predikciju:

> *"Motor će otkazati za 47 ciklusa."*

Za sigurnosno-kritične industrijske primjene (avijacija, energetika) to nije dovoljno. Inženjer koji donosi odluku o zaustavljanju postrojenja treba znati ne samo *šta* model predviđa, nego i *koliko je siguran* u tu predikciju, te da li ta nesigurnost dolazi od šuma u podacima ili od toga što model nije vidio slične slučajeve.

Ovaj rad razvija okvir koji uz svaku predikciju daje i mjeru pouzdanosti:

> *"Preostali životni vijek: 50 ± 13 ciklusa."*

---

## Problem i hipoteza

**Problem:** Deterministički modeli daju samo predikciju RUL-a (Remaining Useful Life) bez mjere pouzdanosti, što nije dovoljno za sigurnosno-kritične industrijske primjene, niti omogućava razlikovanje šuma u podacima od nedostatka znanja modela.

**Hipoteza:** Metode kvantifikacije nesigurnosti zasnovane na različitim probabilističkim principima (MC Dropout, Deep Ensemble, Hibridni BNN) produciraju kvalitativno različite profile nesigurnosti i različit kvalitet kalibracije, iako postižu statistički usporedivu tačnost tačkaste predikcije s determinističkim baseline modelom. Tačnost i kalibracija nesigurnosti su time dvije nezavisne dimenzije kvaliteta modela. Dodatno, MC Dropout i Hibridni BNN, čija se nesigurnost generira kroz stohastičko uzorkovanje, omogućavaju aproksimativnu dekompoziciju na epistemičku i aleatoričku komponentu, za razliku od Deep Ensemble-a. Konformalna predikcija, kao post-hoc tehnika, može sve modele kalibrirati do ciljane statističke pouzdanosti od 95%.

---

## Dataset

**NASA CMAPSS FD001** - simulirani podaci rada turbofan motora do otkaza (jedan operativni uvjet, jedan mod kvara).

- 100 motora u trening skupu (run-to-failure), 100 motora u test skupu
- 21 senzorski kanal + 3 operativna parametra
- Nakon EDA analize: uklonjeno 7 konstantnih senzora: feature set reduciran sa 24 na 17 varijabli
- RUL clipping na 125 ciklusa (piecewise linear model degradacije - motor u ranim fazama ne pokazuje mjerljive znakove degradacije)
- Dužina ulazne sekvence: 40 ciklusa, odabrana analizom osjetljivosti
- Nakon filtriranja motora s nedovoljnim brojem ciklusa za W=40, finalni test skup broji 96 motora

---

## Implementirane metode

Sve četiri metode dijele identičnu LSTM osnovu - arhitektura je kontrolirana varijabla, jedina razlika je UQ mehanizam.

| Metoda | Arhitektura | Izlaz |
|---|---|---|
| **Baseline LSTM** | LSTM(64) → Drop(0.3) → LSTM(32) → Drop(0.3) → Dense(32) → Dense(1) | RUL (tačkasta predikcija) |
| **MC Dropout** | Ista baza, dropout **aktivan u inferenciji** (T=100 stohastičkih prolaza) | mean ± std |
| **Deep Ensemble** | K=5 nezavisno treniranih instanci, svaka daje M=20 prolaza (ukupno 100 uzoraka) | mean ± std |
| **Hibridni BNN** | LSTM slojevi deterministički (dropout 0.4), gusti slojevi **DenseVariational** (reparametrizacijski trik, T=100 prolaza) | mean ± std |

### Konformalna predikcija (Conformal Prediction)

Implementirana kao *post-hoc* kalibracijska tehnika - kalibrira sve modele do ciljane pokrivenosti od 95% bez ponovnog treniranja, uz teorijsku garanciju pokrivenosti.

---

## Eksperimentalna postavka

- 50 epoha treninga (EarlyStopping, patience=10), batch=64, Adam
- Svi hiperparametri (dužina prozora W, dropout stopa p, broj epoha) odabrani OFAT analizom osjetljivosti na validacionom skupu
- Rezultati su izvedeni kroz 10 nezavisnih pokretanja cijelog eksperimenta (različito random sjeme za svako pokretanje), a prikazani su kao mean ± std, kako bi se izbjeglo izvođenje zaključaka iz jednog, potencijalno nereprezentativnog pokretanja
- Vizualizacije koje zahtijevaju kompletan set predikcija po motoru (scatter plotovi, uncertainty profili, dekompozicija, dijagram kalibracije, conformal i bootstrap analize) rađene su na jednom reprezentativnom pokretanju, odabranom kao ono čiji je coverage sva tri UQ modela najbliži prosjeku kroz svih 10 pokretanja

---

## Rezultati (10 nezavisnih pokretanja, mean ± std)

| Model | RMSE | MAE | Coverage (95%) | NLL | MIW | MACE |
|---|---|---|---|---|---|---|
| Baseline LSTM | 14.42 ± 0.28 | 10.24 ± 0.27 | — | — | — | — |
| MC Dropout | 14.35 ± 0.18 | 10.33 ± 0.21 | 0.80 ± 0.03 | 4.34 ± 0.20 | 31.12 ± 1.47 | 0.14 ± 0.03 |
| Deep Ensemble | 14.17 ± 0.11 | 9.99 ± 0.10 | 0.82 ± 0.02 | 4.16 ± 0.14 | 32.26 ± 1.53 | 0.09 ± 0.02 |
| Hibridni BNN | 14.46 ± 0.23 | 10.35 ± 0.22 | 0.85 ± 0.04 | 4.19 ± 0.29 | 37.20 ± 3.61 | 0.09 ± 0.03 |

Bootstrap 95% intervali pouzdanosti za RMSE (1000 iteracija, reprezentativno pokretanje) pokazuju da se intervali sva četiri modela u potpunosti preklapaju - razlike u tačnosti nisu statistički značajne.

### Efekt konformalne predikcije (reprezentativno pokretanje, 77 test motora nakon izdvajanja kalibracionog skupa)

| Model | Coverage (prije) | Coverage (poslije CP) | Width (prije) | Width (poslije CP) |
|---|---|---|---|---|
| MC Dropout | 0.792 | 0.948 | 30.12 | 63.75 |
| Deep Ensemble | 0.818 | 0.922 | 31.71 | 55.57 |
| Hibridni BNN | 0.831 | 0.961 | 38.52 | 68.43 |

Konformalna kalibracija približava pokrivenost nominalnom cilju za sve metode, ali ne izjednačava širinu intervala - Deep Ensemble zadržava najuže (najinformativnije) kalibrirane intervale.

### Dekompozicija nesigurnosti (aleatorička vs. epistemička, aproksimativna heuristika)

| Model | Aleatorička (std, ciklusi) | Epistemička (std, ciklusi) | Udio epistemičke |
|---|---|---|---|
| MC Dropout | 6.30 | 1.47 | 19.10% |
| Hibridni BNN | 8.34 | 1.64 | 16.64% |

Aleatorička komponenta (inherentni šum podataka, varijabilnost degradacije) konzistentno dominira kod oba modela, što je u skladu s fizičkom prirodom procesa degradacije motora. Deep Ensemble je isključen iz dekompozicije jer njegova varijansa proizlazi iz hijerarhijske mješavine neusaglašenosti modela i unutrašnjeg dropout uzorkovanja, pa jednostavna heuristika za njega nema jasno konceptualno značenje.

---

## Ključni nalazi

- **Tačnost i kalibracija su nezavisne dimenzije kvaliteta modela.** Bootstrap analiza potvrđuje da se sva četiri modela statistički ne razlikuju po RMSE-u - uvođenje probabilističkog tretmana ne narušava prediktivnu tačnost. Modeli se, međutim, jasno razlikuju po kvaliteti kalibracije nesigurnosti.
- **Nijedna metoda ne dominira po svim metrikama istovremeno:**
  - **Deep Ensemble** postiže najstabilniju tačnost kroz pokretanja (RMSE std = 0.11) i najuže, najinformativnije intervale - i prije i poslije konformalne kalibracije. Cijena je ~5× veći računski trošak treninga.
  - **Hibridni BNN** postiže najvišu pokrivenost prediktivnih intervala (0.85) i, uz konformalnu kalibraciju, ostaje najbliži nominalnom cilju od 95% (0.961). Po NLL i MACE metrikama statistički je izjednačen s Deep Ensemble-om.
  - **MC Dropout** je najjednostavniji za implementaciju (bez izmjene arhitekture ili treninga), ali je dosljedno najlošije kalibrirana metoda po sve tri kalibracijske metrike (coverage, NLL, MACE).
- **MC Dropout i Hibridni BNN** omogućavaju aproksimativnu dekompoziciju nesigurnosti na epistemičku i aleatoričku komponentu (Deep Ensemble ne, zbog mješovite prirode svoje varijanse), što je korisno za interpretabilnost u industrijskim primjenama - visoka epistemička nesigurnost signalizira da model nije vidio dovoljno sličnih slučajeva, dok visoka aleatorička nesigurnost ukazuje na inherentnu nepredvidivost samog motora.
- **Konformalna predikcija** uspješno kalibrira sve tri metode do pokrivenosti bliske 95%, ali ne rješava pitanje oštrine (širine) intervala - izbor UQ metode ostaje relevantan i nakon kalibracije.

> **Zaključak:** Odabir "najboljeg" modela ovisi o prioritetu primjene: stabilnost tačnosti i uske intervale nudi Deep Ensemble, najvišu pokrivenost Hibridni BNN, a najjednostavniju implementaciju uz mogućnost dekompozicije nesigurnosti MC Dropout. Nalaz je konzistentan sa širom literaturom koja pokazuje da ne postoji univerzalno superiorna UQ metoda za prognostiku preostalog životnog vijeka.

---

## Metrike evaluacije

- **RMSE / MAE**: tačnost tačkaste predikcije
- **Coverage Probability (CP)**: udio stvarnih vrijednosti unutar prediktivnog intervala
- **Mean Interval Width (MIW)**: prosječna širina prediktivnog intervala (uži = precizniji/oštriji)
- **Negative Log-Likelihood (NLL)**: proper scoring rule, simultano vrednuje tačnost i kalibraciju
- **Mean Absolute Calibration Error (MACE)**: prosječno odstupanje nominalne od opažene pokrivenosti kroz cijeli raspon nivoa pouzdanosti

## Tehnologije

Python, TensorFlow/Keras 2.20 (uključujući vlastitu implementaciju `DenseVariational` sloja), scikit-learn, SciPy, NumPy, pandas - razvijeno i testirano u Google Colab okruženju.

## Struktura repozitorija
 
```
/code        Jupyter notebooks:
               - kompletna implementacija (EDA, trening, evaluacija, 10 pokretanja)
               - analiza osjetljivosti hiperparametara (W, dropout stopa p, broj epoha)
/data        CMAPSS FD001 skup podataka
/docs        Pisani rad, literatura i relevantni dokumenti
README.md    Opis projekta
```

## Reference

Ključna literatura na kojoj se rad zasniva: Gal & Ghahramani (2016) - MC Dropout; Lakshminarayanan et al. (2017) - Deep Ensembles; Blundell et al. (2015) - Bayes by Backprop; Kendall & Gal (2017) - aleatorička/epistemička dekompozicija; Saxena et al. (2008) - CMAPSS; Basora et al. (2023) - benchmark UQ metoda u prognostici.