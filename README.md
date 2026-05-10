# Kvantifikacija nesigurnosti u modelima umjetne inteligencije: Okvir za prediktivno održavanje i analizu rizika

Ovaj repozitorij sadrži kod, podatke i dokumentaciju za završni rad.

## 📌 O projektu
Glavni cilj rada nije samo predviđanje kvara (npr. preostali vijek trajanja - RUL), već i **kvantifikacija pouzdanosti** te predikcije. Umjesto determinističkog izlaza "kvar za X ciklusa", model daje probabilistički okvir (npr. "kvar za X +/- Y ciklusa").

Fokus je na razlikovanju dvije vrste nesigurnosti:
* **Aleatorička:** Nesigurnost u samim podacima (šum senzora).
* **Epistemička:** Nesigurnost samog modela (nedostatak znanja o određenim stanjima).

## 🛠 Metode i implementacija
U radu se implementiraju i porede tri ključna pristupa:
1.  **Monte Carlo Dropout:** Korištenje dropout-a tokom faze testiranja za aproksimaciju Bayesovske nesigurnosti.
2.  **Deep Ensembles:** Treniranje više modela sa različitim inicijalizacijama radi procjene varijanse predikcije.
3.  **Bayesovske neuronske mreže (BNN):** Modeliranje težina mreže kao distribucija vjerovatnoće.

## 📊 Dataset
Za eksperimentalni dio koristi se **NASA CMAPSS (Jet Engine Simulated Data)** sa Kaggle-a.  
**Izvor:** [Kaggle - NASA CMAPSS Dataset](https://www.kaggle.com/datasets/behrad3d/nasa-cmaps)  

Skup sadrži:
* Vremenske serije senzorskih mjerenja.
* Operativne parametre mlaznih motora.
* Informacije o kvarovima (Run-to-failure podaci).

## 📂 Struktura repozitorija
* `/code` - Jupyter notebook-ovi i Python skripte za obradu podataka i trening modela.
* `/data` - Linkovi ka datasetu i pre-procesirani podaci.
* `/docs` - Literatura, PDF verzija završnog rada i ostali relevantni dokumenti.
* `README.md` - Opis projekta.

## 🚀 Trenutni status
- [x] Definisanje teme i istraživanje literature.
- [x] Odabir dataseta (CMAPSS).
- [x] Implementacija osnovnog (baseline) modela.
- [ ] Implementacija metoda za kvantifikaciju nesigurnosti.
- [ ] Pisanje finalne teze.