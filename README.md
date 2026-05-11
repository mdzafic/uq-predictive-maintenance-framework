# Završni rad (BSc Thesis)
**Student:** Muhamed Džafić  
**Institucija:** Univerzitet u Sarajevu, Elektrotehnički fakultet  
**Odsjek:** Računarstvo i informatika  
**Tema:** Kvantifikacija nesigurnosti u modelima umjetne inteligencije: okvir za prediktivno održavanje i analizu rizika

## Kratki pregled (Abstract)
Ova teza fokusira se na kvantifikaciju nesigurnosti u modelima umjetne inteligencije, s posebnim naglaskom na primjenu u prediktivnom održavanju i analizi rizika u industrijskim okruženjima. 

Dok klasični AI modeli daju samo tačkaste predikcije (npr. *motor će otkazati za 50 ciklusa*), cilj ovog rada je implementacija metoda koje uz predikciju daju i mjera pouzdanosti (npr. *50 +/- 5 ciklusa*). Razumijevanje razlike između epistemičke (nesigurnost modela) i aleatoričke (šum u podacima) nesigurnosti ključno je za donošenje odluka u sistemima gdje je greška neprihvatljiva.


## Metode i implementacija
U radu se implementiraju i porede tri ključna pristupa:
1.  **Monte Carlo Dropout:** Korištenje dropout-a tokom faze testiranja za aproksimaciju Bayesovske nesigurnosti.
2.  **Deep Ensembles:** Treniranje više modela sa različitim inicijalizacijama radi procjene varijanse predikcije.
3.  **Bayesovske neuronske mreže:** Modeliranje težina mreže kao distribucija vjerovatnoće.

## Dataset
Za eksperimentalni dio koristi se NASA CMAPSS (Jet Engine Simulated Data) sa Kaggle-a. Skup sadrži:
* Vremenske serije senzorskih mjerenja.
* Operativne parametre mlaznih motora.
* Informacije o kvarovima (Run-to-failure podaci).

**Izvor:** [Kaggle - NASA CMAPSS Dataset](https://www.kaggle.com/datasets/behrad3d/nasa-cmaps)

## Struktura repozitorija
* `/code` - Jupyter notebook-ovi i Python skripte za obradu podataka i trening modela.
* `/data` - Linkovi ka datasetu i pre-procesirani podaci.
* `/docs` - Literatura, PDF verzija završnog rada i ostali relevantni dokumenti.
* `README.md` - Opis projekta.