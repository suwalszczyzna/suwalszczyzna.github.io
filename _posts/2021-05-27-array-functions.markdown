---
layout: post
title: "Array functions w JavaScript"
date: 2021-05-27 18:00:00 +0100
categories: js
---
- [filter()](#filter)
  - [Wszyskie samochody marki Honda wyprodukowane przed 2000r](#wszyskie-samochody-marki-honda-wyprodukowane-przed-2000r)
- [map()](#map)
  - [Lista stringów z marką, modelem i ceną sprzedaży](#lista-stringów-z-marką-modelem-i-ceną-sprzedaży)
- [sort()](#sort)
  - [Samochody posortowane po roku produkcji (od najwcześniej urodzonego)](#samochody-posortowane-po-roku-produkcji-od-najwcześniej-urodzonego)
  - [Samochody posortowane po marce](#samochody-posortowane-po-marce)
  - [Samochody posortowane po marce i modelu](#samochody-posortowane-po-marce-i-modelu)
- [reduce()](#reduce)
  - [Suma zysku (suma różnic sellingPrice i cost)](#suma-zysku-suma-różnic-sellingprice-i-cost)
  - [Zliczanie wystąpień danej marki](#zliczanie-wystąpień-danej-marki)


W każdym języku programowania spotykami się z konstruktami zwanymi tablicami, listami itp. Poniżej garść informacji na temat funkcji służących do manipulacji na tablicach (`Arrays`).

Dane, na których będziemy pracować. 20 elementowa lista elementów. Polecam do generowania fakowych danych tą stronę: [mockaroo.com](https://mockaroo.com/)
```javascript
   const cars = [
     {id: 1, brand: "Ford", model: "Fusion", year: 2010, cost: 115773.69, sellingPrice: 123590.69 },
     {id: 2, brand: "Volvo", model: "S80", year: 2010, cost: 114195.88, sellingPrice: 119772.88 },
     {id: 3, brand: "Audi", model: "S6", year: 1995, cost: 130979.2, sellingPrice: 139496.2 },
     {id: 4, brand: "Pontiac", model: "Grand Prix", year: 2005, cost: 36974.48, sellingPrice: 43446.48 },
     {id: 5, brand: "Chevrolet", model: "Silverado 3500", year: 2001, cost: 94786.07, sellingPrice: 103665.07 },
     {id: 6, brand: "Nissan", model: "Titan", year: 2004, cost: 59372.62, sellingPrice: 67693.62 },
     {id: 7, brand: "Lexus", model: "SC", year: 2004, cost: 78654.39, sellingPrice: 82274.39 },
     {id: 8, brand: "Mitsubishi", model: "Montero", year: 2003, cost: 62148.51, sellingPrice: 66114.51 },
     {id: 9, brand: "Maybach", model: "62", year: 2007, cost: 58971.42, sellingPrice: 63019.42 },
     {id: 10, brand: "Volvo", model: "960", year: 1993, cost: 20077.25, sellingPrice: 21284.25 },
     {id: 11, brand: "Dodge", model: "Durango", year: 2002, cost: 118499.48, sellingPrice: 126342.48 },
     {id: 12, brand: "Honda", model: "CR-V", year: 1999, cost: 138907.66, sellingPrice: 146249.66 },
     {id: 13, brand: "Dodge", model: "Dakota", year: 2002, cost: 26968.82, sellingPrice: 30818.82 },
     {id: 14, brand: "Toyota", model: "Tacoma", year: 2010, cost: 110665.34, sellingPrice: 115182.34 },
     {id: 15, brand: "Volvo", model: "S60", year: 2004, cost: 52549.95, sellingPrice: 60064.95 },
     {id: 16, brand: "GMC", model: "3500", year: 1998, cost: 134936.23, sellingPrice: 138935.23 },
     {id: 17, brand: "Maserati", model: "Spyder", year: 2004, cost: 64241.22, sellingPrice: 65604.22 },
     {id: 18, brand: "Honda", model: "S2000", year: 2003, cost: 23455.28, sellingPrice: 27075.28 },
     {id: 19, brand: "Subaru", model: "Loyale", year: 1993, cost: 135813.9, sellingPrice: 140628.9 },
     {id: 20, brand: "Chevrolet", model: "Caprice", year: 1977, cost: 10050.72, sellingPrice: 13606.72 }
  ]
  ```

## filter()
Zwraca przefiltrowaną tablicę

### Wszyskie samochody marki Honda wyprodukowane przed 2000r
```javascript
cars.filter(i => i.year < 2000 && i.brand === 'Honda')
```
## map()
Zwraca przemapowaną tablicę

### Lista stringów z marką, modelem i ceną sprzedaży
```javascript
cars.map(i => `${i.brand} ${i.model}: ${i.sellingPrice}`);
```
Wynik:
```
"Ford Fusion: 123590.69"
"Volvo S80: 119772.88"
"Audi S6: 139496.2"
"Pontiac Grand Prix: 43446.48"
"Chevrolet Silverado 3500: 103665.07"
"Nissan Titan: 67693.62"
"Lexus SC: 82274.39"
"Mitsubishi Montero: 66114.51"
"Maybach 62: 63019.42"
"Volvo 960: 21284.25"
"Dodge Durango: 126342.48"
"Honda CR-V: 146249.66"
"Dodge Dakota: 30818.82"
"Toyota Tacoma: 115182.34"
"Volvo S60: 60064.95"
"GMC 3500: 138935.23"
"Maserati Spyder: 65604.22"
"Honda S2000: 27075.28"
"Subaru Loyale: 140628.9"
"Chevrolet Caprice: 13606.72"
```

## sort()
Zwraca posortowaną listę

### Samochody posortowane po roku produkcji (od najwcześniej urodzonego)

```javascript
cars.sort((a, b) => a.year - b.year)
```

### Samochody posortowane po marce
```javascript
cars.sort((a, b) => {
    if(a.brand < b.brand) return -1;
});
```

### Samochody posortowane po marce i modelu
```javascript
cars.sort((a, b) => {
  if (a.brand === b.brand) {
    return a.model < b.model ? -1 : a.model > b.model ? 1 : 0;
  } else {
    return a.brand < b.brand ? -1 : 1;
  }
});
```

## reduce()
'Redukuje' tablicę. Np liczy jej sumę. Znajdzie zastosowanie wszędzie tam, gdzie trzeba przeiterować po całej tablicy, po drodze zbierając wynik do jakiejś zmiennej.

Syntax:
`reduce(callbackFn, InitValue)`
```
reduce((accumulator, currentValue) => { ... } )
reduce((accumulator, currentValue, index) => { ... } )
reduce((accumulator, currentValue, index, array) => { ... } )
reduce((accumulator, currentValue, index, array) => { ... }, initialValue)
```


Jak wyżej widać, możliwe parametry funkcji `callback` to:

`accumulator` - obiekt, w którym będziemy akumulować wartości

`currentValue` - dostęp do aktualnej wartości podczas iterowania

`index` - aktualny index

`array` - dostęp do funkcji, na której wywoływany jest `reduce`


Oprócz tego, możemy zadeklarować `initialValue` - np kiedy będziemy sumować jakieś wartości możemy wpisać 0 - jeśli sumowanie ma się zacząć od zera.

### Suma zysku (suma różnic sellingPrice i cost)
```javascript
cars.reduce((total, item) => {
  const profit = item.sellingPrice - item.cost;
  return total + profit;
},0);

// wynik: 106844
```
### Zliczanie wystąpień danej marki

```javascript
cars.reduce((result, item) => {
  if(item.brand in result){
    result[item.brand]++;
  }else{
    result[item.brand] = 1;
  }
  return result;
},{}) // tutaj initialVal to {} ponieważ wynikiem funkcji będzie nowy obiekt

// wynik:
{
  "Ford": 1,
  "Volvo": 3,
  "Audi": 1,
  "Pontiac": 1,
  "Chevrolet": 2,
  "Nissan": 1,
  "Lexus": 1,
  "Mitsubishi": 1,
  "Maybach": 1,
  "Dodge": 2,
  "Honda": 2,
  "Toyota": 1,
  "GMC": 1,
  "Maserati": 1,
  "Subaru": 1
}
```