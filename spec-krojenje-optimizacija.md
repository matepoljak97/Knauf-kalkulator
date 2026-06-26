# Specifikacija: Optimizacija krojenja ploča (dvostruko oblaganje)

Cilj: smanjiti broj potrošenih ploča i izbjeći mršave rubne trakove tako da
oba sloja dijele iste ploče i koriste ostatke (cutoffs).

## Problem koji rješavamo
- Kod krojenja sloja po sloj zasebno nastaje tanki rubni komad (npr. 5 cm) koji
  je mehanički loš i tretira se kao otpad.
- Kod dvostrukog oblaganja drugi sloj MORA biti pomaknut u odnosu na prvi
  (spoj ne smije pasti na spoj). Zbog tog pomaka ostaci iz prvog sloja
  najčešće pašu kao korisni komadi u drugom sloju.
- Zato se slojevi ne smiju računati odvojeno — kroje se iz zajedničke zalihe.

## Pravila

### 1. Pomak drugog sloja (offset)
```
offset_drugi_sloj = 0.625   // metara (pola ploče)
// Sloj 1 kreće od x=0; Sloj 2 kreće od x=offset, ostatak slaže redom.
// Tako vertikalni spojevi sloja 2 NE padaju na spojeve sloja 1.
```

### 2. Minimalna širina rubnog komada
```
MIN_KOMAD = 0.30   // metara
// Ako bi zadnji komad u redu bio < MIN_KOMAD:
//   preraspodijeli prvi i zadnji komad da budu jednaki i oba >= MIN_KOMAD.
// Primjer za zid 3.80 m: umjesto 1.25+1.25+1.25+0.05
//   -> 0.65 + 1.25 + 1.25 + 0.65 (oba ruba 65 cm)
// VAŽNO: ako pomakneš ploče, vertikalni spoj mora i dalje pasti na CW profil
//        (na višekratnik rastera). Ako ne pada -> upozorenje korisniku.
```

### 3. Zajednička zaliha ostataka (bin-packing)
```
zaliha_ostataka = []   // lista širina iskoristivih komada (m)

funkcija uzmi_komad(potrebna_sirina):
    // 1) pokušaj naći ostatak iz zalihe koji je >= potrebna_sirina
    najbolji = ostatak iz zalihe s najmanjom širinom koja >= potrebna_sirina
    if (najbolji postoji):
        ukloni najbolji iz zalihe
        rez = najbolji - potrebna_sirina
        if (rez >= MIN_KOMAD) dodaj rez u zaliha_ostataka
        return // bez trošenja nove ploče

    // 2) inače načni novu ploču
    broj_plocа += 1
    rez = SIRINA_PLOCE - potrebna_sirina
    if (rez >= MIN_KOMAD) dodaj rez u zaliha_ostataka
```
SIRINA_PLOCE = 1.25 m, VISINA_PLOCE = 2.00 m (prilagodi stvarnom formatu).

### 4. Tijek izračuna
```
1. Izračunaj raspored komada za SLOJ 1 (od x=0).
2. Izračunaj raspored komada za SLOJ 2 (od x=offset).
3. Spoji sve potrebne komade oba sloja u JEDAN niz potreba.
4. Sortiraj potrebe od najveće prema najmanjoj širini (smanjuje otpad).
5. Za svaku potrebu pozovi uzmi_komad().
6. Ukupno = broj_plocа (stvarno načetih punih ploča).
```

### 5. Visinski spojevi (ako visina zida > visina ploče)
```
- Vodoravne spojeve izmakni cik-cak između susjednih stupaca (ne u liniji).
- Spoj NE smije pasti na ugao otvora (vrata/prozor) -> tamo puca.
  Oko otvora reži ploče u obliku slova "L".
```

## Izlaz
```
Po zidu ispiši:
- broj punih ploča (stvarno potrošeno, zajednički oba sloja)
- broj rezova
- preostali otpad (zbroj komada u zalihi < MIN_KOMAD na kraju)
- upozorenje ako spoj ne pada na profil (raster)
- upozorenje ako spoj pada na ugao otvora
```

## Napomena
Ovaj zajednički pristup smanjuje broj ploča u odnosu na računanje sloja po sloj
jer rubni ostaci jednog sloja postaju korisni komadi drugog sloja. Ne zbrajaj
otpad dvaput.
