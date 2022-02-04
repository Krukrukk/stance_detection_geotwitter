# Stance detection - vaccination in Poland - geotwitter

## 1. Downloading data 
For this purpose, we used 2 scraped data sources: 
- stweet;
- Twitter API (tweepy).

From the 'stweet' we downloaded all tweets that would allow us to define the stance for vaccinations among Poles.

From the Twitter API, we collected information about users who liked tweets.

The process of retrieving this data was initiated during the implementation of the AMC task.

## 2. Preprocessing data

From the raw data, we created a table per user with his attitude towards vaccination 






# Regresja

## Opis ogólny

Naszym głównym zbiorem danych jest zbiór Tweetów z projektu *#SzczepimySię - zwolennicy vs przeciwnicy* z przedmiotu analiza mediów cyfrowych). Zbiór ten zawiera ~36.000 wpisów oraz interakcje użytkowników Twittera z okresu grudzień 2020 - grudzień 2021. Zbiór został przeanalizowany pod względem nastawienia polaków co do szczepionek przeciwko COVID-19, wraz z uzyskanymi danymi geograficznymi użytkowników. Do dalszych analiz postanowiliśmy dołączyć [zbiór](https://dane.gov.pl/pl/dataset/2476/resource/36006,raport-o-liczbie-mieszkancow-zaszczepionych-pierwsza-dawka-oraz-w-peni-zaszczepionych-w-miastach-powiatach-i-gminach-w-dniu-2022-01-23/table?page=1&per_page=20&q=&sort=) udostępniany przez projekt *Otwarte dane* rządu polskiego o statystykach wyszczepień przeciwko COVID-19 (stan na dzień 23.01.2022).

Postanowiliśmy sprawdzić czy istnieje korelacja między nastawieniem polaków co do szczepionek w powiatach oraz poziomem wyszczepienia w poszczególnych powiatach kraju. W tym celu użyliśmy metod regresji w danych przestrzennych poznanych na zajęciach. Wybraną podstawową jednostą korelacji w celu analizy postępu i opisu została miara R-kwadrat (pełne statystyki dostępne w notebooku).

## Wyniki bazowe

Pierwszym krokiem było wykonanie regresji bez specjalnego uwzględniania danych geograficznych w celu uzyskania podstawowych metryk i mierzenia progresu. W tym oba zbiory zostały połączone, a następnie wyektrahowane zostały interesujące nas cechy. W celu ścisłości postanowiliśmy przebadać wyłącznie pełny poziom wyszczepienia. Zbiór danych na którym pracowaliśmy wyglądał następująco:

![Zbiór danych do regresji](./figure/Data_regresja.png)

gdzie `y3classes_sum` to suma z pozytywnych i negatywnych użytkowników w danym powiecie, a `czesc_wyszczepienia` to informacja o odsetku wyszczepień danego powiatu. W celu przeprowadzeniu regresji zostały wybrane 3 cechy zbioru [`y3classes_sum`, `powiat_teryt`, `liczba_ludnosci`], a jako zmienną zależną wspomniana wcześniej `czesc_wyszczepienia`.


Po przeprowadzeniu podstawowej regresji używając modelu **OLS** związek między danymi wykazywał miarę R-squared na poziomie `0.1497`, w przypadku czego nie można mówić o występowaniu jakiejkolwiek zależności.

## Sprawdzenie zależności geograficznej

Następnym krokiem było przeprowadzenie testów, czy w danych zawierają się jakieś dodatkowe zależności geograficzne, które mogą zostać wykorzystane do uzyskania lepszego wnioskowania na zbiorze. W tym celu wyznaczyliśmy zależność między *residuals* modelu a ich *spacial lag* (używając algorytmu KNN z ilością sąsiadów ustawioną na `1`):

![Zależność rezudyali modelu 1](./figure/Residuals_regression_model_1.png)

!!!!!!!! TODO: Co to oznacza? Analiza tego wykresu, chociaż w jednym zdaniu...

Następnie w celu wyznaczenia autokorelacji przestrzennej zwizualizowaliśmy również największych outlierów używając statystyki lokalnej Morana między modelem a algorytmem KNN dla ilości sąsiadów równej `20`:

![Mapa outlierów wg Morana](./figure/Moran_regression_model_1.png)

To nam jednoznacznie ukazało, że w zbiorze można wyznaczyć lepszą korelację używając technik przestrzennych, gdyż wiele powiatów jest nadreprezentowanych lub niedoreprezentowanych przez obecny model.

## Testy zależności geograficznych

W ramach próby poprawy wyników regresji przetestowaliśmy kilka różnych metod włączenia zależności geograficznych do danych. Wszystkie testy były wykonywane niezależnie (każdy osobno w porównaniu do wersji bazowej).

### Czy nastrój jest zaraźliwy?

Jako pierwszą metodę opracowaliśmy mapę sąsiedztwa powiatów względem najbardziej i najmniej pozytywnie nastawionych do szczepień. Zamysłem jest, że obecność skrajnego sąsiada w okolicy może wpływać na nastawienie w danym powiecie (lub po prostu bliskość geograficzna do największej skrajności może mieć wpływ na wyniki).

W tym celu wybraliśmy po 25 najbardziej pozytywnie i negatywnie nastawionych powiatów, a następnie zliczenie dla każdego powiatu ile takich skrajnych przypadków znajduje się w sąsiedztwie wieżowym (*rook*). Jeżeli wokół danej jednostki jest więcej pozytywnych niż negatywnych to przypisujemy wartość `1` (zarażanie pozytywne), jeżeli mniej to `-1` (zarażanie negatywne), a jeżeli po równo (lub nie ma sąsiedztwa) to wartość `0`. Otrzymaliśmy w ten sposób następującą kategoryzację:

![Pozytywne i negatywne zarażanie wieżowe](./figure/Rook_pos_neg_neighborhood.png)

Następnie użyliśmy tych danych do przekazania modelowi OLS informacji o przynależności do sąsiedztwa, co pozwoliło na rozszerzenie regresji o wiele stałych dla każdego sąsiedztwa. Jesteśmy świadomi, że to sąsiedztwo nie jest doskonałe (duże różnice w lokalizacjach pomimo wspólnych cech), więc w finalnej wersji modelu dane zostały użyte w inny sposób. Jednakże nawet w ten sposób udało się podnieść wynik R-kwadrat w porównaniu z wersją bazową do `0.2246`.

### Faktyczne sąsiedztwo (aka województwo)

Następnym krokiem było przetestowanie bardziej geograficznie położonego sąsiedztwa. Naturalnie dla powiatów wybrane zostało tutaj województwo, w którym dany powiat się znajduje. Regresja została rozszerzona w ten sam sposób co w przypadku zarażania nastrojem, lecz dodatkowo pozwoliliśmy na różne współczynniki $\sigma$. Wynik otrzymany był już o wiele lepszy niż w wersji bazowej - R-kwadrat wyniosło `0.6060`.

### Dodanie *spacial lag*

Ostatnim testem było wprowadzenie *spacial lag* dla każdem zmiennej niezależnej danych. Wykorzysaliśmy do tego wyznaczony wcześniej wynik algorytmu KNN, dzięki czemu powstały w zbiorze danych kolejne 3 zmienne niezależne (dla każdej z oryginalnych). Dodanie tyh danych nieznacznie poprawiło wynik R-kwadrat do poziomu `0.1940`.

## Finale

Patrząc na efekty przedstawione wyżej postanowilismy połączyć ich siły w jeden algorytm. Finalnie testowany model (w porównaniu do bazowego) został roszerzony o:

* wykorzystanie sąsiedztwa do ektremalnych powiatów jako dodatkowej zmiennej niezależnej zbioru,
* wykorzystanie województw jako sąsiedztwa na bazie którego dostosowywane są współczynniki stałe regresji,
* dodatkowo dodanie zmiennej niezależnej mówiącej o średnim nastawieniu do szczepionki na osobę powiatu,
* rozszerzenie zbioru o wyznaczenie *spacial lag* dla wszystkich zmiennych niezależnych.

Dodatkowo doszło bardziej **inżynieryjskie** pobawienie się tym, które zmienne niezależne mają być uznane za stałe w obrębie *regime* poskutkowało uzyskaniem miary R-kwadrat fianlnego modelu na poziomie `0.7464`.
