# Ostrzeżenie

Autor tego dokumentu nie jest w jakikolwiek sposób powiązany z ZTM, a informacje służące do jego napisania zostały pozyskane przez analizę kodu wykorzystywanego przez stronę http://www.peka.poznan.pl/vm oraz pakietów wymienianych między serwerem i klientem podczas korzystania z niej.

# Zapytanie

Zapytanie wykonywane jest przez HTTP do hosta www.peka.poznan.pl i zasobu `/vm/method.vm` , z parametrem `ts` przekazywanym przez `GET` i zawierającym wynik javascriptowej `new Date().getTime()`. `Content-Type` zapytania musi być ustawione na `application/x-www-form-urlencoded; charset=UTF-8`, w przeciwnym razie znaki spoza standardowego zakresu ASCII nie będą działały poprawnie.

Wszystkie znaki kodowane są jako UTF-8.

Dane `POST` zapytania instruują serwer odnośnie funkcji, którą ma on wykonać. Nazwa funkcji mieści się w parametrze `method`, natomiast jej parametry jako JSON w `p0`. Implementacja przepuszcza jeszcze to, co idzie do `p0` przez `Object.toJSON`, ale serwer nie zdaje się zwracać na to uwagi.

Wygląda na to, że serwer zawsze zwraca 200 OK, nawet gdy wywoła się funkcję, której nie posiada : jako wynik zapytania jest wtedy zwracany pusty JSON.

Podanie niepoprawnych parametrów do funkcji kończy się zwróceniem obiektu JSON zawierającego komunikat błędu w atrybucie `failure`, np. :
```javascript
{  
   "failure":"error: java.lang.reflect.InvocationTargetException"
}
```

Poprawne wywołanie funkcji kończy się przesłaniem obiektu zawierającego w sobie obiekt `success`, którego znaczenie jest uzależnione od wywołanej funkcji. Obiekt ten jest pomijany w tym dokumencie w opisie wyjścia danej funkcji.

W przykładach wykorzystywana jest funkcja `peka_vm_get`, której definicja w Bashu wygląda następująco :
```bash
peka_vm_get()
{
  curl -H 'Content-Type:application/x-www-form-urlencoded; charset=UTF-8' \
    http://www.peka.poznan.pl/vm/method.vm?ts=$(($(date +'%s') * 1000)) \
    -d "method=$1" \
    -d "p0=$2"
}
```

Jeśli parametr wejściowy jakiejś funkcji jest "wzorcem", należy go rozumieć jak wyszukiwanie z glob-gwiazdkami po obu stronach wzorca. Wielkość liter jest ignorowana. Powinno być to widoczne w przykładach.

# Słowniczek
* *bollard* - coś, co najwygodniej chyba wytłumaczyć jako "słupek". Jest to fizyczne miejsce, w którym zatrzymują się pojazdy. Na jeden przystanek może przypadać więcej niż jeden bollard : jeden dla pojazdów jadących w jednym kierunku, a drugi - w drugim. Dochodzą do tego jeszcze osobne punkty przystankowe dla linii nocnych. Przystanek AWF ma na przykład aż siedem takich punktów : dwa tramwajowe na Królowej Jadwigi, jeden tramwajowy na Garbarach, i autobusowe rozrzucone w różnych miejscach.

# Funkcje
## `getStopPoints`
Pobiera listę przystanków o nazwie pasującej do zadanego wzorca.
### Wejście
* `pattern` - wzorzec, według którego serwer ma szukać przystanku.

### Wyjście
Tablica zawierająca obiekty, w których mamy :
* `symbol` - identyfikator przystanku używany do późniejszych zapytań
* `name` - pełna nazwa przystanku.

### Przykład
```
peka_vm_get getStopPoints '{"pattern":"Pół"}'
```
```javascript
{  
   "success":[  
      {  
         "symbol":"BOZSZ",
         "name":"Bolechowo-Os./Zespół Szkół"
      },
      {  
         "symbol":"PRPLN",
         "name":"Promnice/Północna"
      },
      {  
         "symbol":"POLW",
         "name":"Półwiejska"
      }
   ]
}
```

## `getBollards`
Chyba nieużywana. Dla pustego obiektu przesłanego w zapytaniu pobiera wszystkie bollardy.

## `getBollardsByStopPoint`
Zwraca listę bollardów dla danego przystanku i listę linii odjeżdżających z niego.

### Wejście
* `name` : nazwa przystanku, zwrócona w `getStopPoints`

### Wyjście
Obiekt `bollards`, będący tablicą obiektów `bollard` (opisanym w `getTimes`) i `directions`, który sam jest tablicą obiektów zawierających :
* `returnVariant` : jakiś bool, z tego co widziałem prawie zawsze false.
* `lineName` : nazwa linii odjeżdżającej z danego punktu,
* `direction` : kierunek tejże linii.

### Przykład
```
peka_vm_get getBollardsByStopPoint '{"name":"Termy Maltańskie"}'
```
```javascript
{  
   "success":{  
      "bollards":[  
         {  
            "directions":[  
               {  
                  "returnVariant":false,
                  "direction":"Rondo Rataje",
                  "lineName":"84"
               },
               {  
                  "returnVariant":false,
                  "direction":"Termy Maltańskie",
                  "lineName":"84"
               },
               {  
                  "returnVariant":false,
                  "direction":"Rondo Śródka",
                  "lineName":"84"
               },
               {  
                  "returnVariant":false,
                  "direction":"Nowe ZOO",
                  "lineName":"84"
               }
            ],
            "bollard":{  
               "symbol":"TEMA22",
               "tag":"TEMA01",
               "name":"Termy Maltańskie",
               "mainBollard":false
            }
         }
      ]
   }
}
```

## `getBollardsByStreet`
Zwraca listę bollardów znajdujących się przy zadanej ulicy.

### Wejście
* `name` : nazwa ulicy

### Wyjście
To samo, co przy `getBollardsByStopPoint`.

### Przykład
```
peka_vm_get getBollardsByStreet '{"name":"Lutycka"}'
```
```javascript
{  
   "success":{  
      "bollards":[  
         {  
            "directions":[  
               {  
                  "returnVariant":false,
                  "direction":"Szarych Szeregów",
                  "lineName":"83"
               }
            ],
            "bollard":{  
               "symbol":"LU I21",
               "tag":"LU I01",
               "name":"Lutycka I n/ż",
               "mainBollard":false
            }
         },
         {  
            "directions":[  
               {  
                  "returnVariant":false,
                  "direction":"Szarych Szeregów",
                  "lineName":"83"
               }
            ],
            "bollard":{  
               "symbol":"LUII21",
               "tag":"LUII01",
               "name":"Lutycka II n/ż",
               "mainBollard":false
            }
         },
         {  
            "directions":[  
               {  
                  "returnVariant":false,
                  "direction":"Rondo Śródka",
                  "lineName":"83"
               }
            ],
            "bollard":{  
               "symbol":"LUII22",
               "tag":"LUII02",
               "name":"Lutycka II n/ż",
               "mainBollard":false
            }
         },
         {  
            "directions":[  
               {  
                  "returnVariant":false,
                  "direction":"Rondo Śródka",
                  "lineName":"83"
               }
            ],
            "bollard":{  
               "symbol":"LU I22",
               "tag":"LU I02",
               "name":"Lutycka I n/ż",
               "mainBollard":false
            }
         },
         {  
            "directions":[  
               {  
                  "returnVariant":false,
                  "direction":"Poznań Główny",
                  "lineName":"236"
               }
            ],
            "bollard":{  
               "symbol":"ROOB22",
               "tag":"ROOB02",
               "name":"Rondo Obornicka",
               "mainBollard":false
            }
         },
         {  
            "directions":[  
               {  
                  "returnVariant":false,
                  "direction":"Poznań Główny",
                  "lineName":"246"
               }
            ],
            "bollard":{  
               "symbol":"ROOB21",
               "tag":"ROOB01",
               "name":"Rondo Obornicka",
               "mainBollard":false
            }
         }
      ]
   }
}
```

## `getBollardsByLine`
Zwraca bollardy, przy których zatrzymuje się dana linia, włącznie z jej wariacjami wynikającymi z wyjazdami/zjazdami z/do zajezdni.

### Wejście
* `name` : nazwa linii

### Wyjście
Obiekt `directions`, będący tablicą obiektów zawierających w sobie obiekt `direction` i `bollards`. `direction` zawiera :
* `returnVariant` - chyba `true` dla "powracającego", chociaż trudno mi się domyślić, jakie to ma naprawdę znaczenie.
* `direction` - kierunek jazdy
* `lineName` - nazwę linii.

`bollards` to natomiast tablica obiektów zawierających wszystkie bollardy, przy których zatrzymuje się dana wariacja linii. Każdy z tych obiektów zawiera to samo, co `bollard` znajdujący się w wyniku `getBollardsByStopPoint`, ale z dodanym `orderNo`, czyli - chyba - kolejny numer przystanku dla linii. Atrybut ten pewnie być użyty do zrekonstruowania trasy danej linii i jej wariacji, ale nie udało mi się znaleźć na to jakiegoś algorytmu.

### Przykład
Pominięty ze względu na zbyt duży rozmiar.

## `getLines`
Pobiera linie komunikacji pasujące do zadanego wzorca.

### Wejście
* `pattern`

### Wyjście
Tablica zawierająca obiekty, których `name` zawiera pełną nazwę linii.

### Przykład
```
peka_vm_get getLines '{"pattern":"16"}'
```
```javascript
{  
   "success":[  
      {  
         "name":"16"
      },
      {  
         "name":"616"
      },
      {  
         "name":"716"
      }
   ]
}
```
## `getStreets`
Pobiera listę ulic razem z ich identyfikatorami, pasującymi do zadanego wzorca.

### Wejście
* `pattern` : wzorzec.

### Wyjście
Tablica obiektów zawierających `id` ulicy do użycia w późniejszych zapytaniach, oraz `name`, czyli ich pełną nazwę.

### Przykład
```
peka_vm_get getStreets '{"pattern":"Gło"}'
```
```javascript
{  
   "success":[  
      {  
         "id":3,
         "name":"Głogowska"
      },
      {  
         "id":159,
         "name":"Koziegłowy/Gdyńska"
      },
      {  
         "id":270,
         "name":"Koziegłowy/Piaskowa"
      },
      {  
         "id":167,
         "name":"Koziegłowy/Piłsudskiego"
      },
      {  
         "id":166,
         "name":"Koziegłowy/Poznańska"
      },
      {  
         "id":165,
         "name":"Koziegłowy/Taczaka"
      },
      {  
         "id":319,
         "name":"Kórnik/pl. Niepodległości"
      },
      {  
         "id":277,
         "name":"Luboń/Niepodległości"
      },
      {  
         "id":143,
         "name":"al. Niepodległości"
      },
      {  
         "id":6,
         "name":"zajezdnia Głogowska"
      }
   ]
}
```

## `getTimes`
Pobiera szacowane czasy przyjazdu kolejnych pojazdów na zadany bollard.

### Wejście
* `symbol` : identyfikator bollarda, dla którego pobrać listę przyjazdów. Musi to być identyfikator pobrany z jednej z funkcji `getBollards`, w przeciwnym razie wywołanie kończy się błędem.

### Wyjście
Dwa obiekty : `bollard` opisujący bollarda i zawierający :
 * `symbol` - identyfikator bollarda w systemie,
 *  `tag` - alternatywny identyfikator?,
 *  `name` - nazwę przystanku, do którego należy bollard,
 *  `mainBollard` - na razie widziałem tutaj tylko `false`, ale domyślam się, że może to być `true` dla "jednobollardowych" przystanków.

Oraz obiekt `times` będący tablicą obiektów zawierających :
* `realTime` - czy przyjazd jest podany w oparciu o położenie pojazdu na podstawie odczytu jego lokalizacji,
* `minutes` - ile minut zostało do przyjazdu,
* `direction` - kierunek trasy,
* `onStopPoint` - czy pojazd aktualnie znajduje się na bollardzie,
* `departure` - szacowany czas odjazdu podany jako `yyyy-MM-dd'T'HH:mm:ss.SSS'Z'`. **Nie** jest to ISO 8601, ponieważ końcowe `Z` sugerowałoby wtedy, że jest on podany w UTC. Tak się jednak nie dzieje : godzina podawana jest w czasie lokalnym , czyli CET albo CEST.
* `line` - linia, którą obsługuje pojazd.

### Przykład
```
peka_vm_get getTimes '{"symbol":"AWF03"}'
```
```javascript
{  
   "success":{  
      "bollard":{  
         "symbol":"AWF03",
         "tag":"AWF21",
         "name":"AWF",
         "mainBollard":false
      },
      "times":[  
         {  
            "realTime":true,
            "minutes":0,
            "direction":"Os. Orła Białego",
            "onStopPoint":true,
            "departure":"2016-11-25T21:48:00.000Z",
            "line":"74"
         },
         {  
            "realTime":true,
            "minutes":20,
            "direction":"Os. Orła Białego",
            "onStopPoint":false,
            "departure":"2016-11-25T22:08:00.000Z",
            "line":"74"
         }
      ]
   }
}
```

## `getTimesForAllBollards`
Pobiera czasy przyjazdu na wszystkich bollardach przypisanych do danego przystanku. Swoiste połączenie `getBollardsByStopPoint` i `getTimes`.

### Wejście
* `name` : nazwa przystanku.

### Wyjście
Obiekt `bollardsWithTimes`, będący tablicą struktur opisanych w `getTimes`.

### Przykład
```
peka_vm_get getTimesForAllBollards '{"name":"Katowicka"}'
```
```javascript
{  
   "success":{  
      "bollardsWithTimes":[  
         {  
            "bollard":{  
               "symbol":"KATO22",
               "tag":"KATO02",
               "name":"Katowicka",
               "mainBollard":false
            },
            "times":[  
               {  
                  "realTime":true,
                  "minutes":8,
                  "direction":"Rondo Śródka",
                  "onStopPoint":false,
                  "departure":"2016-11-25T22:48:00.000Z",
                  "line":"57"
               },
               {  
                  "realTime":true,
                  "minutes":13,
                  "direction":"Termy Maltańskie",
                  "onStopPoint":false,
                  "departure":"2016-11-25T22:53:00.000Z",
                  "line":"84"
               }
            ]
         },
         {  
            "bollard":{  
               "symbol":"KATO21",
               "tag":"KATO01",
               "name":"Katowicka",
               "mainBollard":false
            },
            "times":[  
               {  
                  "realTime":false,
                  "minutes":13,
                  "direction":"Rondo Rataje",
                  "onStopPoint":false,
                  "departure":"2016-11-25T22:53:00.000Z",
                  "line":"84"
               },
               {  
                  "realTime":false,
                  "minutes":24,
                  "direction":"Mogileńska",
                  "onStopPoint":false,
                  "departure":"2016-11-25T23:04:00.000Z",
                  "line":"57"
               }
            ]
         }
      ]
   }
}
```

## `getServerTime`
Pobiera aktualny czas na serwerze jako coś, co wygląda na liczbę milisekund od 1. stycznia 1970. Na wejściu pusty obiekt.

### Przykład
```
peka_vm_get getServerTime '{}'
```
```javascript
{"success":1480110116917}
```

## `findMessagesForBollard`
Zwraca wiadomości zapisane przez administrację serwisu, skojarzone z danym bollardem. Z reguły wykorzystywane do zakomunikowania czasowych zmian w rozkładzie. Niestety nie udało mi się znaleźć żadnego przystanku, na którym byłyby one aktywne, więc nie znam formatu wyjścia : dla wszystkich dostaję pustą tablicę.

### Wejście
* `symbol` : identyfikator bollarda

### Wyjście
Najpewniej tablica wiadomości, może zapakowana w jakiś obiekt.

### Przykład
```
peka_vm_get findMessagesForBollard '{"symbol":"RJEZ04"}'
```
```javascript
{"success":[]}
```
