# Analiza sprzedaży – Power BI

## Opis projektu

Projekt przedstawia raport sprzedażowy przygotowany w Power BI na podstawie danych transakcyjnych. Celem było zbudowanie czytelnego i interaktywnego dashboardu umożliwiającego analizę sprzedaży według branży oraz w ujęciu rocznym (rok do roku).

Raport został oparty na poprawnie zbudowanym modelu danych z tabelą faktów „Zamówienia” oraz tabelą kalendarza wykorzystywaną do analizy czasowej.

---

## Zastosowane miary DAX

**Sprzedaż (miara bazowa)**  
Obliczenie wartości sprzedaży jako iloczyn ceny jednostkowej i ilości:

```
Sprzedaż = 
SUMX(
    'Zamówienia',
    'Zamówienia'[Cena jednostkowa] * 'Zamówienia'[Ilość]
)
```

**Sprzedaż z poprzedniego roku (PY)**  
Porównanie wyników rok do roku z wykorzystaniem funkcji inteligencji czasowej:

```
Sprzedaż PY = 
CALCULATE(
    [Sprzedaż],
    PREVIOUSYEAR(Kalendarz[Data])
)
```

**Zmiana sprzedaży vs poprzedni rok (%)**

```
Zmiana Sprzedaży vs PY = 
DIVIDE(
    [Sprzedaż] - [Sprzedaż PY],
    [Sprzedaż PY]
)
```

**Dynamiczny tytuł raportu**  
Tytuł dostosowujący się do wybranego zakresu lat:

```
Kolumnowy tytuł = 
IF(
    SELECTEDVALUE(Kalendarz[Rok]),
    "Sprzedaż w roku " & SELECTEDVALUE(Kalendarz[Rok]),
    "Sprzedaż w latach " 
        & MIN(Kalendarz[Rok]) 
        & " – " 
        & MAX(Kalendarz[Rok])
)
```

---

<img width="2062" height="1162" alt="image" src="https://github.com/user-attachments/assets/b8458eb4-d030-46f0-800f-290c766f3e19" />

## Strona 2 

Druga strona raportu koncentruje się na porównaniu wyników sprzedaży między branżami oraz na wizualnym wyróżnieniu wartości ponadprzeciętnych i skrajnych.

---

## Zastosowane miary DAX

### Średnia sprzedaż wg branży

Miara oblicza średnią sprzedaż dla wszystkich branż niezależnie od aktywnego kontekstu filtrowania.  
Zastosowanie funkcji ALL() usuwa bieżący filtr, dzięki czemu średnia liczona jest globalnie.

```
Średnia Sprzedaż wg Branży = 
AVERAGEX(
    SUMMARIZE(
        ALL(Klient),
        Klient[Branża]
    ),
    [Sprzedaż]
)
```

Miara ta została wykorzystana jako punkt odniesienia w wizualizacji.

---

### Dynamiczne kolorowanie kolumn względem średniej

Kolor kolumny zależy od tego, czy sprzedaż w danej branży jest powyżej czy poniżej średniej.

```
Kolumnowy kolor kolumn = 
SWITCH(
    TRUE(),
    [Sprzedaż] >= [Średnia Sprzedaż wg Branży], "dark green",
    "dark red"
)
```

Dzięki temu:
- branże powyżej średniej są oznaczone kolorem zielonym,
- branże poniżej średniej kolorem czerwonym.

Rozwiązanie zwiększa czytelność raportu i pozwala szybko zidentyfikować lepsze i słabsze segmenty.

---

### Wyróżnienie wartości maksymalnej i minimalnej

W drugim wykresie kolumnowym zastosowano miarę, która wyróżnia najwyższą i najniższą wartość sprzedaży wśród wszystkich branż.

```
Kolumnowy kolor max min = 
VAR max_sprzedaz =
    MAXX(
        SUMMARIZE(
            ALL(Klient),
            Klient[Branża]
        ),
        [Sprzedaż]
    )

VAR min_sprzedaz =
    MINX(
        SUMMARIZE(
            ALL(Klient),
            Klient[Branża]
        ),
        [Sprzedaż]
    )

RETURN
SWITCH(
    TRUE(),
    [Sprzedaż] = max_sprzedaz, "dark green",
    [Sprzedaż] = min_sprzedaz, "dark red"
)
```

Miara:
- usuwa kontekst filtrowania,
- identyfikuje globalną wartość maksymalną i minimalną,
- dynamicznie przypisuje kolor w zależności od wyniku.

---

<img width="2560" height="1440" alt="image" src="https://github.com/user-attachments/assets/61c8a56f-c824-4606-a1ca-d292c21f51d5" />

## Strona 3 – Sprzedaż w latach 2020–2024 (widok struktury)

Trzecia strona raportu przedstawia sprzedaż w ujęciu wieloletnim z podziałem na branże. Celem było zaprezentowanie zmiany struktury sprzedaży w czasie oraz pokazanie, które segmenty rozwijają się najszybciej.

---

## Zakres wizualizacji

Na stronie wykorzystano wykres kolumnowy skumulowany, który prezentuje:

- sprzedaż w latach 2020–2024  
- podział wartości na poszczególne branże  
- zmianę udziału branż w całkowitej sprzedaży  

Dzięki takiej formie wizualizacji możliwe jest jednoczesne przeanalizowanie:
- łącznego wzrostu sprzedaży rok do roku,
- struktury sprzedaży w danym roku,
- dynamiki rozwoju poszczególnych segmentów.

---

## Interaktywność raportu

Strona została wyposażona w fragmentatory:

- Rok  
- Województwo  

Pozwalają one filtrować dane i analizować wyniki w wybranym kontekście.  
Wszystkie wizualizacje reagują dynamicznie na wybrane filtry.

---
<img width="2560" height="1440" alt="image" src="https://github.com/user-attachments/assets/517dfc36-8e0d-4e7b-b85a-a02ae3f75638" />

## Strona 4 – Analiza sprzedaży i realizacja celu (2023)

Czwarta strona raportu koncentruje się na szczegółowej analizie sprzedaży w wybranym roku oraz ocenie realizacji założonych celów sprzedażowych.

---

## 1. Sprzedaż w roku 2023 – analiza branż

Na pierwszym wykresie słupkowym przedstawiono sprzedaż według branż.

Konfiguracja:
- Oś Y: Branża  
- Oś X: Sprzedaż  

Wizualizacja została sformatowana w celu poprawy czytelności osi i etykiet.  
Dodatkowo zastosowano:

- etykiety pokazujące procentową zmianę względem poprzedniego roku,
- formatowanie warunkowe (reguły kolorów),
- słupki błędów oparte na miarze „Sprzedaż PY”.

Miara etykiety zmiany sprzedaży:

```
Etykieta zmiany sprzedaży vs PR = 
SWITCH(
    TRUE(),
    [Zmiana sprzedaży vs PR] > 0, 
        "▲ " & FORMAT([Zmiana sprzedaży vs PR], "#.0%") & " vs PY",
    [Zmiana sprzedaży vs PR] < 0, 
        "▼ " & FORMAT([Zmiana sprzedaży vs PR], "#.0%") & " vs PY",
    [Zmiana sprzedaży vs PR]
)
```

Rozwiązanie to umożliwia szybkie określenie:
- które branże rosną,
- które notują spadek,
- jaka jest skala zmiany rok do roku.

---

## 2. Porównanie sprzedaży 2022 vs 2023

Drugi wykres słupkowy (dolna lewa część strony) przedstawia bezpośrednie porównanie sprzedaży w latach 2022 i 2023.

Wizualizacja:
- prezentuje dwie wartości dla każdej branży,
- umożliwia bezpośrednie zestawienie wyników rok do roku,
- pozwala ocenić skalę wzrostu lub spadku w ujęciu nominalnym.

Dzięki takiej konstrukcji użytkownik może szybko porównać wyniki bez konieczności analizowania osobnych wykresów.

---

## 3. Wykonanie celu sprzedażowego – 2023

Trzeci wykres słupkowy przedstawia poziom realizacji celu sprzedażowego w podziale na kategorie.

Konfiguracja:
- Oś Y: Kategorie  
- Oś X: Wykonanie celu (%) oraz Cel 100%

Zastosowane miary:

**Wykonanie celu 2023**

```
Wykonanie celu 2023 = 
DIVIDE(
    [Sprzedaż],
    SUM('cel 2023'[Cel])
)
```

Dodatkowo:
- utworzono miarę reprezentującą poziom 100% celu,
- dodano linie referencyjne,
- zastosowano czytelne skalowanie procentowe.

Wizualizacja pozwala:
- ocenić, które kategorie zrealizowały plan,
- wskazać segmenty poniżej oczekiwań,
- szybko określić stopień realizacji celu względem wartości docelowej.

---
<img width="2560" height="1440" alt="image" src="https://github.com/user-attachments/assets/b033c1da-20e5-4531-b4f7-500d267befcc" />

## Strona 5 – Analiza trendów sprzedaży w ujęciu miesięcznym

Piąta strona raportu skupia się na wizualizacji trendów sprzedaży w rozbiciu miesięcznym, umożliwiając szybkie wychwycenie odchyleń od średniej i identyfikację wartości ekstremalnych.

---

## 1. Trend sprzedaży – wykres liniowy z kolorem etykiet względem średniej

Na pierwszym wykresie liniowym przedstawiono sprzedaż w kolejnych miesiącach i latach.  

**Konfiguracja:**
- Oś X: Rok i miesiąc (hierarchia)  
- Oś Y: Sprzedaż  

**Dodatkowo zastosowano:**
- zaznaczenie dwóch wybranych miesięcy,
- linię średniej sprzedaży w wybranym okresie,
- etykiety danych dla obu serii (sprzedaż i średnia),
- cieniowanie obszaru pod linią w celu uwidocznienia trendu.

**Miara koloru etykiet względem średniej:**

```DAX
Liniowy kolor etykiet = 
VAR srednia_sprzedaz = 
    AVERAGEX(
        ALLSELECTED(Kalendarz[Rok], Kalendarz[Miesiąc]),
        [Sprzedaż]
    )

VAR kolor =
    SWITCH(
        TRUE(),
        [Sprzedaż] < srednia_sprzedaz, "teal",
        [Sprzedaż] > srednia_sprzedaz, "olive"
    )

RETURN
    kolor

<img width="2560" height="1440" alt="image" src="https://github.com/user-attachments/assets/86479a85-b38d-4aea-b099-3e532bd39e2b" />




