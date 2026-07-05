# Insurance-pricing-model

Projekt przedstawia pełną ścieżkę analityczną budowy silnika taryfikacyjnego na podstawie historycznych danych o szkodowości (Medical Cost Dataset z Kaggle). Celem jest wycena ryzyka i kalkulacja składek dla nowych klientów.

## Cel biznesowy
* Identyfikacja kluczowych czynników ryzyka wpływających na wysokość rocznych roszczeń.
* Przełożenie modelu statystycznego na gotowe narzędzie predykcyjne do wyceny polis.
* Weryfikacja założeń ekonometrycznych i optymalizacja taryfy.

## Technologie i narzędzia
* **Język:** R
* **Biblioteki:** `tidyverse` (`dplyr` do manipulacji danymi, `readr` do importu, `ggplot2` do wizualizacji)
* **Podejście:** Regresja Liniowa (MNK), Transformacje Log-Liniowe, Diagnostyka Gaussa-Markowa (BLUE), Rozkład Gamma

## Główne wnioski z analizy
Wizualizacja szkodowości względem wieku wyraźnie wskazuje na istnienie dwóch odseparowanych segmentów ryzyka, gdzie kluczowym czynnikiem podbijającym koszty jest **palenie tytoniu**.

![wykres_kosztow.png](wykres_kosztow.png)

## Wyniki modelowania i optymalizacji

### 1. Model Bazowy
* Zidentyfikowano wagi ryzyka: z każdym rokiem życia składka rośnie bazowo o ok. 259 USD, a status palacza generuje stałą zwyżkę o ponad 23 800 USD.
* **Problem:** Diagnostyka reszt ujawniła zjawisko heteroskedastyczności (błędy rosnące wraz z ceną polisy) oraz niedoszacowanie ekstremalnie drogich szkód (grube ogony na wykresie Q-Q). Model łamał założenia estymatora BLUE.

![model1.png](model1.png)

### 2. Model Ulepszony (Log-Lin z interakcją)
* Zastosowano logarytm naturalny na zmiennej objaśnianej `log(charges)` oraz wprowadzono efekt synergii `bmi * smoker`.
* **Efekt:** Ustabilizowano wariancję reszt i zneutralizowano wpływ punktów odstających (dźwignia Cooka spadła do bezpiecznego poziomu < 0.05).
* **Wyzwanie metodologiczne:** Przejście z powrotem na skalę oryginalną (charges) za pomocą prostego `exp()` wprowadza systematyczne niedoszacowanie składki (efekt nierówności Jensena). Bez zastosowania korekty (np. estymatora smearing Duana lub czynnika exp(σ²/2)), prognoza dla profilu wysokiego ryzyka (np. 60-letni palacz, BMI=20) jest niedoszacowana o ok. 10.7% (ok. 39 500 USD zamiast poprawnych merytorycznie ok. 43 800 USD).

![4 wykresy](model2.png)

## 3. Model Docelowy (GLM - Rozkład Gamma)
* Wdrożono rynkowy standard taryfikacji ubezpieczeniowej (Uogólniony Model Liniowy) z logarytmiczną funkcją łączącą: `family = Gamma(link = "log")`.
* **Zaleta teoretyczna:** GLM modeluje oczekiwaną wartość bezpośrednio na skali oryginalnej, eliminując problem obciążenia prognoz występujący przy transformacji log-normalnej (brak konieczności stosowania sztucznych korekt transformacji).
* **Rezultat:** Prawidłowo odwzorowano prawostronną skośność rozkładu kosztów oraz wyeliminowano problem sztucznych trendów w dolnych rejestrach reszt, typowy dla klasycznej metody MNK.
![4 wykresy](modelgam.png)
* **Dopasowanie modelu (MAE):** Średni błąd dopasowania w próbie (in-sample MAE) dla modelu GLM wynosi 4094 USD na pojedynczej polisie. Stanowi to czytelną dla zarządu miarę precyzji wyceny ryzyka w obecnym portfelu.
