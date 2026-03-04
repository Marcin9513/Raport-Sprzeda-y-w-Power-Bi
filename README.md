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
