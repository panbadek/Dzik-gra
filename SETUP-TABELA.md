# Wspólna tabela wyników — stan i ostatni krok

Adres bazy jest już wpisany w `index.html`:

```js
var CLOUD_URL = 'https://dzikgoniharcerza-default-rtdb.firebaseio.com';
```

Zostaje **jedna rzecz do sprawdzenia: reguły bazy.**

## Dlaczego to ważne

Jeśli przy zakładaniu bazy wybrałeś **„Start in test mode"**, Firebase wpisał
reguły z datą ważności — działają przez 30 dni, a potem **baza przestaje
przyjmować i oddawać dane**. Tabela zacznie wtedy pokazywać
„🔒 baza odrzuca połączenie". Dlatego warto podmienić je teraz na stałe.

## Reguły do wklejenia

Firebase Console → Twój projekt → **Realtime Database** → zakładka **Rules** →
zaznacz wszystko, wklej poniższe, **Publish**:

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

Bez daty ważności, a przy tym wymuszają poprawny kształt danych: nick to tekst
do 12 znaków, wynik to liczba w rozsądnym zakresie, nic innego nie przejdzie.

## Jak sprawdzić, czy działa

Otwórz grę na telefonie, zagraj jedną rundę i spójrz **pod tabelę** na ekranie
końcowym:

| Napis | Znaczenie |
|---|---|
| 🌍 wspólna tabela — wszystkie telefony | działa, wyniki idą do bazy |
| 🔒 baza odrzuca połączenie — sprawdź reguły w Firebase | reguły złe albo wygasły → wklej te wyżej |
| 📡 brak połączenia | telefon nie ma internetu; wynik zapisał się lokalnie |
| 📱 tabela tylko na tym telefonie | `CLOUD_URL` jest pusty (nie dotyczy, jest wpisany) |

Możesz też podejrzeć dane wprost w Firebase Console → Realtime Database →
zakładka **Data**: po pierwszej rundzie pojawi się tam gałąź
`boards/chase/<nick>`.

## Jak to działa

- **Jeden wiersz na nick** — gracz ma jeden wynik niezależnie od tego, na ilu
  telefonach gra.
- **Słabszy przebieg nie nadpisuje lepszego** — przed zapisem gra czyta wynik
  z bazy i zapisuje tylko wtedy, gdy nowy jest lepszy.
- **Osobna tabela dla każdego trybu** (`boards/chase`, `boards/pursuit`,
  `boards/camp`) — metry i zadania nie są porównywalne.
- **Bez SDK**, zwykłe `fetch` — gra zostaje jednym plikiem bez budowania.
- Brak internetu nie psuje gry: pokazuje ostatnio wczytaną tabelę, a wynik
  i tak ląduje w pamięci telefonu.

## Uwaga o bezpieczeństwie

Adres bazy jest widoczny w źródle strony — inaczej się nie da, bo to
przeglądarka gracza łączy się z bazą. Reguły wyżej ograniczają, **co** można
zapisać, ale ktoś uparty nadal może dopisać zmyślony wynik. Przy grze dla
drużyny to akceptowalne. Gdyby kiedyś przeszkadzało, trzeba by dołożyć
logowanie albo własny serwer pośredniczący.
