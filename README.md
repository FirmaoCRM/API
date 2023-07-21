# Firmao - API 

Instrukcja, która pomoże zintegrować własną aplikację lub serwis z systemem [Firmao.pl](https://firmao.pl/)

API firmao pozwala na integrację dowolnego systemu zewnętrznego z oprogramowaniem Firmao.

# Przykładowe zastosowania API:
<b>Integracja z formularzem na stronie www:</b>
                                             
<ul><li>Tworzenie firmy na podstawie danych z formularza na Twojej stronie www;</li>

<li>Tworzenie zadań przypisanych do Twoich pracowników po wypełneniu formularza;</li>

<li>Integracja ze sklepem internetowym;</li>

<li>Integracja z systemem informatycznym Twojej Firmy;</li>

<li>Wyświetlanie produktów z Firmao na Twojej stronie internetowej.</li></ul>

<b>API uruchamiane jest w systemie z poziomu ustawień firmy (z zakładki API).</b>

## Spis treści
+ [1.Włączenie API i uwierzytelnienie](#token)
+ [2.Dodawanie nowych obiektów (POST)](#POST)
  + [a)Projekty](#f1)
  + [b)Zadania](#f2)
  + [c)Produkty/usługi](#f3)
  + [d)Kontrahent](#f4)
  + [e)Transakcje](#f5)
  + [f)Oferty/Zamówienia/Umowy](#f6)
  + [g)Osoby kontaktowe](#f7)
  + [h)Notatki handlowe](#f8)
+ [3.Modyfikacja obiektów (PUT)](#PUT)
	+ [a)Zmiana nazwy produktu:](#f9)
	+ [b)Zmiana nazwy kontrahenta:](#f10)
	+ [c) Zmiana stanu magazynowego produktu (wymagane włączenie w organizacji ustawienia:
Bezpośrednia modyfikacja stanów magazynowych)](#f11)
+ [4.Pobieranie obiektów (GET)](#GET)
  + [Stronicowanie](#f12)
  + [Sortowanie](#f13)
  + [Filtrowanie](#f14)
  + [Kody odpowiedzi](#f15)
  + [a)Pobranie projektów - projects](#f16)
  + [b)Pobranie zadań - tasks](#f17)
  + [c)Pobranie produktów - products](#f18)
  + [d)Pobranie kontrahentów - customers](#f19)
  + [e)Pobranie transakcji - transactions](#f20)
  + [f)Pobranie transakcji dla kontrahenta - transactions](#f21)
  + [g)Pobieranie pozycji transakcji - transactions](#f22)
  + [h)Pobranie ofert/zamówień - offers](#f23)
  + [i)Pobieranie osób kontaktowych - contacts](#f24)
  + [j)Pobieranie grup kontrahentów - customergroups](#f25)
  + [k)Pobieranie notatek handlowych - salesnotes](#f26)
  + [l)Pobieranie dokumentów magazynowych - storagedocs](#f27)
  + [m)Pobieranie listy plików - documents](#f28)
  + [n)Pobieranie maili - mails](#f29)
+ [5.Kasowanie obiektów (PUT, DELETE)](#Delete)
+ [6.Dokładny opis tworzenia, edycji, kończenia i rozpoczynania czasów pracy](#time)
+ [7.Pobieranie plików PDF](#PDF)
	+ [a)Faktury](#f30)
	+ [b)Oferty/zamówienia/umowy](#f31)
+ [8.Kody odpowiedzi](#codes)
+ [9.Przykładowe skrypty php](#PHP)
	+ [a)Dodawanie kontrahenta](#f31)
	+ [b)Pobranie danych kontrahenta](#f32)
	+ [c)Dodawanie transakcji](#f33)
	+ [d)Wgrywanie pliku php <7.0](#f34)
	+ [e)Wgrywanie pliku php >=7.0](#f35)
+ [10.Więcej opcji API](#MORE)

<a name="token"/>

## 1.Włączenie API i uwierzytelnianie

Aby API działało należy wejść do panelu ustawień Firma -> Ustawienia -> API , włączyć
api, pobrać login oraz wygenerować hasło.

Wszystkie żądania muszą posiadać uwierzytelnianie w postaci:
headera `Authorization` z wartością `Basic` + zakodowowany base64 string username
+":"+ password

przykład:
`Authorization: Basic ai5zbWl0aEBzdGVsbGlzLmJpczoxMjM0NTY=`

gdzie `ai5zbWl0aEBzdGVsbGlzLmJpczoxMjM0NTY` to zakodowany w base64 string
`“j.smith@stellis.bis:123456”`

`username` - login użytkownika API systemu firmao

`password` - hasło użytkownika API systemu firmao

<a name="POST"/>

## 2. Dodawanie nowych obiektów (POST)

Utworzenie nowych obiektów odbywa się za pomocą żądania POST i powinny być
kierowane na adres
`https://system.firmao.pl/{idenfyfikatorOrganizacji}/svc/v1/{xxx}`
`xxx` - nazwa typu obiektu
`idenfyfikatorOrganizacji` - identyfikator organizacji widoczny w urlu po zalogowaniu do
firmao

Typ danych:
ContentType = application/json; charset=UTF-8

Wszystkie dostępne w obiektach pola są opisane w pomocy do importu danych.
Pola dodatkowe mają klucze “customFields.custom5” i przyjmują Stringi. Dostępne pola
można również “podejrzeć” poprzez pobranie danych za pomocą metody GET (z
wyłączeniem pola permission, które jest wyliczane i nie można go podać w żądaniu
POST/PUT).

W odpowiedzi do żądań POST wysyłamy dane w postaci jsona z informacja o
wszystkich polach, które zostały wyliczone/zmienione w przez firmao. Między innymi o
id utworzonego obiektu `[{"modification":"CREATE" …. "objectId":"50007" }]`

Kody odpowiedzi:

API używa standardowych kodów błędów HTTP.

201 - poprawne utworzenie

4xx i 5xx - nastąpił błąd podczas tworzenia obiektu.

W przypadku błędów standardowo pojawia się zrozumiały opis błędu.

Na przykład w przypadku próby utworzenia zadania do nieistniejącego projektu
dostaniemy:

Kod odpowiedzi: 400.

`{"errorCode":"entityNotFoundForId","field":"project","rejectedValue":"50","additionalInfo":
{"id":"50","cls":"project"}}`

W polu autor będzie się pokazywał zawsze użytkownik API.

Przykładowe żądania

<a name="f1"/>

### a)Projekty

`https://system.firmao.pl/{idenfyfikatorOrganizacji}/svc/v1/projects`

```
POST
{
    "description":"Opis",
    "teamMembers":[{"id":6}],
    "restrictedTeamMembers":[]
    ,"managers":[{"id":10}]
    ,"tags":[{"id":3}],
    "startDate":"2012-07-17T00:00:00+02:00",
    "name":"Przykładowy"
}
```

Odpowiedź w przypadku poprawnego utworzenia

Kod 201:

```
{"changelog":[{"deletedWith":"","modification":"CREATE","newValueIsDictionary":false,"o
bjectClass":"task","objectId":50017,"objectLabel":"Przykładowe
zadanie","oldValueIsDictionary":false,"property":""},{"deletedWith":"","modification":"CHA
NGE","newValueClass":"date","newValueDisplay":"2012-07-17T14:00:00.000+02:00","n
ewValueIsDictionary":false,"objectClass":"task","objectId":50017,"objectLabel":"Przykład
owe
zadanie","oldValueClass":"date","oldValueDisplay":"2012-07-17T22:00:00.000Z","oldVal
ueIsDictionary":false,"property":"plannedEndDate"}]}
```

<a name="f2"/>

### b)Zadania

`https://system.firmao.pl/{idenfyfikatorOrganizacji}/svc/v1/tasks`

```
POST
{
    "taskType":"PROJECT",
    "priority":"300",
    "tags":[],
    "responsibleUsers":[{"id":2}],
    "estimatedHours":5,
    "plannedEndDateType":"EXACTLY"
    ,"plannedStartDateType":"EXACTLY"
    ,"description":"Opis",
    "plannedStartDate":"2012-07-17T00:00:00+02:00",
    "plannedEndDate":"2012-07-18T00:00:00+02:00",
    "orderCalculateFromGross":false,
    "orderVatPercent":null,
    "name":"Przykładowe zadanie",
    "status":"WAITING"
}
```
Odpowiedź w przypadku poprawnego utworzenia

Kod 201:

```
{"changelog":[{"deletedWith":"","modification":"CREATE","newValueIsDictionary":false,"o
bjectClass":"task","objectId":50017,"objectLabel":"Przykładowe
zadanie","oldValueIsDictionary":false,"property":""},{"deletedWith":"","modification":"CHA
NGE","newValueClass":"date","newValueDisplay":"2012-07-17T14:00:00.000+02:00","n
ewValueIsDictionary":false,"objectClass":"task","objectId":50017,"objectLabel":"Przykład
owe
zadanie","oldValueClass":"date","oldValueDisplay":"2012-07-17T22:00:00.000Z","oldVal
ueIsDictionary":false,"property":"plannedEndDate"}]}
```

<a name="f3"/>

### c)Produkty i usługi

`https://system.firmao.pl/{idenfyfikatorOrganizacji}/svc/v1/products`

```
POST
{
    "tags":[],
    "purchasePriceGroup.netPrice":10,
    "purchasePriceGroup.grossPrice":12.3,
    "purchasePriceGroup.vat":23,
    "saleable":true,
    "productCode":"PR1",
    "name":"Przykładowy produkt"
}
```

Odpowiedź w przypadku poprawnego utworzenia

Kod 201:

```
{"changelog":[{"deletedWith":"","modification":"CREATE","newValueIsDictionary":false,"o
bjectClass":"product","objectId":50001,"objectLabel":"Przykładowy
8
produkt","oldValueIsDictionary":false,"property":""}]}
```

<a name="f4"/>

### d)Kontrahent

`https://system.firmao.pl/{idenfyfikatorOrganizacji}/svc/v1/customers`

```
POST
{ "bankAccountNumber" : "50 1020 5558 1111 1702 0600 0067",
  "correspondenceAddress.city" : "Pruszków",
  "correspondenceAddress.country" : "Polska",
  "correspondenceAddress.county" : "",
  "correspondenceAddress.postCode" : "05-800",
  "correspondenceAddress.street" : "Stefana Batorego 3/4",
  "customerGroups" : [ ],
  "customerType" : "PARTNER",
  "description" : "Firma specjalizująca się w serwisie komputerów",
  "emails" : [ "kompservice.firmao@mailinator.com",
      "komputery@mailinator.com"
      ],
  "employeesNumber" : 1,
  "faxNumber" : "22 398 78 87",
  "label" : "KOMP-SERVICE",
  "name" : "KOMP-SERVICE Jan Kowalski",
  "nipNumber" : "7291077843",
  "officeAddress.city" : "Pruszków",
  "officeAddress.country" : "Polska",
  "officeAddress.postCode" : "05-800",
  "officeAddress.street" : "Stefana Batorego 3/4",
  "ownership" : "PRIVATE",
  "phones" : [ "223457426" ],
  "status" : null,
  "tags" : [ ],
  "voivodeship" : null,
  "website" : "www.kompservice.bis"
}
```

Odpowiedź w przypadku poprawnego utworzenia

Kod 201:

`{"changelog":[{"deletedWith":"","modification":"CREATE","newValueIsDictionary":false,"o
bjectClass":"customer","objectId":50001,"objectLabel":"KOMP-SERVICE","oldValueIsDi
ctionary":false,"property":""}]}`

Odpowiedź w przypadku powtarzającego się pola (identyfikator lub numer nipu):

Kod 409:

`{"errorCode":"duplicateValueFoundException","objectName":"customer","fieldName":"ni
pNumber","duplicateValues":[{"lineNumber":-1,"objectId":51,"value":"7291077843"}]}`

<a name="f5"/>

### e)Transakcje

`https://system.firmao.pl/{idenfyfikatorOrganizacji}/svc/v1/transactions`

```
POST
{
  "transactionEntries":[
    {
      "classificationNumber":"",
      "discount":null,
      "productCode":"ST0003",
      "quantity":1,
      "unit":null,
      "vatPercent":0,
      "product":52,
```

> w przypadku gdy podamy zamiast liczby tekst (stringa),
zostanie utworzony wpis z tym tekstem niepowiązany z żadnym produktem

```
      "unitNettoPrice":"1000.00"
    }
  ],
  "transactionNettoPrice":"1000.00",
  "transactionBruttoPrice":"1000.00",
  "transactionVatPrice":"0.00",
  "invoiceAnnotations":null,
  "paid":false,
  "currency":"PLN",
  "paymentType":"CASH",
  "restOfPaidValue":1000,
  "paidValue":0,
  "invoiceDate":"2012-08-08",
  "paymentDate":"2012-08-15",
  "transactionDate":"2012-08-08",
  "type":"INVOICE",
  "mode":"SALE",
  “createCustomerIfNeeded”:true, 
```

> Jeśli parametr createCustomerIfNeeded jest
true, zostanie utworzony nowy kontrahent z kolejnymi parametrami
customerName, phones i emails. Jeśli kontrahent o takiej nazwie już istnieje,
zostanie on podpięty do transakcji

```
  “customerName”:”test”,
  “phones”:[
```

>Używać tylko jeśli createCustomerIfNeeded: true

```
  “111222333”
],
“emails:[
```

> Używać tylko jeśli createCustomerIfNeeded: true

```
    “test@test.test”
],
"customerAddress.postCode":"91-477",
"customer":50,
"customerAddress.city":"Łódź",
"customerAddress.country":"Polska",
"customerAddress.street":"Piotrkowska 147/8",
"nipNumber":"1060000062",
"issuingPerson":"Jacob Smith",
"transactionNumber":"1/8/2012"
}
```

Odpowiedź w przypadku poprawnego utworzenia

Kod 201:

`{"changelog":[{"deletedWith":"","modification":"CREATE","newValueIsDictionary":false,"o
bjectClass":"transaction","objectId":50035,"objectLabel":"1/8/2012 - Internet
Center","oldValueIsDictionary":false,"property":""},{"deletedWith":"","modification":"CRE
ATE","newValueIsDictionary":false,"objectClass":"commisionuser","objectId":35,"objectL
abel":"","oldValueIsDictionary":false,"property":""},{"deletedWith":"","modification":"CHA
NGE","newValueClass":"long","newValueDisplay":"10","newValueIsDictionary":false,"obj
ectClass":"customer","objectId":50,"objectLabel":"Internet
Center","oldValueClass":"long","oldValueDisplay":"9","oldValueIsDictionary":false,"prope
rty":"transactionsCount"}], "showCustomVatWarning":"true"}`

Jeżeli transakcja sprzedaży o danym numerze już istnieje, transakcja się nie tworzy a w
odpowiedzi dostajemy:

Kod 400:

`{"errorCode":"transactionNumberAlreadyExists","additionalInfo":{"suggestedNumber":"2/
8/2012"}}`

W przypadku <b>faktury korygującej</b> pozycje muszą zawierać po dwa wpisy dla każdej
pozycji z oryginalnego dokumentu: pozycję z wartościami przed korektą oraz pozycję z
wartościami po korekcie.

Przykład faktury korygującej dla powyższej transakcji:

```
{
  "invoicePatternId": 1,
  "exchangeRate": 1,
  "exchangeRateDate": "2012-08-07",
  "factor": 1,
  "correctionCause": "Błędnie wprowadzone dane.",
  "paid": false,
  "bankAccount": 1,
  "currency": "PLN",
  "paymentType": "CASH",
  "restOfPaidValue": 1000,
  "paidValue": 0,
  "invoiceDate": "2022-04-08",
  "invoiceReceptionDate": "2022-04-08",
  "paymentDate": "2022-04-15",
  "transactionDate": "2012-08-08",
  "type": "CORRECTION",
  "mode": "SALE",
  "customerAddress.postCode": "91-477",
  "customer": 50,
  "customerAddress.country": "Polska",
  "customerAddress.city": "Łódź",
  "customerAddress.street": "Piotrkowska 147/8",
  "nipNumber": "106-00-00-062",
  "recipientPhoneNumber": null,
  "connectedDocDate": "2012-08-08", 
```

> data oryginalnego dokumentu

```
  "issuingPerson": "Jacob Smith",
  "splitPayment": false,
  "transactionNumber": "2022/04/004/KOR",
  "connectedDocNumber": "1/8/2012", 
```

> numer oryginalnego dokumentu

```
  "calculateFromGross": false,
  "transactionEntries": [
    {
```

> pozycja przed

```
      "unit": null,
      "classificationNumber": "",
      "baseEntryId": 50106,
```

> id korygowanej pozycji z oryginalnego dokumentu

```
      "productCode": "ST0003",
      "quantity": 1,
      "entryPaid": "NOT_PAID",
      "entrySize": 1,
      "variant": 1,
      "vatPercent": "0",
      "recordId": 16,
```

> numer identyfikujący pozycję w obrębie danego dokumentu.

```
      "salePriceGroup": "A",
      "unitPurchaseNettoPrice": 0,
      "unitNettoPrice": "1000.00",
      "unitNettoPriceWithoutDiscount": "1000.00",
      "product": 52
    },
    {
```

> pozycja “po”

```
    "unit": null,
    "classificationNumber": "",
    "productCode": "ST0003",
    "quantity": 2,
    "entryPaid": "NOT_PAID",
    "entrySize": 1,
    "variant": 1,
    "vatPercent": 0,
    "recordId": 17,
```

> numer identyfikujący pozycję w obrębie danego dokumentu.

```
    "salePriceGroup": "A",
    "unitPurchaseNettoPrice": 0,
    "unitNettoPrice": "1000.00",
    "unitNettoPriceWithoutDiscount": "0.00",
    "invalidRecordId": 16,
```

> recordId pozycji “przed”

```    
    "product": 52
    }
  ],
  "transactionNettoPrice": "1000.00",
  "transactionBruttoPrice": "1000.00",
  "transactionVatPrice": "0.00",
  "lastSelectedInvoiceReport": "Faktura_korygujaca2"
  }
```

<a name="f6"/>

### f)Oferty/Zamówienia/Umowy

`https://system.firmao.pl/{idenfyfikatorOrganizacji}/svc/v1/offers` (Oferty / Zamówienia)

`https://system.firmao.pl/{idenfyfikatorOrganizacji}/svc/v1/agreements` (Umowy)

```
POST
{
  "entries":[
```

> wpis entries można pominąć w przypadku braku pozycji oferty

```
    {
      "unit":"szt.",
      "product":51, 
```

> w przypadku gdy podamy zamiast liczby tekst (stringa),
zostanie utworzony wpis z tym tekstem niepowiązany z żadnym produktem. W
przypadku podania liczby system wyszukuje istniejący produkt i podpina do
pozycji.

```
      "classificationNumber":"",
      "productCode":"ST0002",
      "quantity":1,
      "vatPercent":22,
      "unitNettoPrice":"818.85"
    }
  ],
  "nettoPrice":"818.85",
  "bruttoPrice":"999.00",
  "vatPrice":"180.15",
  "currency":"PLN",
  "paymentType":"CASH",
  "paymentDate":"2012-11-09",
  "offerDate":"2012-11-09",
  "validFrom":"2012-11-09",
  "type":"OFFER",
```

> OFFER - dla oferty, ORDER - dla zamówienia, AGREEMENT -
  dla umowy,

```
  "mode":"SALE", 
```

> SALE - dla sprzedaży, PURCHASE - dla zakupu

```
  "offerStatus":"NEW",
```

> NEW, SENT, DURING_NEGOTIATIONS, ACCEPTED,
  
> REJECTED, EXECUTED

```
  "customerAddress.postCode":"91-477",
  "daysToDueDate":7,
  "customer":50, 
```

> id kontrahenta w systemie - można użyć parametru
customerName, jeżeli nie chcemy łączyć z kontrahentem

```
  "customerAddress.city":"Łódź",
  "customerAddress.country":"Polska",
  "customerAddress.street":"Piotrkowska 147/8",
  "nipNumber":"1060000062",
  "issuingPerson":"Jacob Smith",
  "number":"1/11/2012/U" 
```

> pole wymagane i musi być niepowtarzalne

```
}
```

Odpowiedź w przypadku poprawnego utworzenia

Kod 201:

`{"changelog":[{"deletedWith":"","modification":"CREATE","newValueIsDictionary":false,"o
bjectClass":"offer","objectId":2,"objectLabel":"1/11/2012/U - Internet
Center","oldValueIsDictionary":false,"property":""},{"deletedWith":"","modification":"CHA
NGE","newValueClass":"long","newValueDisplay":"2","newValueIsDictionary":false,"obje
ctClass":"customer","objectId":50,"objectLabel":"Internet
Center","oldValueClass":"long","oldValueDisplay":"1","oldValueIsDictionary":false,"prope
rty":"offersCount"}]}`

<a name="f7"/>

### g)Osoby kontaktowe

`https://system.firmao.pl/{idenfyfikatorOrganizacji}/svc/v1/contacts`

```
POST
{
  "firstName":"Marzena",
  "lastName":"Wojciechowska",
  "position":"Handlowiec",
  "description":null,
  "emails":[
    "m.wojciechowska@mailinator.com"
  ],
  "phones":[
    "226548484",
    "226548485",
    "500111222"
  ],
  "tags":[
  ],
  "customer":51, 
```

> id kontrahenta w systemie - Pole wymagane

```
  "label":"Marzena Wojciechowska"
}
```

Odpowiedź w przypadku poprawnego utworzenia

Kod 201:

`{"changelog":[{"deletedWith":"","modification":"CREATE","newValueIsDictionary":false,"objectCla
ss":"contact","objectId":50001,"objectLabel":"Marzena
Wojciechowska","oldValueIsDictionary":false,"property":""},{"deletedWith":"","modification":"CHA
NGE","newValueClass":"long","newValueDisplay":"0","newValueIsDictionary":false,"objectClass"
:"customer","objectId":51,"objectLabel":"KOMP-SERVICE","oldValueClass":"long","oldValueDisp
lay":"0","oldValueIsDictionary":false,"property":"mailsCount"},{"deletedWith":"","modification":"CH
ANGE","newValueClass":"long","newValueDisplay":"2","newValueIsDictionary":false,"objectClas
s":"customer","objectId":51,"objectLabel":"KOMP-SERVICE","oldValueClass":"long","oldValueDi
splay":"0","oldValueIsDictionary":false,"property":"contactsCount"},{"deletedWith":"","modificatio
n":"ATTACH","newValueClass":"contact","newValueDisplay":"Marzena
Wojciechowska","newValueId":"50001","newValueIsDictionary":false,"objectClass":"customer","o
bjectId":51,"objectLabel":"KOMP-SERVICE","oldValueIsDictionary":false,"property":"contact_cha
nge"}]}`

<a name="f8"/>

### h) Notatki handlowe

`https://system.firmao.pl/{idenfyfikatorOrganizacji}/svc/v1/salesnotes`

```
POST
  {
    "description":"Testowa notatka",
    "type":"NOTE",
    "customer":51, 
```

> id kontrahenta w systemie - Pole wymagane

```
    "tags":[
  ],
  "date":"2013-04-10T17:12:20+02:00"
}
```

Odpowiedź w przypadku poprawnego utworzenia

Kod 201:

`{"changelog":[{"deletedWith":"","modification":"CREATE","newValueIsDictionary":false,"o
bjectClass":"salesnote","objectId":50011,"objectLabel":"","oldValueIsDictionary":false,"pr
operty":""},{"deletedWith":"","modification":"CHANGE","newValueClass":"long","newValu
eDisplay":"1","newValueIsDictionary":false,"objectClass":"customer","objectId":51,"objec
tLabel":"KOMP-SERVICE","oldValueClass":"long","oldValueDisplay":"0","oldValueIsDicti
onary":false,"property":"salesNotesCount"},{"deletedWith":"","modification":"CHANGE","
newValueClass":"date","newValueDisplay":"2017-09-21T13:49:11.691Z","newValueIsDic
tionary":false,"objectClass":"customer","objectId":51,"objectLabel":"Name","oldValueIsDi
ctionary":false,"property":"lastContactDate"}]}`

<a name="PUT"/>

## 3.Modyfikacja obiektów (PUT)

Modyfikacja obiektów odbywa się za pomocą żądania PUT i powinny być kierowane na
adres

`https://system.firmao.pl/{idenfyfikatorOrganizacji}/svc/v1/{xxx}/{id}`

`xxx` - nazwa typu obiektu

`id` - identyfikator obiektu

`idenfyfikatorOrganizacji` - identyfikator organizacji widoczny w urlu po zalogowaniu do
firmao

Typ danych:

`ContentType = application/json; charset=UTF-8`

Wszystkie dostępne w obiektach pola są opisane w pomocy do importu danych.
Pola dodatkowe mają klucze “customFields.custom5” i przyjmują jako wartości Stringi.
Dostępne pola można również “podejrzeć” poprzez pobranie danych za pomocą metody
GET (z wyłączeniem pola permission, które jest wyliczane i nie można go podać w
żądaniu POST/PUT).

Kody odpowiedzi:

API używa standardowych kodów błędów HTTP.

200 - poprawna edycja

4xx i 5xx - nastąpił błąd podczas edycji obiektu.

<b>Generalnie API jest przystosowane do modyfikacji wielu w jednym żądaniu ale
przetestowane jest głównie dla modyfikacji jednego pola naraz, więc w przypadku
modyfikacji wielu pól w jednym żądaniu mogą wystąpić nieoczekiwane problemy.</b>

<a name="f9"/>

### a)Zmiana nazwy produktu:

Modyfikacja nazwy produktu o id 50001:

`https://system.firmao.pl/{idenfyfikatorOrganizacji}//svc/v1/products/50001`

```
PUT
{
"name":"Przykładowy produkt zmiana nazwy"
}
```

Odpowiedź:

`{"changelog":[{"deletedWith":"","modification":"CHANGE","newValueClass":"string","new
ValueDisplay":"Przykładowy produkt zmiana
nazwy","newValueIsDictionary":false,"objectClass":"product","objectId":50001,"objectLa
bel":"Przykładowy produkt zmiana
nazwy","oldValueClass":"string","oldValueDisplay":"Przykładowy
produkt","oldValueIsDictionary":false,"property":"name"}]}`

<b>W odpowiedzi najważniejsze pola to</b>

<b>`property` - nazwa zmienianego pola</b>

<b>`newValueDisplay` - nowa wartość pola</b>

<b>`oldValueDisplay` - stara wartość pola</b>

<a name="f10"/>

### b)Zmiana nazwy kontrahenta:

Modyfikacja nazwy kontrahenta o id 50001:

`https://system.firmao.pl/{idenfyfikatorOrganizacji}//svc/v1/customers/50001`

```
PUT
{
"name":"KOMP-SERVICE"
}
```

Odpowiedź:

`{"changelog":[{"deletedWith":"","modification":"CHANGE","newValueClass":"string","new
ValueDisplay":"KOMP-SERVICE","newValueIsDictionary":false,"objectClass":"customer"
,"objectId":50001,"objectLabel":"q","oldValueClass":"string","oldValueDisplay":"q","oldVal
ueIsDictionary":false,"property":"name"}]}`

<a name="f11"/>

### c) Zmiana stanu magazynowego produktu (wymagane włączenie w organizacji ustawienia: Bezpośrednia modyfikacja stanów magazynowych)

`https://system.firmao.pl/{idenfyfikatorOrganizacji}/svc/v1/products/50001`

```
PUT
{
    "currentStoreState":50
}
```

Odpowiedź (odpowiedź jest taka duża ponieważ serwer wylicza sam różne dane, w tym
przypadku wylicza wartość produktów w magazynie):

`{"changelog":[{"deletedWith":"","modification":"CHANGE","newValueClass":"float","newV
alueDisplay":"50","newValueIsDictionary":false,"objectClass":"product","objectId":50001,
"objectLabel":"Przykładowy produkt zmiana
nazwy","oldValueClass":"float","oldValueDisplay":"0.000","oldValueIsDictionary":false,"pr
operty":"currentStoreState","subClass":"warehouse","subId":1,"subLabel":"Magazyn
1"},{"deletedWith":"","modification":"CHANGE","newValueClass":"float","newValueDispla
y":"0.00","newValueIsDictionary":false,"objectClass":"product","objectId":50001,"objectL
abel":"Przykładowy produkt zmiana
nazwy","oldValueClass":"float","oldValueIsDictionary":false,"property":"netPriceInStore"}]}`

<a name="GET"/>

## 4.Pobieranie obiektów (GET)

Pobieranie odbywa się za pomocą żądania GET i powinny być kierowane na adres

`https://system.firmao.pl/{idenfyfikatorOrganizacji}/svc/v1/{xxx}`

`xxx` - nazwa typu obiektu

`idenfyfikatorOrganizacji` - identyfikator organizacji widoczny w urlu po zalogowaniu do firmao

<a name="f12"/>

### Stronicowanie

W żądaniach GET mamy do czynienia ze stronicowaniem wyników. Służą do tego
parametry start (od którego wyniku rozpoczynamy listowanie na stronie), limit
(maksymalna ilość wyników na stronie). Maksymalna wartość limitu jest zależna od
wykupionej licencji.

W odpowiedzi z serwera znajduje się pole totalSize, które mówi o całkowitej ilości
obiektów, dzięki czemu wiemy czy istnieją kolejne strony obiektów.

<b>UWAGA</b>: W niektórych przypadkach w odpowiedzi pojawia się totalSize o wartości -1.
Oznacza to, iż istnieje co najmniej tyle wyników ile wynosi limit, czyli
najprawdopodobniej istnieje kolejna strona wyników.

<a name="f13"/>

### Sortowanie

W żądaniu możemy również określić pole po którym sortujemy (parametr sort) oraz
kierunek sortowania (parametr dir). w przypadku braku parametrów sortowania wyniki
zostaną posortowane w sposób domyślny

<a name="f14"/>

### Filtrowanie

W żądaniu możemy określić filtrowanie. Format filtrowania to:

nazwaPola(typPorównania), gdzie nazwa pola to pole po którym filtrujemy a

typPorównania to:

contains - czy pole zawiera wartość (dla niektórych pól obsługuje wartości po
średnikach np “ala;kota”),

notContains - czy pole nie zawiera wartości (dla niektórych pól obsługuje wartości po
średnikach np “ala;kota”),

eq - czy pole jest równe wartości ,

ne - czy pole nie jest równe wartości ,

in - czy pole jest równe wartościom po średnikach np 1;3;5 - obsługiwane najczęściej id

notin - czy nie pole jest równe wartościom po średnikach np 1;3;5 - obsługiwane
najczęściej id

gt - czy pole jest większe (tylko dla pól liczbowych i dat),

ngt - czy pole nie jest większe (czyli mniejsze lub równe) od (tylko dla pól liczbowych i
dat),

ge - czy pole jest większe lub równe (tylko dla pól liczbowych i dat),

nge - czy pole nie jest większe lub równe (czyli mniejsze) od (tylko dla pól liczbowych i
dat),

lt - czy pole jest mniejsze od (tylko dla pól liczbowych i dat)

nlt - czy pole nie jest mniejsze (czyli większe lub równe) od (tylko dla pól liczbowych i
dat)

le - czy pole jest mniejsze lub równe od (tylko dla pól liczbowych i dat)

nle - czy pole nie jest mniejsze (czyli większe) od (tylko dla pól liczbowych i dat)

_Uwaga: nie wszystkie pola każdego obiektu obsługują wszystkie typy filtrowania._

Przykłady filtrowania:

`https://system.firmao.pl/{idenfyfikatorOrganizacji}/svc/v1/projects?id(eq)=15`

> pobiera projekt o id 15.

`https://system.firmao.pl/{idenfyfikatorOrganizacji}/svc/v1/projects?id(ge)=15`

> pobiera projekt o id większym lub równym 15.

`https://system.firmao.pl/{idenfyfikatorOrganizacji}/svc/v1/projects?name(contains)=proj`

> pobiera projekty o nazwie zawierającej ciąg znaków “proj”.

<a name="f15"/>

### Kody odpowiedzi

API używa standardowych kodów błędów HTTP.

`200` - poprawne pobranie danych

`4xx i 5xx` - nastąpił błąd podczas pobierania danych.

<a name="f16"/>

### a) Pobranie projektów - projects

```
GET
https://system.firmao.pl/{idenfyfikatorOrganizacji}/svc/v1/projects?start=0&limit=50&sort
=name&dir=ASC&name(contains)=przy&dataFormat=MEDIUM
```

Odpowiedź:

```
{
  "data": [
    {
      "actualCost": 0,
      "actualCostSummaryBrutto": 0,
      "actualCostSummaryNetto": 0,
      "actualIncomesBrutto": 0,
      "actualIncomesNetto": 0,
      "actualIncomeSummaryBrutto": 0,
      "actualIncomeSummaryNetto": 0,
      "actualOutgoingsBrutto": 0,
      "actualOutgoingsNetto": 0,
      "actualProfitSummaryBrutto": 0,
      "actualProfitSummaryNetto": 0,
      "actualTaskProfit": 0,
      "actualTransactionProfitBrutto": 0,
      "actualTransactionProfitNetto": 0,
      "actualWorkHours": null,
      "approved": false,
      "attachmentsCount": 0,
      "autoCalcEstimatedHours": true,
      "autoCalcPlannedCost": true,
      "autoCalcPlannedIncome": true,
      "balance": 0,
      "budget": 0,
      "contact": null,
      "costRealizationPercent": 0,
      "createdBy": {
        "id": 1,
        "label": "Jacob Smith"
      },
      "creationDate": "2017-09-21T13:08:59Z",
      "customer": null,
      "customFields": null,
      "description": "Opis",
      "endDate": null,
      "estimatedHours": null,
      "finalProfitSummaryBrutto": 0,
      "finalProfitSummaryNetto": 0,
      "finalTransactionProfitBrutto": 0,
      "finalTransactionProfitNetto": 0,
      "hourlyPrice": null,
      "hourlyRate": null,
      "id": 50001,
      "income": 0,
      "incomeRealizationPercent": 0,
      "lastUpdateDate": "2017-09-21T13:08:59Z",
      "managers": [
        {
          "id": 10,
          "label": "Denise Taylor"
        }
      ],
      "margin": null,
      "name": "Przykładowy",
      "overdraftSummaryBrutto": 0,
      "overdraftSummaryNetto": 0,
      "permissions": {
        "project": [
          "basic_update",
          "costs",
          "create",
          "extended_update",
          "read",
          "update"
        ],
        "task": [
          "basic_update",
          "costs",
          "create",
          "read",
          "update"
        ]
      },
      "plannedCost": 0,
      "plannedCostSummaryBrutto": 0,
      "plannedCostSummaryNetto": 0,
      "plannedIncome": 0,
      "plannedIncomesBrutto": 0,
      "plannedIncomesNetto": 0,
      "plannedIncomeSummaryBrutto": 0,
      "plannedIncomeSummaryNetto": 0,
      "plannedOutgoingsBrutto": 0,
      "plannedOutgoingsNetto": 0,
      "plannedProfitSummaryBrutto": 0,
      "plannedProfitSummaryNetto": 0,
      "plannedTaskProfit": 0,
      "plannedTransactionProfitBrutto": 0,
      "plannedTransactionProfitNetto": 0,
      "purchaseTransactionBrutto": 0,
      "purchaseTransactionNetto": 0,
      "restrictedTeamMembers": [],
      "saleTransactionBrutto": 0,
      "saleTransactionNetto": 0,
      "startDate": "2012-07-16T22:00:00Z",
      "tags": [
        {
          "color": "#FFFFFF",
          "id": 3,
          "label": "Projekt zatwierdzony"
        }
      ],
      "teamMembers": [
        {
          "id": 6,
          "label": "Emma Clark"
        }
      ],
        "transactionsBalanceBrutto": 0,
        "transactionsBalanceNetto": 0
      }
      ],
      "dir": "ASC",
      "sort": "name",
      "totalSize": 1
    }
```

<a name="f17"/>

### b) Pobranie zadań - tasks

```
GET
https://system.firmao.pl/{idenfyfikatorOrganizacji}/svc/v1/tasks?start=0&limit=50&sort=n
ame&dir=ASC&name(contains)=zadanie&dataFormat=MEDIUM
```

Odpowiedź:

```
{
  "data": [
    {
      "actualCost": 0,
      "actualEndDate": null,
      "actualStartDate": null,
      "actualWorkHours": 0,
      "additionalCosts": 0,
      "address": null,
      "attachmentsCount": 0,
      "autoCalcActualCost": true,
      "autoCalcIncome": true,
      "autoCalcPlannedCost": true,
      "autoCalcPlannedIncome": true,
      "autoStatus": false,
      "balance": 0,
      "contact": null,
      "createdBy": {
        "id": 1,
        "label": "Jacob Smith"
      },
      "creationDate": "2017-09-21T13:11:51Z",
      "customer": null,
      "customFields": null,
      "description": "Opis",
      "discount": null,
      "estimatedHours": 8,
      "hasEndDateReminder": false,
      "hasStartDateReminder": false,
      "hasSubtasks": false,
      "id": 50033,
      "income": 0,
      "lastUpdateDate": "2017-09-21T13:11:51Z",
      "letter": null,
      "name": "Przykładowe zadanie",
      "orderBruttoPrice": null,
      "orderCalculateFromGross": false,
      "orderNettoPrice": null,
      "orderQuantity": null,
      "orderUnit": null,
      "orderUnitPrice": null,
      "orderVatPercent": null,
      "orderVatPrice": null,
      "parent": null,
      "percentDone": 0,
      "permissions": {
        "task": [
          "basic_update",
          "costs",
          "create",
          "read",
          "update"
      ]
    },
    "plannedCost": 200,
    "plannedEndDate": "2012-07-17T22:00:00Z",
    "plannedEndDateType": "EXACTLY",
    "plannedIncome": 400,
    "plannedStartDate": "2012-07-16T22:00:00Z",
    "plannedStartDateType": "EXACTLY",
    "priority": 300,
    "product": null,
    "project": null,
    "reminderEndDateAdvance": null,
    "reminderStartDateAdvance": null,
    "resource": null,
    "responsibleUsers": [
      {
        "id": 2,
        "label": "Ethan Lam"
      }
    ],
    "status": {
      "group": "OPEN",
      "icon": null,
      "key": "WAITING",
      "label": "Oczekujące",
      "label_de": "Wird erwartet",
      "label_en": "Waiting"
    },
    "subcontact": null,
    "subcontractor": null,
    "subtasksCount": 0,
    "tags": [],
    "taskType": "PROJECT",
    "transactionBruttoPrice": 0,
    "transactionNettoPrice": 0
  }
],
"dir": "ASC",
"sort": "name",
"totalSize": 1
}
```
<a name="f18"/>

### c)Pobieranie produktów - products

```
GET
https://system.firmao.pl/{idenfyfikatorOrganizacji}/svc/v1/products?start=0&limit=50&sort
=name&dir=ASC&dataFormat=MEDIUM
```

Odpowiedź:

```
{
  "data":[
    {
      "classificationNumber":null,
      "createdBy":{
        "id":1,
        "label":"Jacob Smith"
    },
    "creationDate":"2012-07-17T08:39:15Z",
    "currentStoreState":50.00,
    "customFields":null,
    "description":null,
    "endSellingDate":null,
    "grossPrice":12.30,
    "grossPriceInStore":615.00,
    "id":50001,
    "lastUpdateDate":"2012-07-17T08:39:15Z",
    "margin":20.00,
    "name":"Przykładowy produkt zmiana nazwy",
    "netPrice":10.00,
    "netPriceInStore":500.00,
    "overrunState":true,
    "permissions":{
      "product":[
        "buyprice",
        "create",
        "extended_update",
        "read",
        "update"
      ],
      "semiproduct":[
        "create",
        "read",
        "update"
      ],
      "transactionentry":[
        "create",
        "read",
        "update"
      ]
    },
    "productCode":"PR1",
    "saleGrossPrice":14.76,
    "saleGrossPriceInStore":738.00,
    "saleNetPrice":12.00,
    "saleNetPriceInStore":600.00,
    "saleable":true,
    "startSellingDate":null,
    "tags":[
      ],
      "unit":"szt.",
      "vat":23.00,
      "warningStoreState":88.00
      }
    ],
    "dir":"ASC",
    "sort":"name",
    "totalSize":1
  }
```
<a name="f19"/>

### d)Pobranie kontrahentów - customers

```
GET
https://system.firmao.pl/{idenfyfikatorOrganizacji}/svc/v1/customers?start=0&limit=50&s
ort=name&dir=ASC&dataFormat=MEDIUM
```

Odpowiedź:

```
{
  "data":[
    {
      "actualCost":0.00,
      "balance":0.00,
      "bankAccountNumber":"50 1020 5558 1111 1702 0600 0067",
      "correspondenceAddress":null,
      "createdBy":null,
      "creationDate":"2002-12-13T11:40:45Z",
      "customFields":null,
      "customerGroups":[
      ],
      "customerType":"PARTNER",
      "description":"Firma specjalizująca się w serwisie komputerów",
      "email":"kompservice.firmao@mailinator.com",
      "email2":"komputery@mailinator.com",
      "email3":null,
      "emails":[
        "kompservice.firmao@mailinator.com",
        "komputery@mailinator.com"
      ],
      "employeesNumber":1,
      "faxNumber":"22 398 78 87
      ", "hourlyRate":null,
      "id":51,
      "identificationNumber":null,
      "income":0.00,
      "industry":{
        "key":"3",
        "label":"IT"
      },
      "krs":null,
      "label":"KOMP-SERVICE",
      "lastSalesNote":null,
      "lastUpdateDate":"2010-05-29T20:53:30Z",
      "name":"KOMP-SERVICE Jan Kowalski",
      "nipNumber":"7291077843",
      "officeAddress":{
        "city":"Pruszków",
        "country":"Polska",
        "county":null,
        "postCode":"05-800",
        "street":"Stefana Batorego 3/4" },
      "ownership":"PRIVATE",
      "permissions":{
        "vatrecord":[
          "create",
          "read", 
          "update"
      ],
      "task":[
        "basic_update",
        "costs",
        "create",
        "read",
        "update"],
      "accountingoperation":[
        "create",
        "read",
        "update"
      ],
      "customer":[
        "costs",
        "create",
        "read",
        "update"
      ]
    },
    "phone":"22 345 74 26",
    "phoneOther":null,
    "phoneOther2":null,
    "phones":[
      "22 345 74 26"
    ],
    "plannedCost":0.00,
    "plannedIncome":0.00,
    "stats":null,
    "status":null,
    "tags":[
    ],
    "vatUE":"PL 7291077843",
    "voivodeship":null,
    "website":"www.kompservice.bis"
  }
],
"dir":"DESC",
"sort":"id",
"totalSize":1
}
```
<a name="f20"/>

### e)Pobranie transakcji - transactions

```
GET
https://system.firmao.pl/{idenfyfikatorOrganizacji}/svc/v1/customers?start=0&limit=50&s
ort=name&dir=ASC&dataFormat=MEDIUM
```

Odpowiedź:

```
{
  "data":[
    {
      "actualPaymentDate":null,
      "actualPaymentType":null,
      "advanceInvoiceGroupId":null,
      "approved":false,
      "calculateFromGross":false,
      "createdBy":null,
      "creationDate":"2002-12-13T11:40:45Z",
      "currency":"PLN",
      "customFields":null,
      "customer":{
      "emails":[
        "icenter.firmao@mailinator.com",
        "iceneter.biuro1@mailinator.com",
        "iceneter.biuro2@mailinator.com"
      ],"id
      ":50,
      "label":"InternetCenter"
    },"customerAddress":null,
    "customerName":null,
    "description":null,
    "hasReminder":false,
    "id":50,
    "invalidInvoiceDate":null,
    "invalidTransactionNumber":null,
    "invoiceAnnotations":null,
    "invoiceDate":"2010-09-27",
    "issuingPerson":null,
    "lastUpdateDate":"2010-05-29T20:53:30Z",
    "mail":null,
    "mode":"SALE",
    "name":"Sprzedaż DE PC",
    "nipNumber":null,
    "paid":true,
    "paidValue":999,
    "parcelCourierName":null,
    "parcelDeliveryType":null,
    "parcelDescription":null,
    "parcelNumber":null,
    "parcelShipmentDate":null,
    "paymentDate":"2010-09-30",
    "paymentStatus":null,
    "paymentType":"TRANSFER",
    "permissions":{
      "transaction":[
      "create",
      "extended_update",
      "read",
      "update"
    ],
    "transactionentry":[
      "create",
      "read",
      "update"
    ]
  },
  "project":{
    "id":4,
    "name":"GANT USE CASE2"
  },
  "reminderAdvance":null,
  "restOfPaidValue":0,
  "sellerAddress":null,
  "storageStateUpdate":"NO_UPDATE",
  "tags":[
  ],
  "task":{
    "id":24,
    "name":"GANTT UC2 C",
    "status":{
      "group":"OPEN"
      }
    },
    "transactionBruttoPrice":999,
    "transactionDate":"2010-09-24", 
    "transactionGroupBruttoPrice":null,
    "transactionGroupVatPrice":null,
    "transactionNettoPrice":818.85,
    "transactionNumber":"1/1/2010",
    "transactionVatPrice":180.15,
    "type":"INVOICE"
  }
],
"dir":"ASC",
"sort":"name",
"totalSize":1
}
```

<a name="f21"/>

### f) pobranie transakcji dla kontrahenta - transactions

```
GET
https://system.firmao.pl/{idenfyfikatorOrganizacji}/svc/v1/transactions?start=0&limit=50&
customer(eq)=10
```

> gdzie 10 to id kontrahenta

Odpowiedź:

```
{
  "data": [
  {
    "actualPaymentDate": null,
    "actualPaymentType": null,
    "advanceInvoiceGroupId": null,
    "approved": true,
    "attachmentsCount": 0,
    "bankAccount": {
      "id": 1,
      "label": "test"
    },
    "baseBruttoPrice": 0,
    "baseNettoPrice": 0,
    "basePaidValue": 0,
    "baseRestOfPaidValue": 0,
    "baseVatPrice": 0,
    "calculateFromGross": false,
    "commision": 0,
    "commisionRate": 0,
    "connectedDocDate": null,
    "connectedDocNumber": null,
    "connectedDocType": null,
    "contact": null,
    "correctionCause": null,
    "createdBy": {
      "id": 1,
      "label": "Jacob Smith"
    },
    "creationDate": "2017-09-21T13:38:08Z",
    "currency": "PLN",
    "customer": {
      "discount": null,
      "emails": null,
      "id": 1,
      "label": "DELL",
      "salePriceGroup": "A"
    },
    "customerAddress": null,
    "customerBankAccountNumber": null,
    "customerName": "DELL",
    "customFields": null,
    "dateForExchangeRate": "2017-09-21",
    "dateLabel": "SELL_DATE",
    "description": null,
    "documentDate": "2017-09-21",
    "documentGenerated": false,
    "documentNumber": "2017/9/21/21",
    "doNotIncludeInLedgerReport": false,
    "exchangeRate": null,
    "exchangeRateDate": null,
    "factor": 1,
    "finalPartialInvoice": false,
    "hasReminder": false,
    "id": 50034,
    "integrationType": null,
    "invoiceAnnotations": null,
    "invoiceDate": "2017-09-21",
    "invoicePatternId": 1,
    "invoiceReceptionDate": null,
    "issuingPerson": "Jacob Smith",
    "issuingPersonObject": {
      "firstName": "Jacob",
      "id": 1,
      "lastName": "Smith"
      },
      "lastSelectedInvoiceReport": "Faktura",
      "lastUpdateDate": "2017-09-21T13:38:08Z",
      "mail": null,
      "mode": "SALE",
      "name": null,
      "nipNumber": null,
      "paid": false,
      "paidValue": 0,
      "paidValueInvoice": 0,
      "parcelCourierName": null,
      "parcelDeliveryType": null,
      "parcelDescription": null,
      "parcelNumber": null,
      "parcelShipmentDate": null,
      "paymentDate": "2017-09-21",
      "paymentStatus": "NOT_PAID",
      "paymentType": "CASH",
      "permissions": {
        "transaction": [
        "create",
        "extended_update",
        "read",
        "update"
     ]
   },
  "phone": "+48 42 356 53 34",
  "profit": 0,
  "profitCommision": 0,
  "profitCommisionRate": 0,
  "projects": [],
  "purchaseVatCategory": null,
  "rateFromCurrency": null,
  "recipient": null,
  "recipientAddress": null,
  "recipientName": null,
  "recipientNipNumber": null,
  "recipientPhoneNumber": null,
  "reminderAdvance": null,
  "restOfPaidValue": 0,
  "restOfPaidValueInvoice": 0,
  "sellerAddress": {},
  "sellerBankAccountNumber": "PL83 1010 1023 0000 2613 9510 0000",
  "smallTaxPayer": false,
  "storageStateUpdate": "NO_UPDATE",
  "tags": [],
  "task": null,
  "transactionBruttoPrice": 0,
  "transactionDate": "2017-09-21",
  "transactionGroupBruttoPrice": 0,
  "transactionGroupVatPrice": 0,
  "transactionNettoPrice": 0,
  "transactionNettoPriceTax": 0,
  "transactionNumber": "2017/9/21/21",
  "transactionOrInvoiceDate": "2017-09-21",
  "transactionVatPeriod": "09.2017",
  "transactionVatPrice": 0,
  "type": "INVOICE",
  "warehouse": {
  "id": 1,
  "name": "Magazyn 1"
  }
}
],
"dir": "DESC",
"sort": "transactionDate",
"totalSize": 1
}
```

<a name="f22"/>

### g) pobieranie pozycji transakcji - transactions

```
GET
https://system.firmao.pl/{idenfyfikatorOrganizacji}/svc/v1/transactionentries?transaction(e
q)=50
```

> gdzie 50 to id transakcji

Odpowiedź:

```
{
  "data":[
    {
      "balance":null,
      "bruttoPrice":999.00,
      "calculateFromGross":false,
      "classificationNumber":null,
      "description":null,
      "discount":null,
      "entryDate":"2010-09-24",
      "id
      ":50,
      "invalidEntry":null,
      "mode":"SALE",
      "name":null,
      "nettoPrice":818.85,
      "offer":null,
      "permissions":{
        "transactionentry":[
          "create",
          "read",
          "update"
        ]
      },
      "product":{
        "id":51,
        "name":"De PC Pentium 2,74 2GB"
      },
      "productCode":"ST0002",
      "project":null,
      "quantity":1.000,
      "transaction":{
        "customerName":null,
        "id":50,
        "name":"Sprzedaż DE PC",
        "transactionNumber":"1/1/2010"
      },
      "type":"ACTUAL",
      "unit":"szt.",
      "unitBruttoPrice":999.00,
      "unitNettoPrice":818.85,
      "vatPercent":22.00,
      "vatPrice":180.15
      }
      ],
      "dir":"ASC",
      "sort":"transaction",
      "totalSize":1
    }
```

<a name="f23"/>

### h) pobranie ofert/zamówień - offers

```
GET
https://system.firmao.pl/{idenfyfikatorOrganizacji}/svc/v1/offers?start=0&limit=50&sort=n
ame&dir=ASC&dataFormat=MEDIUM

{
  "data":[
    {
      "annotations":null,
      "bruttoPrice":999,
      "calculateFromGross":false,
      "contact":null,
      "createdBy":{
        "id":1,
        "label":"Jacob Smith"
    },
    "creationDate":"2012-11-09T12:06:47Z",
    "currency":"PLN",
    "customFields":null,
    "customer":{
      "email":"icenter.firmao@mailinator.com",
      "id":50,
      "label":"Internet Center"
    },
    "customerAddress":{
      "city":"Łódź",
      "country":"Polska",
      "county":null,
      "postCode":"91-477",
      "street":"Piotrkowska 147/8"
    },
      "customerName":"Internet Center dsfds",
      "daysToDueDate":7,
      "description":null,
      "hasReminder":false,
      "id":1,
      "issuingPerson":"Jacob Smith",
      "lastUpdateDate":"2012-11-09T12:06:48Z",
      "mail":null,
      "mode":"SALE",
      "name":null,
      "nettoPrice":818.85,
      "nipNumber":"1060000062",
      "number":"1/11/2012/U",
      "offerDate":"2012-11-09",
      "offerStatus":"NEW",
      "parcelCourierName":null,
      "parcelDeliveryType":null,
      "parcelDescription":null,
      "parcelNumber":null,
      "parcelShipmentDate":null,
      "paymentDate":"2012-11-09",
      "paymentType":"CASH",
      "permissions":{
        "offer":[
          "create",
          "read",
          "update"
      ],
      "transactionentry":[
        "create",
        "read",
        "update"
      ]
    },
    "project":null,
    "reminderAdvance":null,
    "sellerAddress":{ },
    "tags":[
    ],
    "task":null,
    "type":"AGREEMENT",
    "validFrom":"2012-11-09",
    "vatPrice":180.15
  }
],
"dir":"DESC",
"sort":"offerDate",
"totalSize":1
}
```

<a name="f24"/>

### i) pobieranie osób kontaktowych - contacts

```
GET
https://system.firmao.pl/{idenfyfikatorOrganizacji}/svc/v1/contacts?start=0&limit=50&sort
=firstName&dir=ASC&dataFormat=MEDIUM

{
  "data": [
    {
      "address": null,
      "createdBy": null,
      "creationDate": "2002-12-13T11:40:45Z",
      "customer": {
        "emails": [
          "iceneter.biuro1@mailinator.com",
          "iceneter.biuro2@mailinator.com",
          "icenter.firmao@mailinator.com"
        ],
        "id": 50,
        "label": "Internet Center"
      },
      "customFields": null,
      "departament": "Sekretariat",
      "description": null,
      "email": "anna.wojewodzka@mailinator.com",
      "email2": null,
      "email3": null,
      "emails": [
        "anna.wojewodzka@mailinator.com"
      ],
      "firstName": "Anna",
      "hasReminder": false,
      "id": 51,
      "instantMessager": null,
      "label": "Anna Wojewódzka",
      "lastName": "Wojewódzka",
      "lastUpdateDate": "2010-05-29T20:53:30Z",
      "nextContactDate": null,
      "permissions": {
        "contact": [
          "create",
          "read",
          "update"
        ]
      },
      "phone": "42 688 14 22",
      "phoneOther": null,
      "phoneOther2": null,
      "phones": [
        "42 688 14 22"
      ],
      "position": "Sekretarka",
      "reminderAdvance": null,
      "skypeId": null,
      "tags": []
  },
  {
      "address": null,
      53
      "createdBy": null,
      "creationDate": "2002-12-13T11:40:45Z",
      "customer": {
        "emails": [
          "iceneter.biuro1@mailinator.com",
          "iceneter.biuro2@mailinator.com",
          "icenter.firmao@mailinator.com"
        ],
        "id": 50,
        "label": "Internet Center"
      },
      "customFields": null,
      "departament": null,
      "description": null,
      "email": "m.nowak@mailinator.com",
      "email2": null,
      "email3": null,
      "emails": [
        "m.nowak@mailinator.com"
      ],
      "firstName": "Maciej",
      "hasReminder": false,
      "id": 50,
      "instantMessager": null,
      "label": "Maciej Nowak",
      "lastName": "Nowak",
      "lastUpdateDate": "2017-09-21T14:44:57Z",
      "nextContactDate": null,
      "permissions": {
        "contact": [
          "create",
          "read",
          "update"
      ]
      },
      "phone": "48507306295",
      "phoneOther": "641044884",
      "phoneOther2": null,
      "phones": [
        "48507306295",
        "641044884"
      ],
      "position": "Handlowiec",
      "reminderAdvance": null,
      "skypeId": null,
      "tags": []
    }
],
"dir": "ASC",
"sort": "firstName",
"totalSize": 2
}
```

<a name="f25"/>

### j) pobieranie grup kontrahentów - customergroups

```
https://system.firmao.pl/{idenfyfikatorOrganizacji}/svc/v1/customergroups?start=0&limit=
50&sort=firstName&dir=ASC&dataFormat=MEDIUM

{
  "data": [
    {
      "createdBy": {
        "id": 2,
        "label": "Ethan Lam"
      },
      "creationDate": "2010-07-13T07:00:00Z",
      "hasTransactions": [],
      "id": 1,
      "lastUpdateDate": "2010-07-13T07:00:00Z",
      "name": "Stali klienci",
      "permissions": {
        "customergroup": [
          "create",
          "read",
          "update"
    ]
    },
    "salesmen": [
      {
      "id": 5,
      "label": "Anthony Wilson"
}
]
}
],
"dir": "ASC",
"multiloggingdetected": true,
"sort": "name",
"totalSize": 1
}
```

<a name="f26"/>

### k) pobieranie notatek handlowych - salesnotes

```
https://system.firmao.pl/{idenfyfikatorOrganizacji}/svc/v1/salesnotes?start=0&limit=50&s
ort=firstName&dir=ASC&dataFormat=MEDIUM

{
  "data": [
    {
      "agreement": null,
      "attachmentsCount": 0,
      "contact": null,
      "createdBy": {
        "id": 1,
        "label": "Jacob Smith"
      },
      "creationDate": "2017-09-12T12:06:19Z",
      "customer": null,
      "customFields": null,
      "date": "2017-09-12T12:06:00Z",
      "description": "spotkanie",
      "firstForCustomer": false,
      "id": 50006,
      "lastUpdateDate": "2017-09-12T12:06:18Z",
      "mail": null,
      "offer": null,
      "permissions": {
        "salesnote": [
          "create",
          "read",
          "update"
      ]
      },
      "product": null,
      "project": null,
      "tags": [],
      "task": null,
      "transaction": null,
      "type": "MEETING"
    }
],
"dir": "ASC",
"multiloggingdetected": true,
"sort": "date",
"totalSize": 1
}
```

<a name="f27"/>

### l) pobieranie dokumentów magazynowych - storagedocs

```
https://system.firmao.pl/{idenfyfikatorOrganizacji}/svc/v1/storagedocs?start=0&limit=50&
sort=firstName&dir=ASC&dataFormat=MEDIUM

{
  "data": [
    {
      "attachmentsCount": 0,
      "bankAccount": null,
      "baseBruttoPrice": 29,
      "baseNettoPrice": 23.58,
      "basePaidValue": 0,
      "baseRestOfPaidValue": 29,
      "baseVatPrice": 5.42,
      "calculateFromGross": false,
      "contact": null,
      "createdBy": {
        "id": 1,
        "label": "Jacob Smith"
      },
      "creationDate": "2017-09-13T08:26:34Z",
      "currency": "PLN",
      "customer": {
        "emails": [
          "kompservice.firmao@mailinator.com"
      ],
      "id": 51,
      "label": "KOMP-SERVICE"
      },
      "customerAddress": {
        "city": "Pruszków",
        "country": "Polska",
        "county": null,
        "postCode": "05-800",
        "street": "Stefana Batorego 4"
      },
      "customerBankAccountNumber": "50 1020 5558 1111 1702 0600 0067",
      "customerName": "KOMP-SERVICE Jan Kowalski",
      "customFields": null,
      "dateForExchangeRate": "2017-09-13",
      "description": null,
      "documentDate": "2017-09-13",
      "documentNumber": "2017/09/001/PZ",
      "exchangeRate": null,
      "exchangeRateDate": null,
      "factor": 1,
      "hasReminder": false,
      "id": 1,
      "invoiceAnnotations": null,
      "invoiceDate": "2017-09-13",
      "issuingPerson": "Jacob Smith",
      "issuingPersonObject": {
        "firstName": "Jacob",
        "id": 1,
        "lastName": "Smith"
      },
      "lastSelectedInvoiceReport": "Przyjecie_zewnetrzne",
      "lastUpdateDate": "2017-09-13T08:26:33Z",
      "name": null,
      "nipNumber": "7291077843",
      "paid": false,
      "paidValue": 0,
      "parcelCourierName": null,
      "parcelDeliveryType": null,
      "parcelDescription": null,
      "parcelNumber": null,
      "parcelShipmentDate": null,
      "paymentDate": "2017-09-13",
      "paymentType": "CASH",
      "permissions":{
        "storagedoc":
        [
          "create",
          "read",
          "update"
      ]
      },
      "phone": "22 345 74 26",
      "rateFromCurrency": null,
      "receivingPerson": null,
      "receivingPersonObject": null,
      "recipient": null,
      "recipientAddress": null,
      "recipientName": null,
      "recipientNipNumber": null,
      "recipientPhoneNumber": null,
      "reminderAdvance": null,
      "restOfPaidValue": 29,
      "sellerAddress": {},
      "sellerBankAccountNumber": null,
      "sourceWarehouse": null,
      "storageDocBruttoPrice": 29,
      "storagedocDate": "2017-09-13",
      "storageDocNettoPrice": 23.58,
      "storageDocNumber": "2017/09/001/PZ",
      "storageDocVatPrice": 5.42,
      "storageStateUpdate": "AUTO_UPDATE",
      "tags": [],
      "task": null,
      "type": "OUTSIDE_INCOME",
      "warehouse": {
        "id": 1,
        "name": "Magazyn 1"
      }
    }
],
"dir": "ASC",
"multiloggingdetected": true,
"sort": "name",
"totalSize": 1
}
```

<a name="f28"/>

### m) pobieranie listy plików - documents

```
https://system.firmao.pl/{idenfyfikatorOrganizacji}/svc/v1/documents?start=0&limit=50&d
ataFormat=MEDIUM

{
  "data": [
    {
      "contentType": "image/png",
      62
      "createdBy": null,
      "creationDate": "2017-09-08T07:27:05Z",
      "description": null,
      "fileSize": 15861,
      "isFile": true,
      "name": "specjalista.png",
      "objectClass": "mail",
      "objectDeleted": false,
      "objectId": 23,
      "objectLabel": "Kalendarium Szkoleń - Wrzesień 2017",
      "tags": [],
      "uploaded": true,
      "url": null
    },
    {
      "contentType": "text/plain",
      "createdBy": null,
      "creationDate": "2017-09-08T07:27:08Z",
      "description": null,
      "fileSize": 117,
      "isFile": true,
      "name": "Raport_dostarczenia.txt",
      "objectClass": "mail",
      "objectDeleted": false,
      "objectId": 27,
      "objectLabel": "Mail delivery failed: returning message to sender",
      "tags": [],
      "uploaded": true,
      "url": null
    }
  ],
  "dir": "ASC",
  "sort": "name",
  "totalSize": 32
}
```

<a name="f29"/>

### n) pobieranie maili - mails

```
https://system.firmao.pl/{idenfyfikatorOrganizacji}/svc/v1/mails?start=0&limit=50&sort=s
entDate&dir=DESC&box(eq)=INBOX&dataFormat=MEDIUM

{
  "changelog": [],
  "data": [
    {
      "answerFor": null,
      "attachment": false,
      "bcc": [],
      "box": "INBOX",
      "cc": [
        "mailadress@mailinator.com"
      ],
      "contact": null,
      "createdBy": null,
      "customer": {
        "email": "dell@gmail.com",
        "id": 1,
        "label": "DELL"
      },
      "customFields": null,
      "done": false,
      "folder": null,
      "id": 58,
      "mailAccount": {
        "id": 1,
        "incomingServerProtocol": "IMAP",
        "label": "test@firmao.pl"
      },
      "mailInvitations": [],
      "mailSendingTime": null,
      "project": null,
      "read": true,
      "replyTo": "\"dell@gmail.com\" <dell@gmail.com>",
      "sender": "dell@gmail.com",
      "senderLabel": "dell@gmail.com",
      "sentDate": "2017-09-08T07:56:50Z",
      "status": "RECEIVE_SUCCEED",
      "subject": "as",
      "tags": [],
      "task": null,
      "to": [
        "test@firmao.pl"
      ]
      },
      {
      "answerFor": null,
      "attachment": false,
      "bcc": [],
      "box": "INBOX",
      "cc": [],
      "contact": null,
      "createdBy": null,
      "customer":
        {
        "email": "dell@wp.pl",
        "id": 50002,
        "label": "DELL"
        },
     "customFields": null,
        "done": false,
        "folder": null,
        "id": 24,
        "mailAccount":
        {
          "id": 2,
          "incomingServerProtocol": "IMAP",
          "label": "dell@gmail.com"
        },
        "mailInvitations": [],
        "mailSendingTime": null,
        "project": null,
        "read": true,
        "replyTo": "\"DELL\" <dell@wp.pl>",
        "sender": "dell@wp.pl",
        "senderLabel": "DELL",
        "sentDate": "2017-09-08T07:26:27Z",
        "status": "REPLIED",
        "subject": "Test Firmao",
        "tags": [],
        "task": null,
        "to": [
          "mailadress@firmao.pl",
          "dell@gmail.com"
      ]
    }
  ],
  "dir": "DESC",
  "sort": "sentDate",
  "totalSize": 51
}
```

<a name="f30"/>

### o) pobieranie serii numeracyjnych faktur

`GET: https://system.firmao.pl/{identyfikatorOrganizacji}/svc/v1/invoicenumbers?dataFormat=MEDIUM`

Zapytanie pobierze listę serii numeracyjnych - ich numery id, nazwy i szablony
numeracji dla każdego typu dokumentu (faktura zwykła / proforma / oferta / zamówienie
itd.)

Pobrane w ten sposób id można wykorzystać na przykład do pobrania następnego w
kolejności numeru faktury:

`GET:
https://system.firmao.pl/{identyfikatorOrganizacji}/svc/v1/invoicenumbers/next?objectClass={nazwaObiekt
u}&type={typDokumentu}&mode={trybSprzedaży}&id={idSeriiNumeracyjnej}`

id: uzyskujemy z poprzedniego zapytania

mode:
- SALE - dokument sprzedażowy
- PURCHASE - dokument zakupowy

objectClass wraz ze słownikiem type:
- transaction (Transakcja)
  - invoice (Faktura)
  - receipt (Paragon)
  - proforma (Faktura pro forma)
  - correction (Faktura korekta)
  -  bill (Rachunek)
  - invoicemargin (Faktura marża)
  - advance_invoice (Faktura zaliczkowa)
- agreement (Umowa)
  - agreement (Umowa)
- offer (Oferta/Zamówienie)
  - offer (Oferta)
  - order (Zamówienie)
- storagedoc (Dokument magazynowy)
  - outside_income (Przyjęcie zewnętrzne)
  - outside_release (Wydanie zewnętrzne),
  - inside_release (Rozchód wewnętrzny),
  - inside_income (Przychód wewnętrzny),
  - warehouse_transfer (Przesunięcie międzymagazynowe)
 
Przykładowo, aby dla organizacji stellis, dla linii numeracyjnej o id 2 pobrać kolejny
numer faktury proforma sprzedażowej, należy wysłać następujące zapytanie:

```
GET:
https://system.firmao.pl/stellis/svc/v1/invoicenumbers/next?objectClass=transaction&type=proforma&mod
e=SALE&id=2
```

Dla tej samej linii numeracyjnej i organizacji, pobranie kolejnego numeru zamówienia
sprzedażowego to:

```
GET:
https://system.firmao.pl/stellis/svc/v1/invoicenumbers/next?objectClass=offer&type=order&mode=SALE&i
d=2
```

Pobranie kolejnego numeru umowy:

```
GET:
https://system.firmao.pl/stellis/svc/v1/invoicenumbers/next?objectClass=agreement&type=agreement&mo
de=SALE&id=2
```

<a name="DELETE"/>

## 5. Kasowanie obiektów (PUT, DELETE)

W firmao większość obiektów nie jest kasowana tylko oznaczana jako skasowane i można je
przywrócić. Takie obiekty kasujemy poprzez modyfikacje pola deleted na true tak jak jest to
opisane w rozdziale 3.

Pozostałe obiekty, (które nie posiadają pola deleted) usuwamy metodą DELETE. Są to między
innymi:
- przypomnienia
- załączniki
- pozycje transakcji

Tych obiektów nie można przywrócić.

<a name="time"/>

## 6. Dokładny opis tworzenia, edycji, kończenia i rozpoczynania czasów pracy

Utworzenie nowego czasu pracy <b>tylko z godzina rozpoczęcia</b>:

`https://system.firmao.pl/{idenfyfikatorOrganizacji}/svc/v1/timeentries`

```
POST
{
    "type":"TASK",
    "user":1,
    "dateTimeFrom":"2017-09-15T08:30:00+01:00"
}
```

Odpowiedź:

`{ "changelog" :
[{"deletedWith":"","modification":"ATTACH","newValueClass":"timeentry","newValueDispl
ay":"Jacob Smith, CTE\nod
{[2017-09-15T07:30:00.000Z]}\n","newValueId":"50007","newValueIsDictionary":false,"
objectClass":"user","objectId":1,"objectLabel":"Jacob
Smith","oldValueIsDictionary":false,"property":"timeEntry"}]}`

W odpowiedzi <b>"newValueId":"50007"</b> to identyfikator nowo utworzonego czasu pracy

Utworzenie nowego czasu pracy <b>tylko z godzina zakończenia</b>:

`https://system.firmao.pl/{idenfyfikatorOrganizacji}/svc/v1/timeentries`

```
POST
{
"type":"TASK",
"user":1,
"dateTimeTo":"2017-09-15T16:30:00+01:00"
}
```

Odpowiedź:

`{ "changelog" :
[{"deletedWith":"","modification":"ATTACH","newValueClass":"timeentry","newValueDispl
ay":"Jacob Smith,
CTE\ndo{[2017-09-15T16:30:00.000Z]}\n","newValueId":"50008","newValueIsDictionar
y":false,"objectClass":"user","objectId":1,"objectLabel":"Jacob
Smith","oldValueIsDictionary":false,"property":"timeEntry"}]}`

W odpowiedzi <b>"newValueId":"50008"</b> to identyfikator nowo utworzonego czasu pracy

Zakończenie <b>istniejącego</b> czasu pracy

`https://system.firmao.pl/{idenfyfikatorOrganizacji}/svc/v1/timeentries/50007`

```
PUT
{
"dateTimeTo":"2017-09-15T11:30:00+01:00"
}
```

Odpowiedź:

`{"changelog":[{"deletedWith":"","modification":"CHANGE","newValueClass":"timeentry","
newValueDisplay":"Emma Clark, LEAVE {[2017-09-14T22:00:00.000Z]} -
{[2017-09-15T10:30:00.000Z]}","newValueId":"-1","newValueIsDictionary":false,"objectCl
ass":"user","objectId":6,"objectLabel":"Emma
Clark","oldValueClass":"timeentry","oldValueDisplay":"Emma Clark, LEAVE
{[2017-09-14T22:00:00.000Z]} -
{[2017-09-15T22:00:00.000Z]}","oldValueId":"-1","oldValueIsDictionary":false,"property":"
timeEntry"}]}`

Utworzenie nowego czasu pracy <b>z godziną rozpoczęcia i zakończenia:</b>

`https://system.firmao.pl/{idenfyfikatorOrganizacji}/svc/v1/timeentries`

```
POST
{
"type":"TASK",
"user":1,
"dateTimeFrom":"2017-09-15T08:30:00+01:00",
"dateTimeTo":"2017-09-15T12:00:00+01:00"
}
```

`user` - identyfikator użytkownika w systemie firmao (numer), któremu tworzymy czas
pracy

Odpowiedź:

`{"changelog":[{"deletedWith":"","modification":"ATTACH","newValueClass":"timeentry","n
ewValueDisplay":"Jacob Smith, TASK {[2017-09-15T07:30:00.000Z]} -
{[2017-09-15T11:00:00.000Z]}","newValueId":"50037","newValueIsDictionary":false,"obj
ectClass":"user","objectId":1,"objectLabel":"Jacob
Smith","oldValueIsDictionary":false,"property":"timeEntry"}]}`

<a name="PDF"/>

## 7.Pobieranie plików PDF

<a name="f30"/>

### a)Faktury

`https://system.firmao.pl/{idenfyfikatorOrganizacji}/svc/v1/reports/invoices/transaction/Fa
ktura/{nazwa_pliku}.pdf?original=true&id=1`

Gdzie _<b>Faktura</b>_ to nazwa użytego szablonu, a nazwa pliku to nazwa jaką będzie miał
wygenerowany plik.

<a name="f31"/>

### b)Oferty/zamówienia/umowy

`https://system.firmao.pl/{idenfyfikatorOrganizacji}/svc/v1/reports/invoices/offer/Oferta/{n
azwa_pliku}.pdf?id=1`

Gdzie _<b>Oferta</b>_ to nazwa użytego szablonu, a nazwa pliku to nazwa jaką będzie miał
wygenerowany plik.

<a name="codes"/>

## Kody odpowiedzi

API korzysta ze standardowych kodów HTTP:

`2xx` - operacja bezbłędna

`4xx` - nastąpił błąd

`5xx` - nastąpił błąd

Szczegółowy opis niektórych błędów:

Poprawne żądanie:

`200` - poprawne pobranie bądź edycja błędów

`201` - poprawne utworzenie obiektu

Błędne żądanie:

`400` - błąd wewnętrzny serwera

`401` - żądanie wymaga autoryzacji

`403` - brak uprawnień dostępu

`404` - zasób nieznaleziony

`409` - konflikt, operacja anulowana

`500` - nieoczekiwany błąd.

`501` - funkcja niezaimplementowana

`502` - błąd serwera pośredniczącego (proxy)

W przypadku błędów standardowo pojawia się zrozumiały opis błędu.

Na przykład:

`{"errorCode":"entityNotFoundForId","field":"project","rejectedValue":"50","additionalInfo":{"id":"50
","cls":"project"}}`

<a name="PHP"/>

## 9.Przykładowe skrypty php

<b>UWAGA: Przy kopiowaniu kodu, często ucina myślniki, lub dodaje niepotrzebne
znaki nowej linii.</b>

<a name="f31"/>

### a)dodawanie kontrahenta

```
<?php

$usr = base64_encode("login:haslo");
$url = 'https://system.firmao.pl/{idenfyfikatorOrganizacji}/svc/v1/customers’;

$content = array(
"bankAccountNumber" => "50 1020 5558 1111 1702 0600 0067",
"correspondenceAddress.city" => "Pruszków",
"correspondenceAddress.country" => "Polska",
"correspondenceAddress.postCode" => "05-800",
"correspondenceAddress.street" => "Stefana Batorego 10/4",
"customerGroups" => array(),
"customerType" => "PARTNER",
"description" => "Firma specjalizująca się w serwisie komputerów",
"emails" => array("kompservice.firmao@mailinator.com",
"komputery@mailinator.com"),
"employeesNumber" => "1",
"faxNumber" => "22 398 78 87",
"label" => "KOMP-SERVICE",
"name" => "KOMP-SERVICE Jan Kowalski",
"nipNumber" => "7291077843",
"officeAddress.city" => "Pruszków",
"officeAddress.country" => "Polska",
"officeAddress.postCode" => "05-800",
"officeAddress.street" => "Stefana Batorego 3/4",
"ownership" => "PRIVATE",
"phones" => "223457426",
"status" => null,
"tags" => array(),
"voivodeship" => null,
"website" => "www.kompservice.bis");

$json = json_encode($content);
echo $json.'<br /><nr />';
$curl = curl_init($url);

curl_setopt($curl, CURLOPT_CUSTOMREQUEST, "POST");
curl_setopt($curl, CURLOPT_HEADER, false);
curl_setopt($curl, CURLOPT_RETURNTRANSFER, true);
curl_setopt($curl, CURLOPT_HTTPHEADER,
array("Content-type: application/json; charset=UTF-8", "Authorization: Basic $usr"));
curl_setopt($curl, CURLOPT_POSTFIELDS, $json);
```

> akceptowanie certyfikatów

```
curl_setopt ($curl, CURLOPT_SSL_VERIFYHOST, 0);
curl_setopt ($curl, CURLOPT_SSL_VERIFYPEER, 0);

$json_response = curl_exec($curl);
$status = curl_getinfo($curl, CURLINFO_HTTP_CODE);
if ( $status != 201 && $status != 200) {
die("Error: call to URL $url failed with status $status, response $json_response,
curl_error " . curl_error($curl) . ", curl_errno " . curl_errno($curl));
}
curl_close($curl);
$response = json_decode($json_response, true);
?>
```

<a name="f32"/>

### b)Pobranie danych kontrahenta

```
<?php
$usr = base64_encode("login:haslo");
$url = 'https://system.firmao.pl/{idenfyfikatorOrganizacji}/svc/v1/customers’;
$curl = curl_init($url);

curl_setopt($curl, CURLOPT_CUSTOMREQUEST, "GET");
curl_setopt($curl, CURLOPT_HEADER, false);
curl_setopt($curl, CURLOPT_RETURNTRANSFER, true);
curl_setopt($curl, CURLOPT_HTTPHEADER,
array("Content-type: application/json; charset=UTF-8", "Authorization: Basic $usr"));
curl_setopt($curl, CURLOPT_POSTFIELDS, $json);
//akceptowanie certyfikatów
curl_setopt ($curl, CURLOPT_SSL_VERIFYHOST, 0);
curl_setopt ($curl, CURLOPT_SSL_VERIFYPEER, 0);
$json_response = curl_exec($curl);
$status = curl_getinfo($curl, CURLINFO_HTTP_CODE);
if ( $status != 201 && $status != 200) {
die("Error: call to URL $url failed with status $status, response $json_response,
curl_error " . curl_error($curl) . ", curl_errno " . curl_errno($curl));
}
curl_close($curl);
echo $json_response;
$response = json_decode($json_response, true);
?>
```
<a name="f33"/>

### c) dodawanie transakcji

```
<?php

$usr = base64_encode("login:haslo");
$url = 'https://system.firmao.pl/{idenfyfikatorOrganizacji}/svc/v1/transactions";

$content = array(
"transactionEntries"=>array(array(
"classificationNumber"=>"",
"discount"=>null,
"productCode"=>"ST0001",
"quantity"=>1,
"unit"=>null,
"vatPercent"=>0,
"product"=>51,
"unitNettoPrice"=>"100.00"
)),
"transactionNettoPrice"=>"1000.00",
"transactionBruttoPrice"=>"1000.00",
"transactionVatPrice"=>"0.00",
"invoiceAnnotations"=>null,
"paid"=>false,
"currency"=>"PLN",
"paymentType"=>"CASH",
"restOfPaidValue"=>1000,
"paidValue"=>0,
"invoiceDate"=>"2012-08-08",
"paymentDate"=>"2012-08-08",
"transactionDate"=>"2012-08-08",
"type"=>"INVOICE",
"mode"=>"SALE",
"customerAddress.postCode"=>"91477",
"customer"=>50,
"customerAddress.city"=>"Łódź",
"customerAddress.country"=>"Polska",
"customerAddress.street"=>"Piotrkowska 147/8",
"nipNumber"=>"1060000062",
"issuingPerson"=>"Jacob Smith",
"transactionNumber"=>"12/8/2012"
);
$json = json_encode($content);
echo $json.'<br /><nr />';
$curl = curl_init($url);
curl_setopt($curl, CURLOPT_CUSTOMREQUEST, "POST");
curl_setopt($curl, CURLOPT_HEADER, false);
curl_setopt($curl, CURLOPT_RETURNTRANSFER, true);
curl_setopt($curl, CURLOPT_HTTPHEADER,array("Content-type:application/json;
charset=UTF8","Authorization: Basic $usr"));
curl_setopt($curl, CURLOPT_POSTFIELDS, $json);
//akceptowanie certyfikatów
curl_setopt ($curl, CURLOPT_SSL_VERIFYHOST, 0);
curl_setopt ($curl, CURLOPT_SSL_VERIFYPEER, 0);
$json_response = curl_exec($curl);
$status = curl_getinfo($curl, CURLINFO_HTTP_CODE);
if ( $status != 201 && $status != 200) {
die("Error: call to URL $url failed with status $status, response $json_response,
curl_error " . curl_error($curl) . ", curl_errno " . curl_errno($curl));
}
curl_close($curl);
$response = json_decode($json_response, true);
?>
```

<a name="f34"/>

### d)Wgrywanie pliku php <7.0

```
<?php

$usr = base64_encode("login:haslo");
$url =
'https://system.firmao.pl/{idenfyfikatorOrganizacji}/svc/v1/{object}/{id}/documentsUpload”
;
$curl = curl_init($url);
$parameters = array(
'description'=>date('Y-m-d'),
'file'=>'@'.'c:\test.txt');

curl_setopt($curl, CURLOPT_CUSTOMREQUEST, "POST");
curl_setopt($curl, CURLOPT_POSTFIELDS, $parameters);
curl_setopt($curl, CURLOPT_HEADER, false);
curl_setopt($curl, CURLOPT_RETURNTRANSFER, true);
curl_setopt($curl,
CURLOPT_HTTPHEADER,array("Content-type:multipart/form-data;
charset=UTF8","Authorization: Basic $usr"));
curl_setopt($curl, CURLOPT_SSL_VERIFYHOST, 0);
curl_setopt($curl, CURLOPT_SSL_VERIFYPEER, 0);

$json_response = curl_exec($curl);
$status = curl_getinfo($curl, CURLINFO_HTTP_CODE);
if ($status != 201 && $status != 200) {
die("Error: call to URL $url failed with status $status, response $json_response,
curl_error " . curl_error(
$curl
) . ", curl_errno " . curl_errno($curl));
}
curl_close($curl);
$response = json_decode($json_response, true);
?>
```

<a name="f35"/>

### e)Wgrywanie pliku php >=7.0

```
<?php

$usr = base64_encode("login:haslo");
$url =
'https://system.firmao.pl/{idenfyfikatorOrganizacji}/svc/v1/{object}/{id}/documentsUpload”
;
$curl = curl_init($url);

$parameters = array(
'file' => new CURLFile("$filePath")
81
);

curl_setopt($curl, CURLOPT_CUSTOMREQUEST, 'POST');
curl_setopt($curl, CURLOPT_HEADER, false);
curl_setopt($curl, CURLOPT_RETURNTRANSFER, true);
curl_setopt($curl, CURLOPT_HTTPHEADER, array("Authorization: Basic $usr"));
curl_setopt($curl, CURLOPT_POSTFIELDS, $data);
curl_setopt($curl, CURLOPT_SSL_VERIFYHOST, 0);
curl_setopt($curl, CURLOPT_SSL_VERIFYPEER, 0);

$json_response = curl_exec($curl);
$status = curl_getinfo($curl, CURLINFO_HTTP_CODE);
if ($status != 201 && $status != 200) {
die("Error: call to URL $url failed with status $status, response $json_response,
curl_error " . curl_error(
$curl
) . ", curl_errno " . curl_errno($curl));
}
curl_close($curl);
$response = json_decode($json_response, true);
?>
```

<a name="MORE"/>

## 10.Więcej opcji API

Jeżeli potrzebujesz więcej opcji API lub masz problemy z konfiguracją, napisz do nas
na adres kontakt@firmao.pl
