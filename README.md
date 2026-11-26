# RetailRockets — od obejrzenia produktu do zakupu

Chciałem sprawdzić, co robią w sklepie internetowym osoby, które ostatecznie coś kupują. Czy wracają do sklepu kilka razy? Ile czasu mija od obejrzenia produktu do zakupu?

W Pythonie przeanalizowałem ponad 2,7 mln zdarzeń ze zbioru Retailrocket. Poniżej przedstawiam podsumowanie analizy i pliku `analiza_retailrocket.ipynb`.

#### *🔍 1. Kontrola jakości danych*

| Sprawdzenie | Wynik |
|---|---|
| Braki danych (`null`) | **0** w kolumnach `timestamp`, `visitorid`, `event` i `itemid`. W `transactionid` są **2 733 644** puste wartości, wszystkie przy wyświetleniach lub dodaniach do koszyka. |
| Duplikaty | **0,0167%** wierszy, czyli **460** nadmiarowych powtórzeń. |
| Spójność `transactionid` | Występuje **wyłącznie** przy zdarzeniu `transaction`. Każde zdarzenie zakupu ma identyfikator transakcji. |

> ✅ **Wniosek:** dane nie wymagały dużego czyszczenia.

---

#### *📅 2. Zakres obserwacji*

- **Początek:** 3 maja 2015, godz. 03:00:04 UTC.
- **Koniec:** 18 września 2015, godz. 02:59:47 UTC.
- **Łącznie:** 139 dni kalendarzowych, licząc obie daty.
- Pierwszy i ostatni dzień są **niepełne**. Pominąłem je przy porównywaniu aktywności według dnia tygodnia.

---

#### *🛒 3. Struktura zdarzeń*

| Zdarzenie | Znaczenie | Udział |
|---|---|---|
| `view` | wyświetlenie produktu | **96,67%** (2 664 312 zdarzeń) |
| `addtocart` | dodanie do koszyka | **2,52%** (69 332 zdarzenia) |
| `transaction` | zakup produktu | **0,81%** (22 457 zdarzeń) |

> 💡 **Wniosek:** zdecydowana większość ruchu to oglądanie produktów.

---

#### *📈 4. Aktywność w czasie*

- Na wykresie dziennym porównałem **liczbę zdarzeń** i **liczbę unikalnych użytkowników**.
- Widać **wzorzec tygodniowy**. Średnio najwięcej zdarzeń przypadało na **wtorki**, a najmniej na **soboty**.
- **Pik pod koniec lipca:** 26 lipca 2015 zarejestrowano **32 703 zdarzenia**.
- Ostatni dzień wygląda na gwałtowny spadek, ale zapis obejmuje tylko **około trzech godzin**.

---

#### *👥 5. Ilu użytkowników kupuje?*

| Wskaźnik | Wartość |
|---|---:|
| Unikalni użytkownicy | 1 407 580 |
| Kupujący | 11 719 |
| **Procent kupujących** | **0,83%** |
| Zdarzenia zakupu | 22 457 |
| Unikalne transakcje | 17 672 |

> 💡 **Wniosek:** zakup zarejestrowano u **11 719 spośród 1 407 580 użytkowników**.

---

#### *⏱️ 6. Sesje*

- Nowa sesja zaczyna się po przerwie **dłuższej niż 30 minut** między kolejnymi zdarzeniami tego samego użytkownika. To moje założenie analityczne.
- **Liczba sesji:** 1 761 675, w tym **14 297** z co najmniej jednym zakupem.
- **Konwersja sesji:** 14 297 sesji zakupowych / 1 761 675 wszystkich sesji = **0,81%**.

---

#### *👤 7. Czym różnią się kupujący od pozostałych?*

| Wskaźnik na użytkownika | Kupujący — średnia | Kupujący — mediana | Niekupujący — średnia | Niekupujący — mediana |
|---|---:|---:|---:|---:|
| Liczba zdarzeń | 19,68 | **6** | 1,81 | **1** |
| Liczba sesji | 3,65 | **2** | 1,23 | **1** |
| Liczba różnych produktów, których dotyczyły zdarzenia | 9,54 | **2** | 1,46 | **1** |
| Liczba aktywnych dni | 2,34 | **1** | 1,16 | **1** |

> 💡 **Wniosek:** kupujący częściej wracali do sklepu i wykonywali więcej działań. Szczególnie dobrze widać to przy liczbie zdarzeń. Wśród kupujących średnia wynosiła prawie **20**, ale mediana tylko **6**.

---

#### *🔄 8. Lejek zakupowy*

Sprawdziłem ścieżkę **wyświetlenie → koszyk → zakup** dla tego samego produktu w tej samej sesji.

| Etap | Liczba par | Przejście |
|---|---:|---|
| Wyświetlenie | 2 330 774 | Punkt początkowy |
| Koszyk po wyświetleniu | 45 114 | **1,94%** par z wyświetleniem |
| Zakup po takim koszyku | 14 807 | **32,82%** par z koszykiem po wyświetleniu |

> 💡 **Wniosek:** największy spadek widać między **wyświetleniem a koszykiem**. Tylko około 2 na 100 par przeszły ten etap.

**Co widać po uwzględnieniu wcześniejszych sesji?**

| Gdzie było wcześniejsze zdarzenie? | Wyświetlenie produktu | Dodanie do koszyka |
|---|---:|---:|
| W sesji zakupu | 17 715 (78,88%) | 17 985 (80,09%) |
| Tylko we wcześniejszej sesji | 2 109 (9,39%) | 2 020 (8,99%) |
| Brak wcześniejszego zdarzenia w całym zbiorze | 2 633 (11,72%) | 2 452 (10,92%) |

---

#### *⏳ 9. Ile czasu upływa od obejrzenia produktu do zakupu?*

| Kiedy użytkownik kupił produkt? | Liczba par | Udział dopasowanych par | Mediana czasu od pierwszego wyświetlenia |
|---|---:|---:|---:|
| W sesji pierwszego wyświetlenia | 14 293 | 76,12% | **5,98 min** |
| Po powrocie w innej sesji | 4 485 | 23,88% | **936,26 min**, czyli ok. **15 godz. 36 min** |

> 💡 **Wniosek:** dla około trzech czwartych dopasowanych par zakup nastąpił już podczas pierwszej sesji, w której użytkownik oglądał produkt. Mediana wyniosła wtedy **6 minut**. Przy powrotach połowa zakupów nastąpiła w ciągu **15,5 godziny** od pierwszego wyświetlenia, a najdłuższy odstęp wyniósł **131 dni**.

---

#### *📦 10. Które produkty są często oglądane, ale rzadko kupowane?*

| Produkt (`itemid`) | Wyświetlenia | Unikalni oglądający | Zdarzenia zakupu |
|---|---:|---:|---:|
| `187946` | 3 410 | **2 911** | **0** |
| `5411` | 2 325 | **2 078** | **0** |
| `370653` | 1 854 | **1 577** | **0** |

> 💡 **Wniosek:** produkt `187946` obejrzało prawie 3 tys. użytkowników, bez odnotowania jego zakupu.

---

#### *🧩 11. Segmenty użytkowników*

| Segment | Liczba użytkowników | Udział |
|---|---:|---:|
| Niekupujący, jedna sesja bez koszyka | **1 203 032** | **85,47%** |
| Niekupujący, powracający bez koszyka | 165 683 | 11,77% |
| Niekupujący z koszykiem | 27 146 | 1,93% |
| Kupujący, wiele sesji | 6 400 | 0,45% |
| Kupujący, jedna sesja | 5 319 | 0,38% |

[Dane źródłowe na Kaggle](https://www.kaggle.com/datasets/retailrocket/ecommerce-dataset)