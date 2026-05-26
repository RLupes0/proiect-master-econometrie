# Modelarea Predictivă a Costurilor Medicale folosind Algoritmul XGBoost

[![Python Version](https://img.shields.io/badge/python-3.8%2B-blue.svg)](https://www.python.org/)
[![Machine Learning](https://img.shields.io/badge/framework-XGBoost%20%7C%20Scikit--Learn-orange.svg)](https://scikit-learn.org/)
[![License](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)

## Descrierea Proiectului
Acest proiect implementează un flux complet de analiză statistică și Machine Learning pentru **predicția costurilor medicale individuale** (`charges`) pe baza profilului demografic și a stilului de viață al asiguraților. Scopul principal este transformarea datelor exploratorii în decizii matematice fundamentate, optimizând gestionarea riscului financiar și politicile de tarifare (*pricing*) în asigurările de sănătate.

Prin compararea riguroasă a modelelor parametrice (Regresie Liniară OLS) cu modele non-parametrice bazate pe ansambluri de arbori (Random Forest și XGBoost), proiectul validează **XGBoost** ca fiind soluția optimă, obținând o acuratețe de **80.27%** ($R^2$).

---

## Obiective Cheie
1. **Identificarea Factorilor de Risc:** Analiza impactului și ierarhizarea predictorilor (fumat, BMI, grupe de vârstă) asupra cheltuielilor medicale.
2. **Corectarea Structurală a Datelor:** Preprocesarea variabilelor puternic asimetrice (ajustarea indicelui Skewness de la `1.52` la `-0.09` prin transformare logaritmică).
3. **Modelare Predictivă de Înaltă Performanță:** Maximizarea capacității de generalizare a algoritmilor și minimizarea erorii medii absolute (MAE).
4. **Diagnoza Erorilor:** Evaluarea robustă a reziduurilor pentru testarea ipotezelor de homoscedasticitate și lipsă de polarizare.

---

## Structura și Fluxul Proiectului

### 1. Analiza Exploratorie (EDA)
* **Variabila Țintă (`charges`):** S-a identificat o distribuție non-normală, puternic înclinată spre dreapta, cu o distanță majoră între medie și mediană. Diagrama Box Plot a evidențiat prezența unui volum masiv de outliers în intervalul $35,000 - $64,000.
* **Variabila de Control (`bmi`):** Spre deosebire de costuri, indicele de masă corporală urmează o distribuție normală perfectă, sub formă de clopot (centrată în jurul valorii 30), servind drept referință pentru uniformitate biologică.

### 2. Preprocesare și Feature Engineering
* **Transformare Logaritmică:** Aplicarea funcției $log1p$ ($\ln(x+1)$) asupra costurilor pentru normalizarea distribuției și reducerea impactului valorilor extreme.
* **Segmentarea Vârstei:** Conversia variabilei continue `age` în intervale clinice: *Sub 30*, *31-46*, *47 și peste*.
* **Codificare Categorială:** * **One-Hot Encoding (Variabile Dummy):** Aplicat pentru grupele de vârstă și cele 4 regiuni geografice, prevenind o ierarhizare artificială.
  * **Codificare Binară:** Aplicată pentru variabila `sex` (*female* = 0, *male* = 1) și statusul de fumător.

### 3. Analiza de Corelație și Multicoliniaritate
* **Corelația Pearson:** A demonstrat o legătură liniară majoră între statusul de fumător (`smoker_binary`) și costurile medicale (**0.67**). Genul și regiunea geografică prezintă corelații insignifiante (apropiate de 0).
* **Corelații Structurale:** Explicarea corelațiilor negative rezultate natural din caracterul mutual exclusiv al variabilelor dummy.
* **Tratarea Redundanței:** Testarea multicoliniarității pentru eliminarea predictorilor redundanți și asigurarea stabilității coeficienților.

### 4. Validare Statistică (ANOVA)
Înainte de evaluarea ML, modelul a fost supus testului **ANOVA (Type II)** pe subsetul de antrenament (**80%** din date, $N=1069$), obținând:
* Un indicator **F-statistic de 331.18** și un **p-value global infim ($4.34 	imes 10^{-282}$)**, confirmând semnificația statistică înaltă.
* `smoker_binary` deține cea mai mare sumă a pătratelor (**398.16**), urmat de grupele de vârstă, atestând matematic principalii vectori de variabilitate.
* Toți predictorii majori au înregistrat un $p	ext{-value} < 0.05$, cu excepția regiunii `southwest` ($0.61$), demonstrând o lipsă de aport individual în prezența celorlalți factori.

---

## Evaluarea Comparativă a Modelelor

Performanța algoritmilor a fost evaluată pe subsetul de testare independent (**20%** din date, $N=268$):

| Model | R² Score (Acuratețe) | MAE (Eroarea Medie Absolută) |
| :--- | :---: | :---: |
| **XGBoost Regressor** | **0.8027** | **0.2780** |
| **Regresie Liniară (OLS)** | 0.7982 | 0.3109 |
| **Random Forest Regressor** | 0.7942 | 0.2798 |

### Concluzii Algoritmice:
* **XGBoost** a fost selectat drept model final datorită capacității superioare de a capta interacțiunile complexe non-liniare (ex: intersecția dintre fumat și un BMI ridicat) și a minimizării riguroase a erorilor absolute (MAE minim de **0.2780**).
* Analiza grafică arată că valorile prezise de XGBoost urmăresc fidel diagonala ideală, reducând substanțial erorile sistematice observate la Regresia Liniară.

---

## Analiza Reziduurilor (Diagnoza Finală)
Evaluarea erorilor modelului XGBoost confirmă corectitudinea constructului:
* **Simetrie și Centrare:** Reziduurile ($e = y - \hat{y}$) sunt distribuite uniform în jurul valorii 0, demonstrând un model imparțial (fără sub/supraestimare sistematică).
* **Homoscedasticitate:** Varianta erorilor rămâne stabilă pe întreg spectrul valorilor estimate.
* **Independență:** Absența corelațiilor reziduale cu variabilele independente confirmă că modelul a extras integral informația utilă din date.

---

## Impact și Perspectivă de Business
* **Pricing Dinamic:** Trecerea de la grile rigide la un sistem automatizat de cotare bazat pe riscul individual (ajustări precise bazate pe penalizarea comportamentelor de risc).
* **Strategie de Achiziție:** Utilizarea impactului negativ al segmentului tânăr (`-0.52` în model) pentru campanii agresive de marketing pe segmentul de vârstă sub 30 de ani.
* **Managementul Daunelor:** Identificarea „zonei roșii” (fumători cu BMI ridicat) permite direcționarea programelor preventive de wellness pentru reducerea incidenței cazurilor extreme (outlierii de cost de până la $64,000).

---

## Tehnologii Utilizate
* **Limbaj:** Python 3.x
* **Analiză & Preprocesare:** `pandas`, `numpy`, `scipy`
* **Vizualizare date:** `matplotlib`, `seaborn`
* **Machine Learning & Statistică:** `xgboost`, `scikit-learn`, `statsmodels`

---
*Proiect dezvoltat ca studiu de caz în aplicarea algoritmilor avansați de Machine Learning în Analiza Actuarială și Managementul Riscului.*
