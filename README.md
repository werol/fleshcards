
# Flashcards

Aplikacja służąca do tworzenia własnych zestawów fiszek.

## Informacje ogólne

Głównym założeniem projektu jest synchronizacja bazy lokalnej IndexedDB (która umożliwa użytkownikom korzystanie z aplikacji, gdy są offline) z bazą zdalną. Dokładne działanie aplikacji jest opisane w dalszej części.

Podfolder `backend` zawiera kod napisany w Javie, natomiast w podfolderze `frontend` znajduje się kod w JavaScript (w projekcie wykorzystano bibliotekę React oraz Redux do zarządzania
stanem aplikacji i akcjami).

Pracując nad projektem, aby przyspieszyć pracę, zastosowano `hot reloading` - gdy aplikacja uruchamiana jest na serwerze developerskim, zmiany w plikach frontendowych
oraz w stylach przeładowywują się autoamtycznie bez odświeżania przeglądarki.


## Uruchamianie aplikacji w trybie developerskim (hot reloading)

#### Wymagania
* node 6.0+
* yarn (by możliwe było uruchomienie webpack dev server)
* stworzenie bazy danych MySQL (skrypt w głównym folderze projektu) oraz konfiguracja w pliku `backend/src/main/resources/application.properties`

Uruchomienie metody głównej w klasie FlashcardsManagerApplication spowoduje, że aplikacja będzie dostępna pod adresem:
 
``
http://localhost:3000/
`` 

## Uruchomianie aplikacji w trybie produkcyjnym

Aby uruchomić aplikację w trybie produkcyjnym należy wygenerować plik jar:

``
$ ./gradlew assemble
``

Plik wygeneruje się do folderu `backend/build/libs`, następnie należy go uruchomić komendą:

``
$ java -jar flashcards-0.0.1-SNAPSHOT.jar --spring.profiles.active=production
``

Aplikacja zostanie uruchomiona pod adresem:

``
http://localhost:8080/
`` 

## Uruchomianie aplikacji w trybie produkcyjnym

Aby uruchomić aplikację w trybie produkcyjnym należy wygenerować plik jar:

``
$ ./gradlew assemble
``

Plik wygeneruje się do folderu `backend/build/libs`, następnie należy go uruchomić komendą:

``
$ java -jar flashcards-0.0.1-SNAPSHOT.jar --spring.profiles.active=production
``

Aplikacja zostanie uruchomiona pod adresem:

``
http://localhost:8080/
`` 

## Dokumentacja

### Spis treści
+ [Dokumentacja API](#docAPI)
  + [Użytkownik](#user)
    + [Rejestracja użytkownika](#register)
    + [Logowanie](#login)
    + [Informacje o sesji](#session)
    + [Wylogowywanie](#logout)
    + [Lista wszystkich użytkowników](#users)
  + [Zestawy fiszek](#flashcardSets)
    + [Tworzenie zestawu](#create)
    + [Edycja zestawu](#update)
    + [Usuwanie zestawu](#delete)
    + [Zestaw](#set)
    + [Lista wszystkich zestawów](#sets)
    + [Synchronizacja](#synchronize)
+ [Zrzuty ekranu](#screenshots)
+ [Wykorzystane wzorce](#patterns)
  + [Optimistic Offline Lock](#optimistic)
  + [Wzorzec modułu](#module)
  + [Strategia](#strategy)
  + [Builder](#builder)
  
<a name="docAPI"/>

## Dokumentacja API  
  
<a name="user"/>

### Użytkownik

<a name="register"/>

#### Rejestracja użytkownika

*URL:*  `/api/register`

 *Metoda:* `POST`

*Przykład:*
```javascript
axios.post('/api/register', {
    firstName: 'Fred',
    lastName: 'Flintstone',
    username: 'Fred41'
    email: 'fredflintstone@yahoo.com'
    password: 'IloveWilma123'
  })
  .then(response => {
    console.log(response);
  })
  .catch(error => {
    console.log(error);
  });
```
*Odpowiedź:*

Kod odpowiedzi HTTP: `200 OK`
```javascript
{
    "username": "Fred41",
    "firstName": "Fred",
    "lastName": "Flintstone",
    "email": "fredflintstone@yahoo.com",
    "role": "USER"
}
```
*Zwracane błędy:*

Warunek: Przesyłane dane nie spełniają kryteriów (np. hasło ma mniej niż 8 znaków, nie zawiera dużej litery i cyfry), lub obiekt JSON nie zawiera wszystkich wymaganych pól

Kod odpowiedzi HTTP: `400 Bad Request`

```javascript
{
    "messageKey": "userData.error.badRequest"
}
```
Warunek: Przesyłany username już istnieje

Kod odpowiedzi HTTP: `400 Bad Request`
```javascript
{
    "messageKey": "register.error.usernameExists"
}
```
Warunek: Przesyłany email już istnieje

Kod odpowiedzi HTTP: `400 Bad Request`
```javascript
{
    "messageKey": "register.error.emailExists"
}
```

<a name="login"/>

#### Logowanie

*URL:*  `/api/session`

 *Metoda:* `POST`

*Przykład:*
```javascript
axios.post('/api/login', {
    username: 'Fred41'
    password: 'IloveWilma123'
  })
  .then(response => {
    console.log(response);
  })
  .catch(error => {
    console.log(error);
  });
```
*Odpowiedź:*

Kod odpowiedzi HTTP: `200 OK`
```javascript
{
    "username": "Fred41",
    "token": "6D55B805C77A21230EA9269091FFBE75",
    "authenticated": true
}
```
*Zwracane błędy:*

Warunek: Niepoprawny login lub hasło

Kod odpowiedzi HTTP: `400 Bad Request`

```javascript
{
    "messageKey": "login.error.badLogin"
}
```
<a name="session"/>

#### Informacje o sesji

*URL:*  `/api/session`

 *Metoda:* `GET`

*Przykład:*
```javascript
axios.get('/api/session')
  .then(response => {
    console.log(response);
  })
  .catch(error => {
    console.log(error);
  });
```
*Odpowiedź:*

Kod odpowiedzi HTTP: `200 OK`
```javascript
{
    "username": "Fred41",
    "token": "6D55B805C77A21230EA9269091FFBE75",
    "authenticated": true
}
```
<a name="logout"/>

#### Wylogowywanie

*URL:*  `/api/logout`

 *Metoda:* `DELETE`

*Przykład:*
```javascript
axios.delete('/api/logout')
  .then(response => {
    console.log(response);
  })
  .catch(error => {
    console.log(error);
  });  
```

<a name="users"/>

#### Lista wszystkich użytkowników

*URL:*  `/api/users`

 *Metoda:* `GET`

*Przykład:*
```javascript
axios.get('/api/users')
  .then(response => {
    console.log(response);
  })
  .catch(error => {
    console.log(error);
  });  
```

*Odpowiedź:*

Kod odpowiedzi HTTP: `200 OK`
```javascript
[
    {
        "username": "Fred41",
        "firstName": "Fred",
        "lastName": "Flintstone",
        "email": "fredflintstone@yahoo.com",
        "role": "USER"
    },
    {
        "username": "Wilma",
        "firstName": "Wilma",
        "lastName": "Flintstone",
        "email": "wilmaflintstone@yahoo.com",
        "role": "USER"
    }
]
```
<a name="flashcardSets"/>

### Zestawy fiszek

<a name="create"/>

#### Tworzenie zestawu

*URL:*  `/api/flashcards`

 *Metoda:* `POST`

*Przykład:*
```javascript
axios.post('/api/flashcards', {
    owner: 'Fred41',
    name: 'Animals',
    flashcards: [
      {
        frontSide: 'cat',
        backSide: 'kot',
      }, 
      {
        frontSide: 'dog',
        backSide: 'pies',
      }
    ]
  })
  .then(response => {
    console.log(response);
  })
  .catch(error => {
    console.log(error);
  });  
```
*Odpowiedź:*

Kod odpowiedzi HTTP: `200 OK`
```javascript
{
    "setId": 2,
    "version": 0,
    "owner": "Fred41",
    "name": "Animals",
    "lastModified": 1517603827715,
    "flashcards": [
        {
            "flashcardId": 13,
            "frontSide": "cat",
            "backSide": "kot"
        },
        {
            "flashcardId": 14,
            "frontSide": "dog",
            "backSide": "pies"
        }
    ]
}
```
*Zwracane błędy:*

Warunek: Obiekt JSON nie zawiera wszystkich wymaganych pól

Kod odpowiedzi HTTP: `400 Bad Request`

```javascript
{
    "messageKey": "flashcards.error.badRequest"
}
```

<a name="update"/>

#### Edycja zestawu

*URL:*  `/api/flashcards`

 *Metoda:* `PUT`

*Przykład:*
```javascript
axios.put('/api/flashcards', {
    setId: 2,
    version: 0,
    owner: 'Fred41',
    name: 'Animals',
    flashcards: [ 
      {
        frontSide: 'fish',
        backSide: 'ryba',
      }
    ]
  })
  .then(response => {
    console.log(response);
  })
  .catch(error => {
    console.log(error);
  });  
```
*Odpowiedź:*

Kod odpowiedzi HTTP: `200 OK`
```javascript
{
    "setId": 2,
    "version": 1,
    "owner": "Fred41",
    "name": "Animals",
    "lastModified": 1517604771815,
    "flashcards": [
        {
            "flashcardId": 15,
            "frontSide": "fish",
            "backSide": "ryba"
        }
    ]
}
```
*Zwracane błędy:*

Warunek: Obiekt JSON nie zawiera wszystkich wymaganych pól

Kod odpowiedzi HTTP: `400 Bad Request`

```javascript
{
    "messageKey": "flashcards.error.badRequest"
}
```
<a name="delete"/>

#### Usuwanie zestawu

*URL:*  `/api/flashcards/{setId}`

 *Metoda:* `DELETE`

*Przykład:*
```javascript
axios.delete('/api/flashcards/2')
  .then(response => {
    console.log(response);
  })
  .catch(error => {
    console.log(error);
  });  
```
*Odpowiedź:*

Kod odpowiedzi HTTP: `200 OK`

*Zwracane błędy:*

Warunek: Set o podanym id nie istnieje

Kod odpowiedzi HTTP: `404 Not Found`

<a name="set"/>

#### Zestaw

*URL:*  `/api/flashcards/{setId}`

 *Metoda:* `GET`

*Przykład:*
```javascript
axios.get('/api/flashcards/2')
  .then(response => {
    console.log(response);
  })
  .catch(error => {
    console.log(error);
  });  
```

*Odpowiedź:*

Kod odpowiedzi HTTP: `200 OK`

```javascript
{
    "setId": 2,
    "version": 1,
    "owner": "Fred41",
    "name": "Animals",
    "lastModified": 1517604771815,
    "flashcards": [
        {
            "flashcardId": 15,
            "frontSide": "fish",
            "backSide": "ryba"
        }
    ]
}
```

*Zwracane błędy:*

Warunek: Set o podanym id nie istnieje

Kod odpowiedzi HTTP: `404 Not Found`

<a name="sets"/>

#### Lista wszystkich zestawów

*URL:*  `/api/flashcards`

*Metoda:* `GET`

*Przykład:*
```javascript
axios.get('/api/flashcards')
  .then(response => {
    console.log(response);
  })
  .catch(error => {
    console.log(error);
  });  
```
*Odpowiedź:*

Kod odpowiedzi HTTP: `200 OK`
```javascript
{
    "flashcardSets": [
        {
            "setId": 2,
            "version": 1,
            "owner": "Fred41",
            "name": "Animals",             
            "lastModified": 1517604771815,
            "flashcards": [
                {
                    "flashcardId": 15,
                    "frontSide": "fish",
                    "backSide": "ryba"
                }
            ]
        },
        {
            "setId": 3,
            "version": 1,
            "owner": "Fred41",
            "name": "Numbers",
            "lastModified": 1517431161000,
            "flashcards": [
                {
                    "flashcardId": 5,
                    "frontSide": "one",
                    "backSide": "jeden"
                }
            ]
        }
    ],
    "version": 1517602995656
}
```
<a name="synchronize"/>

#### Synchronizacja

*URL:*  `/api/synchronize`

*Metoda:* `POST`

*Przykład:*
```javascript
axios.post('/api/synchronize', {
    version: 1517431172887,	
    flashcardSets: [{
      setId: 1,
      version: 1,
      name: 'Animals',
      flashcards: [{
        frontSide: 'cat',
        backSide: 'kot'
      }]
    }, {
      name: 'Numbers',
      flashcards: [{
        frontSide: 'one',
        backSide: 'jeden'
      }, {
        frontSide: 'two',
        backSide: 'dwa'
      }]
    }]
  })
  .then(response => {
    console.log(response);
  })
  .catch(error => {
    console.log(error);
  });  
```
*Odpowiedź:*

Kod odpowiedzi HTTP: `200 OK`
```javascript
{
    "flashcardSets": [
        {
            "setId": 2,
            "version": 2,
            "owner": "Fred41",
            "name": "Animals",             
            "lastModified": 1517604771815,
            "flashcards": [
                {
                    "flashcardId": 15,
                    "frontSide": "fish",
                    "backSide": "ryba"
                }
            ]
        }
    ],
    "version": 1517602995656
}
```
<a name="screenshots"/>

## Zrzuty ekranu 

![alt text](https://image.ibb.co/ga6hjR/1.png)
![alt text](https://image.ibb.co/kq8gB6/10.png)
![alt text](https://image.ibb.co/dLYgB6/11.png)
![alt text](https://image.ibb.co/hggtr6/2.png)
![alt text](https://image.ibb.co/jv14ym/3.png)
![alt text](https://image.ibb.co/cTWF4R/4.png)
![alt text](https://image.ibb.co/h2kmB6/5.png)
![alt text](https://image.ibb.co/iPO6B6/6.png)
![alt text](https://image.ibb.co/mEDv4R/7.png)
![alt text](https://image.ibb.co/mRszW6/8.png)
![alt text](https://image.ibb.co/d8Btr6/9.png)

<a name="patterns"/>

## Wykorzystane wzorce

<a name="optimistic"/>

### Optimistic Offline Lock

`backend/src/main/java/flashcards/service/FlashcardSetServiceImpl.java`

<a name="module"/>

### Wzorzec modułu

`frontend/src/indexedDB/dbHandler.js`

<a name="strategy"/>

### Strategia

`frontend/src/ui/handlingIndexedDB/HandlingIndexedDBStrategy.js`
`frontend/src/ui/handlingIndexedDB/OnlineStrategy.js`
`frontend/src/ui/handlingIndexedDB/OfflineStrategy.js`

<a name="builder"/>

### Builder

`frontend/src/ui/component/forms/FormFieldBuilder.js`
