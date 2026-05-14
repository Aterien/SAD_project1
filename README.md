# SAD_project1

A repository dedicated for the first SAD project (Group 6)

Przeanalizowaliśmy zbiór danych zawierający różnorodne informacje na temat gier opublikowanych na znanej nam wszystkim platformie Steam. Zbiór danych zawiera informacje o grach wydanych na platformie Steam od czerwca 1997 r. do połowy marca 2026 r.

Zbiór danych zawiera informacje o grach wydanych na platformie Steam od czerwca 1997 r. do połowy marca 2026 r.

**Surowy zbiór danych w formacie CSV zajmuje 390 megabajtów i zawiera 39 kolumn** z informacjami o grze, takimi jak nazwa, data releasu, wydawca, deweloper, obsługiwane języki, oceny na Metacritic i w sklepie Steam oraz wiele innych.

W naszym projekcie skupiliśmy się na analizie tylko kilku **zmiennych jakościowych i ilościowych.**

Zgodnie z opisem dane zostały zebrane za pomocą **Steam API i Steam Spy**

Początkowo dataset zawieral duzo zbędnych i nieprzydatnych do analizy informacji, np. całą treśc opisu gry, linki do tej strony.

Usunelismy te kolumny z naszego datasetu. Schudł on aż na 350 mb ale nadal zawiera info o wiecej niż 115 000 grach!

Niektóre kolumny, takie jak Genres, Tags, Languages zawierały listy elementów w pythonowej składni, za pomocą tego fragmentu skonwertowalismy je na listy R'owe.

Dodalismy też kolumne positive percent, zawierająca procent pozytywnych recenzji którą faktycznie widzimy w sklepie.

**Ograniczenie** - Dużo gier wgl nie ma żadnych opinii. Jest ich **80590**.

## ANALIZA OPISOWA

Na początek przygotowałem krótki wgłąd w dane.

**Estimated_ownership - szacowana liczba użytkowników posiadających grę.**

Widzymy rozkład liczby gier

Dokładność klasyfikacji do grup nie wynosi 100% — jest to ograniczenie metody Steam Spy. Akceptujemy ten fakt ze względu na brak lepszej alternatywy.

## Regresja liniowa czasu

Chiałem dopasować funkcje regresji do tych danych. Regresją wzgłędem year

Przeanalizowano gry wydane w okresie **2011.5 – 2025.5**, pogrupowane według półroczy.

Model dopasowano w przestrzeni **log-log** przy użyciu wielomianu kwadratowego.

Taka własnie transformacja była w stanie dać sensowną predykcję.

**Czemu od 2011.** Lepsze dopasowanie, zaczął się szybki wzrost liczby gier.

## **LINE assumptions:**

**Liniowość.** Wykres „*Residuals vs Fitted*” pokazuje, że czerwona linia przebiega blisko zera w większości zakresu dopasowania, bez wyraźnego wygięcia, co sugeruje, że kwadratowa transformacja log-log odpowiednio oddaje trend. Łagodne nachylenie w dół na prawym skraju wskazuje na nieznaczne przeszacowanie w ostatnich okresach.

**Niezależność**. Obserwacje to zagregowane dane półroczne z jednej rozwijającej się platformy — kolejne okresy są prawdopodobnie dodatnio skorelowane (co roku dołącza więcej programistów). Nie można twierdzić o formalnej niezależności; model szeregów czasowych byłby bardziej odpowiedni, gdyby istniały obawy dotyczące korelacji szeregowej.

**Normalność**. Wykres Q-Q pokazuje, że większość punktów dobrze pokrywa się z przekątną w środkowym zakresie. Punkty 1 i 4 odbiegały zauważalnie na obu ogonach, co wskazuje na ogony cięższe niż normalnie. Przy n = 29 jest to umiarkowany powód do niepokoju, ale nie ma to krytycznego znaczenia dla wnioskowania, biorąc pod uwagę dużą statystykę F.

**Równa wariancja** (homoscedastyczność). Wskaźnik skali i położenia wykazuje tendencję spadkową — rozrzut reszt jest większy dla małych wartości dopasowanych (wczesne okresy) i zmniejsza się w ostatnich latach. Jest to łagodne naruszenie homoscedastyczności. Punkt 4 (widoczny we wszystkich czterech panelach) jest głównym czynnikiem.

**Point 1** (identifiable via `train_half[4, ]`) appears as an influential outlier across all diagnostic plots and sits beyond Cook's distance boundary in *Residuals vs Leverage*. It corresponds to an early period with anomalously low releases relative to the fitted trend and meaningfully affects the coefficient estimates.

**Współczynniki i dopasowanie modelu:**

Wszystkie trzy współczynniki są wysoce istotne: stała (p \< 2e-16), liniowy człon logarytmiczny log_t (p = 5,35e-06) oraz człon kwadratowy log_t² (p = 5e-04). Statystyka F wynosząca 328,5 przy 2 i 26 stopniach swobody potwierdza, że model jako całość wyjaśnia dane znacznie lepiej niż model bazowy zawierający jedynie punkt przecięcia.

R² = 0,962 (skorygowane 0,959) oznacza, że model wyjaśnia \~96% wariancji w logarytmicznie przekształconych liczbach gier — jest to dobre dopasowanie do rzeczywistych danych liczbowych. Standardowy błąd resztowy wynoszący 0,28 w skali logarytmicznej przekłada się na typowy błąd multiplikatywny wynoszący w przybliżeniu e0,28≈1,32e\^{0,28} \\approx 1,32 e0,28≈1,32, tzn. prognozy są niedokładne o około ±32% w skali pierwotnej.

**Ogólny wniosek:** model dobrze pasuje, a wszystkie zmienne prognostyczne są istotne, ale dwa założenia — niezależność i homoscedastyczność — nie są w pełni spełnione.

**Wyniki należy interpretować raczej jako opisowy model trendu, a nie jako formalny model wnioskowania.**

# Macierz korelacji, druga regresja

**Wybór zmiennej objaśnianej**

Jako miarę popularności gry przyjęto szczytową liczbę jednoczesnych graczy (`Peak_CCU`). Zmienna ta bezpośrednio odzwierciedla zainteresowanie tytułem w czasie rzeczywistym i jest obiektywnym wskaźnikiem sukcesu komercyjnego, niezależnym od deklaracji wydawcy.

**Wybór zmiennych objaśniających**

Dobór zmiennych ilościowych oparto na macierzy korelacji wyznaczonej dla danych przekształconych logarithmicznie (`log1p`). Spośród analizowanych zmiennych najsilniejszą korelację z `log(Peak_CCU)` wykazały:

-   

-   `log(Median_playtime_forever)` — korelacja 0.560 — najsilniejsza zależność w całej macierzy; gry zatrzymujące graczy na dłużej naturalnie generują wyższy jednoczesny ruch

-   

-   `log(Price)` — korelacja 0.242 — umiarkowana dodatnia zależność, prawdopodobnie odzwierciedlająca wyższą jakość produkcji droższych tytułów (AAA)

-   Logarytmiczne przekształcenie obu zmiennych było konieczne ze względu na silnie prawoskośne rozkłady — widoczne na przekątnej macierzy korelacji — oraz obecność ekstremalnych obserwacji odstających.

**Zmienna jakościowa `Primary_tag`**

Gatunek gry włączono do modelu jako zmienną jakościową typu `factor`, kodowaną automatycznie przez R jako zestaw zmiennych zero-jedynkowych (dummy variables) względem poziomu bazowego `FPS`.

### Wyznaczanie gatunku Primary_tag

#### Źródła danych

W zbiorze dostępne są dwie kolumny opisujące gatunek gry:

-   **`Genres`** — oficjalna klasyfikacja Steam, bardzo ogólna (33 unikalne wartości, np. `Action`, `RPG`, `Simulation`)

-   **`Tags`** — tagi przypisywane przez społeczność graczy, znacznie bardziej szczegółowe (450+ unikalnych wartości, np. `Rogue-like`, `Bullet Hell`, `Walking Simulator`

-   Obie kolumny są **list-kolumnami** — każda gra może mieć wiele wartości jednocześnie.

#### Algorytm przypisania gatunku

Gatunek przypisywany jest metodą **priorytetową w dwóch krokach**:

1.  **Tags (główne źródło)** — przeszukujemy listę tagów gry i wybieramy pierwszy pasujący tag z `priority_tags`

2.  **Genres (fallback)** — jeśli żaden tag nie pasuje, szukamy w liście gatunków wg tego samego porządku priorytetów

Lista `priority_tags` zawiera 30 gatunków ułożonych od najbardziej specyficznych do najbardziej ogólnych — np. `Action-Adventure` przed `Action`, `FPS` przed `Shooter`. Dzięki temu bardziej precyzyjny gatunek zawsze wygrywa z ogólnym. Pokrycie wyniosło **93.1%** — 6.9% gier nie pasowało do żadnej kategorii i otrzymało `NA`.

#### Transformacja na factor

```         
Primary_tag = factor(Primary_tag, levels = priority_tags)
```

Zmienna `Primary_tag` jest przekształcana w **factor** z poziomami (`levels`) w kolejności `priority_tags`. Ma to dwa skutki:

-   **Porządek** — poziomy są ustalone, co kontroluje kolejność na wykresach

-   **Kodowanie dummy** — gdy `Primary_tag` trafia do `lm()`, R automatycznie tworzy **k−1 zmiennych zero-jedynkowych** względem poziomu bazowego (reference level)

Poziom bazowy wybierany jest przez `relevel()`:

```         
Primary_tag = relevel(Primary_tag, ref = "RPG"
```

W modelu regresji oznacza to, że intercept odpowiada grze gatunku **RPG**, a każdy współczynnik `γ_k` wyraża różnicę w `log(Peak_CCU)` względem przeciętnej gry RPG przy pozostałych zmiennych stałych.

Dodatkowo wprowadzono zmienną binarną `is_free` odróżniającą gry bezpłatne od płatnych, ponieważ model dystrybucji Free-to-Play reprezentuje odrębną strategię biznesową, której nie można sprowadzić wyłącznie do ceny równej zero.

**Uzasadnienie transformacji logarytmicznej**

Zastosowanie logarytmu po obu stronach równania nadaje modelowi interpretację **elastyczności**: współczynnik β1=0.457\beta\_1 = 0.457 β1​=0.457 oznacza, że 10-krotny wzrost mediany czasu gry wiąże się ze wzrostem Peak CCU o czynnik 100.457≈2.910\^{0.457} \approx 2.9 100.457≈2.9, niezależnie od poziomu wyjściowego. Transformacja ta jednocześnie stabilizuje wariancję reszt i zbliża ich rozkład do normalnego — co potwierdzają wykresy diagnostyczne.

### Ocena modelu regresji log-liniowej

#### Założenia LINE

**Liniowość (Linearity)** Wykres *Residuals vs Fitted* ujawnia wyraźny problem: reszty nie są symetrycznie rozłożone wokół zera — dla małych wartości dopasowanych dominują reszty dodatnie, dla dużych ujemne. Charakterystyczny kształt "trójkąta" wskazuje na **strukturalną asymetrię**, której model nie jest w stanie uchwycić. Liniowość jest spełniona jedynie w przybliżeniu.

**Niezależność (Independence)** Obserwacje to pojedyncze gry — brak oczywistej struktury czasowej czy przestrzennej w danych regresji. Założenie niezależności jest **prawdopodobnie spełnione**, choć gry tego samego wydawcy lub serii mogą być ze sobą skorelowane.

**Normalność reszt (Normality)** Wykres Q-Q pokazuje poważne odchylenia od normalności — **prawy ogon jest ekstremalnie ciężki** (standaryzowane reszty sięgają 5–6 zamiast oczekiwanych \~3). Kilka obserwacji (11566, 7828, 3040) to megahity pokroju CS:GO czy Dota 2, których popularność jest nieporównywalna z resztą rynku. Założenie normalności jest **wyraźnie naruszone**.

**Stałość wariancji (Equal Variance)** Wykres *Scale-Location* pokazuje rosnący trend czerwonej linii — rozrzut reszt rośnie wraz z wartościami dopasowanymi. To klasyczna **heteroskedastyczność**: model gorzej przewiduje gry bardzo popularne niż niszowe. Założenie jest **naruszone**.

------------------------------------------------------------------------

#### Obserwacje wpływowe

Wykres *Residuals vs Leverage* pokazuje kolumnową strukturę punktów — efekt zmiennej `Primary_tag` jako faktora z wieloma poziomami. Linie Cook's distance nie są widoczne w granicach wykresu, co oznacza że **żadna pojedyncza obserwacja nie dominuje** nad współczynnikami w sposób krytyczny. To dobra wiadomość przy n = 13 594.

------------------------------------------------------------------------

#### Ocena kluczowych wartości

**R² = 0.200** — model wyjaśnia jedynie **20% wariancji** `log(Peak_CCU)`. Jest to wynik skromny, ale uczciwy: popularność gry zależy w dużej mierze od czynników nieobecnych w modelu — budżetu marketingowego, streamerów, momentu premiery, marki wydawcy. Adjusted R² = 0.198 potwierdza że dodatkowe zmienne (żanry) wnoszą realną, choć niewielką informację.

**Residual standard error = 1.873** w skali logarytmicznej oznacza typowy błąd predykcji rzędu e1.873≈6.5×e\^{1.873} \approx 6.5\\times e1.873≈6.5× — model może się mylić o czynnik 6 w górę lub w dół. Na rynku gier to ogromna niepewność

**F-statistic = 113.1, p \< 2.2e-16** — model jako całość jest wysoce istotny statystycznie. Przy n = 13 594 nawet słabe efekty będą znaczące.

------------------------------------------------------------------------

#### Od czego zależy Peak CCU najbardziej?

Patrząc na współczynniki, największy wpływ mają:

| Zmienna | Współczynnik | Interpretacja |
|----|----|----|
| `is_freeFree` | **+2.167**\* | Gry F2P mają \~8.7× wyższy Peak CCU niż płatne przy pozostałych czynnikach stałych — najsilniejszy efekt w modelu |
| `log_Price` | +0.544\*\*\* | Elastyczność ceny: droższe gry przyciągają więcej graczy jednocześnie |
| `log_Med_Playtime` | +0.451\*\*\* | Gry angażujące na dłużej mają wyższy CCU |
| `VR` | -1.619\*\*\* | Największy negatywny efekt gatunku — niszowa baza sprzętowa |
| `Visual Novel` | -0.875\*\*\* | Silnie niszowy gatunek |
| `FPS` + `Survival` | \~+0.55\*\*\* | Gatunki z najwyższym CCU względem bazy |

------------------------------------------------------------------------

#### Wniosek

Model jest **statystycznie istotny i poprawnie skonstruowany**, jednak jego zdolność predykcyjna jest ograniczona. Główne problemy to heteroskedastyczność i nienormalność reszt, napędzane przez kilka megahitów z ekstremalnymi wartościami CCU. Wyniki należy interpretować jako **opis przeciętnych tendencji rynkowych**, nie jako narzędzie do przewidywania sukcesu konkretnej gry. Dla lepszego modelu należałoby uwzględnić dane marketingowe, datę premiery względem konkurencji czy przynależność do uznanej serii — informacje niedostępne w tym zbiorze danych.

### Wilcoxon rank-sum test (Peak CCU vs Type)

Wynik testu wskazuje na istotną statystycznie różnicę w medianie Peak_CCU pomiędzy typami gier (p \< 0.001). Oznacza to, że gry singleplayer i multiplayer różnią się poziomem maksymalnej liczby jednoczesnych graczy.

------------------------------------------------------------------------

### Wilcoxon rank-sum test (Median playtime vs Type)

Nie stwierdzono istotnej statystycznie różnicy w medianie czasu gry pomiędzy typami (p = 0.2995). Sugeruje to, że typ gry (singleplayer/multiplayer) nie wpływa istotnie na przeciętny czas gry.

------------------------------------------------------------------------

### Test chi-kwadrat (Estimated owners vs Type)

Wykazano istotną zależność pomiędzy typem gry a kategorią liczby właścicieli (p \< 0.001). Oznacza to, że rozkład liczby właścicieli różni się w zależności od typu gry.

## Korelacja

Wyniki wskazują na umiarkowaną dodatnią korelację między ocenami graczy a ocenami krytyków. Zarówno współczynnik Pearsona (r ≈ 0.60), jak i Spearmana (ρ ≈ 0.58) są statystycznie istotne, co oznacza, że wyższe oceny Metacritic zazwyczaj odpowiadają wyższemu odsetkowi pozytywnych recenzji graczy.

Nie jest to korelacja idealna, ewidentnie gracze i krytycy nie zawsze są zgodni.
