# Disclaimer

The author of this document is not affiliated with ZTM (Poznań Transit Authority) in any way, and the information used to write it has been obtained by analysis of the code used by the site http://www.peka.poznan.pl/vm and packets exchanged by the server and the client while using it.

# Request

The request is made via HTTP to the host www.peka.poznan.pl and its resource `/vm/method.vm`. The original implementation adds a GET parameter `ts` which contains the result of the JavaScript expression `new Date().getTime()`, although it does not seem to be used and is thus skipped in the following examples. The `Content-Type` header must be set to `application/x-www-form-urlencoded; charset=UTF-8`, otherwise characters outside the ASCII range will not work correctly.

All characters are encoded as UTF-8.

The `POST` data in the request tell the server which function it should call. The name of the function is located in the `method` parameter, while the function's parameters are sent JSON-encoded inside the `p0` parameter. The original implementation additionally passes the contents of `p0` via `Object.toJSON`, but the server does not seem to care about this.

It looks like the server always returns 200 OK as the status code, even if a nonexistent function is called : in this case, the result of the request is an empty JSON object.

Specifying incorrect parameters to a function ends up in the server returning a JSON object with the error message in the `failure` attribute, for example :
```javascript
{  
   "failure":"error: java.lang.reflect.InvocationTargetException"
}
```

A correct function call results in an object containing a object with a `success` attribute containing an object, whose meaning depends on the function called.

The examples use the function `peka_vm_get`, whose definition using Bash and curl looks as follows :
```bash
peka_vm_get()
{
  curl -H 'Content-Type:application/x-www-form-urlencoded; charset=UTF-8' \
    http://www.peka.poznan.pl/vm/method.vm \
    -d "method=$1" \
    -d "p0=$2"
}
```

If the input parameter of a function is described as a "pattern", it should be understood as a search with glob-stars on both sides of the pattern string. The search is case insensitive. The examples should show this behaviour.

# Glossary
* *bollard* - this is a physical place where vehicles stop. One stop can consist of more than one bollard : one for vehicles driving in one side of the road, and a second one for the other side. This can be further augmented with separate places for night lines or trams. For example, the AWF stop has seven bollards : two tram bollards on Królowej Jadwigi Street, one tram bollard on Garbary Street, and bus bollards in various places.

# Functions
## `getStopPoints`
Fetches a list of stops whose names match the given pattern.
### Input
* `pattern` - the pattern to look for.

### Output
An array containing objects, which contain :
* `symbol` - the stop identifier to be used in later requests,
* `name` - full name of the stop.

### Example
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
Looks unused. Fetches all bollards with an empty object sent in the request.

## `getBollardsByStopPoint`
Returns the list of bollards for a given stop, and a list of lines that service this stop.

### Input
* `name` : full name of the stop, as returned by `getStopPoints`

### Output
A `bollards` object, which is an array of `bollard` objects (described in `getTimes`), and `directions`, which is an array of objects containing :
* `returnVariant` : some boolean, always false from what I've seen.
* `lineName` : name of the line departing from the given point,
* `direction` : the direction (or, rather, terminus) of the given line.

### Example
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
Returns the list of bollards at a given street.

### Input
* `name` : the name of the street

### Output
The same as `getBollardsByStopPoint`.

### Example
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
Returns bollards where a given line stops, including route variations due to leaving/coming back to the depot.

### Input
* `name` : name of the line

### Output
A `directions` object, which is an array of objects containing `direction` and `bollards`. `direction` contains :
* `returnVariant` - `true` for "returning" variants, but I find it hard to figure out what its real meaning is.
* `direction` - direction (terminus)
* `lineName` - line name.

`bollards` is an array of objects containing all bollards, where a given line's variant stops. Every one of these objects has the same content as the `bollard` object returned by `getBollardsByStopPoint`, but with an additional `orderNo` integer, which is - I guess - the consecutive number of a stop for a given line's variant. This attribute can probably be used to reconstruct the route of the variant, but I haven't found an actual algorithm for this.

### Example
Skipped due to a rather large size.

## `getLines`
Returns lines whose name match the given pattern.

### Input
* `pattern`

### Output
An array of objects containing a `name`, which is the full name of the line.

### Example
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
Returns a list of streets along with their identifiers matching the given pattern.

### Input
* `pattern`

### Output
An array of objects containing the `id` of the street which can be used in later requests and `name`, which is their full name.

### Example
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
Returns the estimated arrival times of vehicles at a given bollard.

### Input
* `symbol` : the `tag` of a bollard (unfortunately not its `symbol`, as one might think) obtained by one of the `getBollards` functions, for which to fetch a list of arrival times.

### Output
Two objects : `bollard` describing the bollard and containing :
* `symbol` - the bollard's identifier in the system,
* `tag` - alternative identifier?,
* `name` - name of the stop that this bollard belongs to,
* `mainBollard` - I've only seen `false` here, but I guess it could be `true` for "one-bollard" stops.

And a `times` object being an array of objects containing :
* `realTime` - whether the arrival time has been estimated based on the device's location,
* `minutes` - how many minutes left until the vehicle's arrival,
* `direction` - the route's terminus,
* `onStopPoint` - whether the vehicle is currently at the bollard,
* `departure` - the estimated time of departure given as `yyyy-MM-dd'T'HH:mm:ss.SSS'Z'`. This is **not** ISO 8601 despite the similarity, since if it were, the trailing `Z` would suggest that the time is in UTC. This is not the case : the time is given in local time, which is either CET or CEST,
* `line` - line served by the vehicle.

### Example
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
Fetches the arrival times for all the bollards assigned to a certain stop. Essentially a mix of `getBollardsByStopPoint` and `getTimes`.

### Input
* `name` : name of the stop.

### Output
A `bollardsWithTimes` object, which is an array of objects described in `getTimes`.

### Example
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
Fetches the current time as something that looks like the number of milliseconds since 1 January 1970. Empty object on input.

### Example
```
peka_vm_get getServerTime '{}'
```
```javascript
{"success":1480110116917}
```

## `findMessagesForBollard`
Returns messages saved by the site's administration, which are connected to a given bollard. Usually used to signal temporary changes in the schedule.

### Input
* `symbol` : the bollard's identifier

### Output
An array of objects containing the following fields :
* `content` : the message to be displayed, as HTML,
* `startHour`, `stopsGroups` and `endHour` : unknown usage, don't appear used in the official implementation,
* `startDate` : the start date of the message's validity,
* `endDate` : like above, but the end date.

The date format is the same as the one used in the `departure` field in the reply of `getTimes`.

### Example
```
peka_vm_get findMessagesForBollard '{"symbol":"RJEZ04"}'
```
```javascript
{  
   "success":[  
      {  
         "content":"INFO: Od 9 kwietnia, w związku z remontem torowiska na ul. 28 Czerwca 1956 r., zmianie ulegną trasy linii tramwajowych nr 2, 9, 10 i 11. Uruchomiona zostanie komunikacja zastępcza. Szczegóły na stronie <a href=\"http://tiny.pl/g5dss\">www.ztm.poznan.pl<\/a>.",
         "startDate":"2017-04-06T01:00:00.000Z",
         "stopsGroups":[  

         ],
         "startHour":60,
         "endDate":"2017-04-11T23:00:00.000Z",
         "endHour":1380
      }
   ]
}
```
