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
