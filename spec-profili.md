# Specifikacija: Modul za izračun profila (pregradni zid CW/UW)

Dodaj postojećoj web aplikaciji modul koji na osnovu dimenzija zida i otvora
izračuna potrebne metalne profile i pričvrsnu robu.

## Ulazni podaci (input)

```
zid = {
  duzina_m: number,        // dužina zida u metrima
  visina_m: number,        // visina zida u metrima
  tip_profila: "CW50" | "CW75" | "CW100",   // default CW75
  oblaganje: "jednostruko" | "dvostruko",   // default jednostruko
  duzina_profila_m: 3 | 4 | 4.5,            // standardna nabavna dužina, default 4
  otvori: [
    { sirina_m: number, visina_m: number, tip: "vrata" | "prozor", x_m: number }
  ]
}
```

## Logika izračuna

### 1. Raster (razmak CW profila)
```
raster = 0.625  // metara, default
if (visina_m > 2.6 || oblaganje === "dvostruko") raster = 0.417
```

### 2. UW profil (pod + strop)
```
ukupna_duzina_UW = duzina_m * 2
// oduzmi širine vrata na podnoj tračnici (vrata nemaju donji UW)
for (otvor of otvori) if (otvor.tip === "vrata") ukupna_duzina_UW -= otvor.sirina_m
broj_UW_komada = ceil(ukupna_duzina_UW / duzina_profila_m)
```

### 3. CW profil (okomiti stupovi)
```
broj_CW_osnovni = floor(duzina_m / raster) + 1
duzina_CW = visina_m - 0.01   // 1 cm zazor za dilataciju
// VAŽNO: raster je konstantan od kraja zida; otvori ga NE pomiču

// pojačanja oko otvora
broj_CW_pojacanja = otvori.length * 2   // po 1 sa svake strane svakog otvora
// za vrata dodatno pojačanje (dupli CW na obje strane)
for (otvor of otvori) if (otvor.tip === "vrata") broj_CW_pojacanja += 2

broj_CW_ukupno = broj_CW_osnovni + broj_CW_pojacanja
```

### 4. Pragovi iznad/ispod otvora
```
// komad UW-a savijen kao prag, širina + 0.20 m za krilca
for (otvor of otvori) {
  prag_duzina = otvor.sirina_m + 0.20
  // prozor ima prag gore i dolje (×2), vrata samo gore (×1)
}
```

### 5. Pričvrsna roba
```
povrsina_m2 = duzina_m * visina_m * 2   // obje strane zida
sloj_faktor = (oblaganje === "dvostruko") ? 2 : 1

vijci_TN = ceil(povrsina_m2 * sloj_faktor * 23)   // 23 kom/m²
pricvrscenja_UW = ceil(ukupna_duzina_UW / 0.6)    // svakih 60 cm u pod/strop
brtvena_traka_m = ukupna_duzina_UW                 // ide pod cijeli UW
```

### 6. Otpad
```
// na sve profile i traku dodaj 10%
faktor_otpada = 1.10
```

## Izlazni prikaz (output)

Ispiši tablicu po zidu:

| Stavka | Količina | Jedinica |
|--------|----------|----------|
| UW profil ({tip}) | broj_UW_komada | kom ({duzina_profila} m) |
| CW profil ({tip}) | broj_CW_ukupno | kom ({duzina_CW} m svaki) |
| — od toga osnovni raster | broj_CW_osnovni | kom |
| — od toga pojačanja oko otvora | broj_CW_pojacanja | kom |
| Vijci TN | vijci_TN | kom |
| Pričvršćenja UW u pod/strop | pricvrscenja_UW | kom |
| Brtvena traka | brtvena_traka_m × 1.10 | m |

## Napomene za prikaz korisniku
- Prikaži upozorenje ako je visina_m > nabavna duzina_CW (treba nastavak/spoj profila)
- Prikaži upozorenje ako otvor pada izvan dužine zida
- Zaokruži sve metre na 2 decimale, komade naviše (ceil)
