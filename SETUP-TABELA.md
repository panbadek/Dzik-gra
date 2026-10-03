# Wspólna tabela wyników — konfiguracja (ok. 3 minuty)

Gra jest statyczną stroną na GitHub Pages, więc żeby **wszystkie telefony
widziały jedną tabelę**, potrzebna jest darmowa baza w chmurze. Poniżej
najprostsza opcja: Firebase Realtime Database (konto Google, bez karty).

Dopóki tego nie ustawisz, gra działa normalnie — tabela jest wtedy tylko
na danym telefonie i gra wprost to pisze pod tabelą.

## Krok po kroku

1. Wejdź na https://console.firebase.google.com i zaloguj się kontem Google.
2. **Add project** → nazwa np. `dzik-gra` → Google Analytics możesz wyłączyć → **Create project**.
3. W menu po lewej: **Build → Realtime Database** → **Create Database**.
   - Lokalizacja: wybierz europejską (np. `europe-west1`).
   - Reguły: wybierz **Start in test mode** (poprawimy je w kroku 5).
4. Skopiuj adres bazy z góry strony. Wygląda tak:
   `https://dzik-gra-default-rtdb.europe-west1.firebasedatabase.app`
5. Zakładka **Rules** → wklej poniższe i kliknij **Publish**.
   Pozwalają każdemu czytać i dopisywać wynik, ale wymuszają poprawny
   kształt danych, więc nikt nie wrzuci tam śmieci:

```json
{
  "rules": {
    "boards": {
      "$mode": {
        ".read": true,
        "$nick": {
          ".write": true,
          ".validate": "newData.hasChildren(['nick','score'])",
          "nick":  { ".validate": "newData.isString() && newData.val().length <= 12" },
          "score": { ".validate": "newData.isNumber() && newData.val() >= 0 && newData.val() <= 100000" },
          "cones": { ".validate": "newData.isNumber() && newData.val() >= 0 && newData.val() <= 100000" },
          "ts":    { ".validate": "newData.isNumber()" },
          "$other": { ".validate": false }
        }
      }
    }
  }
}
```

6. W pliku `index.html` znajdź linię (jest blisko komentarza
   `SHARED SCORE TABLE`):

```js
var CLOUD_URL = '';
```

   i wklej w cudzysłów adres z kroku 4:

```js
var CLOUD_URL = 'https://dzik-gra-default-rtdb.europe-west1.firebasedatabase.app';
```

7. Zapisz, zrób commit i push. Gotowe — pod tabelą pojawi się
   „🌍 wspólna tabela — wszystkie telefony".

## Czego się spodziewać

- Jeden wiersz na nick: gracz ma jeden wynik niezależnie od tego, na ilu
  telefonach gra. Słabszy przebieg nie nadpisuje lepszego.
- Brak internetu w telefonie → gra pokazuje ostatnio wczytaną tabelę
  i pisze „📡 brak połączenia". Wynik i tak zapisze się lokalnie.
- Darmowy limit Firebase (1 GB transferu/mies.) jest przy tej skali
  nie do wyczerpania — jeden odczyt tabeli to kilkaset bajtów.

## Uwaga o bezpieczeństwie

Adres bazy jest widoczny w źródle strony — tak musi być, bo to przeglądarka
gracza łączy się z bazą. Reguły z kroku 5 ograniczają, **co** można zapisać
(poprawny kształt, limity długości i wartości), ale ktoś uparty nadal może
dopisać wymyślony wynik. Przy grze dla drużyny to akceptowalne; gdyby kiedyś
przeszkadzało, trzeba by dołożyć logowanie albo własny serwer pośredniczący.
